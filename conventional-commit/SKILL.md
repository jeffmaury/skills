---
name: conventional-commit
description: >-
  Generate a conventional commit message from staged git changes. Analyzes
  the diff, infers the commit type (feat, fix, chore, etc.), scope, and
  description, adds a sign-off line, a Co-Authored-By agent header, and a
  Fixes #xxxx footer (guessed from the branch name or conversation context).
  Shows the message for confirmation before committing. Use when the user
  asks to commit, generate a commit message, write a commit message, or
  wants to commit staged changes. Trigger keywords: commit, commit message,
  conventional commit, git commit, stage and commit, sign off.
---

# Conventional Commit

Generate a conventional commit message from the currently staged changes,
present it for user approval, then commit if confirmed.

## Workflow

### Step 1: Check for staged changes

Run `git diff --cached --stat` to verify there are staged changes.
If nothing is staged, tell the user and stop.

### Step 2: Gather context

Run these commands to understand what changed:

- `git diff --cached` — the full diff of staged changes
- `git branch --show-current` — the current branch name (used to infer issue number)
- `git config user.name` and `git config user.email` — for the sign-off line

### Step 3: Infer the issue number

Extract an issue number for the `Fixes #xxxx` footer. Check in order:

1. **Conversation context** — if the user mentioned an issue number earlier, use it
2. **Branch name** — look for patterns like `GH-1234`, `issue-1234`, `fix/1234`,
   `feat/1234`, or any sequence of digits that looks like an issue reference
3. If no issue number can be inferred, omit the `Fixes` footer entirely — don't guess

### Step 4: Generate the commit message

Analyze the staged diff and produce a message following the Conventional
Commits specification.

**Subject line format:** `type(scope): description`

- **type** — one of: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`,
  `style`, `perf`, `ci`, `build`, `revert`
- **scope** — the package, module, or area affected (e.g., `renderer`,
  `auth`, `api`). Omit parentheses entirely if the change is too broad
  for a single scope
- **description** — imperative mood, lowercase, no period, under 50 chars

**Body** (optional) — if the change is complex enough to warrant
explanation, add a blank line after the subject and a short paragraph
explaining *what* and *why*. Keep it concise. Omit for trivial changes.

**Footers** — always in this order, each separated by a blank line from
the body:

1. `Fixes #xxxx` (only if an issue number was inferred in Step 3)
2. `Signed-off-by: Full Name <email>` (from git config)
3. `Co-Authored-By: Claude <model> <noreply@anthropic.com>` where `<model>`
   is the model powering the current session (e.g., "Opus 4.6", "Sonnet 4.6").
   Read the model name from the system prompt information.

### Step 5: Present the message and ask for confirmation

Show the full commit message to the user in a code block. Then ask:

- **"Commit"** — proceed to Step 6
- **"Edit"** — ask what to change, revise, and present again
- **"Abort"** — stop without committing

Do NOT commit in the same turn as presenting the message. Stop and wait
for the user's explicit response.

### Step 6: Commit (only after explicit confirmation)

Run `git commit` passing the message via a HEREDOC:

```bash
git commit -m "$(cat <<'EOF'
<the full commit message>
EOF
)"
```

Display the result to the user.

## Example output

```
feat(secret-manager): report provider profiles from OpenShell adapter

Expose provider profile metadata through the OpenShell adapter so that
downstream consumers can discover available secret providers.

Fixes #2390
Signed-off-by: Jeff MAURY <jmaury@example.com>
Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```
