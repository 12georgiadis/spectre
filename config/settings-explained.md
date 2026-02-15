# Settings expliqués

Notre `~/.claude/settings.json` et pourquoi chaque choix.

## Privacy
```json
"env": {
  "DISABLE_TELEMETRY": "1",
  "DISABLE_ERROR_REPORTING": "1",
  "CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY": "1"
}
```
Inspiré de Trail of Bits. Claude Code fonctionne pareil, mais n'envoie plus de données d'usage à Anthropic.

## Agent Teams
```json
"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
```
Active les équipes d'agents (subagents en parallèle).

## Deny rules (minimalistes)
```json
"permissions": {
  "deny": [
    "Bash(mkfs *)", "Bash(dd *)",
    "Bash(curl *|bash*)", "Bash(wget *|bash*)",
    "Bash(git push --force*)", "Bash(git push *--force*)",
    "Read(~/.ssh/**)", "Read(~/.gnupg/**)",
    "Read(~/.git-credentials)", "Read(~/Library/Keychains/**)"
  ]
}
```

### Philosophie
On bloque seulement les commandes **irréversibles** qu'aucun agent ne devrait jamais exécuter, même en mode bypass (`clauded`). Pas de blocage sur `rm -rf` ou `Edit ~/.zshrc` car ce sont des opérations courantes en dev.

La séparation `claude` (avec permissions) vs `clauded` (bypass) gère déjà le controle pour les commandes courantes.

## Statusline
```json
"statusLine": {
  "type": "command",
  "command": "~/.claude/scripts/statusline.sh"
}
```
Affiche : [Modèle] dossier | branche + barre contexte % | $cout | durée | cache% | +lignes/-lignes

## Historique
```json
"cleanupPeriodDays": 365
```
Garde les conversations 1 an au lieu de 30 jours par défaut.

## Thinking
```json
"alwaysThinkingEnabled": true
```
Extended thinking activé en permanence pour de meilleures réponses.
