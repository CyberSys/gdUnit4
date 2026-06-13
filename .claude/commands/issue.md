---
name: issue
description: Fetch and display a GitHub issue from the gdUnit4 repository. Pass the issue number as the argument (e.g. /issue 1157).
context: fork
---

# Issue Viewer

## Input

The issue number is provided as the argument: `$ARGUMENTS`

Fetch the issue from GitHub:

```bash
gh issue view $ARGUMENTS --repo godot-gdunit-labs/gdUnit4 \
  --json number,title,body,state,labels,assignees,milestone,comments
```

If the issue is not found, report the error and stop.

---

## Output

Display the issue in this format:

```
#<number>: <title>

```

After displaying the issue, briefly summarise in 1–2 sentences what the problem or request is, so the context is immediately clear for working on it.

---

## Branch Check

Issue titles follow the pattern '[PREFIX]-[Number]:', e.g. GD-1234, DOC-10, TASK-1001 followed by a title

```
GD-XXX: Brief description of the issue
```

Extract the issue ID from the title using this pattern: the prefix before the first `:` (e.g. `GD-1157`).

Get the current branch first:

```bash
git branch --show-current
```

Then check for uncommitted changes:

```bash
git status --porcelain
```

Compare the current branch name to the extracted issue ID (case-insensitive):

- **Match** — print: `✅ Branch 'GD-XXXX' matches issue GD-XXXX` and stop — no branch switch needed.

- **No match and no uncommitted changes** — print: `⚠️  Current branch '<branch>' does not match issue GD-XXXX`
  Then ask the user: "Do you want to create branch `GD-XXXX` from latest master and switch to it?"
  - If yes:
    ```bash
    git fetch origin master
    git checkout -b GD-XXXX origin/master
    ```
    If the branch already exists locally, switch to it instead: `git checkout GD-XXXX`
    Confirm with: `✅ Switched to branch 'GD-XXXX'`
  - If no: continue without switching

- **No match and uncommitted changes exist** — print:

  ```
  ⚠️  Current branch '<branch>' has uncommitted changes and does not match issue GD-XXXX.

  How do you want to proceed?
    1. Carry changes to new branch — create 'GD-XXXX' from fresh master and bring uncommitted changes along (no commit)
    2. Stash and switch — stash current changes, then create 'GD-XXXX' from fresh master with a clean slate
    3. Abort — do nothing
  ```

  Wait for the user to pick 1, 2, or 3.

  **If 1 (carry changes):**
  ```bash
  git fetch origin master
  git checkout -b GD-XXXX origin/master
  ```
  If the branch already exists locally: `git checkout GD-XXXX`
  The uncommitted changes move to the new branch automatically.
  Confirm with: `✅ Switched to branch 'GD-XXXX' — uncommitted changes carried over.`

  **If 2 (stash and switch):**
  ```bash
  git stash push -m "WIP on <branch> before switching to GD-XXXX"
  git fetch origin master
  git checkout -b GD-XXXX origin/master
  ```
  If the branch already exists locally: `git checkout GD-XXXX`
  Print the stash label from the `git stash push` output (e.g. `stash@{0}: WIP on <branch> before switching to GD-XXXX`).
  Confirm with: `✅ Changes stashed as '<stash-label>'. Switched to clean branch 'GD-XXXX'.`

  **If 3 (abort):**
  Print `Aborted.` and stop.
