# BakedBrie Plugin for ChatGPT and Codex

An unofficial, community-maintained plugin for using [BakedBrie](https://bakedbrie.com/) with Codex and, with an additional authenticated remote MCP bridge, ChatGPT.

Developed by Sasha Kucharczyk.

> **Project status:** early public release (`0.1.0`). The direct BakedBrie MCP integration has been tested successfully in Codex. BakedBrie's current MCP authentication uses a bearer token, so ChatGPT requires an OAuth-capable remote bridge for account-specific access and write actions.

## What this plugin provides

- A BakedBrie skill with safe, repeatable workflow guidance.
- A direct MCP connection to `https://api.bakedbrie.com/mcp` for Codex.
- A repo marketplace entry for easy installation and testing.
- Documentation for a personal OAuth bridge when using BakedBrie from ChatGPT.

The plugin does **not** reimplement BakedBrie. Available tools and behavior are determined by BakedBrie's MCP server.

## Requirements

For the direct Codex path:

- a BakedBrie account;
- a BakedBrie API token;
- Codex with plugin support;
- `BAKEDBRIE_TOKEN` set in the local environment.

For ChatGPT:

- a stable HTTPS MCP endpoint that ChatGPT can reach; and
- OAuth-compatible authentication between ChatGPT and that endpoint.

## Install in Codex

### 1. Create a BakedBrie API token

Create an API token in BakedBrie.

Do not paste the token into an AI conversation, source file, issue, pull request, or Git commit.

### 2. Set `BAKEDBRIE_TOKEN`

Windows PowerShell, current session:

```powershell
$env:BAKEDBRIE_TOKEN = "<your-bakedbrie-token>"
```

Persist it for your Windows user:

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

Restart applications that were already running after changing the environment.

### 3. Add the marketplace

```bash
codex plugin marketplace add sashakucharczyk/bakedbrie-plugin --ref main
```

### 4. Confirm the plugin is available

```bash
codex plugin list --marketplace bakedbrie --available --json
```

### 5. Install the plugin

```bash
codex plugin add bakedbrie@bakedbrie
```

### 6. Test read-only access first

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

Only test write actions after identity, workspace, and permissions are correct.

## Using BakedBrie from ChatGPT

The checked-in MCP configuration is designed for Codex and references the local `BAKEDBRIE_TOKEN` environment variable.

ChatGPT supports authenticated remote MCP servers through OAuth. It cannot simply inherit a local machine's BakedBrie bearer token.

For personal use, the practical pattern is:

```text
ChatGPT
  ↓ OAuth
Personal remote MCP bridge
  ↓ BAKEDBRIE_TOKEN stored as a server secret
https://api.bakedbrie.com/mcp
```

Cloudflare Workers is one reasonable way to host that bridge. The bridge should:

1. expose a Streamable HTTP MCP endpoint, normally ending in `/mcp`;
2. implement an OAuth-compatible authentication flow;
3. restrict access to the intended user;
4. store `BAKEDBRIE_TOKEN` as a deployment secret;
5. forward authorized MCP requests to BakedBrie;
6. never return or log the upstream token.

For Cloudflare, store the BakedBrie token with:

```bash
npx wrangler secret put BAKEDBRIE_TOKEN
```

Then connect the deployed HTTPS `/mcp` endpoint through **ChatGPT → Plugins → Create custom MCP server**, complete OAuth, install the resulting plugin, and test `whoami` before any write operation.

Useful references:

- OpenAI custom MCP server: https://developers.openai.com/api/docs/guides/custom-mcp-server
- OpenAI plugin authentication: https://developers.openai.com/plugins/build/auth
- OpenAI plugin packaging: https://developers.openai.com/plugins/build/plugins
- Cloudflare remote MCP servers: https://developers.cloudflare.com/agents/model-context-protocol/guides/remote-mcp-server/

### Why this repo does not yet use the portable Agent Plugins MCP format

OpenAI now recommends the portable Agent Plugins layout for new packages. That format intentionally does not provide a portable secret-reference field for authenticated remote HTTP MCP servers; authorization is client-managed.

BakedBrie's current direct integration depends on `bearer_token_env_var`, which is supported by the Codex compatibility MCP format used here. Migrating this repo to portable `plugin.json` / `mcp.json` before BakedBrie offers OAuth-compatible authorization would remove the working credential path.

The compatibility package remains supported. A portable migration should happen when BakedBrie provides native MCP OAuth or when a suitable per-user authenticated bridge becomes the canonical endpoint.

## Repository structure

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

The marketplace points to `./plugins/bakedbrie`. That directory is the canonical installable package.

## Security and privacy

Never commit credentials or account-specific connection data.

The repository may contain the **name** `BAKEDBRIE_TOKEN`; it must never contain its value.

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
- account-specific ChatGPT MCP connection IDs or generated `.app.json`;
- captured MCP responses containing private workspace, board, card, or user data.

If a secret is ever committed, rotate it. Removing it from the latest file does not remove it from Git history.

For suspected security issues, see [SECURITY.md](SECURITY.md).

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

Please keep changes small, explain behavior changes clearly, and do not include personal account data or credentials in examples or fixtures.

## Versioning

The current plugin version is `0.1.0`.

Changes are tracked in [CHANGELOG.md](CHANGELOG.md). Releases should use semantic versioning once the public API and installation path stabilize.

## License

This project is licensed under the [MIT License](LICENSE).

## Disclaimer

This is an independent integration. It is not an official BakedBrie, OpenAI, or Cloudflare project. Product names and trademarks belong to their respective owners.
