---
name: levels
description: Reads Owl Mastery's gamma levels for US stocks and indexes, such as call resistance, put support, HVL, gamma wall, net gamma and the 1-day range, and the implied volatility behind them. Use when the user asks where a ticker sits against its levels, what its gamma regime is, what its options market implies, or names a ticker or company and wants its Owl Mastery data.
---

# Read a ticker's levels

The Owl Mastery tools return one snapshot per ticker, computed once a day after the US close. Nothing updates while the market is open. State the `asOf` date in every answer, and say so when it is not the latest session.

## Pick the tools in this order

1. **`search_tickers`** when the user gives a company name or an unsure symbol. Use the symbol it returns. If several match, ask which one the user means.
2. **`get_ticker_levels`** for the answer itself. It takes 1 to 10 symbols, so send every ticker of a request in one call.
3. **`get_gamma_exposure`** only when the user asks about strikes: which strikes hold the most exposure, how it splits across expiries, or how the total moved from the session before.
4. **`get_volatility`** only when the user asks what the options market implies: implied volatility, the 68% and 95% price ranges, or whether volatility is high or low against recent sessions.
5. **`get_market_overview`** or **`get_sectors`** when the user wants the market around the ticker. The `market` skill covers them.

Do not call a tool whose answer the user did not ask for. The first call is usually enough.

## Read the result

Put the close next to each level and give the distance in percent: `(level / close - 1) × 100`. A level above the close has a positive distance. Check these before you write:

- **`gamma` is null.** The ticker has no options data. Report its price and signals and say it has no levels.
- **`notFound` lists symbols.** They are unknown or not built yet. Run each through `search_tickers` before you say it does not exist.
- **Units.** `changePct` and performance `pct` are in percent. Gamma values are dollars of dealer delta per $1 move in the underlying. Every volatility from `get_volatility` is a yearly fraction, so 0.25 means 25%.
- **Gamma wall equals call resistance.** Both are the same strike, so never present the wall as a second confirmation.
- **`front` levels.** They repeat the main levels for the nearest expiry alone. A level held by the nearest expiry stops mattering once that date passes.

For what each level means and how Owl Mastery computes it, read `references/glossary.md`.

## Describe the regime

Compare the close with HVL. Above HVL, dealer hedging is assumed to lean against price moves, which can come with tighter ranges. Below HVL, it is assumed to go with them, which can make moves larger and faster. Neither direction says where price will go. Call resistance is a strike where rallies may slow. Put support is a strike where declines may slow, or speed up once price breaks through it.

## Write the answer

1. The ticker, its close and `asOf`.
2. The levels with their distances, the regime, and the 1-day range, in a short table or list.
3. Anything the user asked for beyond that, from the other tools.
4. One sentence of limits when the answer leans on the levels: they come from open interest and an assumption about dealer positioning, they are a snapshot, and other forces such as news and earnings can outweigh hedging.

Use the site's wording: levels "may" slow a move, hedging "tends to" damp or add to it. The numbers are educational information, not investment advice. If the user asks whether to buy or sell, say the tools show levels and not forecasts, and do not recommend a trade.

## When a call fails

- **401, or "unauthorized".** The API key is missing, wrong or expired. Keys last 14 days. Tell the user to create a new key in Owl Mastery and set it in the `OWLMASTERY_API_KEY` environment variable, then restart the client.
- **"That took too long. Try again."** Retry once.
- **"Something went wrong. Try again."** Retry once, then tell the user the server failed.
