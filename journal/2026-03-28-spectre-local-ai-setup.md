---
name: Spectre Fleet — Local AI Setup Session (28 mars 2026)
description: Inventaire complet de la session sur l'IA locale, hardware comparé, modèles installés, outils configurés, et todo list
type: project
---

# Spectre Fleet — Local AI Setup (28 mars 2026)

## Hardware — Notre setup vs Ryzen AI Max+ 395

**Notre PC-Nomade** : Ryzen 9 9900X + RTX 5090 (32GB VRAM) + 46GB RAM

| | Notre RTX 5090 | Ryzen AI Max+ 395 |
|---|---|---|
| VRAM | 32 GB | ~110 GB unified |
| Modèles <32B | Plus rapide | Correct |
| Modèles 70B+ | Spill RAM (1-2 tok/s) | Tient en mémoire |
| Consommation | 575W | ~120W |

**Verdict** : Pour notre fleet de modèles <=35B, RTX 5090 est plus rapide. Ryzen AI Max+ 395 gagne uniquement sur 70B+.

## Cameras & Audio

**Cameras** : BMPCC 6K (BRAW ou ProRes) + iPhone 15 Pro Max ProRes 422HQ LOG
- BRAW → DaVinci Resolve pour décoder (ou `braw-decode`) → proxy H.264
- ProRes → direct ffmpeg → proxy H.264
- Pipeline IA : extraction keyframes OpenCV → analyse Qwen3.5 vision

**Transcription audio** :
- Contenu anglais (Joshua, FBI, plateaux US) → **Parakeet v3** (Mac, MacWhisper Pro)
- Contenu français (réunions prod, notes) → **Whisper large-v3** (faster-whisper GPU WSL)
- Rushes mixtes FR/EN → Whisper large-v3 par défaut

## Modèles Ollama installés (PC-Nomade, Windows)

```
qwen3.5:35b-a3b   23.9 GB   MoE — meilleur rapport qualité/vitesse
qwen3.5:27b       17.4 GB   dense
qwen3.5:9b         6.6 GB   rapide, autocomplétion
qwen3:32b         20.2 GB   dense, excellent pour code
deepseek-r1:32b   19.9 GB   raisonnement
hermes3:8b         4.7 GB   agents, peu censuré
mistral-nemo       7.1 GB
dolphin3           4.9 GB
```

## SOTA fin mars 2026

### Frontier (cloud)
- **Claude Opus 4.6** : #1 Arena Elo 1504, 80.8% SWE-bench, meilleur code/écriture
- **Gemini 3.1 Pro** : 94.3% GPQA Diamond, meilleur raisonnement, 13/16 benchmarks
- **GPT-5.4** : #1 Terminal-Bench 2.0 (75.1%), fort all-around
- **DeepSeek V4** : 1T params open-weight, challenger open source
- **ARC-AGI-3** (25 mars) : frontier models <1%, nouveau benchmark dur

### Open source local
- **Qwen3.5-397B-A17B** : flagship MoE Alibaba, SOTA open mais non dispo localement sans cluster
- **Notre qwen3.5:35b-a3b** : très bon niveau pour tâches courantes
- **qwen3:32b** : dense, excellent pour code

## Outils configurés cette session

### Installé et fonctionnel
- **Continue.dev** (VSCode extension) — branché sur Ollama, config dans `~/.continue/config.yaml`
  - qwen3.5:35b-a3b pour chat
  - qwen3.5:9b pour autocomplétion inline
  - deepseek-r1:32b pour raisonnement

- **Windows-MCP** (`@cursortouch/windows-mcp`) — MCP global dans `~/.claude.json`
  - Permet à Claude Code de contrôler Windows (click, type, PowerShell, fichiers)
  - Équivalent AppleScript mais pour Windows
  - Activé au prochain redémarrage Claude Code

### À installer manuellement (GUI Windows)
- **Mosh** sur WSL : `sudo apt install mosh` → permet Moshi iOS de se connecter en session persistante
- **Pinokio 7** apps (ouvrir Pinokio sur Windows, cliquer Install) :
  - Hermes Agent (NousResearch) — agent + gateway, accès mobile intégré
  - OpenClaw / ClawdBot — alternative à Claude Code avec modèles open source
  - WanGP — agent vidéo local, Qwen 3.5VL, 8GB VRAM
  - N8N — workflow automation, orchestration Telegram/Discord

### Open WebUI
- Installé sur PC-Nomade (port 3000) — fonctionnel localement
- **Non accessible sur pc-torrent** — Open WebUI pas encore installé là-bas
- Pour accès mobile : Tailscale → `http://100.125.245.108:3000` (mais nécessite d'abord installer Open WebUI sur pc-torrent)
- Credentials : ismael@spectre.local / spectre2026

## Architecture hybride cible

```
Claude Code (tâches complexes, raisonnement, budget OK)
    ↓ si budget limité ou données sensibles
OpenClaw + Hermes Agent (local, Pinokio, zéro coût)
    ↓ orchestration quotidienne
qwen3.5:35b-a3b via Ollama (classification, extraction, RAG)
    ↓ workflows automatisés
N8N (automation) + WanGP (vidéo) + autres apps Pinokio

Contrôle depuis mobile :
iPhone → Moshi iOS → SSH/Mosh via Tailscale → terminal → agents
iPhone → Telegram bot → OpenClaw → Ollama → résultats
```

## Contrôle depuis le phone

- **Moshi iOS** (`getmoshi.app`) — terminal SSH/Mosh, voice input Whisper local, push notifications agent
  - Installer Mosh sur WSL : `sudo apt install mosh`
  - Connexion : Tailscale IP → SSH → Claude Code / OpenCode

- **OpenClaw + Telegram** — parler en langage naturel depuis Telegram, OpenClaw exécute sur la machine
  - Nécessite setup Telegram bot token

## Frameworks agents (si on monte CrewAI/n8n)

| Framework | Usage | Compatibilité Ollama |
|---|---|---|
| CrewAI | Équipes d'agents roleplay, prototypage rapide | Oui |
| LangGraph | Workflows stateful, production | Oui |
| AutoGen/AG2 | Conversationnel, R&D | Oui |
| OpenHands | Agent coding autonome | Oui |

**CrewAI** = point d'entrée recommandé, 2-4h pour premier workflow.

## Todo

- [ ] `sudo apt install mosh` sur WSL (5 min)
- [ ] Ouvrir Pinokio 7 et installer : Hermes Agent, OpenClaw, WanGP, N8N
- [ ] Installer Open WebUI sur pc-torrent (Docker ou pip), ouvrir port 3000 firewall Windows
- [ ] Tester Windows-MCP : redémarrer Claude Code, vérifier que les tools windows-control apparaissent
- [ ] Configurer Telegram bot pour OpenClaw (token Telegram + pairing)
- [ ] Installer Moshi iOS sur iPhone, configurer SSH Tailscale
