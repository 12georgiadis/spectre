J'ai maintenant assez de matière pour une synthèse complète. Voici le rapport.

---

# Photogrammes etalonnés pour training IA : recherche exhaustive

## 1. Format d'input pour training LoRA (kohya_ss, ai-toolkit, SimpleTuner)

**Aucun des trois frameworks ne supporte EXR 32-bit nativement.** Il faut convertir en PNG 8-bit (ou JPEG haute qualité) avant le training.

### Formats supportés

| Format | kohya_ss | ai-toolkit (Ostris) | SimpleTuner (bghira) |
|--------|----------|---------------------|----------------------|
| JPEG (.jpg/.jpeg) | oui | oui | oui |
| PNG (.png, 8-bit) | oui | oui | oui |
| WebP | oui | instable (bugs) | oui |
| BMP | oui | non | oui |
| JPEG XL (.jxl) | non | non | oui (extra optionnel) |
| TIFF | non (pas dans IMAGE_EXTENSIONS) | non | probablement via Pillow |
| EXR (32-bit float) | non | non | non |

**Kohya sd-scripts** : les extensions sont hard-codées dans `library/train_util.py` comme `[".png", ".jpg", ".jpeg", ".webp", ".bmp"]`. Pillow (utilisé en backend) ne lit pas EXR nativement. TIFF n'est pas dans la liste d'extensions autorisées, meme si Pillow le supporte techniquement.

**AI-Toolkit** : le plus restrictif, uniquement JPG/JPEG et PNG. WebP a des bugs connus.

**SimpleTuner** : le plus flexible, supporte JPEG XL en option (`pip install simpletuner[jxl]`), mais pas EXR.

**Recommandation** : convertir les EXR en PNG 8-bit sRGB. Le pipeline de training encode les images en latents via le VAE en fp16/bf16, donc la precision 32-bit de l'EXR n'apporte rien au training. Un PNG 8-bit bien etalonné suffit.

### Repos principaux
- [kohya-ss/sd-scripts](https://github.com/kohya-ss/sd-scripts)
- [ostris/ai-toolkit](https://github.com/ostris/ai-toolkit)
- [bghira/SimpleTuner](https://github.com/bghira/SimpleTuner)

---

## 2. LTX-Video 2 training : format, resolution, color space

### Format d'input
- **Videos** : MP4 (H.264/H.265). Le dataset doit etre homogene (tout video OU tout image, pas de mix).
- **Images** : PNG/JPEG, en utilisant `1` comme frame count dans le bucket de resolution (ex: `960x544x1`).
- **Captions** : fichiers .txt avec le meme nom de base que chaque media. 15-30 fichiers recommandés.
- **Metadata** : CSV, JSON ou JSONL.

### Contraintes de resolution
- Dimensions spatiales : **multiples de 32**
- Nombre de frames : doit satisfaire `frames % 8 == 1` (1, 9, 17, 25, 33, 41, 49, 57, 65, 73, 81, 89, 97, 121...)
- Resolution recommandée : sous 720x1280, frames sous 257
- Exemples courants : `960x544x49`, `768x448x89`, `512x512x121`

### Color space
La documentation officielle de Lightricks ne specifie pas de color space requis. Le modele est entraine sur des videos standard (web, MP4), donc **sRGB / BT.709** est le standard attendu par convention. Pas de traitement HDR ou Log.

### Preprocessing
Le script officiel `process_dataset.py` gere le VAE-encoding des latents. Option `--decode` pour verifier visuellement. Option `--with-audio` pour l'audio-video joint. Le trigger word est prepended automatiquement aux captions.

### Hardware
- H100 80GB recommande pour le full fine-tuning
- 24GB GPU possible pour LoRA avec gradient checkpointing et resolutions reduites
- Temps : 2-8 heures sur 24GB GPU

### Repos
- [Lightricks/LTX-2](https://github.com/Lightricks/LTX-2) (officiel, avec trainer integré)
- [Dataset preparation doc](https://github.com/Lightricks/LTX-2/blob/main/packages/ltx-trainer/docs/dataset-preparation.md)
- [Training doc](https://docs.ltx.video/open-source-model/ltx-2-trainer/ltx-2-training)
- [fal.ai LTX-2 Trainer](https://fal.ai/models/fal-ai/ltx2-video-trainer) (cloud)
- [WaveSpeedAI LTX-2 Trainer](https://wavespeed.ai/blog/posts/introducing-wavespeed-ai-ltx-2-19b-video-lora-trainer-on-wavespeedai/)

---

## 3. Flux 2 training : format photogrammes, color space

### Format
- **PNG et JPEG** : standards universels. PNG recommande pour la qualite sans perte.
- 20-1000 images pour un style LoRA, 15-20 images suffisent pour un character LoRA.
- Captions obligatoires (trigger word + description).

### Resolution
- Multi-resolution supportée : 256, 512, 768, 1024, 1280, 1536 (selon config FLUX Klein)
- Aspect bucketing automatique (pas besoin de crop/resize manuellement)

### Color space
**sRGB attendu.** Les images PNG/JPEG standard sont deja en sRGB. Le VAE de FLUX encode en espace latent 16-channel, la conversion interne est geree par le pipeline.

Un bug connu existe a l'inference (pas au training) : un "dark halo" sur les bords des masques en inpainting, soupçonné comme artefact gamma sRGB. Un workaround existe : sandwich sRGB -> Linear -> k-Sampler -> Linear -> sRGB. Mais ca concerne l'inference, pas la preparation des donnees de training.

**Ne pas soumettre d'images en linear ou Log.** Le VAE n'est pas entraine pour decoder du linear.

### Repos et guides
- [FLUX.2 Training Guide 2026 (Medium)](https://kgabeci.medium.com/flux-2-lora-training-the-complete-2026-guide-from-someone-who-built-the-training-platform-14d0bcb396eb)
- [fal.ai FLUX.2 Trainer](https://fal.ai/models/fal-ai/flux-2-trainer)
- [FLUX Klein Training Docs (BFL)](https://docs.bfl.ai/flux_2/flux2_klein_training_example)
- [Apatero: Flux 2 LoRA Complete Guide](https://apatero.com/blog/flux-2-lora-training-complete-guide-2025)

---

## 4. Impact du color grading sur le training

C'est LA question clé. Synthese des meilleures pratiques 2025-2026 :

### Regle generale

| Type d'image | Utiliser pour training ? | Raison |
|---|---|---|
| **Raw/Log (non etalonné)** | NON | Plat, desature. Le modele apprend un look delave. |
| **Etalonnage neutre (Rec.709 / balance)** | OUI, meilleure pratique | Couleurs realistes, expo correcte, bonne balance des blancs. Meilleure generalisation. |
| **Etalonnage stylistique fort (LUT creative)** | UNIQUEMENT pour un style LoRA intentionnel | Cree un biais couleur fort. A eviter pour les LoRA sujet/personnage. |
| **Mix d'etalonnages dans un dataset** | NON | Introduit de l'incoherence, confond le modele. |

### Pourquoi pas en Log/Raw ?
Le footage Log est volontairement plat, faible saturation, faible contraste. Entrainer la-dessus apprend au modele a produire des images grises et delavees. Les modeles de diffusion sont entraines sur des images "display-referred" (sRGB/Rec.709), pas sur du footage camera raw.

### Pourquoi pas en grade stylistique agressif ?
- Les guides de training listent explicitement les "heavy filters or editing" comme elements a eviter dans les datasets.
- Le color clustering apparait tot dans le training LoRA : si le dataset tire vers le chaud, le modele amplifie. Un grade stylistique etroit augmente le risque d'overfitting.
- Sauf si l'objectif est precisement d'enseigner un style couleur specifique (ex: un LoRA "look Kodak Vision3 250D").

### Ce que font les pros (cinema + IA)
Un paper recent (arXiv, 2025) decrit un pipeline de fine-tuning cinematique de Wan2.1 I2V-14B avec LoRA sur ~50 clips de court metrage. Les chercheurs montrent que le LoRA peut internaliser la "grammaire cinematique" : coherence de temperature de couleur, profondeur de champ, rythme des scenes. Le dataset etait de ~25,000 paires frame-caption (16 minutes de footage total). Source : [Fine-Tuning Open Video Generators for Cinematic Scene Synthesis](https://arxiv.org/html/2510.27364v1)

### Recommandation concrete pour ton workflow
1. Exporter les photogrammes depuis Resolve/Premiere avec un etalonnage **technique** : CST (Color Space Transform) du profil camera vers Rec.709, correction d'exposition et balance des blancs
2. **Ne pas** appliquer de LUT creative/look
3. **Si** tu veux un LoRA qui reproduit un look specifique (ex: ton etalonnage "Goldberg"), alors applique cette LUT de façon consistante sur TOUT le dataset et c'est le but explicite du training
4. Exporter en PNG 8-bit sRGB

---

## 5. Inpainting/Outpainting depuis ComfyUI : garder la coherence couleur

### Le probleme
Apres inpainting, le VAE encode/decode introduit des shifts de couleur. L'image entiere peut subir un changement drastique de teinte par rapport a l'original graded.

### Nodes essentiels

**ColorMatch (KJNodes)** : transfert de caracteristiques couleur d'une image reference vers le resultat. Utilise les algorithmes Reinhard et al., Pitie et al., et MVGD + histogram matching. Ideal pour re-appliquer le look du photogramme original apres inpainting.
- [RunComfy: ColorMatch Node](https://www.runcomfy.com/comfyui-nodes/ComfyUI-KJNodes/ColorMatch)

**VAE Color Corrector (EasyColorCorrector)** : corrige specifiquement les shifts induits par le VAE en comparant l'image pre-VAE et post-VAE. Inclut aussi un mode batch pour les sequences et un module Film Emulation avec highlight rolloff.
- [GitHub: ComfyUI-EasyColorCorrector](https://github.com/regiellis/ComfyUI-EasyColorCorrector)

**INPAINT_ColorCorrection** : ajuste les couleurs en utilisant une image reference avec masque optionnel. Permet de corriger uniquement les zones inpaintees.
- [Documentation](https://comfyai.run/documentation/INPAINT_ColorCorrection)

**ColorMatchImage (ComfyUI-Image-Filters)** : algorithmes avances de matching couleur pour assurer la coherence entre images.
- [RunComfy: ColorMatchImage](https://www.runcomfy.com/comfyui-nodes/ComfyUI-Image-Filters/ColorMatchImage)

### Workflow recommandé

```
Photogramme graded (original)
    |
    v
[Masque inpainting/outpainting]
    |
    v
[FLUX Fill / FLUX Klein] avec mask-aware conditioning
    |
    v
[VAE Decode]
    |
    v
[VAE Color Corrector] (compare original pre-VAE vs resultat post-VAE)
    |
    v
[ColorMatch] (reference = photogramme original graded)
    |
    v
Resultat final (couleur coherente avec l'original)
```

Le workflow **FLUX Klein Unified Image Editing** integre deja un color-match pass en fin de chaine. Il combine mask-aware conditioning, multi-reference latent guidance, et color harmonization.
- [FLUX Klein Workflow (RunComfy)](https://www.runcomfy.com/comfyui-workflows/flux-klein-unified-image-editing-inpaint-remove-outpaint-in-comfyui-advanced-image-restoration)

### Tips
- Denoising strength 0.0-0.3 pour des modifications subtiles qui preservent l'original
- Soft Inpainting pour gradient blending automatique aux bords du masque
- Travailler par petites zones plutot qu'un outpaint massif d'un coup
- Utiliser le FluxGuidance node pour guider l'expansion vers le meme style que l'original

---

## 6. Camera motion LoRA : entrainer des mouvements de camera depuis des rushes reels

### Methodes principales

**A. AnimateDiff-MotionDirector** (SD 1.5, le plus mature)
- Entrainement d'un MotionLoRA depuis une video reelle
- ~300 steps par video avec AdamW, ~150 avec Lion optimizer
- Output : fichier .safetensors compatible MotionLoRA (~128MB par checkpoint)
- Methode la plus simple : via ComfyUI nodes (ADMD), pas besoin de CLI
- Training sur 16 frames (poussable a 24, 32 peut OOM)
- [GitHub: AnimateDiff-MotionDirector](https://github.com/ExponentialML/AnimateDiff-MotionDirector)
- [Guide: Easiest Way to Train Motion LoRA](https://weirdwonderfulai.art/resources/easiest-way-to-train-your-own-motion-lora-for-animatediff/)

**B. LiON-LoRA (ICCV 2025)** : framework academique avance
- Controle decouplé camera + objet via fusion LoRA orthogonale
- Scalabilité lineaire des amplitudes de mouvement
- Controle via token dans le DiT (Diffusion Transformer)
- [Paper: LiON-LoRA (arXiv)](https://arxiv.org/abs/2507.05678)

**C. Video2LoRA (arXiv mars 2026)** : hypernetwork pour poids LoRA par video
- Predit des poids LoRA personnalises pour chaque input semantique
- Pas besoin de training par condition : un seul training, puis generalisation
- Apprend le mouvement, le style, le camera work depuis des videos reference
- [Paper: Video2LoRA (arXiv)](https://arxiv.org/abs/2603.08210)

**D. CameraCtrl (ICLR 2025)** : controle camera precis
- Entraine sur RealEstate10K (~65K clips video)
- Camera encoder + injection de features camera dans le modele
- Approche en 2 phases : d'abord un image LoRA sur les frames du dataset, puis training du modele de controle camera
- [Paper (ICLR 2025)](https://proceedings.iclr.cc/paper_files/paper/2025/file/f98fd73d59d8494489ea970747b91fe4-Paper-Conference.pdf)

**E. Training video LoRA pratique (Wan 2.2, LTX-2)**
- Pour Wan 2.2 : le high-noise transformer (timesteps ~1000 -> 875) controle le mouvement camera et la trajectoire. Entrainer les deux stages (high-noise + low-noise) pour de bons resultats.
- Pour les motion LoRA purs, pas besoin de trigger word : le comportement est encode dans des phrases naturelles ("orbit 360 around the subject").
- [RunPod: Complete Guide to Training Video LoRAs](https://runpod.ghost.io/complete-guide-to-training-video-loras/)
- [Wan 2.2 LoRA Training (RunComfy)](https://www.runcomfy.com/trainer/ai-toolkit/wan-2-2-i2v-14b-lora-training)

### Preparation du dataset camera motion
1. **Selectionner des clips** avec un mouvement de camera clair et repetitif (pan, tilt, zoom, dolly, orbit)
2. **Couper en segments courts** (4-5 secondes, le training tronque automatiquement)
3. **Captionner** avec description du mouvement : "slow pan left across a landscape", "smooth dolly forward through a corridor"
4. **Consistance** : tous les clips doivent montrer le meme type de mouvement si le but est d'enseigner un mouvement specifique
5. **10-50 clips** suffisent pour un motion LoRA viable

---

## Synthese rapide

| Question | Reponse courte |
|---|---|
| EXR 32-bit pour training ? | Non supporté. Convertir en PNG 8-bit sRGB. |
| LTX-Video 2 color space ? | sRGB/BT.709 standard. Pas de Log. |
| Flux 2 color space ? | sRGB. Ne pas soumettre de linear ou Log. |
| Grade avant training ? | Etalonnage technique (Rec.709) = oui. LUT creative = seulement si c'est le but. Log/raw = non. |
| Inpaint graded dans ComfyUI ? | ColorMatch + VAE Color Corrector + FLUX Klein workflow. |
| Camera motion LoRA ? | AnimateDiff-MotionDirector (pratique), Video2LoRA (recherche), CameraCtrl (precis). |

Sources:
- [kohya-ss/sd-scripts (GitHub)](https://github.com/kohya-ss/sd-scripts)
- [ostris/ai-toolkit (GitHub)](https://github.com/ostris/ai-toolkit)
- [bghira/SimpleTuner (GitHub)](https://github.com/bghira/SimpleTuner)
- [Lightricks/LTX-2 (GitHub)](https://github.com/Lightricks/LTX-2)
- [LTX-2 Dataset Preparation](https://github.com/Lightricks/LTX-2/blob/main/packages/ltx-trainer/docs/dataset-preparation.md)
- [LTX-2 Training Docs](https://docs.ltx.video/open-source-model/ltx-2-trainer/ltx-2-training)
- [fal.ai LTX-2 Trainer](https://fal.ai/models/fal-ai/ltx2-video-trainer)
- [fal.ai FLUX.2 Trainer](https://fal.ai/models/fal-ai/flux-2-trainer)
- [FLUX.2 LoRA Training 2026 Guide (Medium)](https://kgabeci.medium.com/flux-2-lora-training-the-complete-2026-guide-from-someone-who-built-the-training-platform-14d0bcb396eb)
- [FLUX Klein Training (BFL Docs)](https://docs.bfl.ai/flux_2/flux2_klein_training_example)
- [Flux VAE sRGB Color Shift Discussion (HuggingFace)](https://huggingface.co/black-forest-labs/FLUX.1-dev/discussions/513)
- [Fine-Tuning Video Generators for Cinematic Synthesis (arXiv)](https://arxiv.org/html/2510.27364v1)
- [LiON-LoRA (arXiv / ICCV 2025)](https://arxiv.org/abs/2507.05678)
- [Video2LoRA (arXiv mars 2026)](https://arxiv.org/abs/2603.08210)
- [AnimateDiff-MotionDirector (GitHub)](https://github.com/ExponentialML/AnimateDiff-MotionDirector)
- [CameraCtrl (ICLR 2025)](https://proceedings.iclr.cc/paper_files/paper/2025/file/f98fd73d59d8494489ea970747b91fe4-Paper-Conference.pdf)
- [ComfyUI ColorMatch Node (KJNodes)](https://www.runcomfy.com/comfyui-nodes/ComfyUI-KJNodes/ColorMatch)
- [ComfyUI-EasyColorCorrector (GitHub)](https://github.com/regiellis/ComfyUI-EasyColorCorrector)
- [INPAINT_ColorCorrection Node](https://comfyai.run/documentation/INPAINT_ColorCorrection)
- [FLUX Klein Unified Editing Workflow](https://www.runcomfy.com/comfyui-workflows/flux-klein-unified-image-editing-inpaint-remove-outpaint-in-comfyui-advanced-image-restoration)
- [WaveSpeedAI LTX-2 Trainer](https://wavespeed.ai/blog/posts/introducing-wavespeed-ai-ltx-2-19b-video-lora-trainer-on-wavespeedai/)
- [RunPod: Complete Guide to Video LoRAs](https://runpod.ghost.io/complete-guide-to-training-video-loras/)
- [Wan 2.2 I2V LoRA Training (RunComfy)](https://www.runcomfy.com/trainer/ai-toolkit/wan-2-2-i2v-14b-lora-training)
- [Easiest Way to Train Motion LoRA (AnimateDiff)](https://weirdwonderfulai.art/resources/easiest-way-to-train-your-own-motion-lora-for-animatediff/)
- [Apatero: LTX-2 LoRA Training Guide](https://apatero.com/blog/ltx-2-lora-training-fine-tuning-complete-guide-2025)
- [Best Practices LoRA Z-Image 2026](https://dev.to/gary_yan_86eb77d35e0070f5/best-practices-for-training-lora-models-with-z-image-complete-2026-guide-4p7h)