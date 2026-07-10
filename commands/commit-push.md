---
description: Commit this session's file changes with a Conventional Commit message and push the current branch.
agent: build
---

Commit and push only file changes made in this session.

Before committing:

- Inspect `git status --short`, `git diff`, `git diff --staged`, and `git log --oneline -10`.
- Identify which changed files were edited or created during this opencode session.
- Treat every other changed, deleted, staged, or untracked file as unknown.
- If unknown files exist, ask the user whether each unknown file should be included, excluded, or whether the command should stop.
- Do not include unknown files without explicit confirmation.
- Do not revert, discard, or modify unknown files.
- If staged changes already exist, verify whether they belong to this session. Ask before committing them when ownership is unclear.
- If no session changes are available after exclusions, stop without committing.

Staging rules:

- Stage only approved files.
- Prefer path-specific `git add -- <path>` commands.
- For files that contain mixed session and non-session edits, stop and ask how to proceed instead of staging the whole file.

Commit message requirements:

- Follow Conventional Commits: `type(scope): summary` or `type: summary`.
- Infer `type` from the actual changes:
  - `feat` for user-visible feature work.
  - `fix` for bug fixes.
  - `docs` for documentation-only changes.
  - `test` for test-only changes.
  - `refactor` for behavior-preserving code changes.
  - `chore` for maintenance/config/tooling.
- Add a scope when it gives useful context.
- Keep summary imperative, concise, and lower-case unless a proper noun is required.
- Include body lines when useful to explain why the change was made, important context, or verification performed.
- Do not use vague messages such as `update files`, `misc changes`, or `wip`.

Wording requirements:

- Use the writing style defined in `@skills/antora-docs/SKILL.md`.
- Prefer direct, instructional prose over marketing language.
- Use a calm expert tone: precise, neutral, and helpful.
- Use active voice where clear.
- Keep paragraphs short, usually one to three sentences.
- Avoid hype, humor, sales copy, rhetorical questions, vague claims, and unexplained abbreviations.
- Do not say `easy`, `simple`, `just`, or `obviously`.
- Use `must`, `should`, and `can` only with exact meanings.
- Prefer these rewrites in commit bodies and summaries when applicable:
  - `This guide will walk you through` -> `This page explains`
  - `Simply run` -> `Run`
  - `Make sure to` -> `Ensure that`
  - `You can easily configure` -> `Configure`
  - `Powerful and flexible` -> describe concrete capability
  - `It is important to note` -> direct sentence
  - `In order to` -> `To`
  - `Utilize` -> `Use`

Before `git commit`:

- Inspect the staged diff with `git diff --staged`.
- Confirm the staged diff contains only approved changes.
- If the staged diff contains unknown or unrelated changes, stop and ask.
- Print the full proposed commit message, including subject and body.
- Ask the user to approve the commit message or provide additional information to include.
- Do not run `git commit` until the user approves the final commit message.

After committing:

- Push the current branch to its upstream.
- If no upstream exists, push with `git push -u origin <current-branch>`.
- Do not force-push unless the user explicitly asked.

Output:

- Commit hash and commit message.
- Pushed branch and remote.
- Files included.
- Unknown files excluded, if any.
- Any verification that was run or skipped.

Use `$ARGUMENTS` only for extra user instructions, such as preferred commit type, scope, target remote, or whether specific unknown files are allowed.
