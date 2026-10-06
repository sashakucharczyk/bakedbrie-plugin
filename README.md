# BakedBrie Plugin for ChatGPT and Codex

An unofficial plugin for using [BakedBrie](https://bakedbrie.com/) from Codex and, with an additional remote MCP bridge, the ChatGPT UI.

Developed by Sasha Kucharczyk.

> **Status:** The direct BakedBrie MCP integration has been tested successfully in Codex. ChatGPT requires an additional remote MCP/OAuth layer because BakedBrie's current MCP authentication uses a bearer token rather than the OAuth flow ChatGPT expects for authenticated custom MCP servers.

## What this plugin does

The plugin combines:

- a BakedBrie skill that gives the model safe, repeatable workflow guidance;
- BakedBrie's existing MCP endpoint at `https://api.bakedbrie.com/mcp`;
- a Git-backed plugin marketplace for installation in Codex and ChatGPT Desktop;
- optional personal remote-MCP wiring for using BakedBrie tools from the normal ChatGPT UI.

The plugin does not reimplement BakedBrie. The available tools and behavior ultimately depend on BakedBrie's MCP server.

## Repository structure

The installable plugin package is intentionally kept in one canonical location:

```text
.agents/
└── plugins/
    └── marketplace.json

plugins/
└── bakedbrie/
    ├── .codex-plugin/
    │   └── plugin.json
    ├── .mcp.json
    └── skills/
        └── bakedbrie/
            └── SKILL.md
```

`.agents/plugins/marketplace.json` points to `./plugins/bakedbrie`.

## Codex architecture

Codex can use BakedBrie's bearer-token authentication directly:

```text
Codex
  ↓
BakedBrie plugin
  ↓
BAKEDBRIE_TOKEN environment variable
  ↓
https://api.bakedbrie.com/mcp
```

The token stays outside the repository.

## Install in Codex

### 1. Create a BakedBrie API token

Create an API token in BakedBrie.

Do not paste the token into an AI conversation or commit it to Git.

### 2. Set `BAKEDBRIE_TOKEN`

Windows PowerShell, current session:

```powershell
$env:BAKEDBRIE_TOKEN = "<your-bakedbrie-token>"
```

To persist it for your Windows user:

```powershell
[Environment]::SetEnvironmentVariable(
    "BAKEDBRIE_TOKEN",
    $env:BAKEDBRIE_TOKEN,
    "User"
)
```

macOS/Linux:

```bash
export BAKEDBRIE_TOKEN="<your-bakedbrie-token>"
```

Restart applications that were already running after setting the variable.

### 3. Add the marketplace

```bash
codex plugin marketplace add sashakucharczyk/bakedbrie-plugin --ref main
```

### 4. Confirm the plugin is available

```bash
codex plugin list --marketplace bakedbrie --available --json
```

### 5. Install it

```bash
codex plugin add bakedbrie@bakedbrie
```

### 6. Run a read-only connection test

Start a fresh Codex session and ask:

```text
Use BakedBrie.

Call `whoami`.

Tell me only:
1. whether the BakedBrie connection succeeded,
2. the workspace name,
3. my role.

Do not create or modify anything.
```

## ChatGPT UI: why extra setup is needed

The checked-in `.mcp.json` is designed for Codex and references the local `BAKEDBRIE_TOKEN` environment variable.

ChatGPT custom MCP connections support authenticated remote MCP servers through OAuth. ChatGPT cannot simply read a local Windows/macOS environment variable or present an arbitrary user-supplied BakedBrie API key to the upstream server.

For personal use, the practical architecture is:

```text
ChatGPT
  ↓ OAuth
Personal remote MCP bridge
  ↓ BAKEDBRIE_TOKEN stored as a server secret
https://api.bakedbrie.com/mcp
```

A small Cloudflare Worker is a convenient way to host that bridge.

OpenAI references:

- https://developers.openai.com/api/docs/guides/custom-mcp-server
- https://developers.openai.com/plugins/build/auth
- https://developers.openai.com/plugins/build/plugins

Cloudflare reference:

- https://developers.cloudflare.com/agents/model-context-protocol/guides/remote-mcp-server/

## Personal Cloudflare bridge

This repository does **not** contain a personal Worker deployment, OAuth credentials, or a ChatGPT MCP connection ID.

For a personal bridge, the Worker should:

1. expose a Streamable HTTP MCP endpoint, normally ending in `/mcp`;
2. implement an OAuth-compatible authentication flow for ChatGPT;
3. restrict access to the intended user;
4. store `BAKEDBRIE_TOKEN` as a Cloudflare secret;
5. forward authorized MCP requests to `https://api.bakedbrie.com/mcp`;
6. never return, log, or expose the upstream BakedBrie token.

### Store the BakedBrie token

From the Worker project:

```bash
npx wrangler secret put BAKEDBRIE_TOKEN
```

Enter the token only when Wrangler prompts for it.

Do not place the value in source code, `wrangler.jsonc`, committed environment files, logs, or this repository.

If the bridge uses OAuth client credentials or a cookie-encryption key, store those with `wrangler secret put` as well.

### Connect the bridge to ChatGPT

After deploying the Worker:

1. In ChatGPT, open **Plugins**.
2. Select **Add/Create custom MCP server**.
3. Enter the deployed HTTPS endpoint ending in `/mcp`.
4. Configure/complete OAuth authentication.
5. Review the risk warning and create the connection as a plugin.
6. Install it and test it in a new conversation.

For the first test, call BakedBrie's `whoami` tool and do not perform writes.

### Combine the ChatGPT connection with this skill

If you want the ChatGPT connection and this repository's BakedBrie skill to appear as one local plugin:

1. create the custom MCP connection in ChatGPT;
2. copy its technical connection ID;
3. use OpenAI Plugin Creator to wire that registered connection into a local copy of `plugins/bakedbrie`;
4. let Plugin Creator create the local `.app.json` and update the local manifest to reference it;
5. reinstall/refresh the plugin in ChatGPT Desktop.

The technical connection ID and resulting `.app.json` are account-specific. They are intentionally excluded from this repository.

Do not modify the checked-in manifest to reference a missing personal `.app.json`; keep the account-specific ChatGPT wiring local.

## Security and privacy

Never commit credentials.

The repository may contain the environment-variable name:

```text
BAKEDBRIE_TOKEN
```

It must never contain its value.

The repository ignores common local secret/configuration files, including:

```gitignore
.env
.env.*
.dev.vars
.app.json
*.key
*.pem
secrets/
.wrangler/
```

Keep these out of Git:

- BakedBrie API tokens;
- OAuth client secrets;
- Cloudflare authentication secrets;
- cookie-encryption keys;
- local `.dev.vars`;
- personal ChatGPT MCP connection IDs or generated `.app.json`;
- captured MCP responses containing account-specific workspace, board, card, or user data.

If a secret is ever committed, deleting it from the latest file is not enough. Rotate it and remove it from Git history.

## Updating the plugin

After changing the checked-in plugin:

```bash
git add .
git commit -m "Update BakedBrie plugin"
git push
```

Refresh the marketplace:

```bash
codex plugin marketplace upgrade bakedbrie
```

If necessary:

```bash
codex plugin remove bakedbrie@bakedbrie
codex plugin add bakedbrie@bakedbrie
```

Restart ChatGPT Desktop after marketplace or plugin-manifest changes.

## Important limitations

- This is an unofficial integration and is not maintained by BakedBrie.
- BakedBrie's MCP API may change independently of this repository.
- The personal Cloudflare bridge pattern is intended for one user's credentials. It is not a production multi-user credential service.
- A broadly distributable ChatGPT integration should ultimately use BakedBrie-native OAuth or another secure per-user authorization architecture rather than sharing one upstream bearer token.

## Safe test prompt

```text
Use BakedBrie.

Call `whoami`.

Tell me only:
1. whether the BakedBrie connection succeeded,
2. the workspace name,
3. my role.

Do not create or modify anything.
```

## Disclaimer

BakedBrie, ChatGPT, Codex, Cloudflare, and other referenced products belong to their respective owners. This repository is an independent integration.
