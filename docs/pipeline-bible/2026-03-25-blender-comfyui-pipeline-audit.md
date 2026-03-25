J'ai maintenant assez de matière pour une synthèse exhaustive. Voici le rapport complet.

---

# Recherche exhaustive : Pipeline Cinema Blender + ComfyUI via Claude Code

## 1. Blender MCP : compositing et color grading depuis Claude Code

Le MCP Blender actif expose `mcp__blender__execute_blender_code`, qui permet d'exécuter **n'importe quel script Python** dans Blender. Cela signifie que tout ce que `bpy` peut faire est accessible depuis Claude Code, y compris la construction programmatique de node trees compositor.

**Nodes couleur disponibles via l'API Python :**
- `CompositorNodeColorBalance` (Lift/Gamma/Gain, Offset/Power/Slope ASC-CDL, White Point)
- `CompositorNodeColorCorrection` (correction tri-tone complète)
- `CompositorNodeCurveRGB` (courbes RGB avec mode `FILMLIKE` qui préserve la teinte)
- `CompositorNodeHueSat` / `CompositorNodeHueCorrect`
- `CompositorNodeExposure` / `CompositorNodeGamma` / `CompositorNodeBrightContrast`
- `CompositorNodeLevels` / `CompositorNodeTonemap` (HDR)
- `CompositorNodeConvertColorSpace` / `CompositorNodeConvertToDisplay` (nouveau Blender 5.0)
- `CompositorNodeSeparateColor` / `CompositorNodeCombineColor`

Concrètement, depuis Claude Code, on peut faire :
```python
# Via mcp__blender__execute_blender_code
tree = bpy.data.node_groups.new("Grade", "CompositorNodeTree")
scene.compositing_node_group = tree
cb = tree.nodes.new('CompositorNodeColorBalance')
cb.correction_method = 'OFFSET_POWER_SLOPE'  # ASC-CDL
curves = tree.nodes.new('CompositorNodeCurveRGB')
curves.mapping.curves[0].points.new(0.25, 0.35)  # S-curve
```

**Limitation critique** : les scopes (vectorscope, waveform, histogramme) sont liés à l'UI Image Editor via `bpy.types.Scopes` mais ne sont pas disponibles comme nodes compositor autonomes. L'add-on [Viewport Scopes](https://superhivemarket.com/products/viewport-scopes) ajoute des scopes temps réel, mais en headless il faut des solutions alternatives (calcul numpy/OpenCV sur les pixels).

---

## 2. Blender comme color grader headless

**Oui, c'est scriptable.** Le pipeline recommandé :

```bash
blender -b grade.blend -P grade_script.py
```

Le flag `-b` (background) est **obligatoire** pour le batch compositor : sans fenêtre visible, le compositor ne se met pas à jour autrement. Le rendu se déclenche via `bpy.ops.render.render()`.

**Changement majeur Blender 5.0** (novembre 2025) :
- `scene.node_tree` remplacé par `scene.compositing_node_group`
- `scene.use_nodes` déprecié (toujours True)
- Le node `Composite` remplacé par `NodeGroupOutput`
- Beaucoup de types renommés (ex: `CompositorNodeMath` devient `ShaderNodeMath`)

**Add-ons pro pour le grading dans Blender :**
- **Lazy Composer 2** (2026) : emulation de pellicule, grain, color balance inspiré de DaVinci Resolve, tourne dans le compositor
- **Render Raw** : accès direct aux pixels bruts, système de grading autonome
- **Analog Film Simulation** : simulation physique des emulsions via Geometry Nodes ou Compositor

**Verdict honnête** : Blender peut faire du color grading basique à intermédiaire (corrections primaires, courbes, CDL, LUTs). Pour du grading avancé (secondaires complexes, qualifiers HSL, power windows fluides, scopes temps réel), DaVinci Resolve reste supérieur. Le workflow recommandé pour un film : Blender pour le compositing 3D + VFX, puis export EXR vers Resolve pour le grade final.

---

## 3. Pipeline EXR multi-layer

**Import/Export multi-layer EXR : entièrement scriptable.**

Le pipeline :
1. Render avec passes (Diffuse, Glossy, AO, Depth, Normal, Mist, Cryptomatte...)
2. Export en OpenEXR MultiLayer (32-bit float)
3. Réimport dans le compositor pour le grade pass par pass
4. Export final en EXR MultiLayer ou format delivery

**Attention au bug connu** : `bpy.types.Image.save_render()` sauvegarde un EXR mono-layer même si les settings disent MultiLayer. Il faut passer par `bpy.ops.render.render()` avec `scene.render.image_settings.file_format = 'OPEN_EXR_MULTILAYER'`.

Pour automatiser l'export multi-pass, le guide de référence est sur [BlenderNotes](https://blendernotes.com/automating-multi-pass-exr-exports-in-blender-with-python-scripts/).

**Outil utile** : [NodeToPython](https://github.com/BrendanParmer/NodeToPython) convertit un node tree existant en script Python reproductible.

---

## 4. Blender <-> ComfyUI : l'écosystème bridge

Quatre projets principaux :

| Projet | Stars | Description |
|--------|-------|-------------|
| [ComfyUI-BlenderAI-node](https://github.com/AIGODLIKE/ComfyUI-BlenderAI-node) (AIGODLIKE) | Le plus mature | Convertit les nodes ComfyUI en nodes Blender natifs. Caméra viewport comme input temps réel. Style transfer, interpolation IA (ToonCrafter), texture generation, ControlNet |
| [ComfyUI-CUP](https://github.com/AIGODLIKE/ComfyUI-CUP) | Bridge | Thumbnails, node diff, queue service entre Blender et ComfyUI |
| [CGA_BlenderBridge](https://www.runcomfy.com/comfyui-nodes/ComfyUI_CGAnimittaTools/cga-blender-bridge) | 3D transfer | Transfert .obj/.fbx/.glb de ComfyUI vers Blender en temps réel |
| [ComfyUI-Blender](https://www.runcomfy.com/comfyui-nodes/ComfyUI-Blender) (alexisrolland) | Server-based | Envoi de requêtes ComfyUI depuis Blender |

**Workflow Render Blender -> ComfyUI -> Post IA -> Retour Blender :**
1. Blender rend en EXR (passes séparées)
2. ComfyUI charge l'image + depth map via nodes LoadImage
3. Post-process IA : style transfer (IPAdapter), upscale (ESRGAN/4x-UltraSharp), color match, film grain
4. Retour via ComfyUI-BlenderAI-node ou fichier disque

**Nodes ComfyUI utiles pour le post-process cinema :**
- [ComfyUI-EasyColorCorrector](https://github.com/regiellis/ComfyUI-EasyColorCorrector) : correction couleur AI-powered, emulation pellicule, correction VAE
- [ComfyUI-ProPost](https://github.com/digitaljohn/comfyui-propost) : film grain paramétrique, LUT .cube, vignette
- [ColorMatch (KJNodes)](https://www.runcomfy.com/comfyui-nodes/ComfyUI-KJNodes/ColorMatch) : transfert de palette couleur (Reinhard, Pitie, MVGD)
- [ComfyUI-post-processing-nodes](https://github.com/EllangoK/ComfyUI-post-processing-nodes) : ColorCorrect, Blend, aberration chromatique
- [ComfyUI-Color_Transfer](https://github.com/45uee/ComfyUI-Color_Transfer) : palette transfer

**Notre setup** : les MCPs `mcp__blender__execute_blender_code` et `mcp__comfyui__enqueue_workflow` + `mcp__comfyui__create_workflow` sont tous actifs. Claude Code peut orchestrer le pipeline complet : scripter le compositor Blender, construire un workflow ComfyUI, l'enqueue, récupérer le résultat.

---

## 5. Blender OCIO / ACES

**Blender 5.0 est un tournant majeur.** Depuis novembre 2025 :

- **ACES 2.0 natif** : views ACES 1.3 et 2.0 intégrées (standard + HDR), en alternative à AgX et Filmic
- **Working space configurable** : Linear Rec.709 (défaut), **ACEScg**, Linear Rec.2020. Se choisit au début du projet
- **HDR** : Rec.2100-PQ et Rec.2100-HLG pour le grading HDR natif, Display P3 pour wide gamut
- **Nouveau node** `ConvertToDisplay` pour conversion manuelle dans le compositor
- **Node ConvertColorSpace** : ajout du choix "Working Space" pour conversion depuis/vers l'espace de travail courant
- **VFX Reference Platform 2025** : toutes les libs alignées pour l'intégration studio

**Pipeline ACES dans Blender :**
1. Définir ACEScg comme working space (`scene.color_management.working_space = 'ACEScg'`)
2. Textures taguées avec leur color space à l'import
3. Rendu en ACEScg
4. Export EXR en ACES2065-1 ou ACEScg
5. Composer/grader dans le compositor avec les nodes de conversion
6. View Transform ACES 2.0 pour le monitoring

Pour les gros studios : installer la config ACES officielle via la variable d'environnement `OCIO`. Pour les indépendants, le built-in de Blender 5.0 couvre la majorité des besoins.

---

## 6. Tracking 2D/3D pour masques de color grading

**Oui, Blender peut faire des "power windows" qui suivent un sujet**, bien que le terme soit propre à DaVinci Resolve.

Le workflow :
1. **Movie Clip Editor** : importer le footage, placer des markers de tracking
2. **Tracking 2D/3D** : tracker les points (Blender utilise le Ceres Solver pour le tracking planaire)
3. **Masques spline** : dessiner des masques Bézier, les animer avec keyframes + données de tracking
4. **Grease Pencil** attaché aux tracks : les strokes suivent le mouvement du tracker automatiquement
5. **Compositor** : utiliser le mask comme entrée d'un node de color grading (Mix, ColorBalance, etc.) pour grader sélectivement

**Blender 5.2** (en développement) ajoute des améliorations au motion tracking.

**Limitation** : pas de qualifier HSL automatique comme dans Resolve. Le masking est manuel (splines). Pour un documentaire avec beaucoup de footage live-action, c'est utilisable mais plus lent que les power windows + tracking de Resolve.

---

## 7. Cas d'utilisation pro documentés

- **Black Mountain (2024)** : horreur, 90% CGI, pipeline 100% Blender + Natron. Prix Best VFX à Slamdance
- **The Last Broadcast (2025)** : sci-fi found footage, Gaffer + Blender pour le lighting/compositing
- Un réalisateur à Austin a monté un pipeline VFX open source complet (Blender + Natron + ACES), économisé $42,000 en licences, livré en avance
- **200+ films et séries** utilisent Blender en 2025 (indie + Netflix selon les sources)
- Le pipeline open source typique : **Blender (3D/VFX) -> EXR -> Natron (compositing) -> DaVinci Resolve (grade final)**

---

## Synthèse pour ton pipeline

Pour un pipeline cinema pro depuis Claude Code :

| Etape | Outil | Via Claude Code |
|-------|-------|-----------------|
| Rendu 3D + passes | Blender | `mcp__blender__execute_blender_code` |
| Compositing VFX | Blender Compositor (scriptable) | `mcp__blender__execute_blender_code` |
| Post-process IA (style, upscale, grain) | ComfyUI | `mcp__comfyui__create_workflow` + `enqueue_workflow` |
| Color grading primaire | Blender Compositor (CDL, curves, color balance) | `mcp__blender__execute_blender_code` |
| Color grading avancé | DaVinci Resolve | Hors MCP (ou resolve-mcp si Studio) |
| Color management | Blender ACES 2.0 natif | Scriptable via bpy |
| Tracking / masques | Blender Movie Clip Editor | Scriptable via bpy |

Le point fort de notre setup : Claude Code orchestre Blender ET ComfyUI en parallèle, ce qui permet d'automatiser des pipelines que les coloristes font manuellement.

---

Sources principales :
- [Blender Python API - CompositorNode](https://docs.blender.org/api/current/bpy.types.CompositorNode.html)
- [Blender 5.0 Color Management](https://developer.blender.org/docs/release_notes/5.0/color_management/)
- [Blender 5.0 ACES 2.0](https://digitalproduction.com/2025/09/02/blender-5-0-ships-built-in-aces-2-0-view-transform/)
- [Blender ACES 2.0 deep dive](https://famousandfaded.com/deep-cuts/blender-5-aces-2-ocio-hdr-support/)
- [Blender 5.0 Python API changes](https://developer.blender.org/docs/release_notes/5.0/python_api/)
- [Blender 5.0 Compositor Migration](https://developer.blender.org/docs/release_notes/5.0/migration/compositor_migration/)
- [ACES open source pipeline (Blender, Natron)](https://community.acescentral.com/t/aces-open-source-software-pipeline-blender-natron/4361)
- [ComfyUI-BlenderAI-node](https://github.com/AIGODLIKE/ComfyUI-BlenderAI-node)
- [ComfyUI-CUP](https://github.com/AIGODLIKE/ComfyUI-CUP)
- [ComfyUI-EasyColorCorrector](https://github.com/regiellis/ComfyUI-EasyColorCorrector)
- [ComfyUI-ProPost](https://github.com/digitaljohn/comfyui-propost)
- [ColorMatch KJNodes](https://www.runcomfy.com/comfyui-nodes/ComfyUI-KJNodes/ColorMatch)
- [Automating Multi-Pass EXR Exports](https://blendernotes.com/automating-multi-pass-exr-exports-in-blender-with-python-scripts/)
- [NodeToPython](https://github.com/BrendanParmer/NodeToPython)
- [Open-Source VFX Pipeline](https://kinonews.pro/open-source-vfx-how-blender-gaffer-and-natron-are-changing-film-production)
- [Lazy Composer 2](https://digitalproduction.com/2026/02/02/lazy-composer-2-teaches-blender-to-pretend-it-shot-film/)
- [Analog Film Simulation](https://digitalproduction.com/2025/10/22/accurate-film-simulation-for-blender-analog-soul-digital-body/)
- [Blender Motion Tracking Manual](https://docs.blender.org/manual/en/latest/movie_clip/index.html)
- [Viewport Scopes add-on](https://superhivemarket.com/products/viewport-scopes)
- [CG Wire - Blender Scripting for Pipelines](https://blog.cg-wire.com/blender-scripting-animation/)
- [Blender Scopes API](https://docs.blender.org/api/current/bpy.types.Scopes.html)