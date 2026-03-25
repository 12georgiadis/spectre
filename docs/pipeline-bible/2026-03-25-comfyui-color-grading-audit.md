I now have all the data needed. Here is the comprehensive report.

---

# ComfyUI comme outil de color grading cinema : audit exhaustif

## Verdict global

ComfyUI n'est **pas un outil de color grading cinema** au sens ou DaVinci Resolve, Baselight ou Nuke le sont. C'est un framework de generation d'images AI (Stable Diffusion) autour duquel une communaute active construit des nodes de post-production. Quelques extensions serieuses emergent, mais l'ensemble reste fragile, jeune, et non utilise par des coloristes professionnels en pipeline cinema reel. Voici le detail question par question.

---

## 1. Bit Depth (16-bit, 32-bit float)

**Architecture interne** : le type `IMAGE` dans ComfyUI est un tenseur PyTorch float32 normalise 0-1. En theorie, la precision est la. En pratique, **plusieurs goulots d'etranglement cassent le pipeline** :
- Le `LoadImage` par defaut utilise PIL, qui ne gere que le 8-bit.
- Le `Save Image` par defaut clamp a 0-1 et exporte en PNG 8-bit.
- Beaucoup de custom nodes convertissent en uint8 en interne (PIL, numpy .astype(uint8)).
- La communaute a signale que meme Radiance, qui se revendique 32-bit float, **reduit a 8-bit dans certains nodes specifiques** a l'interieur de son pipeline.

**Workarounds** :
- [ComfyUI-HQ-Image-Save](https://github.com/spacepxl/ComfyUI-HQ-Image-Save) (99 stars, maj mars 2026) : sauve en TIFF 16-bit et EXR 32-bit float. Le plus fiable pour la sortie high bit depth.
- [HDR VAE Decode](https://github.com/netocg/vae-decode-hdr) : decode le VAE sans clamper les valeurs, preservant les donnees HDR.
- Feature request officielle [#572](https://github.com/comfyanonymous/ComfyUI/issues/572) toujours ouverte, pas de solution native.

**Conclusion bit depth** : possible avec des custom nodes, mais ce n'est pas un pipeline 32-bit float de bout en bout. Il faut assembler les bons nodes et verifier chaque etape. Loin d'un Nuke ou l'ensemble du pipeline est nativement float.

---

## 2. Color Grading Nodes

| Node pack | Stars | Derniere MAJ | Ce qu'il fait |
|-----------|-------|--------------|---------------|
| [**Radiance** (fxtdstudios)](https://github.com/fxtdstudios/radiance) | 182 | mars 2026 | Suite 76 nodes : LGG trackballs, Log Wheels, film stocks (30+ capteurs, 20+ pellicules), camera presets (ARRI Alexa 35, RED V-Raptor, Sony Venice 2, BMPCC), denoise 32-bit, temporal smoothing. Le plus ambitieux. |
| [**ComfyUI-Curve**](https://github.com/aiaiaikkk/ComfyUI-Curve) | 171 | mars 2026 | Courbes Photoshop-style, HSL Mixer, niveaux, color wheels 3 zones, 70+ presets integres. |
| [**ComfyUI-EasyColorCorrector**](https://github.com/regiellis/ComfyUI-EasyColorCorrector) | 137 | mars 2026 | 3-way LGG, courbes perceptuelles, color matching LAB/histogramme, presets cinema (Teal & Orange, Film Noir, Bleach Bypass). |
| [**ComfyUI-Image-Filters** (spacepxl)](https://github.com/spacepxl/ComfyUI-Image-Filters) | 289 | mars 2026 | Keyer luma/sat/chroma, color match, frequency separation, film grain realiste, matte refinement. Adopte par l'org officielle ComfyUI. |
| [**ComfyUI-Olm-LGG**](https://github.com/o-l-l-i/ComfyUI-Olm-LGG) | 13 | mars 2026 | Lift/Gamma/Gain avec color wheels interactives. Simple, une seule fonction, bien fait. |
| [**CRT-Nodes**](https://github.com/PGCRT/CRT-Nodes) | 98 | mars 2026 | 12+ categories d'effets : Levels, Color Wheels, Temp/Tint, glow, glare, aberration chromatique, vignette, grain. |
| [**ComfyUI-post-processing-nodes**](https://github.com/EllangoK/ComfyUI-post-processing-nodes) | 249 | mars 2026 | ColorCorrect : balance, temperature, hue, brightness, contrast, saturation, gamma. |
| [**Nuke Nodes for ComfyUI**](https://github.com/sumitchatterjee13/nuke-nodes-comfyui) | 50 | mars 2026 | Replique les nodes Nuke : Grade node, ACES 2.0 Studio Config integre, LUT loader. |

---

## 3. Debayering (BRAW, RAW camera)

**Pas de support BRAW.** Aucun node ne debayer du Blackmagic RAW ou du ProRes RAW. Le seul node RAW est [comfyui-raw-image](https://github.com/dimtion/comfyui-raw-image) (5 stars, janvier 2026) qui charge des fichiers photo RAW (CR2, NEF, ARW, DNG) via `rawpy`/LibRaw. Il ne gere pas le BRAW, qui necessite le SDK proprietaire Blackmagic.

**Workflow realiste** : pre-convertir les rushes BRAW en EXR ou TIFF 16/32-bit depuis DaVinci Resolve, puis charger dans ComfyUI via les nodes EXR.

---

## 4. Vectorscope / Scopes

**Un seul outil serieux** : [**Radiance**](https://github.com/fxtdstudios/radiance) integre :
- Histogramme
- Waveform
- Vectorscope
- Radiance Viewer (interface style Resolve/Flame avec trackballs LGG + Log Wheels)
- Terminal HUD style Nuke (Python REPL en temps reel)

En dehors de Radiance, **aucun node independant de scopes n'a ete trouve**. C'est un manque significatif pour un usage pro.

---

## 5. Masking / Tracking

C'est le domaine le plus mature grace a l'IA :

| Node pack | Stars | Ce qu'il fait |
|-----------|-------|---------------|
| [**SAM2** (kijai)](https://github.com/kijai/ComfyUI-segment-anything-2) | **1183** | Segment Anything 2, tracking video, propagation de masques sur sequences. De loin le plus utilise. |
| [**ComfyUI-SAM2** (neverbiasu)](https://github.com/neverbiasu/ComfyUI-SAM2) | 253 | Alternative SAM2, integration ComfyUI. |
| [**ComfyUI-Image-Filters** (keyer)](https://github.com/spacepxl/ComfyUI-Image-Filters) | 289 | Keyer luma/sat/channel/greenscreen + matte refinement (closed-form matting). |
| [**ComfyUI_RyanOnTheInside**](https://github.com/ryanontheinside/ComfyUI_RyanOnTheInside) | 766 | FlexMask nodes, manipulation reactive de masques/images/video. |
| [**ComfyUI_LayerStyle**](https://github.com/chflame163/ComfyUI_LayerStyle) | **2952** | Compositing Photoshop-like : blend modes, luminosity masking, layers. Le repo le plus star de toute cette liste. |
| [**Masquerade Nodes**](https://github.com/BadCafeCode/masquerade-nodes-comfyui) | 469 | Operations de masques dependency-free. |
| [**comfyui-tkg-chroma-key**](https://github.com/p1atdev/comfyui-tkg-chroma-key) | 19 | Chroma key par diffusion (paper TKG-DM). |

**Point fort reel** : SAM2 reduit le masking video de heures a minutes. En production VFX, des artistes Flame utilisent deja ComfyUI pour le clean plating et l'extraction de mattes (voir les threads Logik ci-dessous).

---

## 6. LUT Support

| Node pack | Stars | Ce qu'il fait |
|-----------|-------|---------------|
| [**comfyui-propost**](https://github.com/digitaljohn/comfyui-propost) | 204 | Applique des .cube 3D LUTs, support log color space. Le plus mature. |
| [**ComfyUI-OlmLUT**](https://github.com/o-l-l-i/ComfyUI-OlmLUT) | 21 | Applique des .cube avec slider d'intensite. Simple et fiable. |
| [**ComfyUI-lut**](https://github.com/thezveroboy/ComfyUI-lut) | 6 | Extraction de LUT depuis une image. |
| [**lut-maker**](https://github.com/o-l-l-i/lut-maker) (meme auteur qu'OlmLUT) | -- | Outil browser pour creer des .cube. Compatible ComfyUI Essentials. |
| ComfyUI Essentials (ImageApplyLUT) | -- | **Abandonne**, plus maintenu. |

**Limitation** : uniquement .cube. Pas de support .3dl, .csp, .clf, ou .spi3d nativement (sauf Photography Nodes qui supporte .3dl et .1dlut).

---

## 7. ACES / OCIO

| Node pack | Stars | Ce qu'il fait |
|-----------|-------|---------------|
| [**ComfyUI-ACES-IO** (BISAM20)](https://github.com/BISAM20/ComfyUI-ACES-IO) | 4 | Nodes OCIO miroir de Nuke. Tres recent (mars 2026), dev cree en mars 2026. A surveiller mais non prouve. |
| [**Nuke Nodes for ComfyUI**](https://github.com/sumitchatterjee13/nuke-nodes-comfyui) | 50 | ACES 2.0 Studio Config integre, 55 colorspaces hardcodes. |
| [**ComfyUI-color-tools** (APZmedia)](https://github.com/APZmedia/ComfyUI-color-tools) | 3 | OCIO Config Info, conversions OCIO, test patterns (SMPTE bars, ColorChecker). Tres peu adopte. |
| [**comfy_oiio** (melMass)](https://github.com/melMass/comfy_oiio) | 12 | OpenImageIO + OCIO custom configs. Compagnon d'un fork prive pour Nuke. |
| [**Radiance**](https://github.com/fxtdstudios/radiance) | 182 | Conversions colorspace (sRGB, Linear, ACEScg) integrees, depend d'`opencolorio` dans ses requirements. |
| [**CoCoTools_IO**](https://github.com/Conor-Collins/ComfyUI-CoCoTools_IO) | 100 | Colorspace converter (sRGB, Linear, ACEScg, etc.) integre dans les nodes I/O. |

**Etat reel** : le support ACES/OCIO existe mais est eclate entre plusieurs repos tres jeunes (moins de 6 mois pour la plupart). Rien de comparable a l'integration native dans Nuke, Resolve ou Flame.

---

## 8. EXR I/O

| Node pack | Stars | Multi-layer | 32-bit float | Sequences |
|-----------|-------|-------------|--------------|-----------|
| [**ComfyUI-HQ-Image-Save** (spacepxl)](https://github.com/spacepxl/ComfyUI-HQ-Image-Save) | 99 | Non | Oui | Oui (frames) |
| [**ComfyUI-CoCoTools_IO** (Conor-Collins)](https://github.com/Conor-Collins/ComfyUI-CoCoTools_IO) | 100 | **Oui** (extraction layers, Cryptomatte) | Oui | Oui |
| [**ComfyUI-EXR-API** (nofunstudio)](https://github.com/nofunstudio/ComfyUI-EXR-API) | 0 | Oui | Oui | Oui |
| [**Radiance**](https://github.com/fxtdstudios/radiance) | 182 | Oui | Oui | Oui |
| XteVision Pro (Patreon, payant) | -- | **Oui** (full structure multi-layer + Cryptomatte) | 16 ou 32-bit | Oui |

**CoCoTools_IO** est le plus proche d'un Shuffle node Nuke pour l'extraction de layers EXR. C'est l'option la plus solide pour un pipeline VFX reel.

---

## Utilisation par des coloristes pros

### Ce que j'ai trouve

- **Mixing Light** (plateforme de formation pour coloristes pro, Igor Ridanovic) : a publie une [serie d'articles sur ComfyUI](https://mixinglight.com/color-grading-tutorials/comfyui-localhost-overview-digital-post-production-part-1/) en tant qu'outil generatif complementaire. Pas pour le grading, mais pour l'inpainting, le clean plating, le sketch-to-image. L'interet vient de la confidentialite (tout tourne en local, pas de cloud = compatible NDA).

- **Logik Forums (Flame)** : des artistes Flame integrent ComfyUI dans leur pipeline pour l'extraction de profondeur, normales, et mattes AI. Un thread majeur sur l'[integration ComfyUI comme plugin Flame](https://forum.logik.tv/t/how-to-implement-comfyui-as-an-plugin-for-autodesk-flame/12437) mentionne que les EXR 16/32-bit multi-layer fonctionnent entre Flame et ComfyUI. Un artiste VFX senior (Xteve) utilise ComfyUI depuis 3 ans mais precise que ca n'a pas encore fourni le "bread and butter" quotidien.

- **ActionVFX** : propose un [cours "ComfyUI for VFX"](https://courses.actionvfx.com/comfyui-for-vfx) oriente production (LoRAs, IP Adapters, ControlNet, export EXR propre).

- **ACESCentral** : **zero mention de ComfyUI**. Le forum ne discute pas du tout de ce sujet.

- **liftgammagain.com** : **zero mention de ComfyUI**. Les discussions restent centrees sur Resolve, Baselight, Mistika.

- **Blackmagic Forum** : **zero mention de ComfyUI pour le color grading**.

### Ce qui n'existe pas

Aucun coloriste professionnel (DI, finishing, dailies) n'utilise publiquement ComfyUI comme outil de grading dans un pipeline cinema. Les cas d'usage reels sont :
1. Generation d'images AI avec look applique
2. Clean plating / inpainting en pipeline VFX
3. Extraction de mattes AI (SAM2)
4. Pre-visualisation de looks

---

## Synthese : forces et faiblesses

**Forces reelles de ComfyUI pour la post-production :**
- SAM2 pour le masking/tracking video (game changer)
- Pipeline local, gratuit, compatible NDA
- EXR multi-layer I/O fonctionnel (CoCoTools)
- Ecosystem OCIO/ACES emergent
- Radiance comme proof-of-concept d'une suite complete

**Faiblesses bloquantes pour le color grading cinema :**
- Pipeline interne pas vraiment 32-bit float de bout en bout (clamping 0-1, conversions PIL 8-bit dans beaucoup de nodes)
- Pas de debayering BRAW/ProRes RAW
- Scopes uniquement dans Radiance (pas de node independant)
- Pas de timeline / scrubbing video natif pour grading shot-to-shot
- ACES/OCIO immature (repos de quelques semaines/mois, 4 a 50 stars)
- Aucun coloriste pro n'a adopte l'outil
- Radiance a ete signale comme reduisant a 8-bit en interne dans certains de ses nodes, malgre ses revendications 32-bit

**Pour ton pipeline Goldberg** : ComfyUI peut servir pour le masking AI (SAM2), le clean plating, et eventuellement des looks experimentaux en pre-vis. Pour le grading reel de tes rushes BMPCC/iPhone Apple Log, Resolve reste l'outil. L'idee d'une passerelle Resolve -> EXR -> ComfyUI (pour le AI-assisted) -> EXR -> Resolve est viable avec CoCoTools_IO.

---

Sources:
- [FXTD Studios Radiance - GitHub](https://github.com/fxtdstudios/radiance)
- [Radiance - site officiel](https://radiance.fxtd.org/)
- [Radiance - pIXELsHAM review](https://www.pixelsham.com/2026/03/17/fxtd-studios-radiance-a-professional-vfx-grade-32-bit-float-color-science-suite-for-comfyui/)
- [ComfyUI-HQ-Image-Save - GitHub](https://github.com/spacepxl/ComfyUI-HQ-Image-Save)
- [ComfyUI-HQ-Image-Save - pIXELsHAM](https://www.pixelsham.com/2026/03/09/comfyui-hq-image-save-save-and-load-images-and-latents-as-32bit-exrs/)
- [ComfyUI-CoCoTools_IO - GitHub](https://github.com/Conor-Collins/ComfyUI-CoCoTools_IO)
- [ComfyUI-EXR-API - GitHub](https://github.com/nofunstudio/ComfyUI-EXR-API)
- [ComfyUI-Image-Filters - GitHub](https://github.com/spacepxl/ComfyUI-Image-Filters)
- [ComfyUI-Curve - GitHub](https://github.com/aiaiaikkk/ComfyUI-Curve)
- [ComfyUI-EasyColorCorrector - GitHub](https://github.com/regiellis/ComfyUI-EasyColorCorrector)
- [ComfyUI-Olm-LGG - GitHub](https://github.com/o-l-l-i/ComfyUI-Olm-LGG)
- [CRT-Nodes - GitHub](https://github.com/PGCRT/CRT-Nodes)
- [ComfyUI-post-processing-nodes - GitHub](https://github.com/EllangoK/ComfyUI-post-processing-nodes)
- [Nuke Nodes for ComfyUI - GitHub](https://github.com/sumitchatterjee13/nuke-nodes-comfyui)
- [ComfyUI-ACES-IO - GitHub PR](https://github.com/Comfy-Org/ComfyUI-Manager/pull/2674)
- [ComfyUI-color-tools - GitHub](https://github.com/APZmedia/ComfyUI-color-tools)
- [comfy_oiio - GitHub](https://github.com/melMass/comfy_oiio)
- [ComfyUI-OlmLUT - GitHub](https://github.com/o-l-l-i/ComfyUI-OlmLUT)
- [comfyui-propost - GitHub](https://github.com/digitaljohn/comfyui-propost)
- [lut-maker - GitHub](https://github.com/o-l-l-i/lut-maker)
- [ComfyUI-lut - GitHub](https://github.com/thezveroboy/ComfyUI-lut)
- [comfyui-raw-image - GitHub](https://github.com/dimtion/comfyui-raw-image)
- [SAM2 ComfyUI (kijai) - GitHub](https://github.com/kijai/ComfyUI-segment-anything-2)
- [ComfyUI-SAM2 (neverbiasu) - GitHub](https://github.com/neverbiasu/ComfyUI-SAM2)
- [ComfyUI_LayerStyle - GitHub](https://github.com/chflame163/ComfyUI_LayerStyle)
- [Masquerade Nodes - GitHub](https://github.com/BadCafeCode/masquerade-nodes-comfyui)
- [ComfyUI_RyanOnTheInside - GitHub](https://github.com/ryanontheinside/ComfyUI_RyanOnTheInside)
- [comfyui-tkg-chroma-key - GitHub](https://github.com/p1atdev/comfyui-tkg-chroma-key)
- [ComfyUI Higher Bit Depth - Feature Request #572](https://github.com/comfyanonymous/ComfyUI/issues/572)
- [ComfyUI Bit Depth - Impact Pack Issue #369](https://github.com/ltdrdata/ComfyUI-Impact-Pack/issues/369)
- [HDR VAE Decode - GitHub](https://github.com/netocg/vae-decode-hdr)
- [Mixing Light - ComfyUI Series](https://mixinglight.com/color-grading-tutorials/comfyui-localhost-overview-digital-post-production-part-1/)
- [Logik Forum - ComfyUI + Flame](https://forum.logik.tv/t/comfyui-vfx-in-flame/12683)
- [Logik Forum - ComfyUI Plugin for Flame](https://forum.logik.tv/t/how-to-implement-comfyui-as-an-plugin-for-autodesk-flame/12437)
- [Logik Forum - ComfyUI VFX Course](https://forum.logik.tv/t/comfyui-vfx-course/13430)
- [ActionVFX - ComfyUI for VFX Course](https://courses.actionvfx.com/comfyui-for-vfx)
- [ActionVFX - ComfyUI for VFX Blog](https://www.actionvfx.com/blog/comfyui-for-vfx)
- [ComfyUI-for-Nuke - GitHub](https://github.com/vinavfx/ComfyUI-for-Nuke)