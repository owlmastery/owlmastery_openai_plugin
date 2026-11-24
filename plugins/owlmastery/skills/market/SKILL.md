---
name: market
description: "Summarizes the US market from Owl Mastery's daily snapshot: breadth, index moves, top gainers and losers, 52-week highs and lows, sector leaders and laggards, and the user's own watchlists. Use it whenever the user asks how the market, an index or a sector did, which stocks moved most, what led or lagged this week or month, or wants their Favorites, watchlist or portfolio holdings checked, even if they do not say Owl Mastery. Do not use it for one ticker's gamma levels or implied volatility (use the levels skill), for which stocks are in negative gamma, below their put wall or on another screen, or how many closed above or below their HVL (use the screen skill), or to recommend a trade."
---

# Read the market and the user's lists

These tools read the same daily snapshot as the Owl Mastery Overview page. It updates once a day after the US close. State the `asOf` date in every answer.

## Choose the tools

- **`get_market_overview`** for "how did the market do". Pass the range the user names. It defaults to one day (`1d`).
- **`get_sectors`** for which sectors led or lagged, over the same ranges. Pass `sector` with a name from its own answer, such as Information Technology, to list that sector's stocks.
- **`list_watchlists`** when the user asks about their own lists, Favorites or portfolio holdings. It takes no arguments.
- **`get_ticker_levels`** for any ticker the user wants to go deeper on. Follow the `levels` skill from there. It takes up to 10 symbols, so split a longer list into batches.

Do not call a tool whose answer the user did not ask for. For which stocks are in negative gamma, below their put wall or on another screen, or how many stocks closed above or below their levels, the `screen` skill covers it.

## Read the result

- A move runs from the close of the range's first session to the latest close. `changePct` is in percent.
- Breadth counts stocks only. The indexes are reported apart from it.
- The 52-week high and low counts exist on the one-day range only. On any other range they are null, so do not report them.
- A sector's average is equal-weight: every stock in it counts the same.
- A watchlist ticker without a quote has a null `close` and `changePct`. Say it has no quote. Do not guess one.
- The lists belong to the signed-in user. Report only what the user asked for, and do not copy list contents into places the user did not name.

## Write the answer

Give the `url` from the answer, the page on owlmastery.com that shows it. Lead with the breadth and the indexes, then the sectors that led and lagged, then a few named movers. Keep numbers in percent with their range. Say what the data is: a daily snapshot that describes what already happened. It is educational information, not investment advice, and it does not predict the next session.

## When the Owl Mastery tools are missing

If no Owl Mastery tool is available in this session, the connection is not set up. Do not search for other connectors or call other tools, and do not estimate any number. In two or three sentences, tell the user to connect Owl Mastery, give the steps for their client from `references/connect.md`, and link https://owlmastery.com/connect. After connecting, the user asks the question again in a new conversation.

## When a call fails

- **401 or unauthorized.** The connection was disconnected, or is not signed in. Tell the user to connect Owl Mastery again from their client, with the steps in `references/connect.md`, or https://owlmastery.com/connect.
- **An unknown-sector error.** It lists the valid sector names. Pick one of them.
- **429 or too many requests.** Wait a second and retry once.
- **An error that says the data is being rebuilt, the call took too long, or something went wrong.** Wait a moment and retry once. If it fails again, tell the user the server failed and stop.
