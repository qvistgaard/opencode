---
description: Push the current Git branch and create an Azure DevOps pull request with auto-complete.
agent: build
---

Create an Azure DevOps pull request for the current Git branch.

Before creating the PR:

- Inspect `git status`, `git diff`, `git log --oneline -10`, remote tracking, and the diff from the target branch.
- Do not include unrelated worktree changes in the PR summary.
- If there are uncommitted changes, stop and ask whether to commit them. Do not commit unless explicitly asked.
- If the current branch is the target branch, stop and ask for a feature branch.
- Push the current branch to its remote, setting upstream when needed.

PR requirements:

- Use Azure DevOps.
- Source branch is the current branch.
- Target branch is the repository default branch unless the user specifies another branch in `$ARGUMENTS`.
- Enable auto-complete.
- Delete the source branch after merge.
- Prefer squash merge unless the repository policy requires another merge strategy.
- Transition work items only when work items are linked or specified.

Title:

- Follow Conventional Commits: `type(scope): summary` or `type: summary`.
- Infer `type` from the actual changes:
  - `feat` for user-visible feature work.
  - `fix` for bug fixes.
  - `docs` for documentation-only changes.
  - `test` for test-only changes.
  - `refactor` for behavior-preserving code changes.
  - `chore` for maintenance/config/tooling.
- Keep the summary imperative, concise, and lower-case unless a proper noun is required.

Description must include:

- Summary of what changed.
- Details grouped by area or file when useful.
- Behavior impact.
- Verification performed, including tests, builds, docs validation, or why none was run.
- Risks or rollout notes when relevant.
- Linked work items when present or supplied in `$ARGUMENTS`.

Use the Azure DevOps pull request tool with `auto_complete: true`, `delete_source_branch: true`, and `squash: true` unless explicitly told otherwise.

After creating the PR, return the PR URL and a concise summary.
