# Run the cleanup
Warning: This permanently deletes your projects, uncommitted work, OpenCode sessions, configuration, scripts, and other files in your home directory, except the explicitly preserved paths.

The following script preserves:
- `~/.config/gh` — GitHub CLI authentication and configuration.
- `~/.ssh` — SSH keys and configuration.
- `~/.git-credentials` — Git HTTPS credentials, if present.
- `~/.gitconfig` — Git settings, including credential-helper configuration.
It removes other home-directory contents, including any other files inside ~/.config.

Review the preservation list carefully before running it.

```bash
cat > ~/clean-cloud-shell.sh <<'EOF'
#!/usr/bin/env bash
set -Eeuo pipefail

HOME_DIR="$HOME"

echo "This will permanently delete development data from:"
echo "$HOME_DIR"
echo
echo "The following paths will be preserved if present:"
printf '  %s\n' \
  "$HOME_DIR/.config/gh" \
  "$HOME_DIR/.ssh" \
  "$HOME_DIR/.git-credentials" \
  "$HOME_DIR/.gitconfig"
echo

read -r -p 'Type DELETE EVERYTHING to continue: ' CONFIRM

if [[ "$CONFIRM" != "DELETE EVERYTHING" ]]; then
    echo "Cancelled. Nothing was deleted."
    exit 1
fi

# Preserve the GitHub CLI configuration directory, but not other
# contents of ~/.config.
if [[ -d "$HOME_DIR/.config" ]]; then
    find "$HOME_DIR/.config" -mindepth 1 -maxdepth 1 \
        ! -name gh -exec rm -rf -- {} +
fi

# Delete all other top-level home-directory contents except
# the explicitly preserved GitHub authentication paths.
find "$HOME_DIR" -mindepth 1 -maxdepth 1 \
    ! -name .config \
    ! -name .ssh \
    ! -name .git-credentials \
    ! -name .gitconfig \
    -exec rm -rf -- {} +

echo
echo "Cleanup complete."
echo
echo "Remaining home-directory contents:"
ls -la "$HOME_DIR"

echo
echo "Disk usage:"
df -h /
EOF

chmod 700 ~/clean-cloud-shell.sh
bash ~/clean-cloud-shell.sh
```
