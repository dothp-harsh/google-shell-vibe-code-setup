# How to Completely Uninstall OpenCode

### Step 1: Preview the Uninstallation

Open your terminal and run:

```bash
opencode uninstall --dry-run
```

This previews what will be removed without deleting anything.

### Step 2: Uninstall OpenCode

Run the following command and confirm when prompted:

```bash
opencode uninstall
```

### Step 3: Remove Leftover Files

If you want a completely clean removal, first check the file locations:

```bash
opencode debug paths
```

Then manually delete these directories if they exist:

- `~/.local/share/opencode`
- `~/.cache/opencode`
- `~/.config/opencode`
- `~/.local/state/opencode`

**Warning:** These directories may contain your authentication details, configurations, logs, and session data. Back up anything you need before deleting them.

### Step 4: Clean the npm Cache (Optional)

If you installed OpenCode using npm, run:

```bash
npm cache clean --force
```

**Done!** OpenCode and its remaining data should now be removed.
