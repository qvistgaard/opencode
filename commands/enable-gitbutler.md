---
description: Enable GitButler for the current repo and verify the GitButler MCP setup.
---

Enable GitButler for this repository and verify opencode can use the GitButler MCP. First confirm the current working directory is inside a Git repository. Use `/etc/profiles/per-user/wayland/bin/but` when invoking GitButler. Check whether GitButler is already enabled for this repo. If it is not enabled, enable it with the appropriate `but` command for the current repo. Then verify the repo is enabled and report the result. Also verify that the global opencode config contains a `gitbutler` MCP server using `/etc/profiles/per-user/wayland/bin/but mcp --internal`; if the current running opencode session cannot see it yet, explain that opencode must be restarted because MCP config is loaded at startup.
