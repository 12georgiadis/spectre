# tmux Survival Guide

How to never lose a Claude Code session again.

## The golden rule

**Never launch `claude` without tmux.**

```bash
# ALWAYS
tmux new -s nom-du-projet
claude

# NEVER
claude   # naked, no safety net
```

## Daily workflow

### On the Mac (iTerm2)

```bash
# Start your day - create a session
tmux new -s miami

# Launch Claude Code inside
claude

# Need another project? Open a new tmux window
# Ctrl+B c = new window
# Then launch another claude inside
```

### From iPhone/iPad (Blink + Tailscale)

```bash
# 1. Connect
mosh ismaeljoffroychandoutis@100.118.137.41

# 2. Reattach to everything
tmux attach -t miami
```

That's it. Everything is exactly where you left it.

## When things go wrong

### Battery dies / Mac reboots

tmux dies too. But Claude Code conversations are saved. Recovery:

```bash
# See all saved conversations
claude conversations list

# Resume any conversation
tmux new -s nom
claude --resume <conversation-id>
```

### iTerm2 crashes (but Mac is still on)

tmux survives. Just reopen iTerm2:

```bash
tmux attach
# Everything is still running
```

### Blink disconnects / iPhone locks

tmux survives. Just reconnect:

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
| Split horizontal | `Ctrl+B "` |
| Split vertical | `Ctrl+B %` |
| Switch pane | `Ctrl+B arrow` |

On Blink keyboard: `Ctrl` = the `^` key (top left).

## Multiple projects

One tmux session with named windows works best:

```bash
# Create session with first project
tmux new -s work -n miami

# Add more windows
Ctrl+B c    # then rename with Ctrl+B ,
```

Or multiple sessions for big separation:

```bash
tmux new -s film
tmux new -s compta
tmux new -s dev

# List all sessions
tmux ls

# Switch between sessions
Ctrl+B s
```

## Recovery script

If the Mac reboots, run `~/resume-sessions.sh` to recreate all windows from saved conversations. Keep this script updated.

## Discipline

1. **Start of day**: `tmux new -s work` then `claude`
2. **During the day**: `Ctrl+B c` for new windows, `Ctrl+B ,` to name them
3. **End of day**: `Ctrl+B d` to detach (don't kill!)
4. **From phone**: `mosh` then `tmux attach`
5. **After reboot**: `claude --resume <id>` inside tmux
