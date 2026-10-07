# Contributing

Thanks for helping improve the BakedBrie plugin.

## Before opening a change

- Search existing issues and pull requests first.
- Keep credentials and account-specific data out of examples, screenshots, logs, and fixtures.
- For security-sensitive findings, follow [SECURITY.md](SECURITY.md) instead of opening a public issue.

## Development principles

Changes should preserve these behaviors unless the change explicitly intends to replace them:

1. Call BakedBrie's `whoami` before account-specific work.
2. Use BakedBrie's `read_docs` when tool behavior is unclear.
3. Never ask users to paste secrets into chat.
4. Do not perform write, send, approval, connection, or destructive actions without user authorization.
5. Keep the installable package under `plugins/bakedbrie/`.
6. Do not commit personal MCP connection IDs, API tokens, OAuth credentials, or captured private BakedBrie data.

## Pull requests

A good pull request should:

- explain the user problem being solved;
- describe any behavior or installation changes;
- update documentation when setup changes;
- keep the plugin version and changelog accurate for user-visible changes;
- pass the repository validation workflow.

Small, focused pull requests are preferred.

## Testing

For connection-related changes, start with a read-only test:

```text
Use BakedBrie.

Call `whoami`.

Tell me only:
1. whether the BakedBrie connection succeeded,
2. the workspace name,
3. my role.

Do not create or modify anything.
```

Only test write operations in a workspace where the test is authorized.

## Packaging

The current repository intentionally uses the Codex compatibility package because the working BakedBrie MCP connection depends on `bearer_token_env_var`.

Do not replace it with portable Agent Plugins `mcp.json` authentication unless the new path is proven to work end to end for BakedBrie's authenticated MCP server.
