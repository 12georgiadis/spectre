# tmux Survival Guide

How to never lose a Claude Code session again.

## Machines

| Machine | Tailscale IP | Username | Role |
|---------|-------------|----------|------|
| **Mac Mini M4** | `100.84.223.88` | `ismaelstudiominim4` | Serveur principal (toujours allume) |
| MacBook Air M3 | `100.118.137.41` | `ismaeljoffroychandoutis` | Mobile / backup |
| iPhone 15 Pro Max | `100.68.238.125` | - | Client Blink |
| iPad | `100.88.69.47` | - | Client Blink |

## The golden rule

**Never launch `claude` without tmux.**

```bash
# ALWAYS
tmux new -s nom-du-projet
claude

# NEVER
claude   # naked, no safety net
```

## Connect from anywhere

### From iPhone/iPad (Blink + Tailscale)

```bash
# Connect to Mac Mini (main)
# --server flag required because Homebrew PATH not in SSH env
mosh --server=/opt/homebrew/bin/mosh-server ismaelstudiominim4@100.84.223.88

# Or connect to MacBook Air (mobile)
mosh ismaeljoffroychandoutis@100.118.137.41

# Then reattach
tmux attach
```

### From MacBook Air to Mac Mini

```bash
# SSH or mosh
mosh ismaelstudiominim4@100.84.223.88
tmux attach
```

### From Mac Mini to MacBook Air

```bash
mosh ismaeljoffroychandoutis@100.118.137.41
tmux attach
```

## Daily workflow

```bash
# 1. On the Mac Mini, start a tmux session
tmux new -s work

# 2. Launch Claude Code inside
claude

# 3. Need another project? New tmux window
# Ctrl+B c = new window, Ctrl+B , = rename it

# 4. From phone: connect and reattach
# mosh ismaelstudiominim4@100.84.223.88
# tmux attach

# 5. Done for now? Detach (don't kill!)
# Ctrl+B d
```

## When things go wrong

### Closed iTerm / terminal (but Mac still running)

tmux survives! It runs in the background. Just reopen iTerm and:

```bash
tmux attach
# Then launch claude in the window you need
```

### Battery dies / Mac reboots

tmux dies too. But Claude Code conversations are saved. Recovery:

```bash
# See all saved conversations
claude conversations list

# Resume any conversation inside tmux
tmux new -s work
claude --resume <conversation-id>

# Or use the recovery script
bash ~/resume-sessions.sh
tmux attach -t claude-hub
```

### Blink disconnects / iPhone locks

tmux survives. Just reconnect:

```bash
mosh ismaelstudiominim4@100.84.223.88
tmux attach
```

### Can't reach Mac Mini?

Try MacBook Air as fallback:

```bash
mosh ismaeljoffroychandoutis@100.118.137.41
tmux attach
```

## Navigation cheatsheet

| Action | Keys |
|--------|------|
| **See all windows** | `Ctrl+B w` |
| Next window | `Ctrl+B n` |
| Previous window | `Ctrl+B p` |
| Go to window N | `Ctrl+B 0-9` |
| New window | `Ctrl+B c` |
| Rename window | `Ctrl+B ,` |
| **Detach** (leave without killing) | `Ctrl+B d` |
| Switch between sessions | `Ctrl+B s` |
| Split horizontal | `Ctrl+B "` |
| Split vertical | `Ctrl+B %` |
| Switch pane | `Ctrl+B arrow` |

On Blink keyboard: `Ctrl` = the `^` key (top left).

## Multiple projects

One tmux session with named windows works best:

```bash
tmux new -s work -n miami
# Ctrl+B c  then  Ctrl+B ,  to add and rename windows
```

Or separate sessions for big contexts:

```bash
tmux new -s film
tmux new -s compta
tmux ls              # list all
Ctrl+B s             # switch between sessions
```

## Troubleshooting Blink (iPhone/iPad)

### Mouse/touch sends garbage characters (numbers, escape codes)

The touchscreen sends mouse events that tmux interprets as input. Fix:

```bash
# On the Mac (Mini or Air), add to ~/.tmux.conf:
echo 'set -g mouse off' >> ~/.tmux.conf

# Apply immediately without restart:
tmux set-option -g mouse off
```

Also: disable terminal escape sequences in the active session:
```bash
printf '\033[?1000l\033[?1002l\033[?1003l\033[?1006l'
```

### Vertical text / broken display

Terminal size mismatch between Blink and tmux. Fix:
1. `Ctrl+B d` (detach)
2. `tmux attach` (reattach resets dimensions)

### Can't type in Claude Code TUI

Keyboard not reaching the TUI. Fix:
1. `Ctrl+C` to exit Claude Code
2. Relaunch `claude` or `clauded`

### OAuth auth from iPad (no browser)

Claude Code auth requires a browser on the Mac. Options:
- Use AnyDesk from iPad to control Safari on the Mini
- Copy credentials from another authenticated Mac:
  ```bash
  scp ~/.claude/.credentials.json ismaelstudiominim4@100.84.223.88:~/.claude/.credentials.json
  ```
- Auth stays valid as long as the session runs

### mosh: NoMoshServerArgs error

Homebrew's mosh-server is not in the default SSH PATH. Fix:
```bash
mosh --server=/opt/homebrew/bin/mosh-server user@host
```

## Discipline

1. **Start of day**: `tmux new -s work` then `claude`
2. **During the day**: `Ctrl+B c` for new windows, `Ctrl+B ,` to name them
3. **End of day**: `Ctrl+B d` to detach (don't kill!)
4. **From phone**: `mosh` then `tmux attach`
5. **After reboot**: `claude --resume <id>` inside tmux
