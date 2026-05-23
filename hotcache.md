# NoTwTo Hotcache
## Generated: 2026-05-21 ACST
## Version: 1.7 (USDBRL parked · RSI2 backtest · deprecation fixes · bot EURUSD+XAUUSD only)

### Current Status
Account: $3,000.00 AUD (DEMO) | P&L this week: $0.00
Active: DEMO only — live account not funded | Mode: **demo_relaxed** enabled (config.json)
Last trade: None | First session: 2026-05-18 observation only (no signal — EURUSD 2.2 pip range)
Phase 6: COMPLETE (Week 1–4) ✅ — 8 agents, ~70s, smart routing, web research loop
MT5 MCP: CONNECTED ✅ — direct MT5 tool access from Claude Code
ECC Rules: INSTALLED ✅ — global agent orchestration + coding standards active
Git: INITIALIZED ✅ — all project files committed (master branch)
Cooldown active: No

### Recent Changes (v1.7)
- **_m5_candle_stream bug fixed** (commit 8db5ee0) — numpy void rows were passed raw to downstream functions that expected dicts; fixed by converting via `dtype.names`; bot no longer errors mid-session on candle data
- **USDBRL.i PARKED** — two full backtests on Eightcap data confirm no viable strategy:
  - ORB breakout: **27.8% win rate** (56 sessions, Feb–May 2026) — below 33.3% breakeven at 2:1 R:R
  - RSI(2) Mean Reversion (H1): **23.9% win rate** (278 trades / 264 decided, May 2025–May 2026)
  - Root cause: Eightcap USDBRL.i is a CFD product quoted only during US-session (15:30–22:55 UTC), not a B3/WDO direct feed; 2025-2026 BRL depreciation trend kills mean-reversion; no viable intraday strategy found
  - `USDBRL_CFG` removed from `INSTRUMENTS` list in `orb_alert_bot.py`; all code + config preserved
  - `config.json` `usdbrl_status` key added (gitignored file — manual confirmation of park reason)
  - `lessons_learned.md` updated with full post-mortem
  - Revisit only when: BRL regime changes to stable/ranging, OR switch to broker with direct B3/WDO access
- **Bot now monitors EURUSD + XAUUSD only** — USDBRL removed from active session loop
- **WDO session timing confirmed** — Eightcap does not quote USDBRL.i before 15:30 UTC; no candles at 12:00/13:00/14:00/15:00 UTC regardless of B3 WDO open; bot previously set to 13:00 UTC (2.5 h before market open) — fixed in prior version (30aea13)
- **datetime.utcfromtimestamp() deprecation fixed** — all 5 calls in `orb_alert_bot.py` replaced with `datetime.fromtimestamp(ts, tz=UTC)`; also fixed in backtest scripts
- **USDBRL wiki fully updated** — `_Shared/wiki/usdbrl_wiki.md` contains complete research history, session timing confirmation, ORB backtest, RSI(2) backtest, all 5 alternative strategy analyses, and park decision

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
- [ ] Log first real signal in raw/trade_journal.md after first execution
- [ ] Create raw/market_observations.md (human task — log daily ORB range observations)
- [ ] Create raw/agent_decisions.md (human task — first Veto decision triggers this)
- [ ] Decide M5 vs M15 (needs 10+ observation sessions of demo data)
- [ ] Research: backtest 5m ORB on EUR/USD 2022–2025 with NoTwTo exact rules (Sage task)
- [ ] Validate demo_relaxed: track how many sessions fire under relaxed gates vs standard — reassess after 10 sessions
- [ ] A/B track XAUUSD ORB_ATR_GATED vs ASIAN_SWEEP performance over 20+ signals (Sage)
- [ ] USDBRL: source B3/WDO direct-access broker (local Brazilian broker or B3 DMA) before revisiting
- [x] USDBRL.i parked — ORB (27.8%) + RSI(2) (23.9%) both failed; removed from bot ✅ COMPLETE (2026-05-21)
- [x] RSI(2) H1 backtest on USDBRL.i (278 trades, 1-year data) ✅ COMPLETE (2026-05-21)
- [x] datetime.utcfromtimestamp() deprecation fixed in orb_alert_bot.py ✅ COMPLETE (2026-05-21)
- [x] _m5_candle_stream numpy void → dict fix ✅ COMPLETE (2026-05-20, commit 8db5ee0)
- [x] ORB breakout detection fixed (M5 candle close) ✅ COMPLETE (2026-05-20)
- [x] XAUUSD parallel strategies (ORB_ATR_GATED + ASIAN_SWEEP) ✅ COMPLETE (2026-05-20)
- [x] Strategy selector in config.json (xauusd.strategies) ✅ COMPLETE (2026-05-20)
- [x] Backtest baseline recorded (EURUSD 84% / XAUUSD 54% / Combined 69%) ✅ COMPLETE (2026-05-20)
- [x] Sage XAUUSD research → xauusd_strategies.md ✅ COMPLETE (2026-05-20)
- [x] First observation session ✅ COMPLETE (2026-05-18 — EURUSD 2.2 pip, no signal)
- [x] demo_relaxed gates (4 gates) ✅ COMPLETE (orb_alert_bot.py)
- [x] Git initialized + all files committed ✅ COMPLETE (master)
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
