---
name: screen
description: "Finds which US stocks match one of Owl Mastery's daily screens and counts the market against its gamma levels: stocks in negative or positive gamma (below or above their HVL), near the HVL, below put support (put wall), above call resistance (call wall), RSI oversold or overbought, or near the 52-week high, in the whole market or one sector, and how many stocks or what share closed above or below their HVL, call resistance and put support. Use it whenever the user asks which stocks, names or tickers meet one of those conditions, or how much of the market or a sector is in negative gamma, even if they do not say Owl Mastery. Do not use it for one named ticker's levels (use the levels skill), for price moves, breadth or watchlists (use the market skill), or to recommend a trade."
---

# Screen the market against its levels

These tools read the same daily snapshot as Owl Mastery's screen pages and its market gamma report. It updates once a day after the US close. State the `asOf` date in every answer.

## Choose the tools

- **`screen_tickers`** for "which stocks…". Pass `screen` with one of these, and `sector` with a GICS sector name, such as Information Technology, when the user names one:

  | `screen` | A stock is on it when it closed |
  | --- | --- |
  | `negative-gamma` | below its HVL |
  | `positive-gamma` | above its HVL |
  | `near-hvl` | within 0.5% of its HVL, either side |
  | `below-put-support` | below its put support (put wall) |
  | `above-call-resistance` | above its call resistance (call wall) |
  | `rsi-oversold` | with an RSI (14) of 30 or less |
  | `rsi-overbought` | with an RSI (14) of 70 or more |
  | `near-52-week-high` | within 3% of its 52-week high |

  It returns 15 rows unless you pass `limit`, at most 50. Raise it only when the user wants a longer list.
- **`get_market_gamma`** for "how many" or "what share": how many stocks closed above, below or at their HVL, above call resistance and below put support, for the whole market and for each sector, in one call.
- **`get_ticker_levels`** when the user wants to go deeper on a few of the names. Follow the `levels` skill from there.

Map the trader words: short gamma or below the gamma flip is `negative-gamma`, long gamma is `positive-gamma`, the put wall is put support and the call wall is call resistance. Owl Mastery publishes the HVL, not a gamma flip or zero gamma level, so say another site may draw that line elsewhere. For one named ticker, use the `levels` skill instead.

## Read the result

- **`matched` and `covered`.** `criterion` says what a ticker on the screen did. `matched` counts the stocks and indexes on the screen. `covered` counts the stocks that have the data it reads: levels for the gamma screens, RSI for the RSI screens, a 52-week high for the last one. Give the count as "matched of covered", never as a share of every stock.
- **`rows`.** At most `limit` of the matches, most extreme first. When `matched` is larger, say the list shows only the first rows.
- **`value`.** On a gamma screen, the level's distance from the close in percent, `(level / close - 1) × 100`, and `level` is the level's price. On an RSI screen, the RSI (14). On `near-52-week-high`, the close's distance from the 52-week high in percent, 0 or less. `level` is null on the last three.
- **`stale`.** The row's levels are from an earlier session than its close. Name those tickers and say their levels are older.
- **`get_market_gamma` counts.** `market` counts every stock and index together, and `sectors` has one entry per GICS sector. In each, `total` counts every ticker and `withGamma` only those with levels. Take each share of `withGamma`. `above`, `below` and `at` add up to `withGamma`; `at` is a close exactly at the HVL.

## Write the answer

Give the `url` from the answer, the page on owlmastery.com that shows it.

1. The session date, the screen in plain words, and the count, such as "41 of 480 stocks with levels closed below their HVL".
2. A table of the rows: symbol, name, close, change, the level and its distance, or the RSI.
3. One hedged sentence on what the screen may mean:
   - Below the HVL, dealer hedging is assumed to go with moves, which may mean larger moves. Above it, it is assumed to lean against them, which may mean smaller ones. Neither says where price goes next.
   - A close below put support may come with faster moves. A close above call resistance shows the price moved past the largest call strike.
   - An oversold RSI says the recent decline was strong, not that a rebound will follow. An overbought RSI says the advance was strong, not that a decline will follow.

A screen describes options positioning and price signals after the close. It is educational information, not investment advice, and not a list of stocks to buy or sell. If the user asks what to trade, say the screens show levels and not forecasts, and do not recommend a trade.

## When the tools are missing or a call fails

- **No Owl Mastery tool in this session.** The connection is not set up. Do not search for other connectors or call other tools, and do not estimate any number. In two or three sentences, tell the user to connect Owl Mastery, give the steps for their client from `references/connect.md`, and link https://owlmastery.com/connect. After connecting, the user asks the question again in a new conversation.
- **401 or unauthorized.** The user disconnected the app, or never signed in. Give the same steps.
- **A tool error.** It says what to try next. An unknown-screen or unknown-sector error lists the valid names. Pick one of them.
- **429, a call that took too long, or an error that says something went wrong.** Wait a second and retry once. If it fails again, tell the user the server failed and stop.
