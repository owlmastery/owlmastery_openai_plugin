---
name: levels
description: "Reads Owl Mastery's gamma levels and implied volatility for one US stock or index against its close: call resistance, put support, HVL, gamma wall, net gamma, the 1-day range, the strikes with the most exposure, implied volatility and its 68% and 95% price ranges. Use it whenever the user names a ticker or company and asks where it sits against its levels, what its gamma regime is, about its call wall, put wall, gamma flip, zero gamma or dealer hedging, or what its options market implies, even if they do not say Owl Mastery. Do not use it for market-wide, sector or watchlist questions (use the market skill) or to recommend a trade."
---

# Read a ticker's levels

The Owl Mastery tools return one snapshot per ticker, computed once a day after the US close. Nothing updates while the market is open.

## Choose the tools

- **`search_tickers`** when the user gives a company name or an unsure symbol. Use the symbol it returns. If several match, ask which one the user means.
- **`get_ticker_levels`** answers most questions. It takes up to 10 symbols, so send every ticker of a request in one call.
- **`get_gamma_exposure`** only for strikes: which strikes hold the most exposure, how it splits across expiries, or how the total moved from the session before. If it answers with an error, the ticker has no per-strike data. Report what `get_ticker_levels` returned.
- **`get_volatility`** only for what the options market implies: implied volatility, the 68% and 95% price ranges, or whether volatility is high or low against recent sessions.
- **`get_market_overview`** or **`get_sectors`** for the market around the ticker. The `market` skill covers them.

Do not call a tool whose answer the user did not ask for. The first call is usually enough.

## Read the result

State the session the numbers are for. `asOf` is the session of the price. `gamma.date` is the date of the options data behind the levels. Give both when they differ, and say so when the session is not the latest.

Put the close next to each level and give the distance in percent: `(level / close - 1) × 100`. A level above the close has a positive distance. Check these before you write:

- **`gamma` is null.** The ticker has no options data. Report its price and signals and say it has no levels.
- **`notFound` lists symbols.** They are unknown or not built yet. Run each through `search_tickers` before you say it does not exist.
- **Null fields.** A signal `value` or a volatility field can be null when the data was too thin. `implied` is null when the options chain had no usable surface that session. `ranges` is empty and `rank` is null when the ranges could not be built. Report what is there, name what is missing, and do not estimate it.
- **Units.** `changePct` and performance `pct` are in percent. Gamma values are dollars of dealer delta per $1 move in the underlying. Every volatility from `get_volatility` is a yearly fraction, so 0.25 means 25%.
- **Gamma wall equals call resistance.** Both are the same strike, so never present the wall as a second confirmation.
- **`front` levels.** They repeat the main levels for the nearest expiry alone. A level held by the nearest expiry stops mattering once that date passes.

For what each level means, how Owl Mastery computes it, and which result field holds each volatility figure, read `references/glossary.md`.

## Describe the regime

Compare the close with HVL. Above HVL, dealer hedging is assumed to lean against price moves, which can come with tighter ranges. Below HVL, it is assumed to go with them, which can make moves larger and faster. Neither side says where price will go. Call resistance is a strike where rallies may slow. Put support is a strike where declines may slow, or speed up once price breaks through it.

Owl Mastery publishes the HVL, not a gamma flip or a zero gamma level. When the user asks for either, answer with the HVL, say it is the closest Owl Mastery level, and say another site may compute its flip or zero gamma level differently, so the prices can differ. Likewise answer a question about the call wall or the put wall with call resistance or put support, and say the trader names can be computed differently elsewhere.

## Write the answer

1. The ticker, its close and the session date.
2. The levels with their distances, the regime, and the 1-day range, in a short table or list.
3. Anything the user asked for beyond that, from the other tools.
4. One sentence of limits when the answer leans on the levels. The glossary lists them.

Use the site's wording: levels "may" slow a move, hedging "tends to" damp or add to it. The numbers are educational information, not investment advice. If the user asks whether to buy or sell, say the tools show levels and not forecasts, and do not recommend a trade.

## When a call fails

- **401 or unauthorized.** The connection is not signed in, or the sign-in expired. Tell the user to reconnect Owl Mastery in their client's MCP or connector settings.
- **An unknown-ticker error.** Run the symbol through `search_tickers`, then call again with the symbol it returns.
- **An error that says the ticker has no gamma exposure, volatility or data.** The ticker has no such data. Say so and report what the other tools returned. Do not retry.
- **429 or too many requests.** Wait a second and retry once.
- **An error that says the call took too long, or that something went wrong.** Retry once. If it fails again, tell the user the server failed and stop.
