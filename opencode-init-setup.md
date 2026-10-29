# Guide 1 — Initialize a New or Existing Project

**Goal:** Make OpenCode understand the project, organize its development documentation, and maintain persistent memory throughout future development.

## Step 1. Open your project

### For an existing project

```bash
git clone YOUR_REPOSITORY_URL
cd YOUR_PROJECT
```

If the repository is already cloned, open its existing directory instead.

### For a new project

```bash
mkdir my-project
cd my-project
git init
```

Do not overwrite existing files or discard uncommitted changes when working with an existing project.

## Step 2. Initialize OpenCode

Start OpenCode:

```bash
opencode
```

Run its initialization command:

```text
/init
```

This helps OpenCode analyze the repository and create or update its `AGENTS.md` instructions.

## Step 3. Establish the persistent development system

Copy and paste the following prompt into OpenCode.

```md
# Objective: Establish a Persistent Development Knowledge System

Analyze this repository and establish a structured development knowledge system that survives OpenCode reinstalls, lost conversations, usage-limit interruptions, and temporary VM destruction.

First inspect the project and propose an implementation plan. After approval, implement the system without unnecessarily changing application code.

## 1. Audit the Project

Inspect:

- Project architecture, frameworks, languages, dependencies, and runtimes.
- Environment setup, build commands, test commands, and development workflows.
- Existing documentation, agent instructions, implementation plans, audits, bug reports, and historical decisions.
- Git status, branches, remotes, and uncommitted changes.

Preserve useful existing information. Do not overwrite or delete historical documentation without checking its contents.

## 2. Create an Organized Knowledge Base

Use `docs/development/` as the central knowledge base unless the project already has a better established structure.

Organize it around these responsibilities:

- `README.md`: Knowledge-base navigation and operating instructions.
- `PROJECT_CONTEXT.md`: Architecture, environment, conventions, and important implementation details.
- `TASKS.md`: Current tasks, priorities, blockers, and statuses.
- `CURRENT_SESSION.md`: Current progress, unfinished work, and exact next action.
- `sessions/`: Individual session records and an index.
- `plans/features/`: Feature specifications and plans.
- `plans/implementations/`: Implementation plans and execution status.
- `audits/ui-ux/`: UI/UX audit reports.
- `audits/bugs/`: Bug investigations and findings.
- `audits/results/`: Verification and regression results.
- `decisions/`: Important technical and product decisions.
- `improvements/`: Improvement proposals and backlog.
- `archive/`: Superseded plans and historical records.

Create indexes and additional categories only when useful. Integrate existing documentation rather than creating unnecessary duplicates.

Keep the root `AGENTS.md` as a concise entry point that directs OpenCode to the knowledge base and requires it to read relevant instructions before substantial work.

## 3. Establish Mandatory Operating Rules

Document these rules in `AGENTS.md` and the knowledge-base README:

- Read the project instructions and current recovery checkpoint before substantial work.
- Inspect the actual code before modifying it.
- Break complex work into verifiable milestones.
- Update `CURRENT_SESSION.md` after meaningful milestones.
- Update `TASKS.md` whenever task status changes.
- Record important decisions, implementation outcomes, test results, blockers, and the exact next action.
- Never wait until a usage limit is reached to save progress.
- Preserve historical records instead of repeatedly overwriting them.
- Verify completed work instead of trusting old notes blindly.
- Never discard existing user changes or include secrets in documentation or Git commits.
- Do not claim that work has been pushed or backed up remotely unless the operation succeeds.

## 4. Manage Development History

Give substantial sessions, plans, audits, and investigations stable identifiers.

Maintain indexes and cross-references so OpenCode can find related work without reading every historical document.

Track the lifecycle of substantial work:

Audit → Findings → Plan → Implementation → Verification → Closure.

Record which findings are fixed, unresolved, deferred, rejected, or superseded. Preserve the original reports and explain important decisions.

Avoid duplicate documents. Update existing records when appropriate, and create new records when the work represents a distinct investigation or milestone.

## 5. Make Recovery Reliable

The system must allow a new OpenCode session to recover without the original conversation.

Document a recovery procedure that:

1. Reads the root `AGENTS.md`.
2. Reads the knowledge-base README, project context, task board, and current checkpoint.
3. Inspects Git status and recent commits.
4. Opens the relevant plans, audits, and session records.
5. Verifies the current implementation and remaining work.
6. Continues from the last verifiable state.

## 6. Protect Work Across Temporary VMs

Document safe Git checkpoint and recovery procedures.

Inspect existing Git configuration before changing anything. Do not automatically commit, push, change remotes, or perform destructive operations without authorization.

Explain how to preserve approved code and documentation in a remote repository. If automated checkpoints are proposed, explain their permissions, frequency, and failure behavior.

Do not rely on a backup process running only inside a VM to preserve data after the VM disappears.

## 7. Final Verification

Validate the folder structure, indexes, instructions, and recovery procedure.

Preserve existing project files and user changes. Report what was created, what was migrated, what was verified, and what still requires manual configuration.

First present the audit findings and proposed plan. Implement only after approval.
```

## Step 4. Review and save the setup

Once OpenCode has implemented the approved plan:

1. Review the generated `AGENTS.md` and knowledge-base structure.
2. Confirm that existing documentation and audit reports remain intact.
3. Check that `CURRENT_SESSION.md` and `TASKS.md` explain how progress is maintained.
4. Ensure your repository has a working remote.

Save the approved files to Git:

```bash
git add AGENTS.md docs/development
git commit -m "Add persistent development knowledge system"
git push
```

If you are using a new branch, configure its upstream as needed. Review staged changes before committing and exclude secrets and sensitive files.

**Result:** Every future OpenCode session has a consistent set of project instructions, organized development history, and a known location for the current progress checkpoint.

---
