# Guide 2 — Recover When OpenCode Reaches Its Usage Limit

**Goal:** Continue an unfinished task after uninstalling and reinstalling OpenCode, even if its local conversation history has been deleted.

## Step 1. Understand the backup system

Your recovery depends on three independent sources:

| Source                  | What it preserves                                               |
| ----------------------- | --------------------------------------------------------------- |
| Git remote              | Code and documentation that have been committed and pushed      |
| `docs/development/`     | Project context, plans, audits, tasks, and recovery checkpoints |
| OpenCode session export | Conversation history, when successfully exported and backed up  |

**Important:** A checkpoint stored only inside the VM is not protected against VM destruction. Push important progress to your remote repository regularly.

## Step 2. Before the limit is reached

Your project instructions should already require OpenCode to update its checkpoint throughout development.

For every meaningful milestone, it should update:

- `CURRENT_SESSION.md` with the latest verified state and next action.
- `TASKS.md` with completed, pending, and blocked tasks.
- The relevant implementation plan or audit with actual results.
- Session history when a significant milestone or decision warrants a record.

You should not need to wait until the end of a long task to create these records.

### Save the latest checkpoint to Git

When terminal access is available, inspect the changes:

```bash
git status
git diff
```

Review the files before staging them. Then save the approved changes:

```bash
git add -A
git commit -m "Checkpoint development progress"
git push
```

Do not stage secrets, credentials, `.env` files, or sensitive session exports.

If you have authorized an automated checkpoint process, it can perform these steps under your configured rules. Otherwise, you must push the changes yourself.

**A local commit alone is not enough:** it disappears if the VM is destroyed. Verify that the push succeeds.

## Step 3. Export the OpenCode session when possible

If you can still access OpenCode's session data, export the current session using the syntax supported by your installed version.

For versions that support these commands:

```bash
opencode session list
opencode export YOUR_SESSION_ID > session-backup.json
```

Store the resulting backup outside the VM, in private storage.

After reinstalling, you may be able to import it using:

```bash
opencode import session-backup.json
```

Check the installed CLI's help and official documentation to confirm the supported syntax.

**If the usage limit prevents you from accessing the session, skip this step.** Recover from the latest pushed checkpoint instead.

Session backups may contain sensitive prompts, code, or other information. Never upload them to a public repository.

## Step 4. Reinstall OpenCode and restore the project

In the new VM, clone your repository:

```bash
git clone YOUR_REPOSITORY_URL
cd YOUR_PROJECT
```

If the repository is already present, inspect its status before pulling or switching branches.

Restore the required runtime, dependencies, environment variables, and tools using the project's documented setup instructions.

If you have an external session backup, restore it when supported. Otherwise, start a fresh OpenCode session.

## Step 5. Use the recovery prompt

Paste this prompt into the new OpenCode session.

```md
# Recover and Resume Interrupted Development

My previous OpenCode session was interrupted because of a usage limit. Its conversation history may have been deleted.

Recover the project from its persistent documentation, Git history, and actual source code.

## Phase 1: Recover Context

1. Read the root `AGENTS.md`.
2. Read `docs/development/README.md`, `PROJECT_CONTEXT.md`, `TASKS.md`, and `CURRENT_SESSION.md` if present.
3. Inspect Git status, the current branch, recent commits, and available remote changes.
4. Follow references to the active implementation plan, relevant audits, previous sessions, and technical decisions.
5. Inspect the relevant source files to verify what has actually been implemented.

## Phase 2: Identify the Exact State

Determine:

- The original objective and acceptance criteria.
- What has been completed and verified.
- What remains unfinished or partially implemented.
- Which files were modified.
- Any known bugs, failed tests, blockers, or unresolved decisions.
- The exact next action required.

Do not assume that old progress notes are correct if the code contradicts them.

Do not overwrite, discard, or reset existing user changes. Do not repeat completed work unnecessarily.

If information is missing, explain what you can establish and identify any uncertainty before proceeding.

## Phase 3: Resume Development

Continue from the last verifiable state and work toward the original objective.

Follow the existing project instructions, coding conventions, implementation plan, and acceptance criteria.

Update `CURRENT_SESSION.md` after meaningful milestones. Update `TASKS.md`, relevant plans, audit records, and session history when appropriate.

Preserve historical records and avoid unnecessary documentation duplication.

Do not claim that tests pass, work is complete, or backups exist unless verified.

First give me a concise summary of the recovered state and the next action. Then continue implementing the unfinished task.
```

## Step 6. Verify the recovery

Before continuing substantial development, confirm:

- [ ] The correct repository and branch are open.
- [ ] The latest pushed code and checkpoint are present.
- [ ] The current objective and next action are understood.
- [ ] The relevant implementation plan and previous audit findings are available.
- [ ] Existing uncommitted changes have been preserved.
- [ ] The required dependencies and environment are configured.
- [ ] The latest tests and build status have been checked.

**Result:** OpenCode can reconstruct your development context and continue working even when the original AI conversation is permanently lost.

---

# Final Operating Rules

Keep these rules in mind throughout development.

| Situation                        | Required action                                                         |
| -------------------------------- | ----------------------------------------------------------------------- |
| Starting a new project           | Initialize instructions and the development knowledge base              |
| Starting an existing project     | Audit the codebase and preserve existing documentation                  |
| Beginning a complex task         | Identify the objective, plan, and acceptance criteria                   |
| Completing a milestone           | Update the checkpoint and task status                                   |
| Completing an audit              | Record findings, priorities, and verification status                    |
| Reaching a usage limit           | Recover from the latest checkpoint; export the session if possible      |
| Reinstalling OpenCode            | Restore the repository, read the recovery files, and continue           |
| Losing the VM                    | Clone the remote repository and reconstruct the environment             |
| Losing the original conversation | Use the recovery prompt to reconstruct the task from persistent records |

## One final distinction

This setup preserves **development context**, but it does not automatically guarantee that every code change is saved remotely.

For the strongest protection, combine:

1. A persistent development knowledge base.
2. Frequent, verified Git pushes.
3. External OpenCode session backups when available.
4. A reproducible VM setup and recovery procedure.

That gives you a workflow in which a new OpenCode instance can understand your project, find previous plans and audits, identify the current task, and continue without depending on the previous conversation.
