# Owl Mastery for Codex

A Codex plugin and marketplace that connect Codex to the Owl Mastery MCP server. The server gives Codex read-only access to Owl Mastery's daily gamma levels, implied volatility and market data for US stocks and indexes. The plugin adds two skills that teach Codex how to read that data.

**Status.** The server answers at `https://owlmastery.com/mcp` and signs you in with your Owl Mastery account through OAuth. To test against a local API, see [Test against localhost](#test-against-localhost).

## Install

Add the marketplace, then install the plugin:

```bash
codex plugin marketplace add owlmastery/owlmastery_openai_plugin
codex plugin add owlmastery --marketplace owlmastery
```

To install from a local checkout, run `codex plugin marketplace add .` from the repository root. Restart Codex after you install.

## Sign in

You need an Owl Mastery account. Installing the plugin does not sign you in: `codex plugin add` shows `Not logged in` under Auth in `codex mcp list`. Sign in once:

```bash
codex mcp login owlmastery
```

Codex opens your browser at the Owl Mastery sign-in page. Sign in, then select **Allow** on the consent page. On a machine with no browser, add `--no-browser`, open the printed URL elsewhere, and paste the callback URL back. Codex stores the tokens and refreshes them itself. Access tokens last 1 hour and refresh tokens 14 days, renewed on each use. A connection unused for 14 days asks you to sign in again.

You see and disconnect your connections in Owl Mastery under **Profile**, **Connected apps**.
## Check the connection

Run `codex mcp list`. The `owlmastery` row shows `enabled` under Status, and `Not logged in` under Auth until you sign in. The list does not call the server. To test the connection, ask Codex a question such as "Use Owl Mastery to show NVDA's gamma levels."

## What the plugin adds

- **MCP server `owlmastery`.** Streamable HTTP at `https://owlmastery.com/mcp`, with OAuth sign-in. The plugin holds no credential.
- **Skill `levels`.** Explains a ticker's call resistance, put support, HVL, gamma wall, net gamma, 1-day range and implied volatility against its close.
- **Skill `market`.** Summarizes breadth, indexes, movers, sectors and your watchlists.

The server has seven read-only tools. The server describes their inputs itself, so the skills name the tools and leave the schemas out.

| Tool | Answers |
| --- | --- |
| `search_tickers` | Which symbol belongs to a company name or a partial symbol. |
| `get_ticker_levels` | Quote, signals and gamma levels of 1 to 10 tickers. |
| `get_gamma_exposure` | Gamma exposure of one ticker by strike and by expiry. |
| `get_volatility` | Implied volatility, implied price ranges and the volatility rank of one ticker. |
| `get_market_overview` | Breadth, indexes, gainers and losers over a range. |
| `get_sectors` | Sector moves over a range, and the stocks of one sector. |
| `list_watchlists` | Your watchlists, with each ticker's latest close and change. |

All figures are one daily snapshot, computed after the US close.

## Fix a failed call

| Symptom | Cause | What to do |
| --- | --- | --- |
| 401, or "unauthorized" | The connection is not signed in, or the sign-in expired. | Run `codex mcp login owlmastery`. |
| 429, or "Too many requests" | The account went over its rate limit. | Wait a second and retry. |
| A tool answers with an error message | The call timed out or the server failed. | Retry once. If it fails again, try later. |
| `codex mcp list` has no `owlmastery` row | The plugin is not installed or is disabled. | Run `codex plugin list`, then install or enable it. |

## Test against localhost

The bundled server URL is fixed. To test against a local app (`make frontend` serves `/mcp` and `/oauth` on `http://localhost:5173`), add a second server and turn the bundled one off:

```bash
codex mcp add owlmastery-local --url http://localhost:5173/mcp
codex mcp login owlmastery-local
```

```toml
# ~/.codex/config.toml
[plugins."owlmastery@owlmastery".mcp_servers.owlmastery]
enabled = false
```

Remove both when you are done. If you skip the plugin and want only the server, run `codex mcp add owlmastery --url https://owlmastery.com/mcp` and `codex mcp login owlmastery`, and copy the skills from this repository by hand.

## What works where

| Client | Works now | Notes |
| --- | --- | --- |
| Codex CLI and desktop app | Yes | OAuth sign-in with `codex mcp login owlmastery`. |
| ChatGPT web connectors | Expected, not yet confirmed | They use OAuth, which the server now has. It should work once Owl Mastery allows server-to-server traffic through Cloudflare. |
| Claude Code | See the Claude plugin repository | |

## Roadmap

The plugin works for you today. A listing in the public plugin directory, shared by ChatGPT and Codex, needs more. Done so far:

- **OAuth on the server.** OAuth 2.1 with PKCE (`S256`), authorization server metadata, client registration by CIMD or DCR, and a check of each token's audience.
- **A public HTTPS endpoint** at `https://owlmastery.com/mcp`. It stays pending until Owl Mastery allows server-to-server traffic through Cloudflare, and until the domain-verification file that the portal asks for is in place.

These items still block a listing, in order:

1. **A reviewer account.** Review needs a demo account with sample data and no second factor.
2. **Listing assets in this repository.** A square `logo` and `composerIcon`, and a `supportURL`. Add a license too.
3. **A portable manifest.** Move the OpenAI settings to `extensions.com.openai` in a root `plugin.json`, and declare `auth.type: "oauth"` in `mcp.json`.

The server already sets `readOnlyHint`, `destructiveHint` and `openWorldHint` explicitly on every tool, which the portal requires.

The app needs no widget. Every tool returns data that the model can quote, so it is a tool-only app. A widget, such as a levels chart, can follow later.

## Keep the skills in sync

The skills in `plugins/owlmastery/skills/` also ship in the Claude Code plugin repository. After you edit a skill in either repository, copy its `SKILL.md` body and its `references/` folder to the other repository unchanged. The only difference allowed is the `allowed-tools` line in the frontmatter, which only the Claude Code copy has: the Claude Code copy keeps it and the Codex copy leaves it out. The skill text has no client-specific wording, so the body needs no edits. Run `diff -r` on the two `skills/` folders. It must show only the `allowed-tools` lines.

## Layout

```text
.agents/plugins/marketplace.json    the marketplace, named owlmastery
plugins/owlmastery/
  .codex-plugin/plugin.json         the plugin manifest
  .mcp.json                         the MCP server URL
  skills/levels/                    SKILL.md and references/glossary.md
  skills/market/                    SKILL.md
```

This plugin uses the Codex compatibility layout (`.codex-plugin/plugin.json` and `.mcp.json`). This layout keeps working with OAuth in Codex CLI 0.170: the server shows `Not logged in` under Auth until `codex mcp login owlmastery` succeeds. The portable `mcp.json` layout is untested with OAuth.

The data is educational information, not investment advice.
