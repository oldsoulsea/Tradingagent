# Weekly Options Income Screener

A saved Robinhood scan for discovering new candidate tickers to trade
options on — high-quality companies with elevated implied volatility and
liquid options markets. This is a discovery tool, separate from the CSP
strategy in `STRATEGY.md`.

**Scan ID**: `4abfeee3-8645-41b9-b03d-f97845daff6a`
**Title in Robinhood/Legend**: "Weekly Options Income Screener"

Re-run it anytime (in the Robinhood app under Legend scans, or by asking the
agent to run scan `4abfeee3-8645-41b9-b03d-f97845daff6a`) to get a fresh
list — results change daily as prices, volatility, and earnings dates move.

## What it does NOT do

This screener finds *candidates worth a closer look*. It does not add
anything to `WATCHLIST.md` and never authorizes a trade — per `CLAUDE.md`,
adding a ticker to the approved watchlist, and placing any order, both
require the user to explicitly say so.

## Filters

| Filter | Condition | Purpose |
|---|---|---|
| Asset type | STOCK | excludes ETFs/crypto/futures |
| Market cap | > $10B | "quality" floor — large/mid-cap only |
| Net profit margin | > 0% | profitable |
| Return on equity | > 10% | capital-efficiency quality bar |
| Avg options volume (30d) | > 5,000/day | liquid enough to trade weeklies with reasonable spreads |
| Open interest | > 10,000 | liquidity depth |
| Implied volatility (ATM, 30-day) | > 35% | "high IV" gate |
| Earnings date | > 14 days out | excludes imminent-earnings names |

Columns also surfaced (not filtered on): Sector, Historical volatility, P/E,
Forward P/E, Earnings date — useful context when picking among matches.

## Known limitation: no true IV Rank

Robinhood's scanner exposes raw current implied volatility
(`atmIv30Day`) but has **no historical IV time series**, so there is no way
to compute a real IV Rank/Percentile (where current IV sits in its own
52-week range) through this API. The IV filter above screens on the *raw
level* instead, and the results table includes an **IV/HV ratio** (implied
vs. historical/realized volatility) as a rough proxy for "richly priced"
options — not a substitute for true rank.

If you want the real IV Rank number for a specific candidate before
trading it, check it externally (the options-chain view in the Robinhood
app, or a service like Market Chameleon, Barchart, or tastytrade) — this
agent can't compute it.

## Other things to sanity-check on results

- **Sector concentration**: the scan doesn't cap how many results come from
  one sector. Recent runs skewed heavily toward semiconductor/AI-hardware
  names (SMTC, ALAB, SNDK, MRVL, SMCI, STX, WDC, TER, etc.) — correlated
  moves can hit several "different" positions at once.
- **Sector column is unreliable**: it currently returns numeric codes (e.g.
  "311") instead of sector names — an API quirk, not decoded here.
- Passing this screen is not the same as passing `STRATEGY.md`'s entry
  criteria (delta, oversold/breakout signal, yield threshold, sizing). A
  candidate found here still needs to clear those before becoming a trade
  proposal — and, if you want it in rotation, still needs to be added to
  `WATCHLIST.md` explicitly.

## Editing the scan

Ask the agent to adjust filters (e.g. raise/lower the IV bar, add a sector
exclusion, change the earnings buffer) — it can call `update_scan_filters`
on the scan ID above rather than creating a new one.
