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
| IV Rank (`ivRank`, 52-week) | > 50% | real "high IV rank" gate — see below |
| Earnings date | > 14 days out (fixed date, see below) | excludes imminent-earnings names |

Columns also surfaced (not filtered on): Sector, Historical volatility, P/E,
Forward P/E, Earnings date — useful context when picking among matches.

### Earnings filter needs manual refreshing

Robinhood's scanner only supports an absolute date for `FILTER_TYPE_EARNINGS_DATE`,
not a relative "N days from now." So this filter is a **hardcoded cutoff
date** that drifts stale as time passes — it silently stops excluding
near-term earnings once "today" catches up to it. This already happened
once (2026-10-02: the cutoff was still 2026-09-21 from two weeks earlier,
and 5 tickers with earnings inside the 14-day window — ASML, IBKR, DAL, AA,
INFY — slipped through before being caught and fixed).

**Before trusting a run's results, check whether the cutoff needs
bumping.** Current cutoff value: **2026-10-16** (set 2026-10-02, intended
as "14 days out" at that time). Ask the agent to refresh it to
today + 14 days whenever you run this scan after a gap.

## IV Rank (resolved — this now uses the real metric)

Earlier versions of this screener assumed Robinhood's scanner had no
historical IV time series and used raw IV level + an IV/HV ratio as a
proxy. That assumption was wrong: `get_scanner_datapoints` (not available
when this screener was first built) exposes an **`ivRank`** field —
`(iv − iv52WeeksLow) / (iv52WeeksHigh − iv52WeeksLow)`, exactly the
industry-standard definition — as an expression filter. The scan now
filters on `ivRank > 0.5` (50th percentile of the stock's own 52-week IV
range) instead of raw IV level.

This materially changes the result set: raw-IV screening was heavily
skewed toward semiconductor/AI-hardware names simply because that sector
had elevated *absolute* volatility; `ivRank` surfaces different names
entirely (energy, mortgage REITs, fintech, media, healthcare have shown up
in recent runs) because a stock can have low absolute IV but still sit
high in its *own* range, or vice versa.

**Caveat on a similarly-named field**: `atmIv30DayPosInRange` looks like it
should be "30-day IV rank" but is NOT — testing showed it pegged at 1 for
almost every result, meaning it measures where 30-day IV sits in the
*term structure across expirations* (30d/60d/90d/.../720d), not a
historical percentile. Don't use it for IV rank; use bare `ivRank`.

Implied volatility and historical volatility are still shown as columns
for context (IV/HV ratio can be computed from them), just no longer the
filter.

## Other things to sanity-check on results

- **Sector concentration**: the scan doesn't cap how many results come from
  one sector — check the current run rather than assuming. The old
  raw-IV filter skewed heavily toward semiconductors; the `ivRank` filter's
  2026-10-02 run skewed toward energy (PBR, DINO, PSX, MPC, VLO, SU, DVN).
  Either way, correlated moves can hit several "different" positions at
  once.
- **Sector column is unreliable**: it currently returns numeric codes (e.g.
  "311") instead of sector names — an API quirk, not decoded here.
- Passing this screen is not the same as passing `STRATEGY.md`'s entry
  criteria (delta, oversold/breakout signal, yield threshold, sizing). A
  candidate found here still needs to clear those before becoming a trade
  proposal — and, if you want it in rotation, still needs to be added to
  `WATCHLIST.md` explicitly.

## Editing the scan

Ask the agent to adjust filters (e.g. raise/lower the IV Rank bar, add a
sector exclusion, change the earnings buffer). Note: the scan now has an
**expression filter** (`ivRank`) — Robinhood's `update_scan_filters` tool
rejects expression filters, so changes go through `create_scan` with this
scan's `scan_id` (appends a new configuration version; history is kept),
not `update_scan_filters`.

---

# Quality Growth + High IV Rank Screener

A second, separate saved scan — same base filters as above (quality,
liquid options, IV Rank > 50%, no imminent earnings) plus a growth overlay:
companies with growing revenue and positive EPS growth, not just "quality"
by margin/ROE alone.

**Scan ID**: `e3fdb1bb-329f-455b-bf96-9288b42b06c7`
**Title in Robinhood/Legend**: "Quality Growth + High IV Rank Screener"

This scan is independent of the first one — editing one doesn't affect the
other. Same rules apply: discovery only, doesn't touch `WATCHLIST.md` or
place trades.

## Added filters (on top of everything in the first screener)

| Filter | Condition | Purpose |
|---|---|---|
| Revenue growth (`fundamental.quarterlyRevenueGrowth`) | > 5% YoY | "growing revenue" |
| Annual EPS growth (`fundamental.annualEpsGrowth`) | > 0% | EPS growing year-over-year |
| Quarterly EPS growth (`fundamental.quarterlyEpsGrowth`) | > 0% | EPS also growing in the most recent quarter |

**On "steady"**: there's no multi-quarter trend/consistency datapoint
available through this API — no way to check "grew in each of the last 4
quarters," only point-in-time snapshots. Requiring *both* the annual and
most-recent-quarter EPS growth figures to be positive is the closest
approximation of "steady" achievable here: it rules out a name that grew
for the year but is currently declining (or vice versa), but it's not a
real trend check. Treat it as a floor, not proof of consistency.

**Data-quality flag — extreme growth numbers from a small base**: some
matches show absurd-looking growth (e.g. NLY's quarterly EPS growth showed
3,433% and revenue growth 706% in the 2026-10-02 run). This is a real
artifact of growth math on a tiny or near-zero prior-year base, not a
data error — but it's not "steady growth" in the intended sense either.
Sanity-check any triple/quadruple-digit growth number against the
company's actual financials before treating it as a real signal.

## Results snapshot (2026-10-02, 38 matches)

Top names by IV Rank: PBR, DINO, GEN, PSX, MPC, ITUB, CBOE, NU, VLO, NLY,
MPLX, NYT, BSX, TMO, BMY, FTNT. Heavy energy/refiner representation (PBR,
DINO, PSX, MPC, VLO) again — same sector-concentration caveat as the first
screener applies.
