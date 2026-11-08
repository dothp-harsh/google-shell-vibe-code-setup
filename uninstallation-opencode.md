## Step 1: Create a Bash script

Copy and paste this entire block into your Cloud Shell terminal:

```bash
cat > uninstall-opencode.sh <<'EOF'
#!/usr/bin/env bash

set +e

echo "========================================"
echo "       OpenCode Uninstaller"
echo "========================================"

# Step 1: Preview uninstall
if command -v opencode >/dev/null 2>&1; then
    echo ""
    echo "[1/4] Previewing OpenCode uninstallation..."
    opencode uninstall --dry-run

    echo ""
    read -r -p "Proceed with OpenCode uninstall? (y/N): " answer

    if [[ "$answer" =~ ^[Yy]$ ]]; then
        opencode uninstall
    else
        echo "Skipped OpenCode uninstall."
    fi

    # Step 2: Check diagnostic paths
    if command -v opencode >/dev/null 2>&1; then
        echo ""
        echo "[2/4] Checking OpenCode paths..."
        opencode debug paths
    fi
else
    echo "OpenCode command not found. Skipping CLI uninstall."
fi

# Step 3: Remove remaining user data
echo ""
echo "[3/4] Checking common OpenCode directories..."

paths=(
    "$HOME/.local/share/opencode"
    "$HOME/.cache/opencode"
    "$HOME/.config/opencode"
    "$HOME/.local/state/opencode"
)

existing=()

for path in "${paths[@]}"; do
    if [[ -e "$path" ]]; then
        echo "Found: $path"
        existing+=("$path")
    fi
done

if [[ ${#existing[@]} -gt 0 ]]; then
    echo ""
    echo "WARNING: These directories may contain sessions, credentials,"
    echo "configuration, logs, and other personal data."

    read -r -p "Permanently delete these directories? (y/N): " answer

    if [[ "$answer" =~ ^[Yy]$ ]]; then
        for path in "${existing[@]}"; do
            rm -rf -- "$path"
            echo "Removed: $path"
        done
    else
        echo "Skipped data cleanup."
    fi
else
    echo "No remaining OpenCode directories found."
fi

# Step 4: Optional npm cache cleanup
echo ""
echo "[4/4] npm cache cleanup"

if command -v npm >/dev/null 2>&1; then
    read -r -p "Clean the entire npm cache? (y/N): " answer

    if [[ "$answer" =~ ^[Yy]$ ]]; then
        npm cache clean --force
    else
        echo "Skipped npm cache cleanup."
    fi
else
    echo "npm is not installed."
fi

echo ""
echo "========================================"
echo "Cleanup process finished."
echo "========================================"
EOF

chmod +x uninstall-opencode.sh
```

## Step 2: Run the script

Execute:

```bash
./uninstall-opencode.sh
```

Follow the confirmation prompts.

### Important: Check how OpenCode was installed

Your earlier terminal output indicated that you attempted to install `@opencode/cli` through npm.

If OpenCode was installed globally using npm, removing its data directories alone may not uninstall the package. After the script finishes, check:

```bash
command -v opencode
npm list -g --depth=0
```

If `opencode` is still installed globally through npm and you want to remove it, inspect the package list and then uninstall the appropriate package.

**One caution:** Cleaning the npm cache affects other npm packages too, not just OpenCode. You can safely answer `N` to that prompt if you want to preserve the cache.
