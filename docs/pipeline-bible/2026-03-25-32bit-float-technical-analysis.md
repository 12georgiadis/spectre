I now have comprehensive data across all seven dimensions. Let me compile the analysis.

---

# Analyse technique : pourquoi le pipeline ComfyUI perd du bit depth et comment le fixer

## 1. Le VAE de Stable Diffusion / FLUX : format et dynamic range

### Format interne du VAE

Le VAE fonctionne par defaut en **fp32** sur CPU et en **fp16 ou bf16** sur GPU, selon le modele. Le probleme de precision est bien documente :

- **SD 1.5 / SDXL** : le VAE utilise `AutoencoderKL`. En fp16, les valeurs intermediaires peuvent depasser la plage representable de fp16 ([-65504, +65504]), causant des **NaN**. C'est pour ca que `madebyollin/sdxl-vae-fp16-fix` existe : il corrige les couches problematiques.
- **FLUX** : VAE 16 canaux (vs 4 pour SD), compression 12x (vs 48x pour SDXL). Plus de detail preserve, mais meme mecanisme de normalisation en sortie.
- **bf16** est la meilleure alternative : meme plage dynamique que fp32 (exposant identique), mais 16 bits. Pas de NaN, pas d'overflow.

### Le clamp fatal : `process_output`

Voici la ligne exacte dans `comfy/sd.py` qui detruit toute donnee HDR :

```python
self.process_output = lambda image: image.add_(1.0).div_(2.0).clamp_(0.0, 1.0)
```

Trois operations in-place :
1. `.add_(1.0)` : shift [-1, 1] vers [0, 2]
2. `.div_(2.0)` : scale [0, 2] vers [0, 1]
3. **`.clamp_(0.0, 1.0)`** : destruction irreversible de toute valeur hors [0, 1]

Le `process_input` (encode) fait l'inverse : `image * 2.0 - 1.0`

**Point critique** : le decodeur VAE produit des valeurs brutes qui **depassent** la plage [-1, +1]. La couche `conv_out` du decodeur applique une transformation (pas un simple sigmoid) qui compresse les 128 canaux internes vers 3 canaux RGB. Les pics de luminance au-dela de 1.0 existent dans le signal brut mais sont irremediablement clampes.

Pour les VAE de type Stable Cascade et Audio, les lambdas sont des identites (`lambda image: image`), pas de clamp.

### Dtypes supportes

```python
self.working_dtypes = [torch.bfloat16, torch.float32]  # defaut
self.working_dtypes = [torch.float16, torch.bfloat16, torch.float32]  # Hunyuan Video
```

---

## 2. PIL vs OpenCV vs numpy : le goulot d'etranglement 8-bit

### Le probleme exact

**LoadImage** dans `nodes.py` :
```python
image = i.convert("RGB")
image = np.array(image).astype(np.float32) / 255.0
image = torch.from_numpy(image)[None,]
```

PIL ouvre en **uint8** par defaut. Meme si l'image source est 16-bit, `PIL.Image.open()` + `.convert("RGB")` la tronque a 8-bit. Pour les images mode `'I'` (32-bit integer), il y a un hack : `i.point(lambda i: i * (1 / 255))`, qui est mathematiquement incorrect pour du vrai 32-bit.

**SaveImage** dans `nodes.py` :
```python
i = 255. * image.cpu().numpy()
img = Image.fromarray(np.clip(i, 0, 255).astype(np.uint8))
img.save(..., pnginfo=metadata, compress_level=self.compress_level)
```

Double destruction :
1. `np.clip(i, 0, 255)` : clamp
2. `.astype(np.uint8)` : quantification 8-bit
3. Format PNG 8-bit uniquement

### Comment forcer le 16/32-bit

- **PIL** : ne gere pas correctement les images au-dela de 8-bit. `pil_to_tensor()` de torchvision echoue sur le mode `I;16`.
- **OpenCV** : `cv2.imread(path, cv2.IMREAD_UNCHANGED)` charge du 16-bit nativement. `cv2.imwrite()` ecrit du 16-bit PNG/TIFF.
- **tifffile** : ecriture directe float32/float16 TIFF sans passer par PIL.
- **OpenEXR + Imath** : lecture/ecriture 32-bit float native, le standard VFX.
- **OpenImageIO (OIIO)** : la bibliotheque de reference (ILM), supporte tout.

### Nodes ComfyUI qui utilisent PIL vs torch natif

- **LoadImage, SaveImage, PreviewImage** : tous passent par PIL = bottleneck 8-bit
- **VAE Decode/Encode** : torch natif float32, pas de PIL
- **La plupart des custom nodes** : melange. Beaucoup utilisent `tensor2pil` / `pil2tensor` en interne, ce qui clampe silencieusement.

---

## 3. Le type IMAGE dans ComfyUI : HDR ou pas ?

### Specification technique

`IMAGE` = `torch.Tensor`, dtype `float32`, shape `[B, H, W, C]`, convention **[0.0, 1.0]**.

**Point fondamental** : ComfyUI **n'enforce pas** le clamp au niveau du systeme de types. C'est une convention, pas une contrainte. Un tensor IMAGE avec des valeurs a 2.5 ou -0.3 passera entre les nodes sans erreur... tant qu'il ne touche pas un node standard.

### Ce qui clampe

- `SaveImage` / `PreviewImage` : `np.clip(i, 0, 255).astype(np.uint8)` (hard clamp)
- `VAEDecode` : `.clamp_(0.0, 1.0)` (hard clamp)
- Beaucoup de custom nodes : `result.clamp(0, 1)` par convention
- `RemapImageRange` : clamp par defaut (parametre `clamp = True`)

### Ce qui ne clampe pas

- Les operations torch pures (add, mul, conv) ne clampent pas
- Les tensors intermediaires entre nodes specialises (HDR) peuvent contenir des valeurs >1.0
- Un pipeline entierement custom peut maintenir du HDR

### ComfyUI et HDR natif

**Non, ComfyUI ne supporte pas le HDR nativement.** L'equipe de developpement (comfyanonymous) a explicitement refuse d'ajouter le support natif 16/32-bit dans l'issue #572, argumentant que le VAE produit des donnees dans la plage effective d'une image 8-bit, et que sauvegarder en 32-bit ne fait que gonfler le fichier sans gain reel. Position contestable, surtout pour les workflows de compositing.

---

## 4. Radiance FXTD : analyse critique

### Ce que Radiance pretend

76 nodes, pipeline 32-bit float, GPU-accelere (CUDA), film effects avec 30+ capteurs camera et 20+ pellicules, scopes (Histogram, Waveform, Vectorscope), OCIO, ACES 2.0, OpenEXR I/O. Dependances : `opencv-python`, `Pillow`, `imageio`, `OpenEXR`, `Imath`, `opencolorio`, `colour-science`.

Workflow type :
```
[VAE Decode] → [Image to Float32] → [Color Space: sRGB → ACEScg] → [Log Curve Encode: ARRI LogC3] → [Save EXR]
```

### Ce que Radiance fait reellement

**Critique majeure** (source : Alex Villabon, VFX + AI newsletter, janvier 2026) :

> "They claim to be a 32-bit floating-point pipeline, but there are multiple specific nodes where the implementation reduces the data down to 8-bit during processing."

> "Even with their custom sampler and VAE decoder though I'm not getting any values over 1 in the EXRs."

Le probleme : Radiance depend de `Pillow` dans ses dependances. Si certains nodes passent par PIL en interne pour des operations de traitement d'image, la precision tombe a 8-bit a ce point du pipeline, meme si l'entree et la sortie sont en float32. C'est du "container 32-bit, data 8-bit".

### Le "Radiance VAE Decode Pro"

Decrit comme "32-bit floating point VAE decoding with tiling support" mais sans documentation technique detaillee de l'implementation. Vu que les EXR en sortie ne contiennent pas de valeurs >1.0 selon les tests independants, il est probable que le node utilise le meme `process_output` que ComfyUI standard, ou un equivalent qui clampe.

### Verdict

Radiance apporte des outils utiles (viewer, scopes, OCIO, color tools) mais la promesse "32-bit pipeline" est **partiellement du marketing**. Pour du vrai HDR, il faut aller chercher ailleurs.

---

## 5. Solutions techniques concretes pour maintenir le 32-bit

### A. Custom VAE Decode sans clamp

Le projet **vae-decode-hdr** (netocg) est la solution la plus serieuse. Approche :

1. **Forward hooks** sur la couche `conv_out` du decodeur pour capturer les donnees pre/post normalisation
2. **Analyse intelligente** de la transformation du VAE (sigmoid, tanh, ou custom)
3. **MAX pooling** pour la conversion 128 canaux → 3 RGB (au lieu du averaging qui detruit les pics)
4. **4 modes HDR** :
   - Conservative (1.5x expansion) : aspect naturel
   - Moderate (3x) : equilibre qualite/plage
   - Exposure : HDR base sur les stops d'exposition
   - Aggressive : recuperation mathematique maximale
5. **Export EXR lineaire** avec OIIO (OpenImageIO) pour du vrai 32-bit/16-bit float

Workflow : `HDR VAE Decode → Linear EXR Export`

Construit et teste specifiquement pour **FLUX.1 VAE**.

### B. Bypass VAE pour les operations non-generation

Pour le color grading, compositing, upscaling : ne pas passer par le VAE du tout.

```
[Load EXR 32-bit] → [Color Grading nodes (torch natif)] → [Save EXR 32-bit]
```

Les nodes qui manipulent directement les tensors torch sans conversion PIL maintiennent la precision. Le VAE n'intervient que pour la generation/inpainting.

### C. Travailler en linear float, convertir au display

Principe identique a Nuke : tout le pipeline en **scene-referred linear** float32, conversion display (sRGB, Rec.709) uniquement pour la preview/export final.

Nodes necessaires :
- **ComfyUI-ACES-IO** (BISAM20) : transforms OCIO style Nuke, EXR loader/saver, ACES 2.0/1.3/1.2
- **ComfyUI-color-tools** (APZmedia) : `OCIOColorSpaceConverter`, `AdvancedOcioColorTransform`
- **Radiance OCIO Transform** : integration OpenColorIO v2.x (si on fait confiance au reste du pipeline)

### D. OpenEXR I/O natif sans PIL

| Node pack | Format | Precision | Methode |
|-----------|--------|-----------|---------|
| **ComfyUI-HQ-Image-Save** | EXR 32-bit, TIFF 16-bit | float32 / uint16 | OpenEXR lib |
| **vae-decode-hdr** Linear EXR Export | EXR 32-bit | float32 | OpenImageIO (OIIO) |
| **ComfyUI-ACES-IO** EXR Loader/Saver | EXR 32-bit | float32 | PyOpenColorIO + OpenEXR |
| **Orion4D PixelShift** Save Image Advanced | EXR 32-bit, TIFF 32-bit | float32 | imageio/OpenEXR |
| **XteVision Pro** | EXR 16/32-bit | half/float | configurable |
| **SaveImageHighPrec** | PNG 16-bit | uint16 | direct numpy |

### E. Pipeline recommande (production)

```
[KSampler] → [HDR VAE Decode (netocg)] → [OCIO: Linear → ACEScg] → 
[Color Grade (torch ops)] → [OCIO: ACEScg → Linear] → [Save EXR 32-bit (OIIO)]
```

Pour la preview : `[OCIO Display Transform: ACEScg → sRGB] → [Preview]`

---

## 6. Comparaison avec Nuke/Flame

### Comment Nuke maintient le 32-bit

- **Tout est linear float32 en interne**, toujours. Chaque fichier charge est converti en linear 32-bit float a l'entree.
- **Resolution-independent**, 1000+ canaux float32 par pixel possibles
- **OCIO integre nativement** : toutes les conversions color space sont non-destructives
- **Scene-referred par defaut** : les valeurs >1.0 sont preservees, representent des luminances superieures au blanc diffus
- **OpenEXR** est le format natif de travail (cree par ILM, les memes qui ont fait Nuke)
- Pas de PIL, pas de uint8 nulle part dans le pipeline

### Comment Flame fonctionne

- Traditionnellement **display-referred** (DPX, broadcast)
- Capable de scene-referred mais plus complexe a configurer
- Floating-point pipeline disponible mais pas par defaut

### Ce qui manque a ComfyUI pour etre au niveau

| Aspect | Nuke | ComfyUI (actuel) |
|--------|------|-----------------|
| Linear float32 partout | Oui, natif | Non, convention [0,1] clampee |
| OCIO natif | Oui | Non, plugins tiers |
| OpenEXR natif | Oui | Non, plugins tiers |
| Valeurs >1.0 | Preservees | Detruites par defaut |
| Multi-channel (AOVs) | 1000+ canaux | 3 canaux RGB fixes |
| Deep compositing | Oui | Non |
| Color management | OCIO built-in | Aucun nativement |

### Comment reproduire ca dans ComfyUI

On ne peut pas reproduire Nuke dans ComfyUI. Mais on peut s'en approcher pour un sous-ensemble de workflows :

1. **Charger en EXR float32** via ACES-IO ou HQ-Image-Save (pas LoadImage)
2. **Decoder le VAE sans clamp** via vae-decode-hdr
3. **Travailler en ACEScg** via OCIO transforms
4. **Ne jamais toucher un node qui passe par PIL** pour les donnees de travail
5. **Sauvegarder en EXR 32-bit** via un export specialise
6. **Faire le compositing final dans Nuke/Resolve/Flame**, pas dans ComfyUI

---

## 7. ACES dans ComfyUI : etat des lieux

### Ce qui existe

**ComfyUI-ACES-IO** (BISAM20) :
- 13 nodes modelees sur les nodes OCIO de Nuke
- ACES 2.0, 1.3, 1.2 supportes (2.0 et 1.3 embarques avec PyOpenColorIO 2.3+)
- EXR Loader : output `[B, H, W, C]` float32, transforms OCIO optionnels a la lecture
- EXR Saver : compression configurable (ZIP, PIZ, DWAA, etc.)
- Color space picker style Nuke avec navigation par famille
- Video I/O : ProRes 422/4444/XQ (10-bit)
- Config OCIO custom ou built-in

**ComfyUI-color-tools** (APZmedia) :
- `OCIOColorSpaceConverter`, `AdvancedOcioColorTransform`, `OCIOConfigInfo`
- Support config OCIO custom (path configurable)
- Test patterns (ColorChecker, SMPTE bars) pour validation
- VectorScope, Waveform integres

**Radiance** :
- OCIO Transform v2.x, log curve decode (ARRI LogC3, V-Log, S-Log3, RED Log, ACEScct)
- Color space conversions : ACES 2.0, ACEScg, Rec.2020, sRGB, XYZ, linear

### Etat reel vs theorique

**Theorique** : avec ACES-IO + un VAE decode HDR + des nodes OCIO, on pourrait avoir un pipeline ACES fonctionnel dans ComfyUI.

**Reel** :
- Aucun de ces outils n'est dans le core ComfyUI. Ce sont des plugins tiers, maintenus par des individus.
- L'integration entre ces plugins n'est pas testee/garantie.
- Le core ComfyUI (LoadImage, SaveImage, PreviewImage, VAEDecode) casse le pipeline ACES a chaque point de contact.
- Il n'y a **aucune gestion de color space au niveau du framework**. ComfyUI ne sait pas dans quel espace couleur se trouve un tensor IMAGE. C'est a l'utilisateur de le savoir et de ne pas melanger.
- Pas de viewer ACES/OCIO natif pour la preview (il faut Radiance Viewer ou un bridge externe).

**En resume** : ACES dans ComfyUI est techniquement possible mais fragile, non-standard, et demande une discipline rigoureuse pour eviter que le pipeline ne casse silencieusement a un point de conversion.

---

## Synthese des recommandations

Pour un workflow VFX-grade dans ComfyUI aujourd'hui :

1. **VAE Decode** : utiliser `vae-decode-hdr` (netocg), pas le VAEDecode standard
2. **I/O** : `ComfyUI-ACES-IO` pour le load/save EXR avec OCIO
3. **Color management** : definir un OCIO config au debut du workflow, s'y tenir
4. **Eviter** : tout node qui passe par PIL (LoadImage, SaveImage, PreviewImage standard)
5. **Pipeline** : generation dans ComfyUI, compositing dans Nuke/Resolve/Flame
6. **Format d'echange** : OpenEXR 32-bit, linear ou ACEScg
7. **Radiance** : utiliser pour les scopes et le viewer, mais verifier la precision reelle des nodes de traitement

Sources :
- [ComfyUI Images, Latents and Masks docs](https://docs.comfy.org/custom-nodes/backend/images_and_masks)
- [ComfyUI Issue #572 - Higher bit depth export](https://github.com/comfyanonymous/ComfyUI/issues/572)
- [ComfyUI-HQ-Image-Save (pixelsham)](https://www.pixelsham.com/2026/03/09/comfyui-hq-image-save-save-and-load-images-and-latents-as-32bit-exrs/)
- [vae-decode-hdr (netocg)](https://github.com/netocg/vae-decode-hdr)
- [Radiance FXTD Studios](https://github.com/fxtdstudios/radiance)
- [Radiance critique - Alex Villabon VFX+AI](https://alexvillabon.substack.com/p/alex-villabon-vfx-ai-021)
- [SDXL VAE fp16 fix (madebyollin)](https://huggingface.co/madebyollin/sdxl-vae-fp16-fix)
- [sd-webui-mixed-precision-vae](https://github.com/Ttl/sd-webui-mixed-precision-vae)
- [ComfyUI-ACES-IO (BISAM20)](https://github.com/BISAM20/ComfyUI-ACES-IO)
- [ComfyUI-color-tools (APZmedia)](https://github.com/APZmedia/ComfyUI-color-tools)
- [ComfyUI sd.py source (process_output)](https://github.com/Comfy-Org/ComfyUI/blob/master/comfy/sd.py)
- [ComfyUI nodes.py source (SaveImage/LoadImage)](https://github.com/Comfy-Org/ComfyUI/blob/master/nodes.py)
- [Orion4D PixelShift](https://github.com/orion4d/Orion4D_pixelshift)
- [madebyollin notes on SD VAE](https://gist.github.com/madebyollin/ff6aeadf27b2edbc51d05d5f97a595d9)
- [ComfyUI Working with torch.Tensor docs](https://docs.comfy.org/custom-nodes/backend/tensors)
- [ComfyUI Impact Pack Issue #369 - Improving bit depth](https://github.com/ltdrdata/ComfyUI-Impact-Pack/issues/369)
- [Luminance Stack Processor](https://github.com/sumitchatterjee13/Luminance-Stack-Processor)
- [ComfyUI + VFX in Flame - Logik Forums](https://forum.logik.tv/t/comfyui-vfx-in-flame/12683)
- [Nuke features - Foundry](https://www.foundry.com/products/nuke-family/nuke/features)
- [FXTD Radiance documentation](https://fxtd.org/radiance_docs/)
- [ComfyUI-HigherBitDepthSaveImage](https://github.com/aimerib/ComfyUI_HigherBitDepthSaveImage)