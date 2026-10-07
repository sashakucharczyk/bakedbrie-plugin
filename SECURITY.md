# Security Policy

## Supported version

The current `0.1.x` line is the only supported version while this project is in early development.

## Reporting a vulnerability

Please do **not** open a public issue containing:

- API tokens or OAuth secrets;
- exploitable authentication details;
- private BakedBrie workspace data;
- a working proof of concept that could expose or modify another user's data.

When GitHub private vulnerability reporting is available for this repository, use **Security → Report a vulnerability**.

If private vulnerability reporting is not available, contact the maintainer through the GitHub profile and request a private reporting channel. Do not include sensitive details in the initial public message.

## Credential handling

This repository must never contain real BakedBrie bearer tokens, OAuth client secrets, private keys, personal ChatGPT MCP connection IDs, or captured private MCP responses.

If a credential is accidentally committed, assume it is compromised:

1. rotate or revoke it immediately;
2. remove it from the repository;
3. remove it from Git history when appropriate;
4. review logs and downstream systems for misuse.
