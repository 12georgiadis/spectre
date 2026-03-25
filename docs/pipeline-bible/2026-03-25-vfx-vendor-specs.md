Now I have comprehensive data across all seven areas. Here is the full synthesis.

---

# Specs & requirements techniques imposees par les studios VFX a leurs vendeurs

Recherche exhaustive couvrant les specs de livraison, les pipelines couleur, la preparation des plates, les packages de turnover, et les workflows de review/approbation.

---

## 1. Netflix VFX Best Practices & Delivery Specifications

Netflix est le seul studio majeur a publier ses specs VFX en detail et en public. C'est la reference la plus accessible.

### Resolution
- Image active minimale : **UHD (3840 pixels de large)**
- La resolution de travail doit etre egale ou superieure a la resolution de livraison

### Formats de fichier (plates et shots finaux)
- **16-bit EXR (.exr)** si pipeline ACES ou Camera Linear (Linear AWG, etc.)
- Compression : **uncompressed, ZIP, ou PIZ** (lossless uniquement)
- **16-bit DPX (.dpx)** si pipeline Camera Log (LogC, etc.)
- **10-bit DPX** uniquement si la capture originale etait en 10-bit Log
- **LOG EXR refuses**. Fichiers zippes refuses.

### Naming conventions (shot/version/plate)
Format shot : `showID_episode_sequence_scene_shotID` (ex: `AGM_104_TCC_067_0010`)
Format version : shot + `_task_vendorID_v###` (ex: `AGM_104_TCC_067_0010_comp_NFX_v005`)
Format plate : shot + `_PL##_v###.[frames].exr` (ex: `AGM_104_065_010_PL01_v001.[1001-1058].exr`)
Naming avance des plates : `FG01`, `BG01`, `EL01`, `RF01`, `CH` (chromeball), ou noms descriptifs (`smoke01`, `fire03`)
- Les IDs de shots incrementent par **10** (0010, 0020...) pour permettre les insertions
- Version : commence par `v` + 3-4 chiffres
- Caracteres autorises : `[a-z] [A-Z] [0-9] [.] [_] [-]`

### Slates & overlays
**Slate** (1 frame, premiere frame du media) : Show, WIP/FINAL, Version Name, Date (YYYY-MM-DD), Shot/Asset Types, Scope of Work, Submission Note, Vendor, Shot/Asset Name, Frames (first-last + duree), Media Color (colorspace). Templates fournies en Nuke (.nk) et Photoshop (.psd).
**Overlay** (sur toutes les versions non-finales) : Vendor, Show, Date, Image, Version Name, Frames. Ne doit pas etre blanc 100%. Doit rester a l'interieur du letterboxing.

### VFX Archive (Branded Delivery)
Livrables exiges a la fin du projet :
- **Plates** : EXR/DPX selon specs
- **Shots finaux** : meme format, + proxy QT avec couleur cuite, + mattes (embeddees ou separees)
- **3D models** : hero characters, creatures, props, environments uniquement. Fichiers projet + turntable QT. Organisation en dossiers (geo, textures, shaders, rigging, scene files)
- **Scanning data** : LiDAR + photogrammetry (cleaned/processed, pas de raw point clouds). Cyberscans acteurs/creatures/props
- **Nuke scripts / scene files** : finaux uniquement, pas de caches pre-render. Formats acceptes : `.nk`, `.mb`, `.ma`, `.hip`, `.hda`, `.unity`, `.blend`, `.abc`, `.usd`, `.fbx`, `.obj`, `.ztl`
- **Documentation** : VFX Shot/Asset Status Report, VFX Financial Report, Supervisor Onset/Exposure Reports, VFX Breakdown
- **Hardware/Software/Pipeline "snapshot"** : setup general par vendeur
- **VFX Wrap memo** : template fourni par Netflix

### Review (Content Hub)
- Formats proxy : **MP4 ou MOV**, codecs **H.264, ProRes 422 HQ, ProRes 4444, DNxHD**
- Resolution : 640x480 a 4096x2160
- Films Studio/Indie : review en **4K** obligatoire
- Batches multiples le meme jour : suffixer `_01`, `_02`...

Sources Netflix :
- [VFX Best Practices](https://partnerhelp.netflixstudios.com/hc/en-us/articles/360000611467-VFX-Best-Practices)
- [VFX Media Review Delivery Specifications](https://partnerhelp.netflixstudios.com/hc/en-us/articles/360057627253-VFX-Media-Review-Delivery-Specifications)
- [VFX Plate Naming Best Practices](https://partnerhelp.netflixstudios.com/hc/en-us/articles/360055781274-VFX-Plate-Naming-Best-Practices)
- [VFX Shot and Version Naming Recommendations](https://partnerhelp.netflixstudios.com/hc/en-us/articles/360057627473-VFX-Shot-and-Version-Naming-Recommendations)
- [VFX Slates & Overlays Guidelines](https://partnerhelp.netflixstudios.com/hc/en-us/articles/360057627293-VFX-Slates-Overlays-Guidelines)
- [Post Production Branded Delivery Specifications](https://partnerhelp.netflixstudios.com/hc/en-us/articles/7262346654995-Post-Production-Branded-Delivery-Specifications)
- [Situational VFX Delivery Specifications](https://partnerhelp.netflixstudios.com/hc/en-us/articles/21668717059475-Situational-VFX-Delivery-Specifications)
- [Non Graded Archival Master (NAM) Specifications](https://partnerhelp.netflixstudios.com/hc/en-us/articles/21669570043283-Non-Graded-Archival-Master-NAM-Specifications)
- [Content Hub VFX Plate Delivery User Guide](https://partnerhelp.netflixstudios.com/hc/en-us/articles/360055781594-Content-Hub-VFX-Plate-Delivery-User-Guide)

---

## 2. Disney/Marvel, Warner Bros, Amazon, Apple TV+

Ces studios ne publient pas leurs specs VFX specifiques (sous NDA avec les vendeurs). Voici ce qui est publiquement accessible :

### Disney/Marvel
- Pipeline multi-vendeur decentralise, 800+ shots par vendeur possible sur un MCU
- Poussent vers la livraison HDR tout au long du pipeline
- Plates Lab : objectif que les plates livrees aux VFX representent exactement ce qui est vu sur le plateau
- Les specs VFX specifiques sont distribuees directement aux vendeurs sous NDA

### Amazon MGM Studios
- Portal officiel : [portal.amazonstudios.com](https://portal.amazonstudios.com/hc/en-us/sections/15582177580059-Post-Production)
- Archive OCF (Original Capture Files) exigee pendant le tournage
- Livraison : NAM, DSM, Audio/ProTools, Color Grading, Conforms, Editorial, VFX a la fin du post
- LTO et disques durs decourages : solution cloud AWS
- Dolby Vision : Full Range Video/Data levels
- Audio : loudness -27 +/- 2 LU (1770-1 avec Dialogue Gating)
- Tous les vendeurs VFX doivent etre approuves par ecrit par le Post Executive

### Apple TV+
- Portal : [tvpartners.apple.com](https://tvpartners.apple.com/support/3666-asset-requirements-delivery)
- Livraison via packages `.itmsp` avec `metadata.xml`
- Upload via l'outil Transporter (CLI Apple)
- QC obligatoire avant livraison

### Warner Bros Discovery / HBO
- Content Partner Hub : [partnerhub.warnermediagroup.com](https://partnerhub.warnermediagroup.com/)
- SDR : livraison HD minimum (1920x1080), frame rate natif constant, aspect ratio original
- HDR : aligne frame-a-frame avec le SDR. Dolby Vision metadata frame-wrapped dans le MXF
- Dolby Atmos : IAB Level 0, mix 7.1.4 minimum
- Conform IMP : SMPTE ST 2067
- Audio loudness : -24.0 LKFS +/-2.0 LU (ITU-R BS.1770-3/4)
- HBO VFX Production Coordinator Bootcamp (CAVE Academy) couvre les specs, count sheets, workflows editorial

---

## 3. EXR Specifications Standard

### Bit depth
- **16-bit half-float** : standard de livraison universel. 30 stops de dynamic range, 1024 valeurs par stop.
- **32-bit float** : deep compositing, depth buffers, precision extreme
- **32-bit unsigned integer** : cas specifiques

### Channels
- Multi-channel dans un seul fichier : RGB, alpha, specular, diffuse, normals, depth, mattes
- Convention de nommage : **`layer.channel`** (ex: `light1.R`, `light1.specular.G`, `diffuse.R`)
- Layers imbriquees : `L1.L2.channel`

### Compression (tableau recapitulatif)

| Compression | Type | Usage recommande |
|---|---|---|
| **None** | Non compresse | Non recommande |
| **RLE** | Lossless | Zones plates/alpha |
| **ZIP (1 scanline)** | Lossless | General |
| **ZIP (16 scanlines)** | Lossless | Images rendues sans grain |
| **PIZ** | Lossless | **Images avec grain** (le plus efficace) |
| **PXR24** | Legerement lossy | Contribution Pixar, 32-bit > 24-bit |
| **B44/B44A** | Lossy | Playback temps reel |
| **DWAA/DWAB** | Lossy | Contribution DreamWorks, JPEG-like |

**Regle studio** : livraison en **ZIP ou PIZ uniquement** (lossless). Lossy interdit pour les deliverables finaux.

### Metadata EXR obligatoires
- **Timecode** (source, pas timeline)
- **Frame range** (start/end)
- **Shot name** (convention studio)
- **Slate/scene info** (episode, scene, take)
- **Color space / ACES version**
- **Vendor name**, scope of work

Sources :
- [OpenEXR Technical Introduction](https://openexr.com/en/latest/TechnicalIntroduction.html)
- [OpenEXR Codecs Explained](https://www.learnvfx.com/p/openexr-codecs-explained)
- [What Is An EXR File (MASV)](https://massive.io/file-transfer/what-is-an-exr-file/)

---

## 4. Color Pipeline : ACES, CDL, Nuke

### ACES Processing Pipeline
`Camera Data > IDT (Input Device Transform) > Working Space (ACEScg) > Look Transform (optionnel) > RRT+ODT (Output Transform) > Display`

### Color spaces cles
| Color Space | Encoding | Primaries | Usage |
|---|---|---|---|
| **ACES2065-1** | Lineaire | AP0 | Archivage, echange |
| **ACEScg** | Lineaire | AP1 | **Working space VFX/CG** |
| **ACEScc** | Logarithmique | AP1 | Grading |
| **ACEScct** | Log + toe | AP1 | Grading (toe Cineon) |
| **ACESproxy** | Log, integer | AP1 | On-set, SDI |

### OCIO Config
- Fichier config unique lu par tous les outils OCIO-compatibles (Maya, Houdini, Nuke, Mari, Katana, Blender, Substance)
- Configs officielles : [OpenColorIO-Config-ACES GitHub](https://github.com/AcademySoftwareFoundation/OpenColorIO-Config-ACES/releases)
- Config CG (lightweight, sans IDTs camera) vs Config Studio (complete)
- ACES 2.0 : RRT+ODT fusionnes en un seul Output Transform

### CDL (Color Decision List)
- Format ASC CDL : 4 operations reversibles (slope, offset, power, saturation)
- Livraison en sidecar `.cc` ou `.cdl` (jamais cuit dans les plates)
- Pre-grade deconseille ; si applique, doit etre reversible
- Les CDLs et LUTs dailies doivent accompagner les plates pour que le vendeur puisse matcher les proxies editorial

### Nuke et ACES
- Nuke est concu pour travailler en lineaire (natif ACES)
- OCIO integre nativement dans les versions recentes
- Livraison : rendu en EXR lineaire 16-bit, non-clampe, Output Transform applique au viewer uniquement (pas cuit)

### Round-trip testing obligatoire
- Tests entre VFX vendeurs, editorial, et DI **avant le debut de la production VFX**
- Verifier que les pixels non-modifies restent intacts
- Verifier que les normalization grades sont inversibles
- Meme version ACES sur toute la chaine

Sources :
- [ACES and OCIO for VFX Artists (CG Lounge)](https://cglounge.studio/journal/color-pipeline-aces-ocio-for-vfx)
- [ACES VFX Quick Start Guide (PDF)](https://community.acescentral.com/uploads/default/original/1X/25ec1472d70b169ceabb215beacdd501d1a27fac.pdf)
- [ACES Explained (Foundry)](https://www.foundry.com/insights/film-tv/academy-color-encoding-system-explained)
- [Frame.io ACES Workflow Guide](https://workflow.frame.io/guide/aces)
- [ACES chapter (Chris Brejon)](https://chrisbrejon.com/cg-cinematography/chapter-1-5-academy-color-encoding-system-aces/)

---

## 5. Plate Preparation

### Debayer
- Fait par le DI facility ou un vendeur dedie aligne sur le pipeline DI (pas par le vendeur VFX)
- Debayer vers un color space standardise (ACES de preference)
- Un seul debayer pour toute la production, eviter les inconsistances

### Lens distortion
- Charts de distortion par type de lentille, par mode capteur, tournes pendant les tests camera
- Pipeline standard : **plate originale > undistort > camera tracking > CG pipeline (undistorted) > render CG avec overscan > distort le render CG > composite sur la plate originale**
- Overscan requis pour le render CG (sinon les bords sont croppes quand la distortion est re-appliquee)
- Rolling shutter : deux workflows possibles (corriger la plate, ou ajouter le rolling shutter au CG)

### Grain
- Reference grain tournee sur le plateau : grey card + Macbeth chart, un par ISO/ASA
- Workflow standard : **degrain la plate > comp > regrain a la fin**
- Degrain necessaire pour : keys green screen plus propres, paint/cleanup, freeze frames sans grain fige

### Handles (frames supplementaires)
- Standard industrie : **8 ou 12 frames** head + tail
- 8 frames = 16 frames supplementaires total (0.67s a 24fps)
- 12 frames = 24 frames supplementaires total (1s a 24fps)
- Frame 1001 comme debut standard (buffer de 1000 frames avant zero)
- Les handles doivent etre inclus dans les plates livrees

### Scaling
- Methodologie standardisee entre vendeurs (resize filters, ordre des operations)
- Instructions step-by-step du plate resolution au delivery resolution
- Coordination avec Post/VFX Supervisor pour les crops et reframes

Sources :
- [Shooting Plates for VFX (CAVE Academy)](https://caveacademy.com/wiki/onset-production/data-acquisition/data-acquisition-training/shooting-plates-for-vfx/)
- [Lens Distortion Workflow (VFX Camera Database)](https://vfxcamdb.com/lens-distortion-workflow-2/)
- [Shooting Grain Reference (CAVE Academy)](https://caveacademy.com/wiki/onset-production/data-acquisition/data-acquisition-training/shooting-grain-reference/)
- [Why VFX Plates Start at Frame 1001](https://topicroomsvfx.com/articles/why-do-vfx-plates-start-at-frame-1001/)

---

## 6. Review/Approval : ShotGrid, ftrack, Versioning

### Outils principaux
- **Autodesk Flow Production Tracking** (ex-ShotGrid) : tracking tasks/budgets/timelines, review media, versioning, integrations pipeline
- **ftrack** : review frame-accurate dans le navigateur, export sessions en PDF avec thumbnails/timecodes/notes

### Status workflow type
`Prep > WIP > Version submitted > Internal Dailies > Client Review > Approved > Tech Check > Published/Final`

### Hierarchie des statuts dans ShotGrid
- **Asset/Shot Status** : top-level, visible client
- **Discipline Status** : par stage pipeline (Model, Texture, Rigging, Animation, Lighting)
- **Version Status** : par version individuelle + tech checks
- **Published Status** : reference version approuvee

### Versioning
- `v001`, `v002`... incremental
- Version soumise a Dailies interne, si refusee retour en WIP pour nouvelle version
- FINAL declenche PUBLISHED-FINAL au niveau Discipline et Publish

### Integration pipeline
- Nuke Studio / Hiero : import multi-shot depuis ShotGrid/ftrack, comparaison versions sur timeline
- Python API pour automatisation et integration pipeline
- Integrations Maya, Nuke, Hiero, Flame, Photoshop, Jira, Deadline

Sources :
- [Production Statuses (CAVE Academy)](https://caveacademy.com/wiki/production/production-statuses/)
- [Autodesk Flow Production Tracking](https://www.autodesk.com/products/flow-production-tracking/overview)
- [ShotGrid Community Best Practices](https://community.shotgridsoftware.com/t/best-practices-for-publishedfile-and-version-entities/18785)

---

## 7. Turnover Package : ce qu'un studio envoie au vendeur VFX

Un "VFX Turnover" est le package complet que le studio (via l'editorial) envoie au vendeur. Contenu standard :

### Elements principaux
1. **Plates** : image sequences (EXR/DPX) debayerees, en color space standardise, avec handles
2. **CDLs/LUTs** : dailies color reference + QT ou reference frames pour matcher le proxy editorial
3. **Camera data** : slate, scene, shoot location, camera body, lentille, focale, metadata capteur
4. **LiDAR scans** : .e57, OBJ a differents LODs, fichiers RTC originaux. Mesh prefere aux point clouds
5. **HDRIs** : 32-bit, domes, parallax-correct geometry, alignment helpers
6. **Photogrammetry** : modeles des sets et props
7. **Reference photography** : photos du plateau, textures, details

### Documents
8. **Count sheets / Lineup sheets** : chaque shot, travail requis, elements necessaires, timing
9. **Pull lists / EDLs** : instructions pour le post facility (scan/render DPX)
10. **Editorial reference** : locked reference video, slap comps de l'editeur
11. **Framing charts** : resolution capteur exacte vs resolution active vs resolution delivery
12. **Distortion charts** : par lentille, par mode capteur

### Gestion du turnover
- Prepare par l'assistant editeur ou VFX editor en coordination avec le lab
- Transfer via Aspera, GlobalData, MASV Rush, ou solution maison
- Data capture (LiDAR, HDRI, photo) : gere par le departement VFX avec ses propres data wranglers

Sources :
- [Feature Turnover Guide (Evan Schiff, ACE)](https://www.evanschiff.com/articles/feature-turnover-guide-vfx/)
- [Turnover to VFX (FSU Film Handbook)](https://fsufilmhandbook.com/turnover-to-vfx/)
- [VFX Delivery to Editorial (FSU Film Handbook)](https://fsufilmhandbook.com/vfx-delivery-to-editorial/)

---

## Document de reference industrie : VES Transfer Specifications

Le **VES Transfer Spec** est le template de reference pour toute l'industrie, maintenu par le Technology Committee de la Visual Effects Society. Il definit comment les vendeurs VFX doivent ingerer, traiter et livrer les donnees image.

- Package : **Guide** (introduction aux differents aspects) + **Sample** (exemple complet base sur une production reelle)
- Compatible avec le MovieLabs VFX Image Sequence Naming spec
- Format filename MovieLabs : `<showid>_<vfx-sequence>_<vfx-shot>_<image-type>_<vendor-code>_<revision-code>{_<alternate>}{_<camera-reference>}{_<identifying-description>}`
- Objectif : reduire les iterations necessaires pour etablir le pipeline de production

Sources :
- [VES Transfer Specifications (site officiel)](https://vestransferspec.org/)
- [VES Transfer Spec (GitHub)](https://github.com/ves-tech/vestransferspec)
- [MovieLabs VFX Image Sequence Naming (PDF)](https://movielabs.com/prodtech/sdw/vfx/ETC-ImageSequenceNaming-v1.0-063020-FINAL.pdf)
- [MovieLabs VFX Naming and Workflows](https://movielabs.com/interoperable-workflows/vfx-naming-and-workflows/)