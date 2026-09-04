# Strategy: Weekly Cash-Secured Puts

Income strategy: sell cash-secured puts (CSPs) weekly on approved tickers,
sized as a percentage of buying power, targeting a 35% annualized return per
trade, closed early at 80% profit and redeployed.

This document defines the *screening and sizing rules* the agent uses to
build trade candidates. It does **not** grant standing authorization to
trade — every order still requires the user's explicit, in-the-moment
confirmation per the rules in `CLAUDE.md`.

## Account

Trades only in the one Robinhood account flagged `agentic_allowed: true`
(currently nicknamed "Agentic"). Capital in that account is expected to grow
over time; because sizing below is percentage-based, no rule here needs to
change as funds are added.

## Universe

Only tickers listed in `WATCHLIST.md` are eligible. No exceptions unless the
user adds a ticker to that file first.

## Entry criteria

All of the following must hold to open a new CSP:

1. **Instrument**: cash-secured put — collateral (strike x 100/contract)
   fully covered by account buying power, no margin.
2. **Cadence**: new positions opened weekly, targeting the nearest weekly
   expiration (~5-9 calendar days to expiration).
3. **Strike / delta**: ~0.25 delta out-of-the-money put (0.20-0.30 acceptable
   if 0.25 isn't available on the chain).
4. **Setup signal** — at least one of:
   - **Oversold**: price at or below the middle Bollinger Band (20-period
     SMA, daily chart) — i.e. in the lower half of the band range. (Updated
     2026-09-04: originally required price near/below the *lower* band;
     relaxed to the middle band after review.)
   - **Breakout**: price breaking above a clearly defined prior
     resistance / consolidation range (a multi-week trading range or
     horizontal resistance level) — a bullish continuation setup, not a
     mean-reversion one.
5. **Yield threshold**: annualized return on capital >= 35%.
   - `annualized_return = (premium_collected / collateral) * (365 / days_to_expiration)`
   - `collateral = strike_price * 100` per contract.
   - Skip the trade if it doesn't clear this bar, even if delta and signal
     criteria are met.
6. **Watchlist membership**: symbol must be in `WATCHLIST.md`.

## Position sizing

- Each new CSP is capped at **20% of current buying power** in the Agentic
  account (default — edit this number here if you want a different cap).
- No hard cap on concurrent position count; it's bounded naturally by the
  20% sizing rule.
- If 20% of buying power can't cover even one contract's collateral for a
  candidate's strike, skip that ticker that week rather than exceeding the
  cap.

## Exit rule

- **Profit target**: buy-to-close once the position has captured 80% of max
  profit (option value has decayed to <=20% of the credit originally
  collected).
- **Redeployment**: once a position closes (profit target, expiration, or
  assignment), immediately re-screen the watchlist against the entry
  criteria above and open the next qualifying CSP with the freed collateral.

## Weekly workflow

1. Pull current positions/orders in the Agentic account (avoid duplicating
   an existing position in the same ticker).
2. For each watchlist ticker, pull price history + Bollinger Bands to check
   the oversold signal, and check for a qualifying breakout setup.
3. For tickers that clear a signal, pull the options chain, find the
   ~0.25-delta put in the nearest weekly expiration, compute annualized
   yield.
4. Filter to candidates clearing the 35% annualized bar and the 20%
   position-sizing cap.
5. Present candidate trade(s) to the user with full order details + account
   impact, one at a time, for confirmation before placing anything.
6. For existing open positions, check whether any have hit 80% profit
   capture and propose closing + redeploying.

## Open assumptions (confirm or adjust)

- **No stop-loss / defensive-roll rule** is defined for a CSP that moves
  against us before hitting 80% profit. Current assumption: let assignment
  happen if it comes to that (standard CSP philosophy — you're fine owning
  100 shares at your net cost basis). Say the word if you'd rather have a
  defensive roll or stop-loss rule instead.
- **"Weekly expiration"** is assumed to mean the nearest Friday expiration,
  ~5-9 days out at entry — flag it if you meant something else (e.g. always
  exactly 7 DTE).
- **20% per-trade sizing** is a starting default, not something you
  explicitly specified a number for — change it above if you want tighter
  or looser sizing.
