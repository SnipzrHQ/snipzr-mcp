# Snipzr MCP Server

The official [Snipzr](https://www.snipzr.com) MCP server. One hosted endpoint gives any MCP client thirteen tools to create, manage and measure short links: branded domains, batch creation, cookie-less analytics, campaign reports, UTM building and QR codes.

```
https://mcp.snipzr.com/mcp
```

There is nothing to install or run. The server is remote (Streamable HTTP), stateless and available on every paid Snipzr plan. This repository is its public home: setup, the tool reference and the registry metadata. The server itself is part of the Snipzr platform, and its tools are thin adapters over the same handlers that serve the [public REST API](https://www.snipzr.com/docs/api), so the two surfaces always behave identically.

## Quick start

**1. Connect with sign-in (recommended).** Most clients need no token at all: point them at the server address and they open a Snipzr sign-in. The consent screen shows exactly which permissions the application gets and which of your workspaces it may act in; untick anything you do not want, and disconnect it later from Settings.

Claude Code, two commands:

```bash
claude mcp add --transport http snipzr https://mcp.snipzr.com/mcp
claude mcp login snipzr
```

Claude on the web, desktop and mobile: open Settings, then Connectors, then Add custom connector, enter the server address and select Connect. ChatGPT connects the same way in developer mode. Smithery, Glama and other directories connect through the same sign-in.

**2. Or use a token.** In [Settings](https://www.snipzr.com/settings), create a token and copy the secret (it is shown exactly once):

- **User API tokens** act as you. Pick the workspaces the token may act in and what it may do in each one.
- **Workspace API tokens** belong to one workspace and act as that workspace. The right choice for a shared team assistant or a client without a browser.

Cursor, VS Code and any other MCP client take the generic remote config:

```json
{
  "mcpServers": {
    "snipzr": {
      "url": "https://mcp.snipzr.com/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

A client that only speaks stdio can bridge with `npx mcp-remote https://mcp.snipzr.com/mcp --header "Authorization: Bearer YOUR_TOKEN"`.

**3. Confirm it worked.** Ask your assistant to list the Snipzr tools. Thirteen names starting with `snipzr_` means you are connected.

## The thirteen tools

| Tool | What it does | Scope |
| --- | --- | --- |
| `snipzr_create_link` | Create one short link: destination, optional domain, back-half, title | `links:write` |
| `snipzr_create_links_batch` | Create up to 100 short links in one call | `links:write` |
| `snipzr_list_links` | List links with text, domain and campaign filters | `links:read` |
| `snipzr_get_link` | Read one link by its back-half | `links:read` |
| `snipzr_update_link` | Change a destination or title | `links:write` |
| `snipzr_delete_link` | Delete a link, with a confirmation gate | `links:write` |
| `snipzr_get_link_stats` | Per-day clicks and scans, bots counted separately | `stats:read` |
| `snipzr_get_campaign_report` | Clicks rolled up by campaign | `stats:read` |
| `snipzr_get_usage` | Plan, remaining allowance and reset date | `usage:read` |
| `snipzr_list_domains` | Your short domains and their status | `domains:read` |
| `snipzr_get_qr_code` | A scannable QR code (PNG, JPEG or SVG) with colors, a caption and a logo, plus a signed download URL | `links:read` |
| `snipzr_build_utm_link` | Compose a UTM link that follows your workspace's conventions | `links:read` |
| `snipzr_list_workspaces` | The workspaces this connection can act in, with the ids to pass as `workspaceId` | any |

Every tool carries read-only or destructive annotations, so well-behaved clients ask before a delete and never before a read, and every tool publishes an output schema and returns structured content beside the text, so agents read fields instead of parsing prose.

## Access model

- **Scopes** narrow a token to just what it needs: `links:read`, `links:write`, `stats:read`, `usage:read`, `domains:read`. A call outside the token's scopes is refused with a message that names the missing scope.
- **Connections act in the workspaces you chose.** A sign-in connection reaches exactly the workspaces you ticked on the consent screen; a user token reaches the ones picked when it was created. Personal workspace by default. To work in another granted workspace, the assistant passes its id as the `workspaceId` argument of any tool (`snipzr_list_workspaces` lists them). Access follows your live membership there, checked on every call: leave a workspace and the token loses it instantly; get re-added and it resumes at your new role.
- **Workspace tokens** are their one workspace and refuse any other. They are not tied to a person, so people can come and go while automations keep working.
- Refusals carry stable codes (`SCOPE_REQUIRED`, `NOT_GRANTED`, `GRANT_INACTIVE`, `GRANT_REDUCED`, `PLAN_REQUIRED`) so an agent can explain and self-correct.

## Good to know

- Links an agent creates count toward the same monthly allowance as links you create by hand, and requests share the plan's API rate limit. `snipzr_get_usage` reports both before bulk work.
- QR codes come back as an inline image plus a short-lived signed download URL, so an agent can save the file without pushing image bytes through the model.
- Revoking a token or disconnecting an application in Settings cuts the client off immediately, and a disconnected application cannot quietly reconnect: it has to ask you again.
- The server never logs your destinations, and analytics stay cookie-less.

## Links

- Setup with per-client details: [snipzr.com/integrations/mcp](https://www.snipzr.com/integrations/mcp)
- Docs guide: [Connect Your AI With MCP](https://www.snipzr.com/docs/how-to/ai-integrations/connect-your-ai-with-mcp)
- REST API reference: [snipzr.com/docs/api](https://www.snipzr.com/docs/api)

## Support

Questions and problem reports are welcome in this repository's issues, or at support@snipzr.com. Please never post a token secret; if one leaks, revoke or roll it in Settings first.
