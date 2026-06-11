# Letterfest Cursor Plugins

Team marketplace for Letterfest MCP integrations: **Metabase**, **Unleash**, and **Grafana Cloud**.

Config changes ship through this Git repo — update `mcp.json` here, push, and Cursor syncs to the team (enable **Auto Refresh** in the dashboard).

## Plugins

| Plugin | MCP transport | Pre-configured | Per-developer setup |
|--------|---------------|----------------|---------------------|
| **Metabase** | HTTP → `metabase.letterfest.com/api/mcp` | Instance URL | OAuth login (browser) |
| **Unleash** | stdio (`npx @unleash/mcp`) | Base URL, project | PAT in `~/.unleash/mcp.env` |
| **Grafana Cloud** | HTTP → `mcp.grafana.com/mcp` | Hosted MCP URL | OAuth to Grafana Cloud |

## Repository structure

```
.cursor-plugin/marketplace.json   # team marketplace manifest
plugins/
  metabase/                       # Metabase MCP + setup skill
  unleash/                        # Unleash MCP + PAT setup skill
  grafana-cloud/                  # Grafana Cloud MCP + tools skill
```

## Publish to your team

1. Push this repo to GitHub (Letterfest org).
2. In **Cursor Dashboard → Settings → Plugins → Team Marketplaces**, import the repo URL.
3. Mark plugins **Default On** or **Required** for your distribution group.
4. Enable **Auto Refresh** so pushes update developer machines.

## Developer onboarding

### Metabase

1. Plugin installs from team marketplace.
2. First use: approve OAuth login to metabase.letterfest.com when prompted.

### Unleash

1. Plugin installs from team marketplace.
2. One-time: create a PAT in Unleash and save to `~/.unleash/mcp.env`:

```bash
mkdir -p ~/.unleash
printf 'UNLEASH_PAT=<your-token>\n' > ~/.unleash/mcp.env
chmod 600 ~/.unleash/mcp.env
```

3. Restart the Unleash MCP server in Cursor.

### Grafana Cloud

1. Plugin installs from team marketplace.
2. First use: authorize via browser when prompted (Grafana Cloud OAuth).
3. Requires Grafana Cloud account with MCP access (`grafana-assistant-app.cloud-mcp:access`).

## Validate before pushing

```bash
node scripts/validate-template.mjs
```

## Changing config

Edit the relevant `plugins/<name>/mcp.json`, commit, push. Developers reload Cursor or restart the MCP server.

**Never commit secrets** (Unleash PATs, Grafana tokens) to this repo.

## Further reading

- [Cursor Plugins docs](https://cursor.com/docs/plugins)
- [Metabase Cursor plugin](https://github.com/metabase/metabase-cursor-plugin)
- [Unleash MCP](https://github.com/Unleash/unleash-mcp)
- [Grafana Cloud MCP](https://github.com/grafana/ai-marketplace/tree/main/plugins/grafana-cloud-mcp)
