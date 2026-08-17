# IBKR THT Portfolio Dashboard

Interactive portfolio tracker aligned with **Peter DiCarlo’s THT Bull Cycle** methodology (Smart Money Zones, structure, Monthly BX, 33 FVB).

## Live Dashboard

This repository hosts a self-contained HTML dashboard that shows:

- Current IBKR positions
- **Score** column (BUY / HOLD / SELL) based on SMZ / IBZ / Premium + structure rules
- Allocation and NAV charts
- Rules & psychology reminders
- Latest public commentary from @pdicarlotrader (when available)

## Scoring Rules (summary)

| Score | Meaning |
|-------|---------|
| **BUY** | Price inside SMZ (or between SMZ and swing low) + higher-timeframe structure still bullish |
| **HOLD** | Inside IBZ or above SMZ but clearly below premium zone + structure intact |
| **SELL** | At/above premium zone **or** break of structure **or** claimed previous swing high **or** Monthly BX dark red |

## How it is updated

A weekday automation pulls live data from Interactive Brokers, recalculates scores and structure levels, and overwrites the dashboard file.  
Structure-break alerts are sent by email when a key swing low is broken.

## Disclaimer

This is a personal tracking tool.  
It is **not** financial advice. Always do your own research and verify Monthly BX / structure on your own charts.

---

Built with Grok · ibkr-trade-tracker skill
