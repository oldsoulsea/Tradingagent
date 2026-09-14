# Setup Scanner — TradingView Indicator

An all-in-one setup scanner that scores every bar from 0–10 on both bullish and
bearish strength. Works on any market (stocks, crypto, forex) and any timeframe
— optimized for **Daily** and **Weekly** charts.

## What it checks

| Category | Signals | Max Points |
|---|---|---|
| **Trend** | EMA stack (9/21/50/200), price vs anchor EMA, ADX + DI | 3 |
| **Momentum** | RSI zone, MACD cross/direction, Stochastic K/D | 4 |
| **Volume** | Spike vs 20-period MA | 1 |
| **Candlestick** | Engulfing, hammer, morning/evening star | 2 |
| **S/R Context** | Near support/resistance, breakout/breakdown | 1 |

A signal fires when the score meets your threshold (default 5) and exceeds the
opposing score.

## How to install

1. Open [TradingView](https://www.tradingview.com/) and go to any chart.
2. Click **Pine Editor** at the bottom of the screen.
3. Delete the placeholder code and paste the entire contents of
   [`setup-scanner.pine`](setup-scanner.pine).
4. Click **Add to chart**.

## What you see on the chart

- **EMAs** — 9 (blue), 21 (orange), 50 (pink), 200 (purple)
- **BULL / BEAR arrows** — appear when the setup score meets your threshold
- **Green / red background tint** — highlights bars with especially strong scores
- **Support / resistance dots** — derived from 10-bar pivot highs/lows
- **Dashboard** (top-right) — live readout of every component and the composite
  score

## Settings you can tune

| Setting | Default | What it does |
|---|---|---|
| Min Bullish Score | 5 | How many bullish points needed to print a signal |
| Min Bearish Score | 5 | Same for bearish |
| Volume Spike Mult | 1.5× | Volume must exceed MA × this to count as a spike |
| ADX Threshold | 20 | ADX below this = weak/no trend |
| Show EMAs | on | Toggle the four EMA lines |
| Show Signals | on | Toggle the BULL/BEAR arrows |
| Show Dashboard | on | Toggle the score table |

## Alerts

Four alert conditions are built in — set them from the TradingView **Alerts**
dialog (⏰):

- **Bullish Setup** — score meets your threshold
- **Bearish Setup** — score meets your threshold
- **High-Volume Bullish** — bullish setup + volume spike
- **High-Volume Bearish** — bearish setup + volume spike

## Tips

- On **Daily/Weekly**, a score of 6+ with a volume spike is a high-conviction
  setup.
- Use the dashboard to see *why* a signal fired — if trend and momentum agree
  while volume confirms, the setup is stronger.
- Pair with your own support/resistance or supply/demand zones for entries.
