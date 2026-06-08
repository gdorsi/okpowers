# Installation

Installs skills from this repository into the global agent skills directory and symlinks them for Claude Code discovery.

## Target Layout

```
~/.agents/skills/          # Canonical source (copied from this repo)
~/.claude/skills/          # Symlinks to ~/.agents/skills/{skill-name}
```

## Quick Install

Run from the repository root:

```bash
./install-skills.sh
```

Or copy-paste the script below:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_SKILLS_DIR="${PWD}/skills"
AGENTS_SKILLS_DIR="${HOME}/.agents/skills"
CLAUDE_SKILLS_DIR="${HOME}/.claude/skills"

# --- Install to ~/.agents/skills (canonical source) ---
mkdir -p "$AGENTS_SKILLS_DIR"

for skill_path in "$REPO_SKILLS_DIR"/*; do
  [ -d "$skill_path" ] || continue
  skill_name=$(basename "$skill_path")
  target="$AGENTS_SKILLS_DIR/$skill_name"

  if [ -L "$target" ]; then
    rm "$target"
  elif [ -e "$target" ]; then
    rm -rf "$target"
  fi

  cp -R "$skill_path" "$target"
  echo "Installed: $skill_name -> ~/.agents/skills/$skill_name"
done

# --- Symlink into ~/.claude/skills ---
mkdir -p "$CLAUDE_SKILLS_DIR"

for skill_path in "$AGENTS_SKILLS_DIR"/*; do
  [ -d "$skill_path" ] || continue
  skill_name=$(basename "$skill_path")
  link="$CLAUDE_SKILLS_DIR/$skill_name"

  if [ -L "$link" ]; then
    rm "$link"
  elif [ -e "$link" ]; then
    rm -rf "$link"
  fi

  ln -s "$skill_path" "$link"
  echo "Linked: ~/.claude/skills/$skill_name -> ~/.agents/skills/$skill_name"
done

echo "Done. $(ls -1 "$AGENTS_SKILLS_DIR" | wc -l | tr -d ' ') skills installed."
```

## What It Does

1. **Copies** every directory under `./skills/` to `~/.agents/skills/`  
   (This is the canonical source — safe to edit, tracked by you, not by the repo after install.)

2. **Creates** `~/.claude/skills/` if missing.

3. **Symlinks** each skill from `~/.agents/skills/` into `~/.claude/skills/`.

4. **Is idempotent** — running it again cleans up old symlinks/directories and reinstalls cleanly.

## Uninstall

```bash
rm -rf ~/.agents/skills ~/.claude/skills
```

## Notes for Harness Authors

- If your harness already manages `~/.agents/skills/`, skip the copy step and only do the symlink step.
- If you prefer `rsync` over `cp -R`, replace the copy block. The important part is that `~/.agents/skills/` becomes the mutable canonical source.
- Claude Code discovers skills in `~/.claude/skills/` automatically. No additional configuration is needed.
