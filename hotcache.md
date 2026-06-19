# NoTwTo Hotcache
## Generated: 2026-06-19 ACST
## Version: 3.0 (timezone fix done · forward-test reset · first valid trade)

### Current Status
Account: ~$3,027.53 AUD (DEMO) | P&L this week: GBPUSD +19.6 pips
Mode: **DEMO-DIRECT** (config.json phase6.mode = DEMO) — live account not funded
Bot: RUNNING ✅ — Strategy A loop, sleeping to next NY open 21:30 ACST (12:00 UTC)
Auto-start: At-logon launcher + Task Scheduler (dependency-ordered: MT5 → bot)
Forward test: **RESET 2026-06-18** — PF count restarts from 0 (4 pre-fix trades archived INVALID)
Record (post-fix, valid only): **1W – 0L**
Phase 6: CODE INTACT — bypassed in DEMO; re-enables when mode=LIVE + live_confirmed=true
MT5 MCP: CONNECTED ✅ | account 7942393 | Eightcap-Demo
Git: main | Latest push: 1407335 (notwto-hq)

### First Valid Post-Fix Trade ✅
```
2026-06-18 | GBPUSD.i LONG | [DEMO-DIRECT] [ORB] 15UTC | ticket 235373783
Entry 1.32285 | SL 1.32181 | TP1 1.32381 | TP2 1.32481 | lots 0.01
Outcome: WIN — TP2 hit | +19.6 pips
```
This is the FIRST trade counted toward the new (post-timezone-fix) forward test.
All trades before f19c035 are archived INVALID in lessons_learned.md — do not count them.

### Timezone Fix — DONE ✅
- **f19c035** — centralized timezone handling; ORB was reading a dead bar at the wrong
  UTC offset (server is not UTC; misread "02UTC" dead-bar). Now derives offset from MT5
  server time and stamps the ORB candle explicitly.
- **fda5cc1** — explicit-timestamp ORB candle + never-zero pip value guard.
- Net effect: ORB range now marks the correct first-5-candle window at NY open.

### Active Strategy (Strategy A only)
```
Strategy A [ORB]   │ EURUSD.i + GBPUSD.i | NY open 21:30 ACST (12:00 UTC) | ORB floor 4.0 pips
Strategy B [TOKYO] │ PARKED — no edge (60d PF 0.65)
XAUUSD             │ PARKED PERMANENTLY (2026-06-12) — no viable strategy found
```
Sizing: **risk % of live balance** (DEMO 2.5%; LIVE hard cap 1.5%), lot rounded DOWN.
Pip value: **dynamic** from MT5 `trade_tick_value` (fixed prior 9× under-sizing); never zero.
Intraday exit: time-based — close at 12h held OR Friday NY-close (no weekend hold).

### ORB Execution Mode
```
Config state              │ ORB behavior
──────────────────────────┼────────────────────────────────────────────
DEMO (current)            │ signal → Discord [DEMO-DIRECT] → mt5.order_send() directly
LIVE + live_confirmed     │ signal → Discord [SIGNAL_FIRED] → Phase 6 (8 agents, ~70s)
```
Re-enable Phase 6: set `phase6.mode="LIVE"` AND `phase6.live_confirmed=true` in config.json
(two-key lock — must be set manually, never automatically).

### System Rules (trading safety — config.json)
Max risk/trade: 1.5% (LIVE) | Max daily loss: 4.0% | Max signals: 5/day
Min R:R: 2.0 | Min Veto confidence: 70% | Lot: always round DOWN
Veto is absolute — never skippable/bypassable.

### Why Gold & Tokyo S/R Are Parked
- **XAUUSD**: every "edge" was a small-sample mirage — Tokyo S/R, ORB 2R, fade,
  RSI mean-rev (H1+H4) all fail on 180d + realistic cost (best fade PF ≈ 0.99). Forex only.
- **Tokyo S/R (v7 bot)**: 60d PF 0.65; best variant (TP=3.0) only WR 77% / EV −0.04R.
- **USDBRL.i**: ORB 27.8% / RSI(2) 23.9% — both failed; needs B3/WDO direct broker.

### Pending / Watch
- [ ] Accumulate post-fix valid sample — only 1 trade (1W) so far; need a meaningful
      count before re-judging PF and considering live readiness.
- [ ] Broker live account — FP Markets / Eightcap application in progress; stay DEMO.
- [x] Timezone fix (server UTC offset + explicit ORB candle) ✅ f19c035 / fda5cc1
- [x] Forward-test reset — pre-fix trades archived INVALID, PF restart ✅ 2026-06-18
- [x] First valid post-fix trade logged — GBPUSD WIN +19.6 pips ✅ 2026-06-18
- [x] Dynamic pip value (MT5 trade_tick_value, never-zero) ✅
- [x] Intraday time-based exit (12h / Fri NY-close) ✅
- [x] Dependency-ordered launcher + At-logon auto-start ✅

### What NOT to do right now
- Do NOT fund / switch to live account — broker not approved, still DEMO.
- Do NOT count pre-f19c035 trades — they are archived INVALID.
- Do NOT re-activate XAUUSD or Tokyo S/R — both parked for lack of edge.
- Do NOT set `"mode": "LIVE"` or `"live_confirmed": true` automatically (two-key manual lock).
- Do NOT make Veto skippable/optional/bypassable.
- Do NOT relax trading-safety hard rules without explicit boss confirmation.

### Boss Profile (for Cloud reference)
Name: Notto | Adelaide, Australia (ACST = UTC+9:30)
Available: Mon–Fri from ~17:00 ACST | PRIMARY session: NY open 21:30 ACST | London 16:30 = skip
Goal: AI trading company → financial independence in Thailand
Communication: Thai (เจ้านาย) + English for code/logs
