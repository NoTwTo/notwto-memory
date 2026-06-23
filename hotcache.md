# NoTwTo Hotcache
## Generated: 2026-06-23 ACST
## Version: 3.1 (forward test 1W-1L · margin≠risk blocker found)

### Current Status
Account: **$2,946.67 AUD** (DEMO) | P&L: GBPUSD +19.6 / EURUSD −77 pips
Mode: **DEMO-DIRECT** (config.json phase6.mode = DEMO) — live account not funded
Bot: RUNNING ✅ — Strategy A loop, sleeping to next NY open 21:30 ACST (12:00 UTC)
Auto-start: At-logon launcher + Task Scheduler (dependency-ordered: MT5 → bot)
Forward test: **RESET 2026-06-18** — PF count restarts from 0 (4 pre-fix trades archived INVALID)
Record (post-fix, valid only): **1W – 1L**
Phase 6: CODE INTACT — bypassed in DEMO; re-enables when mode=LIVE + live_confirmed=true
MT5 MCP: CONNECTED ✅ | account 7942393 | Eightcap-Demo | leverage 1:30 (ASIC)
Git: main | Latest push: 1407335 (notwto-hq)

### Post-Fix Forward Test — 1W – 1L
```
2026-06-18 | GBPUSD.i LONG  | [ORB] 15UTC | WIN  — TP2  | +19.6 pips | clean ORB
????-??-?? | EURUSD.i       | [ORB]       | LOSS — SL   | −77   pips | false breakout, counter-trend
```
GBPUSD = first trade counted toward the new (post-timezone-fix) forward test.
EURUSD loss = false breakout against the prevailing trend (price broke ORB, reversed, hit SL).
All trades before f19c035 are archived INVALID in lessons_learned.md — do not count them.

### ⚠️ Live-Readiness Blocker — Margin ≠ Risk (found 2026-06-23)
- **Symptom**: risk-based sizing is **not margin-aware**. A **tight SL** → formula returns a
  **large lot** (to hold $ risk constant) → notional exceeds free margin → **2nd trade of the
  session can't open** (MT5 retcode **10019** No money / not enough margin).
- **Concept**: risk (SL-based, our 1.5%/2.5% rule) ≠ margin (notional ÷ leverage). Under
  **ASIC 1:30**, EURUSD 1.0 lot locks ≈$3,333 margin no matter how tight the SL.
- **TODO before live**: margin pre-check (`order_calc_margin` vs `margin_free`); cap lot by
  min(risk-lot, margin-lot) rounded DOWN; add SL floor; reserve headroom for concurrent signals.
- **Backlog (NOT decided, n=2 noise)**: track whether counter-trend ORB entries lose more →
  maybe add a trend filter. Tag each trade trend/counter-trend; revisit at meaningful sample.

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
- [ ] **Margin guard before live** — sizing not margin-aware (retcode 10019 on trade 2 when
      SL tight). Add margin pre-check + lot cap + SL floor. See blocker note above.
- [ ] Accumulate post-fix valid sample — only 2 trades (1W-1L) so far; need a meaningful
      count before re-judging PF and considering live readiness.
- [ ] Trend filter? (backlog, undecided) — the one loss was counter-trend; n=2 too small.
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
