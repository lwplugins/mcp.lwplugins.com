# Changelog

## 2.0 — Auto-discovery (2026-05-16)

**The big shift: it just works with any plugin you install.**

When you install a WordPress plugin that exposes abilities through the WordPress Abilities API (like `lw-site-manager-extra`), its tools show up in your AI client automatically — you no longer have to wait for us to ship an update. Reconnect your MCP session and the new tools are there.

### What you can do now that you couldn't before

- **Fluent Forms** — when [LW Site Manager Extra](https://github.com/lwplugins/lw-site-manager-extra) is active:
  - Create, update, duplicate, set status, JSON import/export forms
  - List / view / status-change / delete submissions; add internal notes; bulk-approve / decline / delete; CSV + JSON export with date filters
  - Read and write form-level settings and email notifications
  - Configure reCAPTCHA / hCaptcha / Turnstile / CleanTalk / MailChimp / autosave / email summary
  - Enable or disable any Fluent Forms module / addon
  - Manage Pro integration feeds (Mailchimp, Slack, Hubspot, ActiveCampaign, Zapier, …) per form
  - Read Pro payment transactions and subscriptions
- **Core WordPress 6.9** — `get-site-info` and `get-environment-info` are now visible too.
- Any **third-party plugin** that registers abilities via the Abilities API works the same way without us having to add anything.

### What changed for existing setups

- **Tool names changed.** Tools used to be grouped — for example `wp_posts` with `action: "list"`. They're now individual, one per ability: the same call is `site_manager_list_posts`. Most users won't notice: Claude / Cursor / ChatGPT re-discover the tool list at session start. Only hard-coded automation needs an update.
- **5-minute discovery cache.** After installing a new plugin on your WordPress site, its tools become visible to the AI on the next reconnect, or after the cache expires. You don't have to do anything from your end.

### Reliability fixes (2.0.1 → 2.0.3)

Three small follow-ups for delete and bulk operations that came up during live testing:

- **Deleting** a form, notification, submission, or feed now works reliably. (Earlier some delete calls were rejected by WordPress because the server used the wrong HTTP method.)
- **Bulk submission actions** ("mark these 12 entries as approved", "delete these entries") now work reliably.
- These fixes self-heal even when your WordPress site sits behind a CDN that strips error headers — the server figures out the right method from the error body.

---

## 1.2 — `/healthz` for self-hosters (2026-02-26)

If you run the server yourself with Docker, there's now a `/healthz` endpoint for container health checks, and the Express app explicitly allow-lists the hostnames it will accept — safer multi-tenant deployment.

---

## 1.1 — Header authentication (2026-02-26)

Credentials moved from the URL into HTTP headers — `X-WP-URL`, `X-WP-USER`, `X-WP-API-KEY`. Query parameters still work for clients that can't set headers, but headers are now the recommended path: cleaner config files, less chance of leaking the key in browser history, referrers, or proxy logs.

---

## 1.0 — Initial release (2026-02-26)

The MCP server went live at **`https://mcp.lwplugins.com`**. From day one you could ask an AI client (Claude, Cursor, ChatGPT, anything MCP-compatible) to:

- **Posts, pages, comments, media, users, taxonomies, custom fields, settings** — full CRUD, restore from trash, duplicate, bulk actions, set featured images, manage permalinks, reading, discussion.
- **WordPress maintenance** — check and install plugin / theme / core updates, run site health checks, read the error log, flush caches, optimize and clean up the database, create and restore backups, manage plugins (install, activate, deactivate, delete) and themes.
- **WooCommerce** — list and manage products (with stock, categories, variations, bulk actions), handle orders (status changes, notes, refunds), and pull sales / top-sellers / revenue / low-stock / orders-total / products-total reports.
- **Multiple WordPress sites** through a single endpoint — one MCP server entry per site, each with its own credentials.
