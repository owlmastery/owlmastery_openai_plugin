# Owl Mastery for Codex

A Codex plugin and marketplace that connect Codex to the Owl Mastery MCP server. The server gives Codex read-only access to Owl Mastery's daily gamma levels, implied volatility and market data for US stocks and indexes. The plugin adds two skills that teach Codex how to read that data.

**Status.** The server answers at `https://owlmastery.com/mcp` once Owl Mastery routes it. Until then, test against a local API (see [Test against localhost](#test-against-localhost)).

## Install

Add the marketplace, then install the plugin:

```bash
codex plugin marketplace add owlmastery/owlmastery_openai_plugin
codex plugin add owlmastery --marketplace owlmastery
```

To install from a local checkout, run `codex plugin marketplace add .` from the repository root. Restart Codex after you install.

## Create and set an API key

The server accepts a bearer API key from a registered Owl Mastery user. A key starts with `owl_`. You create it in your Owl Mastery profile while signed in, and it works for 14 days. After that, calls answer 401 and you create a new key.

Set the key in the `OWLMASTERY_API_KEY` environment variable of the shell that starts Codex:

```bash
export OWLMASTERY_API_KEY=owl_...
```

Never write the key into a file in this repository or into a Codex config file. The plugin reads the variable by name. After you change the variable, restart Codex from that shell.

## Check the connection

Run `codex mcp list`. The `owlmastery` row shows `OWLMASTERY_API_KEY` under Bearer Token Env Var and `Bearer token` under Auth. The list does not call the server, so it cannot tell you whether the key works. To test the key, ask Codex a question such as "Use Owl Mastery to show NVDA's gamma levels." A 401 answer means the key is missing, wrong or expired.

## What the plugin adds

- **MCP server `owlmastery`.** Streamable HTTP, with `bearer_token_env_var` set to `OWLMASTERY_API_KEY`. Codex reads the variable when it connects.
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
| 401, or "unauthorized" | The key is missing, wrong or expired. | Create a new key in your Owl Mastery profile, set `OWLMASTERY_API_KEY`, and restart Codex. |
| 429, or "Too many requests" | The account went over its rate limit. | Wait a second and retry. |
| A tool answers with an error message | The call timed out or the server failed. | Retry once. If it fails again, try later. |
| `codex mcp list` has no `owlmastery` row | The plugin is not installed or is disabled. | Run `codex plugin list`, then install or enable it. |

## Test against localhost

The bundled server URL is fixed. To test against a local API on `http://localhost:8080`, add a second server and turn the bundled one off:

```bash
codex mcp add owlmastery-local --url http://localhost:8080/mcp --bearer-token-env-var OWLMASTERY_API_KEY
```

```toml
# ~/.codex/config.toml
[plugins."owlmastery@owlmastery".mcp_servers.owlmastery]
enabled = false
```

Remove both when you are done. If you skip the plugin and want only the server, run the `codex mcp add` command against `https://owlmastery.com/mcp` and copy the skills from this repository by hand.

## What works where

| Client | Works now | Notes |
| --- | --- | --- |
| Codex CLI and desktop app | Yes | Bearer token from an environment variable. |
| ChatGPT web connectors | Not yet | They offer OAuth or no authentication only, and the server has no OAuth yet. |
| Claude Code | See the Claude plugin repository | |

## Roadmap

The plugin works for you today. A listing in the public plugin directory, shared by ChatGPT and Codex, needs more. These items block it, in order:

1. **OAuth on the server.** The submission portal supports no authentication, OAuth, or both. It has no connection flow for an API key. The server needs OAuth 2.1 with PKCE (`S256`), authorization server metadata, client registration by CIMD or DCR, and a check that each token's audience is the `resource` the client sends.
2. **A public HTTPS endpoint** at `https://owlmastery.com/mcp`, with the domain-verification file that the portal asks for.
3. **A reviewer account.** Review needs a demo account with sample data and no second factor.
4. **Listing assets in this repository.** A square `logo` and `composerIcon`, and a `supportURL`. Add a license too.
5. **A portable manifest.** Once OAuth exists, move the OpenAI settings to `extensions.com.openai` in a root `plugin.json`, and declare `auth.type: "oauth"` in `mcp.json`.

The server already sets `readOnlyHint`, `destructiveHint` and `openWorldHint` explicitly on every tool, which the portal requires.

The app needs no widget. Every tool returns data that the model can quote, so it is a tool-only app. A widget, such as a levels chart, can follow later.

## Keep the skills in sync

The skills in `plugins/owlmastery/skills/` also ship in the Claude Code plugin repository. After you edit a skill in either repository, copy its `SKILL.md` body and its `references/` folder to the other repository unchanged. The only difference allowed is the `allowed-tools` line in the frontmatter, which only the Claude Code copy has: the Claude Code copy keeps it and the Codex copy leaves it out. The skill text has no client-specific wording, so the body needs no edits. Run `diff -r` on the two `skills/` folders. It must show only the `allowed-tools` lines.

## Layout

```text
.agents/plugins/marketplace.json    the marketplace, named owlmastery
plugins/owlmastery/
  .codex-plugin/plugin.json         the plugin manifest
  .mcp.json                         the MCP server and its bearer variable
  skills/levels/                    SKILL.md and references/glossary.md
  skills/market/                    SKILL.md
```

This plugin uses the Codex compatibility layout (`.codex-plugin/plugin.json` and `.mcp.json`). In Codex CLI 0.170, the portable `mcp.json` has no field for a bearer environment variable, and a server declared there shows Auth as unsupported.

The data is educational information, not investment advice.
