# NoTwTo Hotcache
## Generated: 2026-05-25 ACST
## Version: 1.8 (Strategy B dual-direction · BUY at support · multi-entry tracking · backtest scripts)

### Current Status
Account: $3,000.00 AUD (DEMO) | P&L this week: $0.00
Active: DEMO only — live account not funded | Mode: **demo_relaxed** enabled (config.json)
Last trade: None | Strategy A (ORB) + Strategy B (Tokyo S/R) both active
Phase 6: COMPLETE (Week 1–4) ✅ — 8 agents, ~70s, smart routing, web research loop
MT5 MCP: CONNECTED ✅ — direct MT5 tool access from Claude Code
ECC Rules: INSTALLED ✅ — global agent orchestration + coding standards active
Git: main branch | Latest push: 5bedb38
Cooldown active: No

### Recent Changes (v1.8)
- **Strategy B BUY direction fully active** (commit 5bedb38) — channel trading now bidirectional:
  - SELL: M5 high touches resistance (±1.5 pts) + bearish pattern (shooting star / bearish engulfing)
  - BUY: M5 low touches support (±1.5 pts) + bullish pattern (hammer / bullish engulfing / pin bar)
  - Each direction waits M1 confirm (bar close beyond M5 high/low) before entry
  - Max 3 entries total per session (SELL+BUY combined); each has independent TP/SL
  - BUY: TP = resistance, SL = 12.5 pts below entry
  - SELL: TP = support, SL = 12.5 pts above entry
- **Multi-entry tracking fixed** — replaced single `open_entry` var with `open_entries` list; previously a 2nd signal would silently overwrite the 1st before it resolved
- **`_tokyo_sr_trades` now populated** — every entry written on take; result updated (win/loss) when TP/SL hit or at session end (conservative loss); terminal summary now shows all entries correctly
- **Terminal summary format** (example):
  ```
  ✅ SELL ~4574 | TP hit +6.4 pts | Pattern: [shooting_star]
  ✅ BUY  ~4566 | TP hit +8.1 pts | Pattern: [hammer]
  ❌ NO SIGNAL — No touch signal (channel: sup=4550.00 res=4574.00 width=24.0pts)
  ```
- **Backtest/audit scripts added** (commit 5a3523e, `_Config/`):
  - `strategy_backtest.py` — 4-strategy comparison (Silver Bullet, London ORB, Asian Range, EMA trend) — 60-day
  - `smc_ob_backtest.py` — SMC Order Block H4+M5 — 60-day EURUSD
  - `london_orb_audit.py` — London ORB audit GBPUSD.i + USDJPY.i — 60-day
  - `orb_pipeline_audit.py` — deterministic 8-agent pipeline filter replay — 30-day (no Claude API)
- **Tokyo S/R window**: 02:00–03:30 UTC (11:30–13:00 ACST); pre-scan 01:45 UTC; XAUUSD only

### XAUUSD Strategy Summary (code reference)
```
Strategy          | Trigger                          | Filter           | Thread
──────────────────|──────────────────────────────────|──────────────────|──────────
ORB_ATR_GATED     | M5 candle close beyond ORB H/L   | ATR > threshold  | Thread 1
ASIAN_SWEEP       | Sweep of Asian H or L + reversal | NY session only  | Thread 2
```
Config key: `xauusd.strategies` array in `config.json`
Discord tag: `[ORB_ATR_GATED]` or `[ASIAN_SWEEP]` prefix on each embed

### demo_relaxed Gate Summary (code reference)
```
Gate                  | Standard          | demo_relaxed ON
──────────────────────|───────────────────|──────────────────────────────
Min ORB floor         | 2.0 pips          | 1.0 pip (_DEMO_MIN_ORB_PIPS)
Spread limit EURUSD   | 1.5 pips          | 2.5 pips (demo_max_spread_pips)
Nova BLOCK            | suppress signal   | WARN label, proceed
Nova CONFLICT         | suppress signal   | CONFLICT label, keep ORB dir
```
config.json key: `"demo_relaxed": true` (gitignored — manual toggle only)

### Backtest Baseline (all real MT5 data)
| Instrument | Strategy         | Win Rate | Trades | Period         | Source |
|------------|------------------|----------|--------|----------------|--------|
| EURUSD     | ORB (M5)         | 84%      | ~30    | 30-day         | xauusd_strategies.md |
| XAUUSD     | ORB (M5)         | 54%      | ~30    | 30-day         | xauusd_strategies.md |
| USDBRL.i   | ORB (M5)         | 27.8%    | 50     | Feb–May 2026   | usdbrl_wiki.md — FAILED |
| USDBRL.i   | RSI(2) MR (H1)   | 23.9%    | 264    | May 2025–2026  | usdbrl_wiki.md — FAILED |

### Wiki State
- [[usdbrl_wiki]]: COMPLETE — full research + 2 backtests + park decision
- [[xauusd_strategies]]: ATR-Gated ORB + Asian Sweep research, A/B framework, backtest baseline
- [[patterns_wiki]]: 4 strategies documented — 5m ORB ★★★★★ (primary), ICT Silver Bullet, EMA, SMC
- [[agents_wiki]]: Veto, Atlas, Shield, Sage, Cloud frameworks populated
- [[performance_wiki]]: Baseline metrics, 0 trades, research benchmarks
- [[index]] + [[log]]: Up to date

### Confirmed Core Setup (EURUSD — unchanged)
```
Session: NY open 21:30 ACST (PRIMARY) | London 16:30 ACST (secondary — only if home early)
Range:   Mark H/L of first 5 candles
Confirm: ORB break (M5 candle close) + FVG on M5 + liquidity sweep
Entry:   Pullback into FVG in break direction
SL:      Below swing low of sweep
TP1:     1:1 R:R | TP2: 2:1 R:R minimum
Filter:  No red-folder news ≤10 min before / ≤15 min after | spread ≤1.5 pip (≤2.5 pip demo) | confluence ≥ 4/7 (Demo Aggressive)
```

### Current Market Context
Demo observation phase underway. First session (2026-05-18): EURUSD 2.2 pip ORB — no breakout, no signal.
Boss available from ~17:00 ACST (day job). London open (16:30) is too early — skip.
NY open (21:30 ACST) is PRIMARY session.

### System Rules Reminder
Risk: 1.5% max ($45 AUD on $3k) | Min confidence: 70% | R:R: 2:1 min | Cooldown: 120min after loss
Max signals: 3/day | Leverage: 30:1 | No news ≤10 min before / ≤15 min after | Pairs: EUR/USD primary

### Pending Items
- [ ] Run backtest scripts (strategy_backtest.py, smc_ob_backtest.py, london_orb_audit.py, orb_pipeline_audit.py) — need MT5 connected
- [ ] Log first real signal in raw/trade_journal.md after first execution
- [ ] Create raw/market_observations.md (human task — log daily ORB range observations)
- [ ] Validate demo_relaxed: track how many sessions fire under relaxed gates vs standard — reassess after 10 sessions
- [ ] A/B track XAUUSD ORB_ATR_GATED vs ASIAN_SWEEP performance over 20+ signals (Sage)
- [ ] USDBRL: source B3/WDO direct-access broker before revisiting
- [x] Strategy B BUY direction fully active — dual SELL+BUY channel trading ✅ COMPLETE (2026-05-25, commit 5bedb38)
- [x] _tokyo_sr_trades population + multi-entry tracking fix ✅ COMPLETE (2026-05-25)
- [x] Terminal summary shows all entries per session ✅ COMPLETE (2026-05-25)
- [x] Backtest/audit scripts × 4 added to _Config/ ✅ COMPLETE (2026-05-25)
- [x] USDBRL.i parked — ORB (27.8%) + RSI(2) (23.9%) both failed; removed from bot ✅ COMPLETE (2026-05-21)
- [x] datetime.utcfromtimestamp() deprecation fixed ✅ COMPLETE (2026-05-21)
- [x] _m5_candle_stream numpy void → dict fix ✅ COMPLETE (2026-05-20, commit 8db5ee0)
- [x] ORB breakout detection fixed (M5 candle close) ✅ COMPLETE (2026-05-20)
- [x] XAUUSD parallel strategies (ORB_ATR_GATED + ASIAN_SWEEP) ✅ COMPLETE (2026-05-20)
- [x] Backtest baseline recorded (EURUSD 84% / XAUUSD 54%) ✅ COMPLETE (2026-05-20)
- [x] First observation session ✅ COMPLETE (2026-05-18 — EURUSD 2.2 pip, no signal)
- [x] orb_alert_bot → phase6_pipeline.py handoff ✅ COMPLETE (2026-05-16)
- [x] Phase 6 Python + MT5 + Anthropic API architecture ✅ COMPLETE
- [x] MT5 MCP server connected + 4 priorities verified ✅ COMPLETE (2026-05-18)

### What NOT to do right now
- Do NOT fund live account — demo phase not yet started
- Do NOT trade outside London open (16:30) or NY open (21:30) windows
- Do NOT execute without Veto APPROVED (≥70% confidence, ≥4/7 confluence in demo_relaxed)
- Do NOT use ICT Silver Bullet or EMA as primary setups — 5m ORB only until proven
- Do NOT trade USD/BRL until switched to B3/WDO direct broker
- Do NOT skip journaling — every trade logged in raw/trade_journal.md
- Do NOT go live before 20+ demo trades with consistent discipline score
- Do NOT set `"mode": "LIVE"` or `"live_confirmed": true` in config.json automatically
- Do NOT disable XAUUSD strategy threads independently without updating config.json xauusd.strategies

### Boss Profile (for Cloud reference)
Name: Notto | Adelaide, Australia (ACST = UTC+9:30)
Available: Mon–Fri from ~17:00 ACST | PRIMARY session: NY open 21:30 ACST | London open 16:30 = skip (too early)
Goal: AI trading company → financial independence in Thailand
Communication: Thai + English for logs
