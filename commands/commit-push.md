---
description: Commit this session's file changes with a Conventional Commit message and push the current branch.
agent: build
---

This command requests a commit and a push. Do both.

1. Commit using the Commit Workflow and Commit Message Rules in the `git` skill. Use the commit approval step.
2. After the commit, push using the Push Rules in the `git` skill. The push was requested by this command, so do not treat it as an extra action or ask whether to push.
3. Ask the push question only to choose the push option, such as the upstream or new-branch option. Do not ask whether to push at all.

Use `$ARGUMENTS` only for extra user instructions, such as preferred commit type, scope, target remote, or whether specific unknown files are allowed.
