# LW Site Manager — Remote MCP Server

Manage any WordPress site with AI through a hosted MCP server at **`https://mcp.lwplugins.com`**.

No self-hosting required — connect any MCP-compatible client directly to our endpoint with your WordPress credentials.

> **Auto-discovery (v2.0):** every ability registered in your WordPress instance via the Abilities API automatically becomes an MCP tool. Install a new plugin that ships abilities (e.g. `lw-site-manager-extra`) — its tools appear on the next reconnect without any server-side change. See [CHANGELOG.md](CHANGELOG.md) for version history.

## Requirements

- **WordPress 6.9+** with the [Abilities API](https://developer.wordpress.org/reference/)
- **[LW Site Manager](https://github.com/lwplugins/lw-site-manager)** plugin installed and activated
- **Application Password** created (Users → Edit User → Application Passwords)
- *(Optional)* **[LW Site Manager Extra](https://github.com/lwplugins/lw-site-manager-extra)** for premium-plugin abilities (FluentForm, more coming)

## Quick Start

### 1. Install the WordPress plugin

Download and activate [LW Site Manager](https://github.com/lwplugins/lw-site-manager) on your WordPress site.

### 2. Create an Application Password

Go to **Users → Your Profile → Application Passwords**, enter a name (e.g. "MCP") and click **Add New**. Copy the generated password.

### 3. Connect your MCP client

Choose the method that matches your client:

#### Claude Code (CLI)

```bash
claude mcp add wordpress \
  --transport http \
  --url https://mcp.lwplugins.com/mcp \
  --header "X-WP-URL: https://yoursite.com" \
  --header "X-WP-USER: your-username" \
  --header "X-WP-API-KEY: xxxx xxxx xxxx xxxx xxxx xxxx"
```

#### Claude Desktop / Claude.ai

Add to your MCP config (`claude_desktop_config.json` or Claude.ai settings):

```json
{
  "mcpServers": {
    "wordpress": {
      "type": "url",
      "url": "https://mcp.lwplugins.com/mcp",
      "headers": {
        "X-WP-URL": "https://yoursite.com",
        "X-WP-USER": "your-username",
        "X-WP-API-KEY": "xxxx xxxx xxxx xxxx xxxx xxxx"
      }
    }
  }
}
```

#### Cursor

Add to `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "wordpress": {
      "url": "https://mcp.lwplugins.com/mcp",
      "headers": {
        "X-WP-URL": "https://yoursite.com",
        "X-WP-USER": "your-username",
        "X-WP-API-KEY": "xxxx xxxx xxxx xxxx xxxx xxxx"
      }
    }
  }
}
```

#### ChatGPT / OpenAI Codex

In the ChatGPT interface: **Settings → MCP Servers → Add Server** and paste this URL:

```
https://mcp.lwplugins.com/mcp?wp_url=https://yoursite.com&wp_user=your-username&wp_api_key=xxxx+xxxx+xxxx+xxxx+xxxx+xxxx
```

#### Other MCP clients — query params

If your client doesn't support custom headers, use the query params URL:

```
https://mcp.lwplugins.com/mcp?wp_url=https://yoursite.com&wp_user=your-username&wp_api_key=xxxx+xxxx+xxxx+xxxx+xxxx+xxxx
```

### 4. Start using it

```
"List active plugins"
"Create a new post titled Hello World"
"Check for available updates"
"Show me the latest WooCommerce orders"
"List Fluent Forms submissions from the last week"
```

## Authentication

Credentials are passed via HTTP headers (preferred) or query parameters — they never appear in tool schemas, keeping your LLM context clean.

| Method | Name | Description |
|--------|------|-------------|
| Header | `X-WP-URL` | Your WordPress site URL |
| Header | `X-WP-USER` | WordPress username |
| Header | `X-WP-API-KEY` | Application Password |
| Query | `wp_url` | Your WordPress site URL |
| Query | `wp_user` | WordPress username |
| Query | `wp_api_key` | Application Password |

Headers take priority over query params. The `tools/list` endpoint works without credentials (returns an empty list until credentials are provided).

## Multiple Sites

Add separate MCP server entries for each WordPress site:

```json
{
  "mcpServers": {
    "site-blog": {
      "type": "url",
      "url": "https://mcp.lwplugins.com/mcp",
      "headers": {
        "X-WP-URL": "https://blog.example.com",
        "X-WP-USER": "admin",
        "X-WP-API-KEY": "xxxx xxxx xxxx xxxx xxxx xxxx"
      }
    },
    "site-shop": {
      "type": "url",
      "url": "https://mcp.lwplugins.com/mcp",
      "headers": {
        "X-WP-URL": "https://shop.example.com",
        "X-WP-USER": "admin",
        "X-WP-API-KEY": "yyyy yyyy yyyy yyyy yyyy yyyy"
      }
    }
  }
}
```

## Available Tools (auto-discovery)

The tool list is **dynamic** — it is generated on each session from the abilities your WordPress site exposes via `/wp-abilities/v1/abilities`. There's no fixed catalogue: install a plugin → its tools appear; uninstall it → they disappear.

### How tool names are derived

The ability name's `/` and `-` characters are replaced with `_`, lowercased:

| Ability name | MCP tool name |
|---|---|
| `site-manager/list-posts` | `site_manager_list_posts` |
| `site-manager/ff-update-form` | `site_manager_ff_update_form` |
| `core/get-site-info` | `core_get_site_info` |
| `hellopack/check-updates` | `hellopack_check_updates` |

### Method routing (transparent to you)

Each ability is invoked with the HTTP method the WP Abilities REST controller expects, based on its annotations:

| Annotation | HTTP method |
|---|---|
| `readonly: true` | `GET` (input via query string) |
| `destructive: true` AND `idempotent: true` | `DELETE` (input via query string) |
| otherwise | `POST` (input via JSON body) |

On a 405 response, the client retries with the next method advertised by the `Allow` header (and falls back to parsing the error body if the header is stripped by a CDN).

### What you get out of the box

Once **LW Site Manager** is active, you immediately have tools for: WordPress core updates, plugins, themes, backups, site health, posts, pages, comments, media, users, taxonomies, post/user/term meta, settings, and (when WooCommerce is active) products, orders, and reports.

Add **LW Site Manager Extra** to extend the surface with premium-plugin abilities — currently FluentForm (forms, submissions, global settings, modules, Pro integration feeds, payments). More integrations are added there over time.

You can list everything available at runtime by asking the model to call `tools/list`, or via:

```bash
curl -X POST https://mcp.lwplugins.com/mcp \
  -H "Content-Type: application/json" \
  -H "X-WP-URL: https://yoursite.com" \
  -H "X-WP-USER: your-username" \
  -H "X-WP-API-KEY: xxxx xxxx xxxx xxxx xxxx xxxx" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Discovery cache

Discovered ability lists are cached in memory for 5 minutes per `wpUrl + wpUser` key. After installing a new plugin, the new tools become visible on the next reconnect (or after the cache expires).

## Security

- Credentials are transmitted over HTTPS and never stored on the server
- Each request is independently authenticated against your WordPress site
- Application Passwords can be revoked at any time from WordPress admin
- The server acts as a transparent proxy — no data is cached or logged

## Disclaimer

This service is provided "as is", without warranty of any kind. Use it at your own risk. The authors are not responsible for any data loss, security incidents, or damages resulting from the use of this service. Always create backups before performing destructive operations. By using this service, you acknowledge that you are solely responsible for any actions performed on your WordPress sites through this MCP server.

## License

MIT
