# WordPress + Elementor MCP Container

A Dockerized WordPress + Elementor environment pre-wired with **Model Context Protocol (MCP)** servers. Point your favorite AI coding harness (for example [opencode](https://opencode.ai)) at it, and the agent can build a complete, fully functional WordPress site for you — straight from your design files.

The stack uses **free Elementor**. No Elementor Pro or other paid plugins are required.

## Why WordPress, when AI can generate a whole site?

Many companies have their website content creation process built around WordPress.

The biggest bottlenecks in creating WordPress websites have been design work and manual page building. By combining AI design tools with the WordPress + Elementor MCPs, every business owner can now create the website of their dreams.

WordPress itself will eventually become obsolete as more lightweight CMS systems powered by AI tools take over. Until then, this project hopefully helps you design and launch your dream website in no time.

## What it does

On `docker compose up`, the stack:

1. Starts **MariaDB** and **WordPress**.
2. Runs a one-shot **wp-setup** container that installs WordPress, configures permalinks, and installs and activates:
   - **Elementor** (plus Essential Addons, ACF, Rank Math SEO and WP-Optimize)
   - **emcp-tools** — the Elementor MCP plugin (tools appear as `emcp-tools_*`)
   - **mcp-adapter** — the official WordPress MCP plugin (posts, media, options, users, and more)
3. Exposes the MCP servers through `wp-cli`, so any MCP-capable AI client can drive the site.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with the Compose plugin
- An MCP-capable AI client (opencode, Claude Desktop, Cursor, …)

## Quick start

```bash
cp .env.example .env   # then edit the passwords in .env
docker compose up -d
```

- WordPress: http://localhost:8080
- Admin login: `WP_ADMIN_USER` from `wordpress.env`, `WP_ADMIN_PASSWORD` from `.env`

You do not need to touch the WordPress admin console yourself while the site is being built — but feel free to browse around and watch the progress. A build can easily take several hours, depending on your design.

Once the site is built, you can take over and tweak it manually in WordPress and Elementor, or keep editing it through the provided MCPs with AI.

## Workflow: design → site

1. Place your design files in **`import/`** — for example HTML exports from an AI design tool (Claude Design, Stitch, and others). Ask the design tool to document the design so another AI can build it, and include the images and other assets.
2. Start the stack and connect your AI harness (see below).
3. Ask the agent to read the designs in `import/` and build the site with the MCP tools: pages, containers, atomic Elementor widgets, media, navigation, SEO, and more.

The site is built through the Elementor MCP tools — no hand-written HTML/CSS hacks — producing a site that a human webmaster can maintain normally in WordPress.

## Connecting your AI harness

MCP server definitions for opencode are provided in `opencode.jsonc`:

- **emcp-tools** — Elementor MCP (build pages, containers, widgets)
- **wordpress** — WordPress MCP via mcp-adapter (posts, media, options)
- **playwright** — Playwright MCP for headless-browser QA (navigate, click, screenshot; screenshots are saved to `qa/shots/`)

The servers run through the stack's wp-cli:

```bash
docker compose run --rm -T wpcli wp mcp-adapter serve --server=emcp-tools-server --user=siteXadmin
```

Any MCP-capable client can wrap that command as a local MCP server. Restart your client after adding the configuration.

## Configuration

| File | Purpose |
|---|---|
| `.env` | Secrets only (DB, WP admin and MCP app passwords). Copy from `.env.example`. |
| `wordpress.env` | Site settings: URL, title, locale, plugins, theme, comments. Safe to commit. |
| `docker-compose.yml` | The stack itself. |
| `opencode.jsonc` | MCP server configuration for opencode. |

## Useful commands

```bash
# Manual WP-CLI
docker compose run --rm wpcli wp plugin list

# Headless QA (screenshots + design audits)
# Note: the harness scripts (capture.js, audit.js) are generated per site during
# the build (see ELEMENTOR-PLAYBOOK.md §5); qa/ is gitignored.
docker compose --profile tools run --rm qa-browser sh -c "npm ci --no-progress && node capture.js && node audit.js"
```

## Project layout

- `import/` — your design files (HTML exports, prototypes, assets)
- `qa/` — Playwright QA harness, screenshots (`qa/shots/`) and reports (gitignored; the harness scripts are generated during the build)
- `wp-content/` — themes, plugins and mu-plugins; bind-mounted and editable on your host
- `ELEMENTOR-PLAYBOOK.md` — the tested build sequence and tool-bug workarounds (read this before any Elementor MCP work)
- `AGENTS.md` — instructions for AI agents working in this repo

## Monitoring progress

While the site is being built, you can watch the work in the WordPress admin:

- **EMCP Tools dashboard → History** — see what the MCPs are doing:
  http://localhost:8080/wp-admin/admin.php?page=emcp-tools-history
- **EMCP Themer → All Templates** — the header and footer templates:
  http://localhost:8080/wp-admin/edit.php?post_type=emcp_theme_template
- **Pages** — pages appear here as the agent creates them:
  http://localhost:8080/wp-admin/edit.php?post_type=page

## Example build

Once the stack is up, import your designs and plan the website build with your AI harness.

In this example, I used Claude Design to create a website with one long main page, a blog index page and a blog article page. I exported everything from Claude Design and asked it to document the design properly so another AI could rebuild it in WordPress and Elementor — that documentation makes it much easier for the AI to follow the design. Remember to export the images and any other assets as well.

I then asked the agent: *"I have added my designs to the import folder, plan how you can build them on my WP+Elementor setup."* It inspected the import folder, asked a few questions about the site, and presented a build plan. After I answered the questions and approved the plan, it built the site. (At the time of writing, I use opencode with GLM-5, "default" variant.)

For this example design, the build used about 280K tokens, cost roughly 10 USD, and took about 30 minutes.

## Stop / reset

```bash
docker compose down      # stop (data is kept in volumes)
docker compose down -v  # stop and wipe the DB + WordPress core
```

## License

MIT — see [LICENSE](LICENSE).
