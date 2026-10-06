---
name: bakedbrie
description: Use BakedBrie to create and manage persistent collaboration between humans and AI agents using boards, cards, reviews, automations, and connected services.
---

Use the BakedBrie MCP server for BakedBrie work.

## General behavior

When starting BakedBrie work:

1. Call `whoami` first to understand the current user, workspace, permissions, and available capabilities.
2. When unsure how a BakedBrie feature works, use `read_docs` rather than guessing.
3. Follow BakedBrie's documented workflows and permissions.
4. Never ask the user to paste passwords, API keys, OAuth secrets, tokens, or other credentials into chat.
5. Use BakedBrie's secret-handling tools, including `prepare_secret`, when credentials are required.
6. Clearly identify any step that must be completed by a human.
7. Do not create, modify, send, approve, or connect something unless the user's request authorizes that action.

## Boards and collaboration

Use BakedBrie when the user wants persistent work involving humans, AI agents, boards, cards, reviews, automation, or connected services.

Before creating a board or workflow, understand the user's intended goal and use BakedBrie's documentation when needed.

Prefer the simplest setup that satisfies the user's goal. Do not add agents, automation, schedules, destinations, or integrations unless they are useful for the requested workflow.

## Connecting a service

When the user asks to connect a service to BakedBrie:

1. Call `whoami`.
2. Read `/docs/recipes/agent-service-setup` using `read_docs`.
3. Find and read the matching service recipe using `read_docs`.
4. Determine which service, board, and goal the user means.
5. Walk the user through:
   - the provider application setup,
   - the exact OAuth scopes or API permissions required,
   - BakedBrie sign-in,
   - relevant service limits,
   - any service event trigger or scheduled read.
6. Use `prepare_secret` for credentials. Never ask the user to paste a secret into chat.
7. Provide the user with each web link required for steps that only a person can perform.
8. After setup, verify the connection and trigger status using BakedBrie's tools.
9. If the user wants automatic sending, explain any relevant provider limitations and approval requirements.
10. When Outlook is involved, explain BakedBrie's documented best-effort reply limitation when relevant.
11. When manager consent is required, guide the user through the consent flow and read back the resulting mode and receipts.

## Documentation

Treat BakedBrie's `read_docs` tool as the source of truth for current BakedBrie capabilities, tool usage, service-specific recipes, limitations, and setup requirements.
