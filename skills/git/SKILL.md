---
name: git
description: Use when the user asks to commit, push, pull, switch or create branches, stash, or manage worktrees in a git repository, including requests such as "commit", "commit and push", or "push".
---

# Git

Act on natural-language git requests directly. The user's request is the instruction. Do not ask for the exact git command.

## Requests

- "commit": commit this session's changes, after approval.
- "commit and push": commit after approval, then push the current branch.
- "push": push the current branch without committing.
- "pull", "switch to <branch>", "create branch <name>", "stash": run the matching git action.

## Commit Workflow

1. Run `git status --short`, `git diff`, and `git diff --staged`.
2. Stage only files changed in this session, using `git add -- <path>`. Do not use `git add .` or `git add -A`.
3. Treat any other changed, staged, or untracked file as unknown. Ask before including it. Do not revert, discard, or modify it.
4. If a file has both session and non-session edits, stop and ask how to proceed.
5. Run `git diff --staged` again and confirm that it contains only approved changes.
6. Write the commit message using the rules below.
7. Show the staged file list, any unknown files left out, and the full commit message. Ask for approval with the options `Approve commit`, `Revise message`, and `Cancel`.
8. Run `git commit` only after `Approve commit` is selected. A free-text "yes" does not count as approval.
9. Show the commit hash and message in the reply.

## Commit Message Rules

- Format: `type(scope): summary`, or `type: summary` when a scope adds no value.
- Types: `feat` for user-visible features, `fix` for bug fixes, `docs` for documentation only, `test` for tests only, `refactor` for behavior-preserving changes, and `chore` for maintenance, config, or tooling. Infer the type from the actual diff.
- Summary: imperative, lower-case unless a proper noun is required, 72 characters or fewer, no trailing period.
- Body: add one when the reason or context is not obvious from the diff. Wrap lines at 72 characters. Explain why, not what. List verification performed, if any. Skip the body for small, self-explanatory changes.
- Language: English. Use direct, neutral, active-voice prose. Keep paragraphs to one to three sentences.
- Do not use "easy", "simple", "just", or "obviously". Do not use hype or filler.
- Rewrite vague summaries such as "update files", "misc changes", or "wip" before asking for approval.
- Rewrites to apply: "In order to" becomes "To", "Utilize" becomes "Use", "Make sure to" becomes "Ensure that", "Simply run" becomes "Run".

## Pull Request Rules

- Title: follow the Commit Message Rules, including format, types, summary length, English, and banned words.
- Description: use direct, neutral, active-voice English with short paragraphs. Include these sections, and omit any section with no content:
  - Summary: what changed, in one to three sentences.
  - Changes: grouped by area or file.
  - Behavior impact.
  - Verification: tests, builds, or docs checks run, or why none were run.
  - Risks and rollout: when relevant.
  - Work items: when linked or supplied.
- Do not include unrelated worktree changes in the description.
- If the branch has uncommitted session changes, ask whether to commit them first, using the Commit Workflow. Do not commit without approval.
- If the current branch is the target branch, stop and ask for a feature branch.

## Push Rules

- Push the current branch. Use `git push -u origin <branch>` when no upstream exists.
- Never force-push. Use `git push --force-with-lease` only when the user explicitly asks.

## Pull, Branch, and Stash Rules

- Use `git pull --ff-only`. Do not use plain `git pull`.
- If a rebase or merge conflicts, stop and ask how to proceed.
- Do not run `git reset --hard`, `git clean`, `git stash drop`, `git stash clear`, or delete a branch unless the user explicitly asks.

## Hooks

- Do not skip hooks with `--no-verify` unless the user asks.

## Worktrees

- Use `tools.opencode.worktree_create`, `tools.opencode.worktree_list`, and `tools.opencode.worktree_remove`.
- Set `force` on removal only when the user confirms forced removal.

## Output

- Report the branch, remote, commit hash, and message when applicable.
- List files included in the commit and any unknown files left out.
- Report any verification skipped.
