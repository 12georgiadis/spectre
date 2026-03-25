Now I have comprehensive data. Here is the full analysis.

---

## Recherche exhaustive : alternatives open source a DaVinci Resolve pour l'etalonnage cinema

---

### 1. DARKTABLE

**Ce que c'est :** Raw developer photo, open source (GPL), avec un module *Color Balance RGB* inspire du grading cinema (lift/gamma/gain perceptuel en JzAzBz).

**Color grading video/cinema ?**
- Darktable est concu pour la photo, pas pour la video. Il n'y a pas de timeline, pas de playback video.
- Cependant, il est possible de traiter des **sequences d'images** extraites d'une video via `darktable-cli`.

**CLI et batch processing :**
- `darktable-cli input.dng output.exr --style "MonGrade" --hq true` fonctionne image par image.
- Pour traiter une sequence : boucle shell, un appel par frame. Pas de mode daemon (demande depuis 2018, jamais implemente).
- Performance : chaque appel relance le moteur complet. Lent pour des milliers de frames.
- Export EXR supporte (linear HDR), mais mono-couche uniquement. Bug connu (2024) : les metadonnees EXIF sont perdues a l'export EXR.

**Pipeline darktable vers EXR :**
```bash
for file in /frames/*.tiff; do
    darktable-cli "$file" "${file%.*}.exr" --style "CinemaGrade" --hq true
done
```
Viable mais lent. Pas de gestion native OCIO (darktable a son propre pipeline couleur interne).

**Niveau pro :** Amateur/prosumer photo. Jamais utilise en production cinema.
**Automatisable depuis Claude Code :** Oui, via shell scripting sur `darktable-cli`. Mais pipeline fragile.
**Verdict :** Utilisable comme raw developer dans une chaine (raw camera vers EXR lineaire), pas comme outil de grading video.

---

### 2. BLENDER (Compositeur)

**Ce que c'est :** Le compositeur node-based de Blender, avec OCIO integre nativement (Filmic, AgX, ACES via config custom).

**Capacites color grading :**
- Nodes : Color Balance (lift/gamma/gain), RGB Curves, Color Correction, Hue Saturation, Mix
- Vectorscope et histogramme integres
- Import/export EXR 16-bit et 32-bit float
- OCIO configurable via variable d'environnement `$OCIO` : possibilite de charger ACES, Filmic, AgX, configs custom
- Add-ons : *Colorist Pro* (ACES one-click), *Film Emulation Nodes* (halation, grain, vignette), *Cinematic Color Grading Node* (Blender 5.0/AgX)

**Films pro utilisant Blender pour le compositing :**
- Plus de 200 films et series en 2025, principalement indie et Netflix originals
- Utilise pour le VFX/compositing, rarement pour l'etalonnage final seul
- Pas de cas documente de Blender utilise pour le grading sur un film Oscar VFX. Les blockbusters restent sur Nuke + Resolve/Baselight.
- Pipeline open source viable : Blender (3D/comp) vers Natron (comp avance) vers Resolve (grade final)

**Automatisable depuis Claude Code :**
- Oui, via `blender --background --python script.py` : on peut scripter tout le compositing en Python
- Le MCP Blender est deja configure dans ton setup

**Niveau pro :** Semi-pro pour le compositing. Indie films : oui. Oscar VFX : non.
**Pipeline reproductible :** Oui, entierement scriptable en Python.

---

### 3. NATRON

**Ce que c'est :** Compositeur open source node-based (GPLv2), fork/clone de Nuke. Pipeline 32-bit float, OCIO natif, support EXR/DPX via OpenImageIO et FFmpeg.

**Etat du projet (2025-2026) :**
- Developpement actif mais **tres lent**, communaute benevole, pas de sponsor corporate
- v2.5.1 pre-release (sept 2024), v2.6.0-alpha1 (Apple Silicon beta)
- Nouveau site web lance en juin 2025
- Recherche active de developpeurs C++
- Plus de financement Inria depuis 2018

**Capacites color grading :**
- OCIO natif avec configs custom (ACES, etc.)
- 32-bit float tout le pipeline
- Noeuds de grading : ColorCorrect, Grade, ColorLookup (courbes), Saturation
- Support OpenFX : plugins Nuke compatibles (certains)
- Formats : EXR, DPX, TIFF, H264, DNxHR via FFmpeg/OpenImageIO

**Limitations critiques :**
- "Working with color is not so convenient like in Nuke" (revue G2)
- Performance inferieure a Nuke, instabilite sur les node graphs complexes
- Usabilite et UX faibles
- Pas de GPU compositing (OpenGL 2.0 pour le viewport seulement)

**Niveau pro :** Indie/etudiant. Quelques studios l'utilisent en production low-budget. Pas de credits Oscar/Cannes.
**Automatisable depuis Claude Code :** Oui, Natron a un mode CLI (`NatronRenderer`) et supporte le scripting Python.
**Verdict :** Le plus proche de Nuke en open source, mais en pratique trop lent et fragile pour une production cinema serieuse.

---

### 4. OpenColorIO (OCIO)

**Ce que c'est :** La bibliotheque standard de l'industrie pour la gestion couleur dans les pipelines VFX/cinema. Projet Academy Software Foundation. Utilisee par Netflix, Disney, ILM, Weta, tous les studios.

**Integration Python CLI :**

```python
import PyOpenColorIO as OCIO

config = OCIO.Config.CreateFromFile("aces_1.3_config.ocio")

# Log to Linear
processor = config.getProcessor("ACEScct", "ACEScg")
cpu = processor.getDefaultCPUProcessor()

# Appliquer a un array numpy
import numpy as np
pixels = np.array([0.5, 0.3, 0.2], dtype=np.float32)
cpu.applyRGB(pixels)
```

**Capacites cles :**
- Transformation log vers linear vers display
- Application de LUTs (`.cube`, `.3dl`, CTF, CLF)
- Baking de LUTs : `ociobakelut --format houdini --inputspace lg10 --outputspace srgb8 output.lut`
- Grading transforms : `GradingPrimaryTransform`, `GradingToneTransform`, `GradingRGBCurveTransform`
- CDL natif (Slope/Offset/Power via `CDLTransform`)
- Contexts pour grades par shot
- ACES 2.0 complet depuis OCIO 2.4
- `ocioconvert` : outil CLI pour appliquer des transforms a des images

**Niveau pro :** **Le standard absolu.** Chaque film Oscar/Cannes passe par OCIO d'une facon ou d'une autre.
**Automatisable depuis Claude Code :** Parfaitement. `pip install PyOpenColorIO`, tout est scriptable.
**Verdict :** Ce n'est pas un grading tool autonome, mais c'est **la colonne vertebrale** de tout pipeline de grading pro. Indispensable.

---

### 5. COLOUR-SCIENCE (Python)

**Ce que c'est :** Bibliotheque Python comprehensive pour la science de la couleur. 4000+ stars GitHub, maintenue activement (releases 2025).

**Capacites pour un pipeline de grading :**
- **LUT I/O** : `colour.io.read_LUT()` / `colour.io.write_LUT()` (formats .cube, SPImtx, CLF via extension)
- **LUT3D** : `colour.LUT3D()` creation, manipulation, inversion (Shepard interpolation)
- **Application de LUT** : `LUT.apply(RGB)`
- **Espaces couleur** : ACEScc, ACEScct, ACEScg, ARRI LogC3/4, S-Log3, Cineon, DaVinci Intermediate, Canon Log, etc.
- **EXR** : lecture native, ecriture via `colour.io.write_image_OpenImageIO()`
- **OCIO integration** : `colour.io.process_image_OpenColorIO()`
- **Colour correction** : `colour.colour_correction()` avec matrices ColorChecker
- **Transfer functions** : encoding/decoding pour tous les log/gamma standards cinema

**Ce qui manque nativement :**
- Pas de lift/gamma/gain ou ASC CDL built-in (il faut implementer la formule SOP en NumPy, trivial)
- Pas de film emulation built-in (mais les transfer functions permettent de construire des LUTs custom)
- Pas d'interface de courbes interactives (c'est une lib de calcul, pas un UI)

**Formule CDL en NumPy (trivial) :**
```python
import numpy as np
def apply_cdl(rgb, slope, offset, power, saturation=1.0):
    out = np.clip((rgb * slope + offset), 0, None) ** power
    luma = 0.2126 * out[...,0] + 0.7152 * out[...,1] + 0.0722 * out[...,2]
    out = luma[...,None] + saturation * (out - luma[...,None])
    return np.clip(out, 0, 1)
```

**Niveau pro :** Utilise en recherche, pipeline Netflix/Disney (en coulisses). Pas un outil de grading direct.
**Automatisable depuis Claude Code :** Parfaitement. `pip install colour-science[optional]`.
**Verdict :** Excellent pour construire un pipeline Python custom (LUT generation, matching camera, conversion d'espaces). Combine avec OCIO et OpenImageIO, on peut tout faire.

---

### 6. FFmpeg

**Ce que c'est :** Le couteau suisse de la video. Filtres de color grading disponibles en CLI.

**Filtres de grading :**
| Filtre | Usage |
|--------|-------|
| `lut3d` | Appliquer un .cube/.3dl (le plus utile) |
| `curves` | Courbes via fichier .acv ou parametres inline |
| `colorbalance` | Shadows/midtones/highlights par canal RGB |
| `eq` | Contrast, brightness, gamma, saturation |
| `colortemperature` | Kelvin warm/cool |
| `colorspace` | Conversion BT.709/BT.2020/sRGB |
| `zscale` | Conversion colorspace haute qualite (via libzimg) |
| `tonemap` | HDR vers SDR (hable, mobius, reinhard) |
| `colorchannelmixer` | Matrice de melange RGB |

**Exemple pipeline CLI :**
```bash
ffmpeg -i input.mov \
  -vf "lut3d=file=grade.cube,curves=preset=lighter,eq=contrast=1.1:saturation=0.9" \
  -c:v prores_ks -profile:v 4444 -pix_fmt yuva444p10le output.mov
```

**Limitations :**
- Pas de scopes visuels, pas de preview en temps reel
- `tonemap` est mono-thread, preuve de concept
- `zscale` pas GPU-accelere (mais rapide en CPU)
- Risque de clipping couleur si primaires converties avant tonemap
- Pas de grading interactif : tout est "render and check"

**Niveau pro :** Utilise dans tous les pipelines pro pour le transcodage et les conversions d'espace. Pour le grading creatif : outil de batch/automation, pas d'etalonnage artistique.
**Automatisable depuis Claude Code :** Parfaitement. Deja installe sur toutes tes machines.
**Verdict :** Indispensable comme glue de pipeline (LUT application en batch, conversions d'espace, transcode). Ne remplace pas Resolve pour le grading artistique.

---

### 7. RAWPY / LIBRAW

**Ce que c'est :** Wrapper Python pour LibRaw. Debayering de fichiers raw camera (NEF, CR2, ARW, DNG, etc.).

**Pipeline :**
```python
import rawpy
raw = rawpy.imread('frame.dng')
rgb = raw.postprocess(
    output_color=rawpy.ColorSpace.raw,  # linear
    output_bps=16,
    no_auto_bright=True,
    gamma=(1, 1)  # linear
)
# Puis ecrire en EXR via OpenImageIO ou colour-science
```

**Capacites :**
- Debayering haute qualite (AHD, DHT, etc., sauf GPL demosaics exclus car rawpy est MIT)
- White balance, matrice couleur camera-vers-XYZ
- Sortie lineaire pour integration pipeline VFX
- Support de centaines de formats raw camera

**Limitations :**
- Pas de video raw (BRAW, R3D) : LibRaw traite des images fixes raw
- Pour le cinema digital (RED, ARRI, Blackmagic) : il faut les SDK proprietaires respectifs
- Pas de debayering CinemaDNG en temps reel

**Niveau pro :** Standard pour le traitement raw photo en Python. Pour le cinema : utile uniquement si on travaille avec des sequences DNG (certains workflows Blackmagic, Magic Lantern).
**Automatisable depuis Claude Code :** Oui. `pip install rawpy`.

---

## SYNTHESE : Pipeline open source reproductible

Voici comment ces outils s'assemblent en un pipeline automatisable depuis Claude Code :

```
Camera Raw (DNG/TIFF)
    |
    v
[rawpy/darktable-cli] --> debayering, white balance --> EXR lineaire
    |
    v
[OpenColorIO] --> Log-to-Linear, espace de travail (ACEScg)
    |
    v
[colour-science + NumPy] --> CDL (slope/offset/power), LUT custom, matching
    |
    v
[FFmpeg lut3d + colorspace] --> Application LUT, conversion d'espace, transcode
    |
    v
[Blender/Natron] --> Compositing noeuds (si necessaire)
    |
    v
[OCIO display transform] --> Rec.709 / DCI-P3 / HDR deliverable
```

### Tableau comparatif

| Outil | Niveau | Oscar/Cannes | Automatisable CC | Pipeline EXR 32-bit |
|-------|--------|-------------|-------------------|---------------------|
| darktable-cli | Photo prosumer | Non | Oui (lent) | Oui (mono-couche) |
| Blender comp | Semi-pro | Non (VFX oui) | Oui (Python) | Oui |
| Natron | Indie | Non | Oui (Python/CLI) | Oui |
| **OCIO** | **Standard industrie** | **Oui, partout** | **Oui (Python)** | **N/A (lib couleur)** |
| **colour-science** | Recherche/pipeline | Indirect (Netflix) | **Oui (Python)** | **Oui (via OIIO)** |
| **FFmpeg** | **Standard industrie** | **Oui (transcode)** | **Oui (CLI)** | Oui (lecture) |
| rawpy/LibRaw | Photo standard | Non | Oui (Python) | Non natif |

### Conclusion

**Aucun outil open source ne rivalise avec DaVinci Resolve pour le grading interactif et artistique.** Resolve reste imbattable pour le travail de coloriste (scopes, noeuds, tracking, HDR, GPU temps reel). Version gratuite suffisante pour 90% des usages.

En revanche, un **pipeline CLI/Python automatise** combinant OCIO + colour-science + FFmpeg permet de :
- Appliquer des grades (CDL, LUTs) en batch a des milliers de plans
- Convertir des espaces couleur (log vers linear, ACES, Rec.709)
- Generer des LUTs custom
- Integrer le grading dans un workflow Claude Code scriptable

C'est complementaire a Resolve, pas un remplacement. Le workflow optimal pour ton setup : grading artistique dans Resolve sur le PC Windows (RTX 5090), automation pipeline en Python/OCIO/FFmpeg depuis Claude Code.

---

Sources:
- [darktable CLI documentation](https://docs.darktable.org/usermanual/development/en/special-topics/program-invocation/darktable-cli/)
- [darktable Color Balance RGB](https://docs.darktable.org/usermanual/development/en/module-reference/processing-modules/color-balance-rgb/)
- [darktable EXR export issue #16121](https://github.com/darktable-org/darktable/issues/16121)
- [darktable EXR layers issue #13546](https://github.com/darktable-org/darktable/issues/13546)
- [Blender Color Management OCIO](https://docs.blender.org/manual/en/2.79/render/post_process/color_management.html)
- [Blender Filmic](https://sobotka.github.io/filmic-blender/)
- [AgX/Filmic LUTs for Resolve](https://www.creativeshrimp.com/agx-filmic-luts.html)
- [Film Emulation Nodes for Blender](https://studioimi.gumroad.com/l/filmemulationnodes)
- [Cinematic Color Grading Node Blender 5.0](https://www.vfxmed.com/2025/12/cinematic-color-grading-node-for-blender-5-0-free-download/)
- [Open-Source VFX: Blender, Gaffer, Natron](https://kinonews.pro/open-source-vfx-how-blender-gaffer-and-natron-are-changing-film-production)
- [Natron GitHub](https://github.com/NatronGitHub/Natron)
- [Natron About](https://natrongithub.github.io/about)
- [ACES pipeline Blender + Natron](https://community.acescentral.com/t/aces-open-source-software-pipeline-blender-natron/4361)
- [OpenColorIO documentation](https://opencolorio.readthedocs.io/en/latest/)
- [OCIO Usage Examples (Python)](https://opencolorio.readthedocs.io/en/latest/guides/developing/usage_examples.html)
- [OCIO Grading Transforms](https://opencolorio.readthedocs.io/en/latest/api/grading_transforms.html)
- [OCIO Baking LUTs](https://opencolorio.readthedocs.io/en/latest/tutorials/baking_luts.html)
- [OCIO Tool Overview](https://opencolorio.readthedocs.io/en/latest/guides/using_ocio/tool_overview.html)
- [colour-science PyPI](https://pypi.org/project/colour-science/)
- [colour-science GitHub](https://github.com/colour-science/colour)
- [colour.LUT3D documentation](https://colour.readthedocs.io/en/develop/generated/colour.LUT3D.html)
- [camera-match Python library](https://github.com/ethan-ou/camera-match)
- [FFmpeg color grading guide](https://gabor.heja.hu/blog/2024/12/10/using-ffmpeg-to-color-correct-color-grade-a-video-lut-hald-clut/)
- [FFmpeg color space journey (Canva)](https://www.canva.dev/blog/engineering/a-journey-through-colour-space-with-ffmpeg/)
- [FFmpeg HLG color grading](https://www.binarytides.com/color-grading-hlg-videos-with-ffmpeg/)
- [FFmpeg tonemap filter](http://underpop.online.fr/f/ffmpeg/help/tonemap.htm.gz)
- [rawpy GitHub](https://github.com/letmaik/rawpy)
- [rawpy API documentation](https://letmaik.github.io/rawpy/api/rawpy.RawPy.html)
- [Best Color Grading Software 2026](https://www.colorgradingcentral.com/color-grading-software/)
- [OCIO Compatible Software](https://opencolorio.readthedocs.io/en/latest/guides/using_ocio/compatible_software.html)
- [ASC CDL in-depth (Pomfort)](https://pomfort.com/article/an-in-depth-look-at-asc-cdl-based-color-controls/)
- [cdl_convert Python library](https://cdl-convert.readthedocs.io/en/latest/usage_cc.html)