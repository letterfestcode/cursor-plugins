---
name: setup-unleash-mcp
description: One-time Unleash MCP setup for Letterfest. Guides developers to create a personal access token and store it locally — never in git or the team marketplace.
---

Letterfest Unleash MCP is pre-configured with the shared instance URL. Each developer must provide their own **Personal Access Token** locally.

## What is already configured (team marketplace)

- Instance: `https://unleash.letterfest.com`
- Default project: `default`
- MCP command: `npx @unleash/mcp@latest`

Secrets are **not** in the team repo. They live in `${userHome}/.unleash/mcp.env` on each machine (typically `~/.unleash/mcp.env`).

## One-time developer setup

1. Log in to https://unleash.letterfest.com
2. Go to **Profile → Personal Access Tokens** and create a token with permission to manage feature flags
3. Create the local env file:

```bash
mkdir -p ~/.unleash
cat > ~/.unleash/mcp.env <<'EOF'
UNLEASH_PAT=<paste-token-here>
EOF
chmod 600 ~/.unleash/mcp.env
```

Only `UNLEASH_PAT` is required in the env file — `UNLEASH_BASE_URL` and `UNLEASH_DEFAULT_PROJECT` come from the team plugin config.

4. Restart the Unleash MCP server in **Cursor Settings → Tools & MCPs**, or reload the window.

## Troubleshooting

- `UNLEASH_PAT is required` — the env file is missing, empty, or MCP was not restarted after creating it
- Node.js 22+ is required for `@unleash/mcp`

Never commit PATs to git, the team marketplace repo, or chat.
