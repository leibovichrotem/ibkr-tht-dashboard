# IBKR Portfolio Tracker — Hourly (data.js only)

**Name:** IBKR Portfolio Tracker - Market Hours  
**Cadence:** Mon–Fri hourly (15:00–23:30 Asia/Jerusalem)

## CRITICAL ARCHITECTURE (do not break)

- `index.html` is a **pure renderer**. NEVER edit, patch, or overwrite it.
- All numbers live only in `data.js` — **single source of truth**.
- Automation **only** writes/overwrites `data.js`, then `git add data.js` + commit + push.
- Browser Finnhub overlay may update on-screen Last/Chg%/Unrealized; it does **not** write back to data.js.

---

**Prompt:**

```
IBKR Portfolio Tracker — hourly refresh of data.js ONLY + git push.

## 0) Deploy key (REQUIRED first)
mkdir -p /root/.ssh ~/.ssh
cat > /root/.ssh/grok_deploy_key << 'KEYEOF'
# PASTE THE EXISTING DEPLOY KEY HERE (never commit the key to the repo)
# Same key already used by hourly automation — keep it only in the Automation prompt secret/config
KEYEOF
chmod 600 /root/.ssh/grok_deploy_key
cp /root/.ssh/grok_deploy_key ~/.ssh/grok_deploy_key 2>/dev/null || true
export GIT_SSH_COMMAND="ssh -i /root/.ssh/grok_deploy_key -o IdentitiesOnly=yes -o StrictHostKeyChecking=no"

## 1) Pull IBKR data
- get_account_positions
- get_account_summary
- get_account_trades period=YEAR_TO_DATE
- get_pa_performance_all_periods (TWR)

## 2) closed[] — deterministic aggregation
- SELL fills only.
- Group by order_id: sum realized_pnl, sum size, latest trade_time as date, size-weighted avg exit.
- Each row: { date:"YYYY-MM-DD", sym, qty, exit, pnl, cost, pct }
- Sort newest first.
- n = closed.length MUST equal kpis.n. Never invent trades.

## 3) kpis — compute server-side from closed[] only
- wins = pnl>0, losses = pnl<0
- winRate, profitFactor (null if no losses), avgWin, avgLoss, expectancy, avgWinPct, avgLossPct, totalRealized
- Browser must NOT recalculate KPIs from partial data.

## 4) positions[] from IBKR
- Fields: symbol, qty, avg, last, upnl, upnlPct, cost, mv, pctNet, chg:0, peter:null
- Sort by cost descending (UI also sorts by cost).

## 5) Peter public notes on holdings (required)
- X search: from:pdicarlotrader ($SYM OR $sym) since ~14 days, PUBLIC posts only.
- For each held symbol with a substantive public post:
  peter: { sum: "1-2 sentence paraphrase", url: "https://x.com/pdicarlotrader/status/<id>", when: "Aug 19" }
- Human-readable when (e.g. "Aug 19"), not ISO.
- If no public post -> peter: null. Do NOT invent.

## 6) recs[] — Recommendations (not held)
- X search: from:pdicarlotrader (buy|watchlist|adding|smart money|zone|structure) last 3-5 days, PUBLIC only.
- Exclude symbols currently in positions[].
- Each: { date:"Aug 20, 2026", sym, kind:"buy"|"watch"|"note", sum, url }
- Human-readable date. Empty array is OK if none found.

## 7) Performance (TWR vs SPY)
- From get_pa_performance_all_periods: portfolio m1/mtd/ytd/y1/all as percent (cps * 100).
- SPY proxy: keep structure:
  perfCompare: { portfolio:{m1,mtd,ytd,y1,all}, spy:{m1,mtd,ytd,y1,all:null} }
- equityReturns: { labels:["MM/DD",...], portfolio:[cum%...], spy:[cum%...] } same length, rebased to 0 at start.
- Optional: keep performance: { ytd:{labels,nav}, mtd:{labels,nav} } for compatibility.

## 8) meta
- generatedAt: ISO with +03:00
- generatedAtHuman: "YYYY-MM-DD HH:MM IDT"
- nav, cash, grossPositions, unrealized, positionsCount, source, note

## 9) Write ONLY data.js — full schema every time
window.__DATA__ = {
  meta: { ... },
  positions: [ ... ],      // with peter objects where found
  closed: [ ... ],
  kpis: { ... },
  recs: [ ... ],
  perfCompare: { portfolio:{}, spy:{} },
  equityReturns: { labels:[], portfolio:[], spy:[] },
  performance: { ytd:{}, mtd:{} }
};
Never omit recs, peter, perfCompare, or equityReturns (use [] / null / empty arrays if needed).

## 10) Push data.js only
cd /tmp/ibkr-tht-dashboard || (git clone git@github.com:leibovichrotem/ibkr-tht-dashboard.git /tmp/ibkr-tht-dashboard && cd /tmp/ibkr-tht-dashboard)
git pull origin main
# place the newly written data.js in repo root
git config user.email roti246@gmail.com
git config user.name "Grok Automation"
git add data.js
git commit -m "IBKR hourly data refresh $(date -u +%Y-%m-%dT%H:%MZ)" || true
GIT_SSH_COMMAND="ssh -i /root/.ssh/grok_deploy_key -o IdentitiesOnly=yes -o StrictHostKeyChecking=no" git push origin main

## 11) Reply short
NAV, unrealized, closed n + WR, Peter notes count, recs count, push ok.
Live: https://leibovichrotem.github.io/ibkr-tht-dashboard/
```

## Never
- Edit or overwrite index.html
- Hardcode KPI numbers that disagree with closed[]
- Drop peter / recs / perfCompare / equityReturns from the schema
- Leave closed[] incomplete or invent trades / Peter posts
