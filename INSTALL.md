# Installation

Installs skills from the remote repository into the global agent skills directory and symlinks them for Claude Code discovery. No local clone required.

## Target Layout

```
~/.agents/skills/          # Canonical source (copied from remote repo)
~/.claude/skills/          # Symlinks to ~/.agents/skills/{skill-name}
```

## Quick Install

Copy-paste this script into your terminal:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_URL="git@github.com:gdorsi/okpowers.git"
AGENTS_SKILLS_DIR="${HOME}/.agents/skills"
CLAUDE_SKILLS_DIR="${HOME}/.claude/skills"
TMP_DIR=$(mktemp -d)

echo "Fetching skills from ${REPO_URL}..."
git clone --depth 1 "$REPO_URL" "$TMP_DIR/repo" >/dev/null 2>&1

# --- Install to ~/.agents/skills (canonical source) ---
mkdir -p "$AGENTS_SKILLS_DIR"

for skill_path in "$TMP_DIR/repo/skills"/*; do
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

rm -rf "$TMP_DIR"
echo "Done. $(ls -1 "$AGENTS_SKILLS_DIR" | wc -l | tr -d ' ') skills installed."
```
