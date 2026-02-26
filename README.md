# LW Site Manager — Remote MCP Server

Manage any WordPress site with AI through a hosted MCP server at **`https://mcp.lwplugins.com`**.

No self-hosting required — connect any MCP-compatible client directly to our endpoint with your WordPress credentials.

## Requirements

- **WordPress 6.9+** with the [Abilities API](https://developer.wordpress.org/reference/)
- **[LW Site Manager](https://github.com/lwplugins/lw-site-manager)** plugin installed and activated
- **Application Password** created (Users → Edit User → Application Passwords)

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

Headers take priority over query params. The `tools/list` endpoint works without credentials.

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

## Available Tools (16 tools, 117 actions)

### WordPress Core

| Tool | Actions | Description |
|------|---------|-------------|
| `wp_updates` | check, update_plugin, update_theme, update_core, update_all, check_db_updates, update_db, update_all_dbs, get_db_plugins | Updates management |
| `wp_plugins` | list, activate, deactivate, install, delete | Plugin management |
| `wp_themes` | list, activate, install, delete | Theme management |
| `wp_backup` | create, status, cancel, list, restore, delete | Backup management |
| `wp_maintenance` | health_check, error_log, optimize_db, cleanup_db, flush_cache | Site maintenance |

### Content

| Tool | Actions | Description |
|------|---------|-------------|
| `wp_posts` | list, get, get_types, create, update, delete, restore, duplicate, bulk, set_terms, get_terms | Posts |
| `wp_pages` | list, get, create, update, delete, restore, duplicate, hierarchy, reorder, set_homepage, set_posts_page, front_page_settings, templates, set_template | Pages |
| `wp_comments` | list, get, counts, create, update, delete, approve, spam, bulk | Comments |
| `wp_media` | list, get, upload, update, delete | Media library |

### Data

| Tool | Actions | Description |
|------|---------|-------------|
| `wp_users` | list, get, create, update, delete, reset_password, get_roles | Users |
| `wp_taxonomy` | list_categories, get_category, create_category, update_category, delete_category, list_tags, get_tag, create_tag, update_tag, delete_tag | Taxonomies |
| `wp_meta` | get_post, set_post, delete_post, get_user, set_user, delete_user, get_term, set_term, delete_term | Meta fields |
| `wp_settings` | get_general, update_general, get_reading, update_reading, get_discussion, update_discussion, get_permalink, update_permalink | Settings |

### WooCommerce

| Tool | Actions | Description |
|------|---------|-------------|
| `wc_products` | list, get, create, update, delete, duplicate, update_stock, list_categories, list_variations, bulk | Products |
| `wc_orders` | list, get, update_status, list_statuses, create_refund, list_notes, add_note, bulk | Orders |
| `wc_reports` | sales, top_sellers, orders_totals, revenue_stats, low_stock, products_totals | Reports |

## Security

- Credentials are transmitted over HTTPS and never stored on the server
- Each request is independently authenticated against your WordPress site
- Application Passwords can be revoked at any time from WordPress admin
- The server acts as a transparent proxy — no data is cached or logged

## Self-Hosting

If you prefer to run your own instance, see the [self-hosted documentation](https://github.com/lwplugins/mcp.lwplugins.com).

## Disclaimer

This service is provided "as is", without warranty of any kind. Use it at your own risk. The authors are not responsible for any data loss, security incidents, or damages resulting from the use of this service. Always create backups before performing destructive operations. By using this service, you acknowledge that you are solely responsible for any actions performed on your WordPress sites through this MCP server.

## License

MIT
