# WordPress + Elementor MCP Container

A Dockerized WordPress + Elementor environment pre-wired with **Model Context Protocol (MCP)** servers, so you can point your favorite AI coding harness (e.g. [opencode](https://opencode.ai)) at it and have the agent build a complete, fully functional WordPress site for you.

## What it does

On `docker compose up`, the stack:

1. Starts **MariaDB** and **WordPress**.
2. Runs a one-shot **wp-setup** container that installs WordPress, configures permalinks, and installs/activates:
   - **Elementor** (+ Essential Addons, ACF, Rank Math SEO, WP-Optimize)
   - **emcp-tools** — the Elementor MCP plugin (tools appear as `emcp-tools_*`)
   - **mcp-adapter** — the official WordPress MCP plugin (posts, media, options, users, etc.)
3. Exposes the MCP servers through `wp-cli` so any MCP-capable AI client can drive the site.

## Workflow: design → site

1. Place your design files in **`import/`** — e.g. HTML exports from an AI design tool (Claude Design, Stitch, etc.).
2. Start the stack and connect your AI harness.
3. Ask the agent to read the designs in `import/` and build the site with the MCP tools (pages, containers, atomic Elementor widgets, media, navigation, SEO, …).

Everything is built through the Elementor MCP tools — no hand-written HTML/CSS hacks — producing a site a human webmaster can maintain normally in WordPress.

## Quick start

```bash
cp .env.example .env    # fill in passwords
docker compose up -d
```

- WordPress: http://localhost:8080
- Admin login: user from `wordpress.env` (`WP_ADMIN_USER`), password from `.env` (`WP_ADMIN_PASSWORD`)

## Connecting your AI harness

MCP server definitions are provided in `opencode.jsonc` for opencode:

- **emcp-tools** — Elementor MCP (build pages, containers, widgets)
- **wordpress** — WordPress MCP via mcp-adapter (posts, media, options)
- **playwright** — Playwright MCP for headless-browser QA (navigate, click, screenshot; shots saved to `qa/shots/`)

The servers are served through the stack's wp-cli, e.g.:

```bash
docker compose run --rm -T wpcli wp mcp-adapter serve --server=emcp-tools-server --user=siteXadmin
```

Any MCP-capable client (opencode, Claude Desktop, Cursor, …) can wrap that command as a local MCP server. Restart your client after adding the config.

## Configuration

| File | Purpose |
|---|---|
| `.env` | Secrets only (DB + WP admin + MCP app passwords). Copy from `.env.example`. |
| `wordpress.env` | Site settings: URL, title, locale, plugins, theme, comments. Safe to commit. |
| `docker-compose.yml` | The stack itself. |
| `opencode.jsonc` | MCP server config for opencode. |

## Useful commands

```bash
# Manual WP-CLI
docker compose run --rm wpcli wp plugin list

# Headless QA (screenshots + design audits)
docker compose --profile tools run --rm qa-browser sh -c "npm ci --no-progress && node capture.js && node audit.js"
```

## Project layout

- `import/` — your design files (HTML exports, prototypes)
- `qa/` — Playwright QA harness, screenshots (`qa/shots/`) and reports
- `wp-content/` — themes & plugins, bind-mounted and editable on your host
- `ELEMENTOR-PLAYBOOK.md` — tested build sequence and tool-bug workarounds (read this before any Elementor MCP work)
- `AGENTS.md` — instructions for AI agents working in this repo

## Stop / reset

```bash
docker compose down            # stop (data kept in volumes)
docker compose down -v         # stop and wipe DB + WP core
```
