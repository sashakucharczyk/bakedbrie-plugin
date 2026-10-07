# Changelog

All notable user-facing changes to this project will be documented here.

The project is still early. Versioning follows semantic-versioning conventions where practical.

## [Unreleased]

- Public repository readiness and contribution tooling.

## [0.1.0] - 2026-10-07

### Added

- BakedBrie skill for safe human/agent collaboration workflows.
- Direct BakedBrie MCP configuration for Codex using `BAKEDBRIE_TOKEN`.
- Repo marketplace metadata.
- Installation and ChatGPT remote-bridge documentation.

### Security

- Local secret files and account-specific ChatGPT mappings are excluded from Git.
- The canonical skill instructs agents not to request credentials in chat.
