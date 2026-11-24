---
name: market
description: Summarizes the US market from Owl Mastery's data: breadth, index moves, top gainers and losers, 52-week highs and lows, sector moves, and the user's own watchlists. Use when the user asks how the market or a sector did, which stocks moved most, or wants their watchlist checked against Owl Mastery's levels.
---

# Read the market and the user's lists

These tools read the same daily snapshot as the Overview page. They update once a day after the US close. State the `asOf` date in every answer.

## Pick the tools in this order

1. **`get_market_overview`** for "how did the market do". Pass `range`, one of `1d`, `1w`, `2w`, `1m`, `3m`, `6m`, `ytd` or `1y`. It defaults to `1d`. `limit` sets how many gainers and losers come back, 1 to 25, default 10.
2. **`get_sectors`** for which sectors led or lagged, over the same ranges. Pass `sector` with a name from its own answer, such as Information Technology, to list that sector's stocks.
3. **`list_watchlists`** when the user asks about their own lists, Favorites or portfolio holdings. It takes no arguments and returns the caller's lists with each ticker's close and change.
4. **`get_ticker_levels`** for any ticker the user wants to go deeper on. Follow the `levels` skill from there. Send up to 10 symbols per call, so split a longer list into batches.

## Read the result

- A move runs from the close of the range's first session to the latest close. `changePct` is in percent.
- Breadth counts stocks only. The indexes are reported apart from it.
- The 52-week high and low counts exist on `1d` only. On any other range they are null, so do not report them.
- A sector's average is equal-weight: every stock in it counts the same.
- A watchlist ticker without a quote has a null `close` and `changePct`. Say it has no quote. Do not guess one.
- The lists belong to the signed-in user. Report only what the user asked for, and do not copy list contents into places the user did not name.

## Write the answer

Lead with the breadth and the indexes, then the sectors that led and lagged, then a few named movers. Keep numbers in percent with their range. Say what the data is: a daily snapshot that describes what already happened. It is educational information, not investment advice, and it does not predict the next session.

If a call fails with 401, the API key is missing, wrong or expired. Keys last 14 days. Tell the user to create a new one and set `OWLMASTERY_API_KEY`.
