# Glossary of levels, signals and volatility

Definitions follow the Owl Mastery methodology page. Every level is a strike price, computed over all strikes and summed across expiries, from open interest. The levels are dated with the options data they come from.

## Gamma exposure (GEX)

Dollars of delta (hedging) exposure per $1 move in the underlying. Calls count as positive and puts as negative, and net exposure is call plus put. It rests on a common simplification: dealers are long the calls and short the puts their customers trade. Open interest does not show who holds each side, so real dealer positioning can differ.

## Levels

| Level | Definition |
| --- | --- |
| Call resistance | The strike with the largest net exposure. Rallies may slow there. |
| Put support | The strike with the most negative net exposure. Declines may slow there, or speed up once price breaks through it. |
| HVL (high volatility level) | The strike where the running sum of net exposure, from the lowest strike up, is lowest. It may mark the line between gamma regimes. Common names for similar ideas are gamma flip and zero gamma, but Owl Mastery publishes neither. |
| Gamma wall | The strike with the largest signed gamma times open interest, puts counted as negative. It is the same strike as call resistance. |
| 1-day range (`oneDayMin`, `oneDayMax`) | `spot ± spot × IV30 / √252`. An estimate of a typical day, not a limit. Price can close outside it. |
| Front levels | Call resistance, gamma wall, HVL and put support for the nearest expiry alone. |
| Top strikes | Strikes that are not already a level, ranked by `|net GEX × net DEX|`, with five strikes inside the 1-day range first. |

"Call wall" and "put wall" are trader names for the strikes with the most call or put gamma or open interest. Other providers compute them differently. The matching Owl Mastery levels are call resistance and put support.

## Regimes

- **Positive gamma** (close above HVL, positive net GEX). Dealers are assumed net long gamma. Their hedging leans against moves, selling as price rises and buying as it falls. It can come with smaller daily ranges.
- **Negative gamma** (close below HVL, negative net GEX). Dealers are assumed net short gamma. Their hedging goes with moves, which can make them larger and faster.

Neither regime says which way price will go.

## Signals

Computed from one year of daily bars. A signal is null when the stock has too few bars.

| Signal | Formula | Label |
| --- | --- | --- |
| RSI (14) | Wilder's RSI over 14 bars | Overbought at 70 or more, Oversold at 30 or less |
| vs 50D average | `(close / mean of the last 50 closes - 1) × 100` | Above, Below or At |
| Volatility (20D) | Sample standard deviation of the last 20 daily log returns, times `√252 × 100` | None |
| From 52W high | `(close / highest high of the last 252 bars - 1) × 100` | None |

## Implied volatility

The volatility that option prices imply. It reflects what the market prices in, not what has happened.

- **IV30** is the implied volatility of the listed expiry closest to 30 trading days out. It sets the 1-day range.
- **The 30-day figure** from `get_volatility` is the at-the-money implied volatility interpolated to 30 calendar days. It differs from IV30. Do not use one for the other.
- **The 21-session and 5-session figures** come from the options surface, over the next 21 and 5 US trading sessions.
- **The 68% and 95% ranges** are the prices one and 1.96 standard deviations either side of the median close that the implied volatility prices for each horizon. The spread is on a log scale, so each range reaches a little further above the close than below it.
- **IV percentile** is the share of past sessions whose 21-session implied volatility was below today's. **IV rank** is where today's value sits between the lowest and highest of those sessions, from 0 to 100. Both use at most the last 92 sessions.
- **Range track record.** A session's window counts as inside when the close 21 sessions later stayed within the 68% range that session priced. Windows overlap and are not independent, so read the count beside every share.

## Limits to carry into an answer

- Open interest is counted once a day, so same-day positions never appear in it.
- Gamma changes as price, time to expiry and implied volatility change. The exposure at the close can differ from the exposure the next session.
- A failed update keeps the previous figures. Check the date.
- Hedging is one force among many. No level or signal says where price will go.
