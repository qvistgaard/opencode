# General Instructions

When planning code changes, preview intended patches and code changes before applying them. This applies in both build mode and plan mode.

For non-trivial code changes, include a concise plan before editing and describe files/functions expected to change. Before applying patches, show the relevant patch or code-change preview so the user can review intent.

Before applying changes, present patch set grouped by file. Require user to review and verify each file's proposed changes individually before applying them.

## Pre-Approved Read Operations

- `kubectl get *` and `kubectl describe *` are allowed without asking for permission. Treat these as read-only cluster inspection commands.
- Azure DevOps MCP read operations are allowed without asking for permission. Create, update, comment, and pull request operations still require normal dry-run or confirmation behavior.
