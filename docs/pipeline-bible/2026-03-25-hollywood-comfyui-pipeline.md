I now have comprehensive data across all six research areas. Let me compile the definitive report.

---

# Recherche exhaustive : ComfyUI dans les pipelines cinema/VFX professionnels

## 1. PIPELINES HOLLYWOODIENS AVEC IA GENERATIVE

### Studios majeurs : approche proprietary, pas ComfyUI direct

Les grands studios VFX n'utilisent pas ComfyUI en production finale. Ils construisent des outils propriétaires :

**DNEG / Brahma** : En fevrier 2025, Brahma (la division AI de DNEG, valorisee 1.43 milliard USD) a acquis Metaphysic, la startup qui a de-age Tom Hanks dans *Here* de Zemeckis. L'ensemble reunit 800+ ingenieurs. Leur plateforme CLEAR permet le developpement AI in-house. Pas de ComfyUI, pas de Stable Diffusion en production : tout est custom, neural rendering proprietaire.
- Source : [Variety](https://variety.com/2025/film/global/metaphysic-acquired-dneg-ai-here-1236311233/), [BusinessWire](https://www.businesswire.com/news/home/20250218546095/en/), credibilite : elevee (annonces officielles + couverture presse majeure)

**Netflix / Eyeline Studios (Scanline VFX)** : Pour *El Eternauta* (avril 2025), Eyeline Studios (le labo innovation interne) a produit les premiers plans IA generes en "final pixel" dans une serie Netflix originale. Un building collapse a Buenos Aires realise 10x plus vite et a une fraction du cout traditionnel. Technologies : 3D Gaussian Splatting, simulation hybride + inpainting par diffusion. Eyeline Labs a publie un paper SIGGRAPH Asia 2025 : "Detail Enhanced Gaussian Splatting for Large-Scale Volumetric Capture".
- Sources : [postPerspective](https://postperspective.com/ai-in-vfx-netflix-tests-the-creative-waters/), [Financial Content](https://markets.financialcontent.com/bpas/article/tokenring-2026-1-8-the-ai-revolution-in-cinema-how-netflixs-el-eternauta-redefined-the-vfx-pipeline), credibilite : elevee (declarations Ted Sarandos + paper SIGGRAPH)

**Metaphysic / *Here*** : 53 minutes de face replacement complet dans le film. Pipeline : training neural network sur images de reference jeunesse des acteurs + systeme temps reel "Metaphysic Live" sur le plateau (6 frames de delai). Preview on-set + raffinement post-prod pour le 4K cinema. Pas de Stable Diffusion : modeles proprietaires entraines sur footage licencie.
- Sources : [fxguide podcast #379](https://www.fxguide.com/fxpodcasts/a-deep-dive-into-the-filmmaking-of-here-with-kevin-baillie/), [Creative Bloq](https://www.creativebloq.com/ai/this-is-how-ai-helped-radically-de-age-tom-hanks-in-the-new-movie-here), credibilite : elevee (interview Kevin Baillie, VFX supervisor)

**Studio Freewillusion (Seoul/LA)** : Pipeline AFX lance en decembre 2025, teste sur le film coreeen *Run to the West* (350+ plans VFX en 50% moins de temps) et le film Hollywood *KOBOLD* (2026). Trois modules : neural rendering re-lighting, Xpanza (outpainting video), Taylor Dub (lip-sync multilingue).
- Source : [Globe and Mail / AccessNewswire](https://www.theglobeandmail.com/investing/markets/markets-news/ACCESS%20Newswire/36384995/), credibilite : moyenne (communique de presse, pas encore de review independante)

**Disney / OpenAI** : Partenariat d'un milliard USD (decembre 2025). ILM a debute le "generative neural rendering" avec *Star Wars: Field Guide* en 2025. Disney a construit l'Office of Technology Enablement (OTE, 100+ experts) pour integrer l'IA a l'echelle de l'entreprise, entrainant les modeles exclusivement sur son propre IP.
- Source : [Financial Content / Tokenring](https://markets.financialcontent.com/wral/article/tokenring-2026-1-2-the-magic-of-the-machine-how-disney-is-reimagining-entertainment-through-generative-ai-integration), credibilite : moyenne-haute (analyse business, confirmee par Disney SEC filings)

### SIGGRAPH 2025 : les presentations cles

- Atelier hands-on ComfyUI : "Innovating With Generative AI: A Hands-On ComfyUI Workshop"
- HBO *The Penguin* : workflow AI pour le Gotham inonde
- *Here* de Zemeckis : presentation par Kevin Baillie + Jo Plaete (Metaphysic)
- Autodesk Flow Studio (ex-Wonder Dynamics) : pipeline VFX cloud avec AI
- "AssetDropper" : premier framework generatif d'extraction d'assets depuis des images de reference
- Sources : [SIGGRAPH 2025 post-conference](https://s2025.siggraph.org/siggraph-2025-ignites-global-ai-dialogue-across-art-science-and-industry/), [Life in the Machine recap](https://lifeinthemachine.substack.com/p/siggraph-2025-recap), credibilite : elevee

### VFX Voice Roundtable (16 leaders industrie)

Participants : Metaphysic, Digital Domain, DNEG, Scanline/Eyeline, The Mill, SideFX, Spin VFX, Tippett Studio, Ingenuity/Ghost VFX, et d'autres. ComfyUI est explicitement mentionne comme "integral to AI pipelines". Outils cites : Metaphysic Live, Midjourney, FLUX, Runway 3, Nuke Copycat, ComfyUI. Consensus : "AI and Film 3.0", au-dela de la peur initiale.
- Source : [VFX Voice roundtable](https://vfxvoice.com/ai-vfx-roundtable-revolutionizing-imagery-the-future-of-ai-and-newer-tech-in-vfx/), credibilite : elevee (magazine officiel VES)

---

## 2. PIPELINES INDIE / HACK SERIEUX

### Doug Hogan / Groove Jones

Doug Hogan est la figure cle du pont ComfyUI-VFX professionnel. 20 ans d'experience (credits : *Space Jam: A New Legacy*, *Scoob!*), Senior Creative Technologist chez Groove Jones, consultant pour ComfyUI sur les features studio-grade, sponsorise par Dell et The Foundry.

Son cours ActionVFX "Introduction to ComfyUI for VFX" (15 modules) inclut des EXR plates, des workflows pour clean plates, mattes ControlNet, relighting via depth maps, set extensions par inpainting, et export EXR pour compositing Nuke/AE/Resolve. Il a aussi construit un "Nuke MCP" (AI Copilot) connectant Nuke a Claude via Model Context Protocol.
- Sources : [befores & afters](https://beforesandafters.com/2025/10/24/introduction-to-comfyui-for-vfx-by-doug-hogan/), [ActionVFX](https://courses.actionvfx.com/comfyui-for-vfx), [Groove Jones GenVFX](https://www.groovejones.com/genvfx-pipeline-development), credibilite : elevee (credits verifiables, sponsors majeurs)

### Victor Perez (ComfyUI x NUKE)

VFX Supervisor avec 28 ans d'experience Hollywood. Son cours "Generative AI Workflows for Visual Effects: ComfyUI x NUKE" (410 GBP) enseigne l'integration ComfyUI comme extension du compositing Nuke. Disponible en 6 langues, oriente studio avec volume discounts.
- Source : [victorperez.co.uk](https://victorperez.co.uk/products/comfyui-x-nuke), credibilite : elevee (credits verifiables, Foundry certified)

### Eran Dinur / fxphd (ComfyUI + InvokeAI)

VFX Supervisor Emmy/VES Award (*Hereditary*, *Wolf of Wall Street*, *Uncut Gems*, *Boardwalk Empire*). Son cours fxphd AIF201 couvre : Maya + Nuke + ComfyUI ensemble, utilisation de beauty/Z-depth/normal passes + cryptomattes pour guider la generation AI, creation d'assets 3D via HunYuan, InvokeAI pour le matte painting.
- Source : [fxphd](https://www.fxphd.com/details/713/), credibilite : elevee

### Plugin Flame-ComfyUI (Xtevision)

Plugin payant pour Autodesk Flame 2025.x/2026.x qui fait un roundtrip EXR : export des clips Flame en sequences EXR, envoi au serveur ComfyUI local pour traitement AI, reimport automatique dans Flame. "1326 lignes de Python".
- Source : [Logik Forums](https://forum.logik.tv/t/comfyui-vfx-in-flame/12683), [Patreon Xtevision](https://www.patreon.com/Xtevision/shop/), credibilite : moyenne-haute (utilise en production par des artistes Flame)

### Sumit Chatterjee (Nuke-ComfyUI ACES 2.0 nodes)

Developpeur de custom nodes qui bridgent Nuke et ComfyUI avec integration ACES 2.0 studio color management, LUT loader style Nuke, write node specialise, et gestion des valeurs negatives lors de la conversion log-to-linear sur des images generees par AI.
- Source : [LinkedIn post](https://www.linkedin.com/posts/sumit-chatterjee-b565b58b_comfyui-vfx-colormanagement-activity-7413893027322245120-M4tX), credibilite : moyenne (individu, mais travail technique specifique et visible)

---

## 3. LE PROBLEME DU 8-BIT DANS COMFYUI : ANALYSE TECHNIQUE

### Les goulots d'etranglement precis

Le probleme du 8-bit dans ComfyUI a **quatre sources distinctes** :

**A. Le VAE decode clamp a [0,1]**
Le code de decodage VAE fait `decoded = decoded.div(2).add(0.5).clamp(0, 1)`. Tout ce qui est en dehors de [0,1] est irremediablement perdu. Ce n'est pas un simple clamping : la couche `conv_out` du VAE applique une normalisation de type sigmoide qui compresse les valeurs HDR. C'est la source #1 de perte de dynamic range.
- Source : [Explaining the SDXL latent space (HuggingFace)](https://huggingface.co/blog/TimothyAlexisVass/explaining-the-sdxl-latent-space), [vae-decode-hdr GitHub](https://github.com/netocg/vae-decode-hdr), credibilite : haute (analyse technique detaillee)

**B. PIL (Python Imaging Library)**
PIL ne gere correctement que les images 8-bit. C'est confirme dans les issues du ComfyUI-Impact-Pack. Toute operation passant par PIL (resize, paste, filtres, alpha) tronque a 8-bit. La solution : remplacer PIL par OpenCV ou PyTorch pour du float32/uint16.
- Source : [ComfyUI-Impact-Pack Issue #369](https://github.com/ltdrdata/ComfyUI-Impact-Pack/issues/369), credibilite : haute (issue technique directe)

**C. Le Save Image node natif**
Le node Save Image de ComfyUI normalise TOUS les pixels a [0,1] avant d'ecrire en PNG 8-bit. Meme si votre pipeline interne est en float32, le moment ou vous sauvegardez, c'est fini.
- Source : [vae-decode-hdr documentation](https://github.com/netocg/vae-decode-hdr), credibilite : haute

**D. Precision fp16 du VAE**
Le VAE SDXL genere des NaN en fp16 a cause de valeurs d'activation internes trop grandes. Un fix existe (`madebyollin/sdxl-vae-fp16-fix`) mais avec de legeres divergences. Le fp16 peut causer du color banding sur les degradees, les ciels, les surfaces lisses.
- Source : [sdxl-vae-fp16-fix (HuggingFace)](https://huggingface.co/madebyollin/sdxl-vae-fp16-fix), credibilite : haute

### Representatin interne du latent space

SDXL : 4 canaux a 1/8e resolution (1024x1024px image = 128x128x4 latent). Pendant l'inference, les valeurs vont de -30 a +30, restreintes a environ -4 a +4 au decodage. En fp16, chaque pixel latent peut encoder 16,384 valeurs distinctes par canal, ce qui est suffisant pour du 8-bit output (256 valeurs) mais pas pour du HDR.

FLUX : 16 canaux latents (vs 4 pour SDXL), potentiellement plus d'information encodee.

### Contournements existants

| Solution | Approche | Limite |
|----------|----------|--------|
| `--fp32-vae` flag | Force le VAE en fp32 | Plus de VRAM, pas de vrai HDR |
| HDR VAE Decode node | 4 modes (Conservative/Moderate/Exposure/Aggressive) de recovery HDR | Reconstruction mathematique, pas de vraie info HDR |
| ComfyUI-HQ-Image-Save | Export EXR 32-bit | Container haute fidelite, mais le contenu reste limite par le VAE |
| Radiance FXTD Studios | Pipeline float32 complet avec ACES 2.0 | **Des utilisateurs rapportent que certains nodes reduisent quand meme a 8-bit en interne** |
| Soft clamping pendant le denoising | Shrink les valeurs vers la moyenne pendant le sampling | Evite le hard clipping mais reduit le contraste |

---

## 4. PIPELINE 3D/ANIMATION vers ComfyUI

### Pixar / Disney Research

Deep learning denoising : Disney Research + Pixar + UC Santa Barbara ont developpe un denoiser neuronal entraine sur des millions d'exemples de *Finding Dory* pour du rendu production-quality accelere. Pas de ComfyUI, mais le concept est le meme : AI pour accelerer le rendu.
- Source : [Disney Research](https://la.disneyresearch.com/innovations/denoising/), credibilite : elevee

### Blender Studio

- AI Render : addon qui genere des images AI basees sur un prompt + la scene Blender, avec animation des parametres SD
- Dream Textures : Stable Diffusion directement dans Blender pour textures + concept art
- Conference Blender 2025 : presentation de trois studios francais (Cube Creative, Autour de Minuit, Normaal) sur leurs pipelines open-source
- Roadmap 2026 : NPR (rendu non-photorealiste), Cycles texture cache, ameliorations majeures du Video Sequencer
- Source : [Blender Conference 2025](https://conference.blender.org/2025/presentations/4004/), [Blender roadmap 2026](https://www.blender.org/development/projects-to-look-forward-to-in-2026/), credibilite : elevee

### Unreal Engine + ACES

Unreal Engine utilise le filmic tonemapper ACES par defaut depuis UE 4.15. OCIO est integre. Le workflow : Unreal (sRGB-Linear interne, ACES tonemapper) --> export EXR lineaire (tonemapper desactive via OCIO config) --> Nuke --> DaVinci Resolve. Des tutoriels communautaires Epic Games documentent le pipeline ACES complet UE --> Nuke --> DaVinci.
- Sources : [Epic Games ACES course](https://dev.epicgames.com/community/learning/courses/qEl/), [Virtual Production Indie Film Guide](https://vpifg.com/guides/unreal-ocio/), credibilite : elevee

### Neural Rendering (2026)

3D Gaussian Splatting : 100-200x plus rapide que les implementations NeRF originales de 2020. Neural Texture Compression (NVIDIA Blackwell / RTX 5090) : compresse les textures a 4-7% de l'empreinte VRAM originale. Le rendu hybride (simulation classique + inference neuronale) est en passe de devenir le defaut pour le temps reel.
- Source : [SuperRenders Farm](https://superrendersfarm.com/article/gpu-ai-render-trends-2026-neural-rendering-render-farms), credibilite : moyenne-haute

---

## 5. PIPELINE ACES END-TO-END AVEC ComfyUI

### La reponse courte : personne n'a encore un pipeline ACES 32-bit float de bout en bout passant par ComfyUI.

Voici pourquoi, et voici le meilleur compromis actuel :

### Le probleme fondamental

Le latent space de Stable Diffusion / FLUX ne travaille pas en ACES. Il travaille dans un espace latent compresse 4 ou 16 canaux, sans notion de color space standardise. Le VAE decode produit des valeurs dans un espace proche du sRGB, clampees a [0,1]. Le passage par la generation AI est une **rupture de la chaine colorimetrique ACES**.

### Le meilleur compromis pro (mars 2026)

Le pipeline le plus avance combine ces elements :

1. **FXTD Studios Radiance** (79 nodes) : pipeline float32 avec ACES 2.0, DWG, AWG4, OpenColorIO v2.x, presets camera (dont Blackmagic Pocket 4K/6K). Export EXR multi-canal 32-bit PIZ. LUT baking en .cube compatible Resolve. Inclut un Nuke Bridge pour transfer live bidirectionnel.
- Source : [GitHub fxtdstudios/radiance](https://github.com/fxtdstudios/radiance), [radiance.fxtd.org](https://radiance.fxtd.org/), credibilite : haute (outil open-source, specifique et documente)

2. **ComfyUI-HQ-Image-Save** : Save/Load EXR 32-bit, Save/Load Latent en EXR. Recommande `--fp32-vae`. Caveat explicite : "you only get true HDR/linear benefits if your graph is actually producing/maintaining values the way you expect."
- Source : [GitHub spacepxl/ComfyUI-HQ-Image-Save](https://github.com/spacepxl/ComfyUI-HQ-Image-Save), credibilite : haute

3. **ComfyUI-CoCoTools_IO** : EXR multicouche, cryptomatte, conversion colorspace via colour-science + OpenColorIO. Charge et ecrit des EXR comme Nuke.
- Source : [GitHub Conor-Collins/ComfyUI-CoCoTools_IO](https://github.com/Conor-Collins/ComfyUI-CoCoTools_IO), credibilite : haute

4. **HDR VAE Decode** : Bypass de la normalisation sigmoide du conv_out, preservation float32, 4 modes HDR. Workflow recommande : HDR VAE Decode --> Linear EXR Export --> compositing pro. NE PAS utiliser le Save Image natif.
- Source : [GitHub netocg/vae-decode-hdr](https://github.com/netocg/vae-decode-hdr), credibilite : moyenne-haute (projet individuel mais techniquement solide)

5. **Nuke-ComfyUI ACES 2.0 nodes** (Sumit Chatterjee) : Bridge Nuke-ComfyUI avec color management ACES 2.0, gestion log-to-linear pour images generees.
- Source : [LinkedIn](https://www.linkedin.com/posts/sumit-chatterjee-b565b58b_comfyui-vfx-colormanagement-activity-7413893027322245120-M4tX), credibilite : moyenne

### Pipeline recommande pour Goldberg Variations (BRAW 6K)

```
BRAW 6K (Blackmagic Pocket) 
  --> DaVinci Resolve (decode BRAW, RCM ou ACES) 
    --> Export EXR 32-bit lineaire (plates pour AI)
      --> ComfyUI (Radiance + HQ-Image-Save + HDR VAE Decode)
        [LoRA / inpainting / outpainting / generation]
        [--fp32-vae, ACES color space via Radiance]
      --> Export EXR 32-bit lineaire (output AI)
    --> Nuke ou Resolve (compositing, conform)
  --> DaVinci Resolve (grade final, delivrables)
```

**Le point de rupture reste le VAE** : quand l'image passe par le latent space SD/FLUX, les informations au-dela de [0,1] sont perdues ou approximativement reconstruites. Radiance et HDR VAE Decode tentent de recuperer du headroom, mais c'est une reconstruction mathematique, pas une preservation reelle de la dynamic range du BRAW original.

---

## 6. NETFLIX / AMAZON / APPLE : SPECS DE LIVRAISON + IA GENERATIVE

### Netflix

**Specs techniques** : Livraison IMF (SMPTE ST 2067-21:2016/2020/2023), plus haute qualite et resolution (HD + UHD). IMF crees directement depuis l'outil de grading color ou depuis un mezzanine intermediaire.
- Source : [Netflix Partner Help Center](https://partnerhelp.netflixstudios.com/hc/en-us/articles/7262346654995-Post-Production-Branded-Delivery-Specifications), credibilite : haute (source officielle)

**Politique IA generative** (aout 2025) : Netflix ne cite AUCUN outil specifique (pas de ComfyUI, pas de SD). Les principes : ideation = low risk, character designs/key visuals = escalation requise, digital replicas = consentement obligatoire. Environnements enterprise-secured obligatoires. Pas de formation sur IP non-possedee. Les plans AI dans le deliverable final necessitent notification precoce + guidance legal.
- Source : [Netflix Partner Help Center](https://partnerhelp.netflixstudios.com/hc/en-us/articles/43393929218323-Using-Generative-AI-in-Content-Production), [CineD](https://www.cined.com/netflix-publishes-generative-ai-guidelines-for-content-production/), credibilite : haute

**Cas pratiques Netflix** : *Happy Gilmore 2* (de-aging), *Billionaires' Bunker* (wardrobe/set design pre-prod), *El Eternauta* (VFX final pixel), localization agent (sous-titres multilingues en heures au lieu de semaines).

### Amazon Prime Video

AI pour recommandations, synopsis courts, dialogue enhancement, audio descriptions. Raf Soltanovich (tech lead) decrit un "flywheel entre tech et creativite". Pas de specs publiques sur l'IA generative dans la production.
- Source : [CNBC](https://www.cnbc.com/2025/10/22/netflix-all-in-on-leveraging-ai-in-its-streaming-platform.html), credibilite : moyenne

### Apple TV+

Aucune information publique trouvee sur les politiques ou specs AI generative d'Apple TV+. Apple reste plus prive que Netflix/Amazon sur sa roadmap technologique de production.

### Framework "Authorized AI" (industrie)

Emerge comme prerequis non-negociable pour les deliverables animation/VFX. Le partenariat Disney-OpenAI et l'acquisition Sony-Bandai Namco signalent que les majors construisent des pipelines de training AI licencies et clean. Les studios utilisant des outils AI non-autorises risquent des problemes de completion bond et de distribution.
- Source : [Vitrina AI report](https://vitrina.ai/blog/how-generative-ai-disrupting-animation-production-pipelines-2026/), credibilite : moyenne-haute

---

## SYNTHESE POUR LE PROJET GOLDBERG VARIATIONS

**La realite pragmatique** : Aucun studio hollywoodien n'utilise ComfyUI en "final pixel" sur des longs metrages. Les majors ont des outils proprietaires (Brahma/Metaphysic, Eyeline/Scanline, Disney OTE). Mais ComfyUI est massivement adopte dans le tier juste en-dessous : superviseurs VFX independants, petits studios, pre-production, et certaines taches specifiques (clean plates, mattes, roto, set extensions).

**Pour ton cas specifique** (documentaire auteur, BRAW 6K, LoRA/inpainting/outpainting/deepfakes) :

1. **Radiance FXTD Studios** est l'outil le plus important a installer. C'est le seul qui tente un pipeline ACES 32-bit float complet dans ComfyUI, avec preset Blackmagic Pocket 4K/6K.

2. **Le passage par le VAE est une rupture incontournable de la chaine 32-bit**. Accepter cette realite et compenser : export/import en EXR 32-bit autour de ComfyUI, utiliser HDR VAE Decode pour maximiser le headroom, et faire le grade final dans DaVinci Resolve.

3. **Pour les LoRA face** : training sur 20-30 images (2-4h GPU), bien superieur aux methodes single-image (IP-Adapter/FaceID). Trigger word specifique, dataset varie (eclairages, angles). Combinable avec FaceDetailer + inpainting pour les ajustements.

4. **Pour la conformite diffusion (Netflix/festivals)** : documenter l'usage AI, obtenir les consentements pour les digital replicas, garder les traces de provenance des modeles utilises.

Sources (selection des plus importantes) :

- [FXTD Studios Radiance (GitHub)](https://github.com/fxtdstudios/radiance)
- [ComfyUI-HQ-Image-Save (GitHub)](https://github.com/spacepxl/ComfyUI-HQ-Image-Save)
- [HDR VAE Decode (GitHub)](https://github.com/netocg/vae-decode-hdr)
- [ComfyUI-CoCoTools_IO (GitHub)](https://github.com/Conor-Collins/ComfyUI-CoCoTools_IO)
- [Netflix GenAI Guidelines](https://partnerhelp.netflixstudios.com/hc/en-us/articles/43393929218323-Using-Generative-AI-in-Content-Production)
- [Netflix IMF Delivery Specs](https://partnerhelp.netflixstudios.com/hc/en-us/articles/7262346654995-Post-Production-Branded-Delivery-Specifications)
- [ActionVFX ComfyUI for VFX (Doug Hogan)](https://courses.actionvfx.com/comfyui-for-vfx)
- [befores & afters - ComfyUI for VFX](https://beforesandafters.com/2025/10/24/introduction-to-comfyui-for-vfx-by-doug-hogan/)
- [Victor Perez ComfyUI x NUKE](https://victorperez.co.uk/products/comfyui-x-nuke)
- [fxphd AIF201 (Eran Dinur)](https://www.fxphd.com/details/713/)
- [VFX Voice AI Roundtable](https://vfxvoice.com/ai-vfx-roundtable-revolutionizing-imagery-the-future-of-ai-and-newer-tech-in-vfx/)
- [fxguide - How GenAI is used in VFX (VES Panel)](https://www.fxguide.com/quicktakes/how-gen-ai-is-being-used-in-vfx-right-now-ves-panel/)
- [Explaining the SDXL Latent Space (HuggingFace)](https://huggingface.co/blog/TimothyAlexisVass/explaining-the-sdxl-latent-space)
- [SIGGRAPH 2025 recap](https://s2025.siggraph.org/siggraph-2025-ignites-global-ai-dialogue-across-art-science-and-industry/)
- [El Eternauta AI VFX breakdown](https://markets.financialcontent.com/bpas/article/tokenring-2026-1-8-the-ai-revolution-in-cinema-how-netflixs-el-eternauta-redefined-the-vfx-pipeline)
- [Brahma/DNEG acquires Metaphysic](https://www.businesswire.com/news/home/20250218546095/en/)
- [Metaphysic de-aging in Here (fxguide)](https://www.fxguide.com/fxpodcasts/a-deep-dive-into-the-filmmaking-of-here-with-kevin-baillie/)
- [Studio Freewillusion AFX pipeline](https://www.morningstar.com/news/accesswire/1112603msn/studio-freewillusion-launches-production-ready-ai-vfx-pipeline-for-hollywood-filmmakers)
- [Groove Jones GenVFX](https://www.groovejones.com/genvfx-pipeline-development)
- [Logik Forums - ComfyUI + Flame](https://forum.logik.tv/t/comfyui-vfx-in-flame/12683)
- [Flame ComfyUI Plugin (Patreon)](https://www.patreon.com/Xtevision/shop/comfyui-vfx-integration-plugin-for-flame-1444438)
- [ACES 2.0 + ComfyUI nodes (LinkedIn)](https://www.linkedin.com/posts/sumit-chatterjee-b565b58b_comfyui-vfx-colormanagement-activity-7413893027322245120-M4tX)
- [Radiance Official Site](https://radiance.fxtd.org/)
- [PixelSHAM coverage of Radiance](https://www.pixelsham.com/2026/03/17/fxtd-studios-radiance-a-professional-vfx-grade-32-bit-float-color-science-suite-for-comfyui/)
- [Unreal Engine ACES pipeline](https://dev.epicgames.com/community/learning/tutorials/Zdew/unreal-engine-ue-to-nuke-to-davinci-aces-color-pipeline)
- [Disney-OpenAI partnership](https://www.contentgrip.com/disney-openai-partnership/)
- [Blender Conference 2025 - Open Source Pipelines](https://conference.blender.org/2025/presentations/4004/)
- [NVIDIA + ComfyUI at GDC 2026](https://blogs.nvidia.com/blog/rtx-ai-garage-flux-ltx-video-comfyui-gdc/)
- [SDXL VAE fp16 fix (HuggingFace)](https://huggingface.co/madebyollin/sdxl-vae-fp16-fix)
- [ComfyUI-Impact-Pack bit depth issue](https://github.com/ltdrdata/ComfyUI-Impact-Pack/issues/369)