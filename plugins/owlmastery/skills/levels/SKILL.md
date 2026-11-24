---
name: levels
description: "Reads Owl Mastery's gamma levels and implied volatility for one or a few named US stocks or indexes against their close: call resistance, put support, HVL, gamma wall, net gamma, the 1-day range, the strikes with the most exposure, implied volatility, its history and its 68% and 95% price ranges, and shows a ticker's price, gamma, volatility, odds of reaching a price or gamma history as a chart. Use it whenever the user names a ticker or company, or a few to compare, and asks about its price, how it moved or how it did over a period, where it sits against its levels, what its gamma regime is, about its call wall, put wall, gamma flip, zero gamma or dealer hedging, what its options market implies, the odds or probability that it reaches a price, or how its gamma exposure changed over past sessions, or asks to show, chart, graph, plot, draw or explore any of these, even if they do not say Owl Mastery. Do not use it to find which stocks match a condition such as negative gamma or below the put wall (use the screen skill), for market-wide, sector or watchlist questions (use the market skill) or to recommend a trade."
---

# Read a ticker's levels

The Owl Mastery tools return one snapshot per ticker, computed once a day after the US close. Nothing updates while the market is open.

## Choose the tools

- **`search_tickers`** when the user gives a company name or an unsure symbol. Use the symbol it returns. If several match, ask which one the user means.
- **`get_ticker_levels`** answers most questions. It takes up to 10 symbols, so send every ticker of a request in one call.
- **`get_gamma_exposure`** only for strikes: which strikes hold the most exposure, how it splits across expiries, or how the total moved from the session before. If it answers with an error, the ticker has no per-strike data. Report what `get_ticker_levels` returned.
- **`get_volatility`** only for what the options market implies: implied volatility, the 68% and 95% price ranges, or whether volatility is high or low against recent sessions. Pass `history` with a number of sessions (at most 92) only when the user asks how volatility moved over time, such as whether implied volatility is rising or whether it overstated the moves that followed. Leave it out otherwise.
- **`get_gamma_history`** for the numbers behind a question about how one ticker's gamma exposure or levels moved over past sessions: one row per session, over `sessions` of 20 (the default), 60, 120 or 250. Ask for the shortest period that covers the question.
- **A `show_` tool** when the question is about one ticker's price, its gamma or levels, its implied volatility, the odds of reaching a price, or its gamma history. Call it even when the user does not say chart. [Show a chart](#show-a-chart) names the tool for each.
- **`get_market_overview`** or **`get_sectors`** for the market around the ticker. The `market` skill covers them. For which stocks match a condition, such as negative gamma, the `screen` skill covers it.

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
- **`history` rows.** Oldest first. `iv21` is the 21-session implied volatility on that date. `rv21` is the volatility the stock then realized over the next 21 sessions, and `ret21` its return over them as a fraction. Both are null for the last 21 sessions, which have not traded out yet. Compare `iv21` with `rv21` only on rows that have both, and say how many rows that is.

For what each level means, how Owl Mastery computes it, and which result field holds each volatility figure, read `references/glossary.md`.

## Show a chart

When the question is about one ticker, show it as a chart, even when the user does not ask for one. Call `show_price_chart` for its price, how it moved or how it did over a period. Call `show_gamma_chart` for its gamma, exposure or levels. Call `show_volatility_chart` for its implied volatility, expected move or price range. Call `show_probability` for the odds that it reaches a price, or closes above or below it. Call `show_gex_explorer` for how its gamma exposure or levels changed over past sessions or across expirations. Call these two only when the question asks for them. When a question covers more than one of these, such as "how is AAPL doing", call each chart that answers part of it. Pass one ticker per call. When the user compares several tickers, answer with the table and no charts.

Each takes an optional choice. `range` for price: `6m` by default, or the period the user names, such as `1m`, `3m`, `ytd` or `1y`. `view` for gamma: `key-levels` by default, `by-strike`, `regime`, `calls-puts` or `change`. `chart` for volatility: `cone` by default, `track`, `range-held`, `rank` or `term`. `period` for the GEX explorer: `60` sessions by default, `20`, `120` or `250`. `target` and `sessions` for the odds: the price and how many sessions ahead, 1 to 21. Pick a view only when the user names it.

- **Do not mention the chart's display.** Do not say a chart is shown above, and do not hedge about whether the app draws charts. Apps that draw it show it with your answer, and the others show its text. Write the answer as if the numbers stand alone.
- **Always pass on the `url`.** It is the page on owlmastery.com that shows the chart.
- **Give the numbers.** The answer carries the key figures, such as the levels or the implied ranges. State them with the link, so the answer works in a terminal, which draws no chart.
- **Do not call a `get_*` tool for those numbers.** Call `get_gamma_exposure` or `get_volatility` only for what the chart answer lacks, such as the strikes with the most exposure or the volatility history.
- **Pass the price and the date the user names to `show_probability`.** `target` is the price in dollars, and `sessions` is how many sessions ahead, 1 to 21, where a month is 21. Without them the odds are for a close about 5% above the latest close, 21 sessions ahead. State the target and the session with the odds, and say the model is log-normal with no drift. An error that says the target is outside the explorer's range names the range: tell the user, and offer a price inside it. For a date more than 21 sessions ahead, say the odds stop at 21 sessions. Do not work out other odds yourself.
- **`show_gex_explorer` answers the latest session.** Its numbers are that session's spot, levels and exposure. For how they moved over the period, call `get_gamma_history` too, and describe the change from its rows with their dates. Do not describe a trend from the explorer's answer alone.

## Describe the regime

Compare the close with HVL. Above HVL, dealer hedging is assumed to lean against price moves, which can come with tighter ranges. Below HVL, it is assumed to go with them, which can make moves larger and faster. Neither side says where price will go. Call resistance is a strike where rallies may slow. Put support is a strike where declines may slow, or speed up once price breaks through it.

Owl Mastery publishes the HVL, not a gamma flip or a zero gamma level. When the user asks for either, answer with the HVL, say it is the closest Owl Mastery level, and say another site may compute its flip or zero gamma level differently, so the prices can differ. Likewise answer a question about the call wall or the put wall with call resistance or put support, and say the trader names can be computed differently elsewhere.

## Compare tickers

When the user names two or more tickers to compare, send them in one `get_ticker_levels` call, at most 10 symbols per call. Answer with one table: a row per ticker with its close, each level's distance in percent and its regime. Compare distances, not dollar prices, because the tickers trade at different prices. Rank the tickers only by what the user asked, such as which sits closest to its call resistance, and give the session date once if every ticker shares it.

## Write the answer

1. The ticker, its close and the session date.
2. The levels with their distances, the regime, and the 1-day range, in a short table or list.
3. Anything the user asked for beyond that, from the other tools.
4. The `url` of the ticker's page from the answer, so the user can open it.
5. One sentence of limits when the answer leans on the levels. The glossary lists them.

Use the site's wording: levels "may" slow a move, hedging "tends to" damp or add to it. The numbers are educational information, not investment advice. If the user asks whether to buy or sell, say the tools show levels and not forecasts, and do not recommend a trade.

## When the tools are missing or a call fails

- **No Owl Mastery tool in this session.** The connection is not set up. Do not search for other connectors or call other tools, and do not estimate any number. In two or three sentences, tell the user to connect Owl Mastery, give the steps for their client from `references/connect.md`, and link https://owlmastery.com/connect. After connecting, the user asks the question again in a new conversation.
- **401 or unauthorized.** The user disconnected the app, or never signed in. Give the same steps.
- **A tool error.** It says what to try next, such as `search_tickers` for an unknown ticker. Do that. An error that says the ticker has no gamma exposure, volatility or data means it has none. Say so, report what the other tools returned, and do not retry.
- **429, a call that took too long, or an error that says something went wrong.** Wait a second and retry once. If it fails again, tell the user the server failed and stop.
