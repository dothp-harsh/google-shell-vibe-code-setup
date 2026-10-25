# Setting Up a Vibe-Coding Environment with OpenCode in Google Cloud Shell

This guide assumes you are using Google Cloud Shell as your cloud development machine and want to run OpenCode as your AI coding agent. The recommended approach is to start with the **terminal-based setup** and avoid installing a full desktop or VNC environment unless you later find a real need for it.

---

## 1. Understand Cloud Shell’s Limits First

Cloud Shell is useful, but it is not a permanently allocated development VM. These limits directly affect how you should work:

| Limit                           |               Value | Practical impact                                                                 |
| ------------------------------- | ------------------: | -------------------------------------------------------------------------------- |
| Maximum session length          |            12 hours | You cannot use it as an always-on remote machine                                 |
| Weekly interactive quota        | 50 hours by default | Long daily sessions can exhaust the quota                                        |
| Persistent storage              |     5 GB in `$HOME` | Code and configs persist, but `node_modules` and build artifacts fill it quickly |
| Non-interactive session timeout |          40 minutes | Do not rely on long-running background non-interactive processes                 |
| Inactive home deletion          |      After 120 days | Open Cloud Shell occasionally if you want to keep `$HOME`                        |

Your `$HOME` previously had about **4.4 GB free**. That is enough for small and medium projects, but you should keep only the active repository in Cloud Shell and push work to GitHub regularly.

---

## 2. Install OpenCode Correctly with npm 12

You previously hit this error:

```text
Error: @opencode/cli's postinstall script was not run.
```

This happens because npm 12 blocks install scripts by default. OpenCode needs its `postinstall` script to link the correct platform binary.

Install OpenCode while explicitly allowing scripts only for `@opencode/cli`:

```bash
npm install -g --allow-scripts=@opencode/cli @opencode/cli@2.1.4
```

Verify:

```bash
opencode --version
```

If it still fails, check the current configuration:

```bash
npm config get allow-scripts
npm list -g @opencode/cli
```

If you want future global installs of OpenCode to allow its install script automatically:

```bash
npm config set allow-scripts=@opencode/cli --location=user
```

**Security note:** `--allow-scripts` allows a package to execute code during installation. Only use it for packages you trust. Do not use blanket options such as `--dangerously-allow-all-scripts`.

---

## 3. Authenticate GitHub in Cloud Shell

OpenCode does not manage Git credentials itself. It calls the system `git`. Therefore, GitHub authentication must be configured in Cloud Shell before OpenCode can push changes.

### Check GitHub CLI

```bash
gh --version
```

### Log in with browser/device code

```bash
gh auth login
```

Choose these options:

1. **Account:** `GitHub.com`
2. **Protocol:** `HTTPS`
3. **Authenticate Git with your GitHub credentials:** `Yes`
4. **Authentication method:** `Login with a web browser`

The terminal will show a one-time code and ask you to open:

```text
https://github.com/login/device
```

Complete the authorization in your normal browser, then return to Cloud Shell.

### Configure Git to use GitHub CLI credentials

```bash
gh auth setup-git
gh auth status
```

`gh auth setup-git` configures Git to use GitHub CLI credentials for HTTPS operations. This is what allows `git push` to work without asking for a password every time.

### Set your commit identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your-github-email@example.com"
```

These settings identify your commits. They do not authenticate you.

### Clone your repository

```bash
mkdir -p ~/projects
cd ~/projects
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
```

Replace `USERNAME/REPOSITORY` with your actual repository path.

---

## 4. Connect OpenCode to an AI Model Provider

OpenCode is the coding agent, not the model itself. You need to connect it to a model provider.

Start OpenCode inside the repository:

```bash
cd ~/projects/REPOSITORY
opencode
```

Inside the OpenCode TUI, run:

```text
/connect
```

Select your provider and complete the authentication flow. Credentials are stored in OpenCode’s local auth storage, not in your project repository.

If you prefer environment variables for a session, you can export your API key before starting OpenCode, for example:

```bash
export ANTHROPIC_API_KEY="your-key"
```

However, environment variables do not persist across Cloud Shell sessions. The `/connect` flow is generally more convenient for repeated use.

---

## 5. Initialize Project Context with `AGENTS.md`

When you first open OpenCode inside a repository, run:

```text
/init
```

OpenCode will analyze the repository and generate or update an `AGENTS.md` file. This file is important because it gives the agent project-specific instructions.

A useful `AGENTS.md` should include:

- The actual test command used by the project
- The build command
- The lint command
- Important directories or files that should not be modified
- Known project conventions
- Commands that must not be run without approval

Minimal example:

```markdown
## Project instructions

- Use the repository's existing build and test commands.
- Inspect nearby code before changing a shared interface.
- Do not add dependencies without stating why.
- Do not modify files outside the scope of the requested task.
- Report any checks that could not be run.
```

`AGENTS.md` guides the agent, but it does not replace your review of the final diff.

---

## 6. Set Permission Boundaries

The biggest risk in vibe coding is not that the agent writes bad code. The bigger risk is that the agent runs a destructive command or pushes changes without your review.

You can configure OpenCode to ask before running shell commands. Create or edit `opencode.json` in the project root:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "bash": "ask"
  }
}
```

This makes OpenCode ask for approval before executing shell commands.

If you only want to require approval for pushes, you can use a more targeted permission rule:

```json
{
  "agents": {
    "build": {
      "permissions": [
        {
          "action": "shell",
          "resource": "git push *",
          "effect": "ask"
        }
      ]
    }
  }
}
```

With this rule, normal build and test commands can run, but `git push` requires your confirmation.

**Important:** `"bash": "ask"` is not a complete sandbox. It only controls OpenCode’s shell tool. You still need to protect sensitive credentials and review changes before pushing.

---

## 7. Recommended Vibe-Coding Workflow

OpenCode has two especially useful modes:

| Mode    | Behavior                           | When to use                                                |
| ------- | ---------------------------------- | ---------------------------------------------------------- |
| `plan`  | Read-only exploration and planning | Understanding an unfamiliar codebase and designing changes |
| `build` | Implements changes and runs tools  | After you approve the plan                                 |

Recommended sequence:

1. Start in `plan` mode.
2. Ask OpenCode to analyze the repository and propose a plan.
3. Review the plan yourself.
4. Switch to `build` mode.
5. Let OpenCode implement the changes and run tests.
6. Review the diff carefully.
7. Only then allow `git push`.

A useful instruction template for `build` mode:

> Locate the implementation and existing tests related to [feature/bug]. Reproduce the current behavior with the smallest possible change, then modify only the target behavior while leaving other paths unchanged. Run the project’s existing relevant test command and show the diff and test output. Do not add undocumented dependencies. If authentication fails, branch protection blocks the push, or the push would overwrite remote work, stop and explain instead of forcing the push.

This gives the agent clear scope, verification requirements, and push safety boundaries.

---

## 8. Persistence and Storage Strategy

Cloud Shell’s `$HOME` is only 5 GB, and sessions can end at any time. Do not treat `$HOME` as the only copy of your work.

Check space usage:

```bash
du -sh ~/projects/* 2>/dev/null | sort -rh | head
df -h "$HOME"
```

Recommended practices:

- Push meaningful changes to GitHub regularly.
- Keep only the active project in `$HOME`.
- Delete old local clones when you no longer need them.
- Clean npm cache when needed:

```bash
npm cache clean --force
```

- Avoid keeping multiple large `node_modules` directories in `$HOME`.
- Treat GitHub as the source of truth.

---

## 9. Do Not Install a Desktop or VNC Yet

You previously looked at `dorowu/ubuntu-desktop-lxde-vnc` and a `nomachinewindows10` script. Those setups add a full graphical desktop stack, consume memory and storage, and do not turn Cloud Shell into a genuine Windows VM.

OpenCode is a terminal TUI tool. Cloud Shell already provides:

- A terminal
- A browser-based editor
- Web Preview for local web apps
- Git, Node.js, npm, and Docker

For this workflow, installing a desktop or VNC is unnecessary overhead. Start with the terminal setup first.

---

## 10. Full Command Sequence

```bash
# 1. Install OpenCode with its postinstall script allowed
npm install -g --allow-scripts=@opencode/cli @opencode/cli@2.1.4
opencode --version

# 2. Authenticate GitHub
gh auth login
gh auth setup-git
gh auth status
git config --global user.name "Your Name"
git config --global user.email "your-github-email@example.com"

# 3. Clone your repository
mkdir -p ~/projects
cd ~/projects
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY

# 4. Start OpenCode
opencode

# Inside OpenCode:
# /connect   -> configure your AI model provider
# /init      -> generate AGENTS.md
# Use plan mode first, then build mode
```

---

## Final Summary

Google Cloud Shell plus OpenCode is a practical free setup for vibe coding, but it is a temporary workstation with a 12-hour session limit, a 50-hour weekly quota, and 5 GB of persistent storage. It is not a permanent cloud development server.

The three most important habits for this environment are:

1. **Push your code to GitHub regularly.**
2. **Set OpenCode permissions to ask before risky commands or pushes.**
3. **Write a real `AGENTS.md` with accurate project commands and constraints.**

Start with the terminal-based OpenCode setup. Do not install a desktop or VNC environment unless you later have a specific need for it.
