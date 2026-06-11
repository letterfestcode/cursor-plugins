---
name: setup-metabase-mcp
description: Read these instructions before using Metabase MCP tools. Letterfest Metabase is pre-configured; OAuth login is required on first use.
---

Read these steps before querying Metabase data via MCP.

**Location**: MCP config is `../../mcp.json` relative to this SKILL.md (the `plugins/metabase/` folder in the Letterfest team marketplace repo).

## Letterfest setup

This plugin is pre-configured for `https://metabase.letterfest.com/api/mcp`. **Do not ask the user for a Metabase URL.**

## Required actions

1. Read `../../mcp.json` and confirm the URL is set (not a placeholder).
2. If the MCP server shows **Needs Auth**, call `mcp_auth` for the Metabase MCP server so the user can log in via OAuth.
3. Proceed with Metabase MCP tools once connected.

## Prerequisites

- Metabase v60 or higher
- User has a valid login to metabase.letterfest.com
- Metabase Admin has enabled MCP and allowed Cursor (Admin → AI → MCP)

Do not attempt MCP tool calls until authentication is complete.
