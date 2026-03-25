# Analyse LUT comparative : 7 LUTs x 2 plans
**Date** : 25 mars 2026
**Outil** : `lut-analyzer.py` (CIE Lab, Delta E 2000, vectorscope I-line)

## Sources testees

| Plan | Fichier | Type | Timestamp |
|------|---------|------|-----------|
| Plan A | `5C25DDD0-A725-4FBE-A894-40AF1F54059F.mp4` | Interieur jour | t=120s |
| Plan B | `0DDF4394-FD6C-43EC-848B-21021BF253E8.mp4` | Exterieur jour | t=60s |

## LUTs analysees (7)

| # | LUT | Origine | Date ajout |
|---|-----|---------|------------|
| 1 | `240326_LUTS_DINOS-65` | Dinos | 25/03 00:41 |
| 2 | `Dinos_CYANFROID_131.71` | Dinos | 25/03 00:41 |
| 3 | `HADRI Technicolor_DD01_classic_fpe_r709` | Hadrien | 25/03 00:47 |
| 4 | `HADRI-LUT-GDB-+GREEN-65-HV_1` | Hadrien (NOUVELLE) | 25/03 04:42 |
| 5 | `HADRI-LUT-GDB-65-HV_1` | Hadrien (NOUVELLE) | 25/03 04:15 |
| 6 | `HADRI-LUT-LOW-CONTRASTE-GDB-65-HV_5` | Hadrien (NOUVELLE) | 25/03 04:15 |
| 7 | `night_cyanfroid_balam` | Balam | 25/03 00:41 |

Note : `Goldberg-variation.cube` exclue (corrompue). Les LUTs 4, 5, 6 sont les nouvelles d'Hadrien.

---

## Resultats Plan A : Interieur jour

### Tableau comparatif

| Metrique | ORIGINAL | DINOS-65 | CYANFROID | HADRI Tech | HADRI +GREEN | HADRI GDB-65 | HADRI LOW-C | night_cyan |
|----------|----------|----------|-----------|------------|-------------|--------------|-------------|------------|
| **Y moyen** | 0.580 | 0.598 | 0.595 | 0.640 | 0.577 | 0.575 | 0.619 | 0.616 |
| **Contraste** | 11.8:1 | 41.9:1 | 37.5:1 | **148.1:1** | 24.7:1 | 21.1:1 | 11.4:1 | 31.1:1 |
| **Saturation** | 0.254 | 0.342 | 0.406 | 0.385 | 0.284 | 0.296 | 0.281 | 0.398 |
| **Temp R-B** | +0.038 | -0.007 | +0.129 | +0.105 | +0.080 | +0.078 | +0.059 | +0.116 |
| **Delta E vs orig** | -- | 4.02 | 7.91 | 7.26 | 3.67 | 3.80 | 3.53 | 7.77 |

### Skin tones (zone centre-haut)

| Metrique | ORIGINAL | DINOS-65 | CYANFROID | HADRI Tech | HADRI +GREEN | HADRI GDB-65 | HADRI LOW-C | night_cyan |
|----------|----------|----------|-----------|------------|-------------|--------------|-------------|------------|
| **Lab peau** | 59,+7.8,+4.6 | 62,+9.5,+3.1 | 59,+13.5,+13.0 | 64,+14.7,+10.5 | 56,+13.2,+9.6 | 57,+13.0,+9.3 | 62,+9.1,+6.5 | 62,+12.3,+12.7 |
| **Delta E skin** | 10.65 | 10.10 | 5.63 | **4.52** | 9.24 | 8.36 | 7.83 | **3.57** |
| **I-line dev** | 8.5 (OK) | 15.7 (WARN) | **0.9 (OK)** | 5.7 (OK) | 5.4 (OK) | 5.7 (OK) | 5.7 (OK) | **0.3 (OK)** |
| **Sat UV** | 10.77 | 11.46 | 22.00 | 21.57 | 18.93 | 18.65 | 13.53 | 21.06 |

### Zones (ombres / tons moyens / hautes lumieres)

| Zone | ORIGINAL | DINOS-65 | CYANFROID | HADRI Tech | HADRI +GREEN | HADRI GDB-65 | HADRI LOW-C | night_cyan |
|------|----------|----------|-----------|------------|-------------|--------------|-------------|------------|
| Ombres % | 15.7 | 24.3 | 26.2 | 23.4 | 26.6 | 24.7 | 14.7 | 24.6 |
| Midtones % | 57.0 | 19.2 | 19.9 | 19.8 | 20.1 | 25.8 | 44.1 | 19.2 |
| Highlights % | 27.3 | 56.5 | 53.8 | 56.8 | 53.3 | 49.4 | 41.1 | 56.3 |

**Verdict Plan A** :
- Meilleur skin tone (Delta E) : **night_cyanfroid_balam** (3.57)
- Meilleur I-line : **night_cyanfroid_balam** (0.3)
- Meilleur contraste : **HADRI Technicolor** (148.1:1)

---

## Resultats Plan B : Exterieur jour

### Tableau comparatif

| Metrique | ORIGINAL | DINOS-65 | CYANFROID | HADRI Tech | HADRI +GREEN | HADRI GDB-65 | HADRI LOW-C | night_cyan |
|----------|----------|----------|-----------|------------|-------------|--------------|-------------|------------|
| **Y moyen** | 0.547 | 0.553 | 0.548 | 0.591 | 0.535 | 0.532 | 0.582 | 0.570 |
| **Contraste** | 20.0:1 | 50.8:1 | 45.8:1 | **996.3:1** | 25.9:1 | 24.2:1 | 19.6:1 | 43.4:1 |
| **Saturation** | 0.347 | 0.420 | 0.487 | 0.482 | 0.331 | 0.349 | 0.376 | 0.471 |
| **Temp R-B** | +0.024 | -0.028 | +0.101 | +0.079 | +0.063 | +0.062 | +0.042 | +0.083 |
| **Delta E vs orig** | -- | 4.45 | 7.88 | 7.62 | 3.36 | 3.45 | 3.37 | 7.40 |

### Skin tones

| Metrique | ORIGINAL | DINOS-65 | CYANFROID | HADRI Tech | HADRI +GREEN | HADRI GDB-65 | HADRI LOW-C | night_cyan |
|----------|----------|----------|-----------|------------|-------------|--------------|-------------|------------|
| **Lab peau** | 52,+9.6,+0.7 | 55,+10.7,-2.3 | 51,+13.0,+6.3 | 57,+13.7,+5.9 | 51,+11.7,+5.8 | 51,+11.8,+5.6 | 55,+11.2,+1.9 | 55,+11.6,+6.3 |
| **Delta E skin** | 16.62 | 16.54 | 14.23 | **10.56** | 14.83 | 14.44 | 13.94 | **11.46** |
| **I-line dev** | 25.3 (BAD) | 40.1 (BAD) | 11.4 (WARN) | 13.1 (WARN) | 11.1 (WARN) | 11.6 (WARN) | 21.4 (BAD) | **9.6 (OK)** |
| **Sat UV** | 9.70 | 9.45 | 16.19 | 16.88 | 14.68 | 14.67 | 11.96 | 15.21 |

### Zones

| Zone | ORIGINAL | DINOS-65 | CYANFROID | HADRI Tech | HADRI +GREEN | HADRI GDB-65 | HADRI LOW-C | night_cyan |
|------|----------|----------|-----------|------------|-------------|--------------|-------------|------------|
| Ombres % | 20.4 | 29.2 | 31.3 | 27.9 | 32.3 | 30.4 | 19.1 | 29.6 |
| Midtones % | 58.3 | 20.6 | 21.6 | 21.3 | 20.5 | 25.6 | 43.2 | 20.5 |
| Highlights % | 21.2 | 50.2 | 47.0 | 50.8 | 47.2 | 44.1 | 37.7 | 49.9 |

**Verdict Plan B** :
- Meilleur skin tone (Delta E) : **HADRI Technicolor** (10.56)
- Meilleur I-line : **night_cyanfroid_balam** (9.6)
- Meilleur contraste : **HADRI Technicolor** (996.3:1)

---

## Synthese comparative : les 3 nouvelles LUTs d'Hadrien

### HADRI-LUT-GDB-65-HV_1 (la "base")
- **Contraste modere** : 21-24:1 (entre DINOS-65 et CYANFROID)
- **Skin tone** : Delta E 8.36 (int) / 14.44 (ext) = correct en interieur, insuffisant en exterieur
- **I-line** : 5.7 (int, OK) / 11.6 (ext, WARN) = bon en interieur
- **Saturation** : retenue (0.296 int, 0.349 ext), proche de l'original
- **Temperature** : legerement chaude (+0.078 / +0.062), equilibree
- **Profil** : LUT polyvalente, pas de defaut majeur, pas de point fort spectaculaire

### HADRI-LUT-LOW-CONTRASTE-GDB-65-HV_5 (variante low contrast)
- **Contraste** : le plus bas de tous (11.4 int / 19.6 ext), quasiment identique a l'original
- **Skin tone** : Delta E 7.83 (int) / 13.94 (ext) = meilleur des 3 Hadrien en interieur
- **I-line** : 5.7 (int, OK) / 21.4 (ext, BAD) = problematique en exterieur
- **Saturation** : la plus retenue (0.281 int), look naturel/desature
- **Midtones** : conserve 44% (int) / 43% (ext) = bien plus de midtones que les autres LUTs
- **Profil** : LUT "film" douce, preservant les details dans les tons moyens. Ideale pour de la correction fine en post

### HADRI-LUT-GDB-+GREEN-65-HV_1 (variante verte)
- **Contraste** : modere (24.7 int / 25.9 ext)
- **Skin tone** : Delta E 9.24 (int) / 14.83 (ext) = intermediaire
- **I-line** : 5.4 (int, OK) / 11.1 (ext, WARN) = legerement meilleur que GDB-65
- **Ombres** : les plus froides des 3 Hadrien en interieur (-0.045) = separation ombres/lumieres nette
- **Saturation** : la plus retenue des 3 (0.284 int)
- **Profil** : variante avec split toning vert dans les ombres. Esthetique cinema verite

---

## Recommandations par type de plan

### Interieur jour
**Recommandation : night_cyanfroid_balam ou HADRI Technicolor**

| Critere | Gagnant | Score |
|---------|---------|-------|
| Delta E skin | night_cyanfroid_balam | 3.57 |
| I-line | night_cyanfroid_balam | 0.3 |
| Contraste | HADRI Technicolor | 148.1:1 |

night_cyanfroid_balam domine en precision skin tone (Delta E 3.57, I-line 0.3). En interieur, la lumiere est controlee et la LUT excelle a restituer les carnations. HADRI Technicolor offre le contraste le plus cinematographique si le plan le demande.

Parmi les nouvelles d'Hadrien : **HADRI-LUT-LOW-CONTRASTE** est la meilleure pour l'interieur (Delta E 7.83, I-line OK, midtones preserves a 44%). Son look doux et naturel respecte l'ambiance interieure.

### Interieur nuit
**Recommandation : night_cyanfroid_balam**

Pas de plan nuit teste, mais les donnees d'interieur jour suggerent fortement night_cyanfroid_balam : sa precision I-line (0.3) est imbattable pour les skin tones en basse lumiere, ou la moindre deviation chromatique se voit. La saturation UV elevee (21.06) garantit que les carnations restent lisibles meme dans les ombres.

Alternative : **HADRI-LUT-GDB-+GREEN** pourrait fonctionner de nuit grace a ses ombres froides (-0.045 R-B) qui creent une separation lumiere chaude/ombre froide typique du cinema nocturne.

### Exterieur jour
**Recommandation : HADRI Technicolor ou night_cyanfroid_balam**

| Critere | Gagnant | Score |
|---------|---------|-------|
| Delta E skin | HADRI Technicolor | 10.56 |
| I-line | night_cyanfroid_balam | 9.6 |
| Contraste | HADRI Technicolor | 996.3:1 |

En exterieur, toutes les LUTs peinent davantage sur les skin tones (Delta E minimum 10.56 vs 3.57 en interieur). HADRI Technicolor reprend l'avantage grace a son contraste extreme et son meilleur Delta E skin. night_cyanfroid_balam reste le seul a atteindre le seuil I-line "OK" (9.6).

Parmi les nouvelles d'Hadrien : **HADRI-LUT-GDB-+GREEN** est la plus pertinente en exterieur (I-line 11.1, le meilleur des 3 Hadrien). Le split toning vert dans les ombres fonctionne bien avec la lumiere naturelle.

### Avertissement : HADRI Technicolor contraste
Le ratio de contraste extreme de HADRI Technicolor (148:1 int, 996:1 ext) signale un ecrasement des noirs et/ou un clipping des blancs (min 1% a 0.00-0.01). A utiliser avec precaution, ou combinee avec un lift des noirs en Resolve pour recuperer du detail dans les ombres.

---

## Classement global

| Rang | LUT | Forces | Faiblesses |
|------|-----|--------|------------|
| 1 | **night_cyanfroid_balam** | I-line parfait (0.3), meilleur skin tone interieur | Saturation elevee, pas adapte a tous les looks |
| 2 | **HADRI Technicolor** | Contraste cinema, meilleur skin tone exterieur | Ecrasement noirs/blancs, necessite correction fine |
| 3 | **HADRI-LUT-LOW-CONTRASTE** | Naturel, midtones preserves, polyvalent | I-line mauvais en exterieur (21.4) |
| 4 | **HADRI-LUT-GDB-+GREEN** | Split toning ombres, bon en exterieur | Intermediaire partout |
| 5 | **HADRI-LUT-GDB-65-HV_1** | Equilibre, pas de defaut | Pas de point fort non plus |
| 6 | **Dinos_CYANFROID** | Bon I-line (0.9 int) | Saturation forte, chaud |
| 7 | **240326_LUTS_DINOS-65** | Delta E modere | I-line mauvais (15.7), froid |

## Fichiers de reference
- JSON interieur : `/tmp/lut-analyze-interieur-v2/lut-analysis.json`
- JSON exterieur : `/tmp/lut-analyze-exterieur/lut-analysis.json`
- PNG compares : `/tmp/lut-analyze-interieur-v2/` et `/tmp/lut-analyze-exterieur/`
- Script : `~/Projects/Films/goldberg/tools/lut-analyzer.py`
