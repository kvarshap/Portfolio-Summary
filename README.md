# Portfolio Summary

A mobile-friendly, **serverless** view of my stock portfolio. It reads my Google Sheet (published as CSV) straight from the phone browser and answers one question: *what needs my attention right now?*

No backend, no build step, no dependencies — one `index.html` hosted on GitHub Pages.

> **Not financial advice.** This is a personal tracking tool. Prices come from Google Sheets (`GOOGLEFINANCE`) and can lag by minutes. Always confirm the live price in your broker app before placing an order.

## What it shows

| Tab | Purpose |
|---|---|
| **🏠 Home** (default) | Top gainers/losers today, 52-week high/low lists (within 10%), excluded & no-price stocks, SAFE vs RISK split, data health |
| **📋 Holdings** | Every stock with search, filters (Held, SELL, BUY, In loss, RISK, Watchlist) and sorting |
| **⚡ Act** | Stocks needing attention, most urgent first: triggers hit, sheet SELL/BUY, near 52-week high/low, big movers |
| **🔔 Alerts** | All buy/sell triggers by closeness to trigger, plus ⚙ data-source settings |

The header always shows portfolio value, total P&L, today's change and amount invested. Stock cards show CMP, average price and share count. Tap any stock card for 52-week range, trigger distances and a Google Finance link.

## How it works

```
Google Sheet ──(Publish to web, CSV)──▶ phone browser ──▶ index.html
 data.csv   = source of truth                (fetches on open / refresh)
 alerts.csv = my buy/sell triggers
```

- `data.csv` is the source of truth. The page only reads it; it never writes back.
- The last successful download is cached in the phone's `localStorage`, so the page still opens offline (with a banner saying the data is old).
- **When it fetches:** only when the page is opened and when you tap ⟳ (at most once every 5 minutes). There is no background polling or timed refresh. Each fetch is two small requests, one per sheet.
- Sheet links are **not** stored in this repo. They are pasted once in the page's settings and kept only in that browser.

## Setup

1. **Publish the sheets.** In Google Sheets: *File → Share → Publish to web* → choose the tab → format **CSV** → Publish. Do this for the data tab and (optionally) the alerts tab. Copy both links.
2. **Enable GitHub Pages.** Repo *Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: `main`, folder `/ (root)` → Save*. After a minute or two the site is live at `https://<username>.github.io/Portfolio-Summary/`.
3. **Open the site on your phone** and paste the two links when the settings sheet appears (reachable later at *Alerts → Data source settings*).
4. **Add to Home Screen** (iOS: Share → Add to Home Screen; Android Chrome: ⋮ → Add to Home screen).

## Sheet format

**data.csv** — required columns: `TICKER`, `Stock Name`, `Quantity`, `Avg Cost Price`, `CMP`.
Also used when present: `Include`, `Profit/Loss`, `Profit/Loss(%)`, `HIGH52`, `LOW52`, `CLOSEYEST`, `CHANGE`, `CHANGEPCT`, `Action`, `LEVEL` (`SAFE`/`RISK`), `Sector Name`.
**`Include` (optional; a tick box, or `1`/`0`):** rows marked `0`, `FALSE`, `No` or `N` are left out of every list and signal, and are only shown in the *Excluded & no price* table on Home. `1`, `TRUE`, a blank cell or no such column means included. Alerts for excluded tickers are ignored. Excluded holdings **still count in the header totals (value, P&L, today, invested) and the risk split**; everything else (gainers/losers, 52-week lists, Act, Holdings, Alerts) ignores them.
Rows with a missing or `#N/A` price are shown with `-` in the same table. `Quantity = 0` means watchlist.

**alerts.csv** — columns: `TICKER`, `Date`, `Buy Trigger Price`, `Sell Trigger Price`, `Last Action`, `Last Price`. A trigger of `0` means "not set".

## How the signals are decided

- **Buy trigger hit:** price ≤ buy trigger. **Sell trigger hit:** price ≥ sell trigger *and* the stock is held.
- **Sheet SELL / BUY:** taken as-is from the `Action` column in `data.csv`.
- **Near 52-week high:** price ≥ 95% of the 52-week high (held stocks only). **Near 52-week low:** price ≤ 105% of the 52-week low.
- **Big mover:** daily change of 5% or more, up or down.
- **Act-tab order:** SELL-type lists run from the biggest profit down to losses; BUY-type lists run from the biggest loss up to profits (by P/L %; Big movers by today's %).
- **Home tab (market-movers for your own stocks):** Top gainers and Top losers today (held stocks, up to 25 rows each), plus **52-week high** and **52-week low** lists of every stock within 10% of that level, closest first (no row limit), and an **Excluded & no price** table. Change `NEAR52` / `MORE_CAP` at the top of the script to adjust.
- Each stock is listed once on the Act tab, under its most urgent reason (in the order above); its tags show every reason that applies.

## Privacy

- This repo contains **no portfolio data and no sheet links**. `.gitignore` blocks `*.csv` and `.env` so they can't be committed by accident.
- The page is marked `noindex`, so search engines are asked not to list it.
- Note that anyone who knows your *published sheet link* can read that sheet. Treat those links like a password and re-publish the sheet to get a new link if one leaks.

## Troubleshooting

| Symptom | Fix |
|---|---|
| "Failed to fetch" / banner says offline or blocked | Check the link under *Alerts → Data source settings*; make sure the sheet is published as **CSV**, not as a web page. As a fallback, load the CSV files from the phone in the same settings sheet. |
| Numbers look old | Tap ⟳. Google refreshes published sheets only every few minutes, and `GOOGLEFINANCE` prices can lag. |
| A stock is missing | Check whether `Include` is `0`/`FALSE`, or its row has an empty or `#N/A` price. Both are listed in the *Excluded & no price* table on Home. |
| Alert never fires | Trigger of `0` is ignored, and alert tickers must match `data.csv` exactly. |

## Local testing

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

Put test CSVs next to `index.html` and point the settings to `http://localhost:8000/data.csv` (never commit them).
