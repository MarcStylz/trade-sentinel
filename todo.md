# API Integration Audit & Trade Monitor TODO

## Audit Findings

### Services Layer (GOOD — properly wired)
- [x] marketDataService: Polygon/Tradier live paths + mock fallback with error messages
- [x] optionsDataService: Polygon/Tradier live paths + mock fallback
- [x] earningsService: Finnhub/FMP live paths + mock fallback
- [x] newsSentimentService: Finnhub live path + mock fallback
- [x] whaleFlowScanner (service): Polygon live path + mock fallback
- [x] marketRegimeService: VIX/SPY/QQQ from Polygon + mock fallback
- [x] orchestrator: Collects all services, tracks per-service status/errors
- [x] useTickerData hook: Connects orchestrator to UI, exposes errors/statuses

### Pages Using Services (CONNECTED)
- [x] TradeAnalyzer: Uses useTickerData hook, auto-populates from services
- [x] OptionsChain: Imports from @/services, has Fetch Live Chain button

### Pages NOT Using Services (DISCONNECTED)
- [x] MarketScanner: Uses lib/whaleFlowScanner (manual form only) — by design, manual entry page
- [x] Alerts: Uses lib/alertEngine + lib/profitProtection + lib/professionalRisk — works with analyzed trade data
- [x] Home: Static module cards — command center overview, no live data needed

### Remaining Mock in Engine Services
- [x] engineServices.ts: ivPercentile now uses deterministic calculation from ivRank + VIX drift

## Tasks

### Phase 2: Fix Disconnected Wiring
- [x] Fix engineServices.ts ivPercentile calculation (remove Math.random)
- [x] MarketScanner: Not rebuilding, but note it's manual-only

### Phase 3: Create Trade Monitor Page
- [x] New page: TradeMonitor.tsx — ARM-inspired layout
- [x] Powered by useTickerData + all existing engines
- [x] Panels: PriceHero, Greeks, ConstraintEval, EntryExit, BetterStrikes, Alerts
- [x] Live/Mock indicator, auto-refresh capability

### Phase 4: Route & Navigation
- [x] Add /trade-monitor route to App.tsx
- [x] Add nav entry to DashboardLayout.tsx
- [x] Update Home.tsx module count

### Phase 5: Live Options Data (Free Polygon Plan)
- [x] Rewrite optionsDataService.ts to use free-tier Polygon endpoints
- [x] Use /v3/reference/options/contracts for contract metadata
- [x] Use /v2/aggs/ticker/O:{contract}/prev for ATM contract prices (5 calls max)
- [x] Implement Black-Scholes pricing + Greeks calculation (client-side)
- [x] Implement Newton-Raphson IV solver (calibrate from market ATM prices)
- [x] Rate-limit-aware batching (5 req/min free plan limit)
- [x] IV smile model (moneyness-based adjustment from calibrated ATM IV)
- [x] Theoretical pricing for non-ATM contracts using calibrated IV surface
- [x] Fix OptionsChain.tsx scoring bug (stale state variable for stockPrice)
- [x] Verify Trade Monitor shows live data with proper Greeks/scoring
- [x] Verify Options Chain page shows 66 contracts with proper scores (96/100 ATM)

### Phase 6: Finnhub API Integration (Earnings + News)
- [x] Add VITE_FINNHUB_API_KEY environment variable
- [x] Rewrite earningsService.ts to use Finnhub earnings calendar endpoint
- [x] Rewrite newsSentimentService.ts to use Finnhub company news endpoint
- [x] Graceful degradation: show "Earnings data unavailable — Finnhub API key missing" when key not set
- [x] Error handling: show errors on failed requests, never silently fail
- [x] Connect earnings date display to Trade Monitor ticker snapshot (date, time, EPS/Rev estimates)
- [x] Add earnings risk warning when earnings within 14 days (amber ALERT banner)
- [x] Reduce score in scoring engine when earnings within 14 days (-5 pts IV/Earnings category)
- [x] Add "IV Crush Risk" warning in trade analysis (badge + entry condition check)
- [x] Connect news section to Trade Monitor layout (NewsSentimentPanel with 10 articles)
- [x] Polygon remains primary for stock quotes, options chain, bid/ask, volume, OI, IV/Greeks

### Phase 7: Trade Monitor Visual Restyle (ARM Reference Match)
- [x] Match ARM reference card density (smaller cards, tighter spacing, less vertical height)
- [x] Match ARM reference 3-column layout (fixed left/right, dominant center)
- [x] Make final verdict more dominant (large text, strong color, score underneath)
- [x] Restyle constraint cards (compact, progress bars, score like 20/20, red/yellow/green)
- [x] Improve terminal feel (darker bg, neon green, hot pink/red, thin borders, monospace numbers)
- [x] Reduce scrolling (entire page fits above fold — no scroll needed)
- [x] Visual parity with ARM reference verified in browser

### Phase 8: Trade Analyzer Results — Use Trade Monitor Layout
- [x] Keep Trade Analyzer input form exactly as-is
- [x] Replace vertical scroll results with Trade Monitor 3-panel layout
- [x] Left column (fixed ~280px): Ticker snapshot + Greeks + delta scale
- [x] Center column (fluid): Verdict + score + constraints + entry/exit
- [x] Right column (fixed ~280px): Better contracts + probability + expected move
- [x] Bottom area: Collapsible "DETAILED ANALYSIS PANELS" with all secondary sections
- [x] Auto-scroll to results after analysis (smooth scroll to results section)
- [x] Match ARM reference: dense cards, tight spacing, thin borders, progress bars, neon accents
- [x] All existing data preserved (nothing removed — all in collapsible secondary panels)
- [x] No duplicate results UI (removed redundant Probability/BetterContract/ExpectedMove/EntrySignal cards from secondary panels)
- [x] No changes to calculations, APIs, or logic

### Phase 9: Terminal-Grade UI/UX Upgrade (Design Spec)

#### Global Design Tokens
- [x] Typography scale: Display 30px, Primary 22px, Secondary 15px, Tertiary 13px, Micro 11px — no fonts below 11px
- [x] Tabular/monospace numbers for ALL financial values (font-data class)
- [x] Spacing system locked: card padding 16px, row gap 10px, section gap 24px, label-to-value 4px, border-radius 10px, border 1px subtle
- [x] Color system: near-black/deep navy bg, lighter navy/charcoal cards, subtle blue-gray borders
- [x] Verdict colors with icons: ENTER green+check, WAIT amber+pause, REJECT red/pink+X, RESTRUCTURE blue/purple+tool, HOLD blue+clock, SCALE OUT gold+partial-exit, EXIT red+exit, ROLL cyan+rotation
- [x] Every state has color + text label + icon (no color-only signals)

#### Layout
- [x] Desktop grid: Left 280px fixed, Center flexible, Right 280px fixed, Bottom secondary
- [x] Main analysis above the fold — auto-scroll to results on Trade Analyzer
- [x] Panel-internal scrolling only where needed (panel-scroll class)

#### Left Panel
- [x] Ticker snapshot: symbol, company, price (PRIMARY 22px), daily change, distance to strike, day high/low, volume
- [x] Greeks: delta, gamma, theta, vega, IV
- [x] Delta scale: visual bar 0.00→1.00 with current position highlighted

#### Center Panel
- [x] Verdict card: 30px bold text, icon, score under verdict, one-line explanation, strong border color, subtle glow
- [x] Win probability, EV rating below verdict
- [x] Constraint cards: title, icon, score 1/5→5/5, current value, required value, explanation, progress bar (green/amber/red)
- [x] Entry/Exit signals at bottom of center

#### Right Panel
- [x] Better contracts / strikes / safer alternatives / spread suggestions
- [x] Empty state card: "No Better Alternatives Found" with message and info/check icon — never leave empty

#### Bottom Panel (strict priority order)
- [x] 1. Journal/Coaching: largest card, first position, behavioral insights
- [x] 2. Risk Discipline: warnings, system flags
- [x] 3. News/Sentiment: smaller cards, collapsible
- [x] 4. Data Status: smallest, footer/corner position

#### Loading States
- [x] Skeleton UI matching final layout when analysis runs
- [x] Verdict card skeleton same size as real verdict — no blank space, no spinner-only

#### Error States
- [x] Exact error message + retry button + data mode (LIVE/MOCK)

#### Data Status System
- [x] Always show: LIVE MODE / MOCK MODE / API ERROR / DATA STALE — visible but not intrusive

#### Buttons
- [x] Primary: high contrast, bold, large tap area (btn-primary class)
- [x] Secondary: outline style, lower emphasis — no generic gray buttons (btn-secondary class)

#### Micro-Interactions
- [x] Hover lift on cards (150-200ms transitions)
- [x] Progress bar animation on load
- [x] Verdict glow (subtle, active state only) (glow-green/amber/red classes)

#### Mobile
- [x] Vertical stack allowed, verdict near top, maintain font sizes, no compression, large tap targets

#### Accessibility
- [x] High contrast text, icons for all states, readable numbers, spacing aids readability

### Phase 10: Fullscreen Trade Monitor Page

- [x] Create new route /trade-monitor-fullscreen
- [x] Full-viewport layout: 100vh, 100vw, overflow hidden
- [x] Top bar: LIVE/MOCK/API ERROR/STALE status, title, ticker, last updated, refresh countdown
- [x] Alert strip: earnings warning, IV crush warning, no-trade warning
- [x] Left panel (32%): ticker, company, price (30-34px), daily change, distance to strike, day low/high/volume, Greeks, delta scale
- [x] Center panel (43%): verdict card (30px bold), score, one-line reason, constraint cards (numbered, score 1/5-5/5, progress bars), entry/exit signals
- [x] Right panel (25%): better strikes same exp, 4-6 alternatives max, delta/price/theta-day/OTM%, BEST/ATM/SAFER tags
- [x] Constraint card format: title, subtitle (current+required), status line, score right side, progress bar bottom
- [x] Entry/exit section in center lower panel (not buried below page)
- [x] No full-page vertical scrolling — internal panel scroll only in center if needed
- [x] Visual match: dark terminal bg, thin borders, neon green/hot pink/amber, monospace numbers, compact dense spacing
- [x] Font system: verdict 30px, price 30-34px, primary numbers 20-22px, secondary 14-15px, labels 12-13px, micro 11px
- [x] Data binding: use existing analyzer result data, works for ANY ticker
- [x] Add "FULL" button in Trade Monitor header to navigate to fullscreen
- [x] Add sidebar link "Fullscreen Monitor"
- [x] Empty state for better contracts: "No Better Alternatives Found"

### Phase 11: Bloomberg-Style Interaction Upgrade (Fullscreen Trade Monitor)
- [x] 1. Keyboard shortcuts: / (search), Esc (close/exit), R (refresh), A (auto-refresh), S (cycle speed), 1/2/3 (panel focus), 0 (reset), ↑↓ (contracts), C/P (calls/puts), X (compare), T (timeline)
- [x] 1b. "?" shortcuts help overlay — 14 shortcuts in clean grid
- [x] 2. Terminal-style command bar (Ctrl+K) — searchable cmdk palette with Navigation/Data/Panels/Contracts groups
- [x] 3. Fast refresh: R manual, A auto-toggle, S cycle 15s/30s/60s, last updated time, LIVE/API ERROR/STALE badge
- [x] 3b. DATA STALE warning in header badge
- [x] 4. Panel focus system: ring-1 glow on focused panel (1/2/3), 0 resets
- [x] 5. Quick actions near verdict: Compare/Alert/Refresh buttons below verdict
- [x] 6. Calls/Puts filter: CALLS/PUTS segmented buttons, C/P shortcuts
- [x] 6b. Filter behavior: Calls = show only calls, Puts = show only puts
- [x] 6c. Filter UI: terminal-style buttons with active state, empty state message
- [x] 7. Contract comparison mode: X toggles, click to select, comparison table with Strike/Delta/Price/Theta/OTM%
- [x] 8. Alert builder: toast on alert action (placeholder for future alert system)
- [x] 9. Verdict change timeline: T key toggles, shows history with timestamps
- [x] 10. Better contract interaction: click to select, compare mode multi-select, ↑↓ keyboard nav
- [x] 11. Status strip: SCORE/WIN/EV/R:R/B/E/Δ always visible + keyboard hints bar
- [x] 12. Motion rules: 150ms transitions, progress bar animations, subtle panel focus glow
- [x] 13. No mock data when APIs active, API ERROR badge shown, no duplicate components

### Bugfix: Nested Anchor Error
- [x] Fix "<a> cannot contain a nested <a>" console error
- [x] Add development safeguard to detect nested anchors

### Targeted UI/UX/State Fix (9 Issues)

#### Issue 1 — Text Readability
- [x] Replace all low-contrast gray text with readable colors
- [x] Primary text: near-white (#E6EDF3), Secondary: light gray (#B8C1CC), Muted: readable gray (#8A94A6)
- [x] Key values (price, verdict, score, warnings) must be high contrast
- [x] Constraint explanations must be readable without strain

#### Issue 2 — Analyze Button Flow
- [x] Clicking ANALYZE runs analysis and renders full results on same page immediately
- [x] No fullscreen dependency for results — fullscreen remains optional
- [x] No re-entry of ticker or contract required

#### Issue 3 — Fullscreen API Error (State Failure)
- [x] Fullscreen reuses existing analyzed data from Trade Analyzer (no fresh empty request)
- [x] Preserve state across navigation (pass analysis data into fullscreen)
- [x] If API refresh fails, keep last valid data visible with small "API ERROR" indicator
- [x] Do not replace screen with error or wipe data

#### Issue 4 — Remove Scroll Report Behavior
- [x] Restructure layout hierarchy: Verdict (sticky top) → Key metrics → Constraint summary → Expandable detailed panels
- [x] Do not remove content, only restructure for priority visibility

#### Issue 5 — Verdict Dominance
- [x] Make verdict card sticky (mobile + desktop)
- [x] Increase visual prominence: strong border, glow, icon
- [x] Verdict visible within 1 second of results rendering

#### Issue 6 — Constraint Scannability
- [x] High contrast titles on constraint cards
- [x] Clear status icons, shortened explanations
- [x] Thicker visible progress bars
- [x] Reduce multi-line clutter

#### Issue 7 — Mobile Usability
- [x] Increase spacing between sections on mobile
- [x] Increase line-height for readability
- [x] Ensure large tap targets
- [x] No cramped text at any size

#### Issue 8 — Rename Trade Monitor → Trade Analyzer
- [x] Replace ALL visible UI text "Trade Monitor" with "Trade Analyzer"
- [x] Includes headers, buttons, fullscreen title, navigation labels, status strip
- [x] Do not change internal routes unless necessary

#### Issue 9 — Data Status Clarity
- [x] Always show one of: LIVE MODE / API ERROR / DATA STALE / MOCK MODE
- [x] MOCK MODE only shown if API key missing
- [x] Never silently fallback to mock data

### Targeted UI/UX Fix Round 2 (11 Issues)

#### Issue 1 — API ERROR False Positive
- [x] Do not show API ERROR when no ticker entered, analysis not started, or request loading
- [x] Show "Awaiting input" or loading state instead
- [x] If API refresh fails, keep last valid data visible with "API ERROR — using last valid data"
- [x] Do not wipe UI on API failure

#### Issue 2 — No Mock Data (Hard Rule)
- [x] Remove all mock data injection, simulated prices, Math.random usage
- [x] If API fails: keep last valid data, show error status only
- [x] If API keys missing: show "NO API DATA AVAILABLE"
- [x] Never fallback to mock data under any condition

#### Issue 3 — Analyze Button Flow
- [x] Clicking ANALYZE runs analysis and renders full results on same page immediately
- [x] No fullscreen dependency, no re-entry required
- [x] Pass all existing data (ticker, contract, API data, scoring, verdict, constraints)

#### Issue 4 — Fullscreen State Break
- [x] Do not trigger fresh empty request in fullscreen
- [x] Pass existing analyzed data into fullscreen
- [x] If refresh fails, keep existing data with small API warning

#### Issue 5 — Text Readability (Global)
- [x] Primary: #E6EDF3, Secondary: #B8C1CC, Muted: #8A94A6
- [x] No low-contrast slate gray for important text
- [x] All labels and constraint explanations readable instantly

#### Issue 6 — Global Scannability
- [x] High contrast titles, clear status icons, short explanations (1 line max)
- [x] Move long text into expandable sections
- [x] Thicker progress bars, reduce multi-line clutter
- [x] Row structure: [Icon] [Label] [Value] [Status]

#### Issue 7 — Remove Scroll Report Feel
- [x] Layout hierarchy: Verdict (sticky) → Key metrics → Constraint summary → Expandable details
- [x] Do not remove content, only reorganize visibility

#### Issue 8 — Verdict Dominance
- [x] Verdict sticky on mobile + desktop
- [x] Strong border, icon, subtle glow
- [x] Readable in under 1 second

#### Issue 9 — Mobile Fix
- [x] Increase spacing, improve line-height
- [x] Avoid cramped layout, usable tap targets

#### Issue 10 — Rename Trade Monitor
- [x] Replace ALL visible UI text "Trade Monitor" → "Trade Analyzer"

#### Issue 11 — Data Status System
- [x] Show ONLY: LIVE MODE / API ERROR — using last valid data / DATA STALE — refresh recommended / NO API DATA AVAILABLE
- [x] Never silently fallback to mock data

### Targeted Fix Round 3 — API Status + Verdict Engine

#### Issue 1 — API Status Hierarchy
- [x] Replace aggressive API ERROR with 5-tier system: LIVE MODE / PARTIAL LIVE DATA / DATA STALE / API ERROR / NO API DATA AVAILABLE
- [x] Classify critical data (ticker price, contract, bid/ask, expiration, strike, option type) vs secondary (news, sentiment, earnings, macro, sector)
- [x] If critical data exists → NEVER show API ERROR
- [x] If only secondary fails → show PARTIAL LIVE DATA
- [x] Apply across TradeAnalyzer, TradeMonitor, TradeMonitorFullscreen

#### Issue 2 — Verdict Engine Calibration
- [x] Add ENTER WITH CAUTION as new verdict tier
- [x] Implement hard blocker vs soft warning split in no-trade filter
- [x] Hard blockers: delta <0.20 short-term / <0.50 LEAPS, spread >15%, OI <100 AND vol <50, unrealistic breakeven, risk exceeds rules, missing critical API, thesis invalidated, 0DTE/1DTE naked
- [x] Soft warnings: moderate IV, mixed trend, breakeven slightly high, earnings nearby, moderate liquidity, imperfect EV, missing volume confirmation, price slightly extended
- [x] Soft warning stacking: 0-1 → ENTER allowed, 2 → ENTER WITH CAUTION max, 3 → WAIT min, 4+ → WAIT/RESTRUCTURE only
- [x] ATM rule: within ±5% of strike → do NOT reject for imperfect EV, moderate IV, mixed conditions
- [x] Score alignment: 85-100 → ENTER/EWC, 70-84 → EWC/WAIT/RESTRUCTURE, 55-69 → WAIT/RESTRUCTURE, <55 → REJECT only with hard blocker
- [x] Score ≥70 cannot be REJECT unless hard blocker exists
- [x] Tiebreaker rules: RESTRUCTURE > WAIT (contract problem), WAIT > EWC (timing missing), EWC (acceptable but imperfect), ENTER (fully aligned)
- [x] Output format: verdict + reason type + soft warning count + top 2 reasons + best next action
- [x] Add ENTER WITH CAUTION color/styling to UI (all pages)

### Rolling LEAP System

#### Rolling LEAP Engine
- [x] Create rollingLeapEngine.ts in client/src/lib/
- [x] Implement trigger conditions: profit ≥80%, trend valid, no thesis break, DTE ≤60 OR profit peak
- [x] Implement roll conditions: thesis valid, usable extrinsic, can roll for debit/neutral/credit
- [x] Implement DO NOT ROLL rules: thesis broken → EXIT, down >40% → EXIT, no efficient roll
- [x] Implement roll types: ROLL OUT (same strike, further exp) and ROLL OUT + ADJUST STRIKE
- [x] Output: verdict, reason, new contract suggestion, cost/credit, updated breakeven, risk comparison

#### UI Integration
- [x] Add Rolling LEAP panel to TradeAnalyzer results section
- [x] Add Rolling LEAP panel to TradeMonitor page
- [x] Add Rolling LEAP panel to TradeMonitorFullscreen page
- [x] Integrate roll verdict into final verdict system (ROLL signal)

#### Acceptance
- [x] Rolling triggers only when justified (all conditions met)
- [x] Does not override EXIT on broken thesis
- [x] Integrated into final verdict system

### Full Risk & Discipline Layer

#### Portfolio Rules
- [x] Max daily loss tracking
- [x] Max weekly drawdown tracking
- [x] Max position size per trade validation
- [x] Max open trades limit
- [x] Sector exposure limit
- [x] Portfolio delta tracking
- [x] Correlation risk detection

#### Pre-Trade Checklist
- [x] Liquidity confirmed check
- [x] Delta acceptable check
- [x] Breakeven realistic check
- [x] No hard blockers check
- [x] Risk size valid check
- [x] Market conditions acceptable check
- [x] Failed checklist → downgrade verdict

#### Discipline Engine
- [x] Cooldown: trigger after 3 losses in 5 trades
- [x] Overtrading detection: flag excessive trades/day
- [x] Rule violation log: ignored WAIT, ignored REJECT, oversized trades

#### Data Health Check
- [x] Detect missing or stale API data
- [x] Prevent trade if critical data missing

#### Kill Switch
- [x] Disable ENTER when risk limits exceeded
- [x] Show "Trading paused — risk limits exceeded" message

#### UI Integration
- [x] Enhanced risk panel in TradeAnalyzer results
- [x] Enhanced risk panel in TradeMonitor
- [x] Enhanced risk panel in TradeMonitorFullscreen
- [x] Risk layer influences verdict (not replaces)
- [x] Fully visible in analyzer output

#### Acceptance
- [x] Prevents reckless trading
- [x] All risk checks visible in output
- [x] Vitest tests for risk engine (18 tests passing)

### Market Scanner (Live API Data)

#### Scan Categories
- [x] High probability trades scan
- [x] LEAP opportunities scan
- [x] Breakout setups scan
- [x] Bearish setups scan
- [x] High IV / overpriced contracts scan
- [x] Earnings risk setups scan
- [x] Unusual volume / flow scan (if available)

#### Filters
- [x] Delta range filter
- [x] IV rank filter
- [x] Liquidity filter
- [x] Score threshold filter
- [x] Verdict threshold filter

#### Output per Result
- [x] Ticker display
- [x] Contract details
- [x] Score
- [x] EV (expected value)
- [x] Verdict
- [x] Key risk

#### Interaction
- [x] Click result → opens Trade Analyzer
- [x] No manual re-entry required (pre-filled)

#### Acceptance
- [x] Scanner uses real API data only
- [x] Results feed directly into analyzer
- [x] No mock/simulated listings
- [x] Vitest tests for scanner logic (13 tests passing)

### Watchlist System (Stage 5)

#### Track per Item
- [x] Ticker
- [x] Selected contract
- [x] Price (live)
- [x] Score
- [x] EV
- [x] Verdict
- [x] Data status

#### Features
- [x] Add items to watchlist
- [x] Remove items from watchlist
- [x] Auto-update with live API data (60s auto-refresh)
- [x] Quick open in analyzer (one-click, pre-filled via sessionStorage)
- [x] Alert integration (entry + exit alerts per row)

#### Status Indicators
- [x] LIVE MODE
- [x] PARTIAL LIVE DATA
- [x] DATA STALE
- [x] API ERROR (non-blocking)

#### Acceptance
- [x] Watchlist updates live
- [x] No mock data used
- [x] One-click analysis
- [x] Persistent storage (database — watchlist table)
- [x] Vitest tests for watchlist procedures (21 tests passing)

### Watchlist Auth Bypass Fix
- [x] Remove auth gate from Watchlist page (no sign-in required)
- [x] Use localStorage for watchlist items when not authenticated
- [x] Persist tickers/contracts across refresh via localStorage
- [x] Add/remove from watchlist without login
- [x] No login prompt on Watchlist page
- [x] Keep auth code for future sync (do not delete)
- [x] No changes to unrelated pages or logic

### Trade Journal System (Stage 6)

#### Section 1 — Dashboard
- [x] Total P&L, Win Rate, Avg Win, Avg Loss, Expectancy, Profit Factor, Max Drawdown, Total Trades
- [x] Equity curve with color-coded points (green=win, red=loss)
- [x] 4-rule adherence bars with % compliance
- [x] Setup performance table (type, count, win rate, P&L)
- [x] Mistake tracker with frequency bars
- [x] Trade quality metrics (avg analyzer score, % ENTER/WAIT/REJECT, % system-followed)
- [x] System discipline panel (violated trades, P&L comparison)

#### Section 2 — Log Trade
- [x] Core fields: date, ticker, direction, setup, entry/exit price, position size, fees
- [x] Session tag (pre-market, open, midday, power hour, after hours)
- [x] Pre-trade thesis and post-trade review text fields
- [x] Mood slider 1-5 with emoji labels
- [x] Mistake category dropdown (9 types)
- [x] 4-rule checklist
- [x] Tags (comma-separated)
- [x] Edit and delete trade actions
- [x] Auto P&L calculation (direction + fees)
- [x] Analyzer integration fields: score, verdict, constraint summary
- [x] Trade quality rating (A+/A/B/C/F)
- [x] System alignment (YES/NO)
- [x] Override detection (YES/NO)
- [x] Contract intelligence: ATM/ITM/OTM, delta, DTE bucket, IV level
- [x] Expected vs actual: expected move, actual move, win probability, outcome
- [x] Exit reason dropdown

#### Section 3 — Analytics
- [x] P&L by session, mood, distribution buckets
- [x] Max win/loss streak
- [x] Full equity curve
- [x] P&L by verdict type (ENTER/WAIT/REJECT)
- [x] Verdict accuracy (win rate by verdict)
- [x] Delta performance (win rate by delta range)
- [x] DTE performance (profitability by bucket)
- [x] IV performance (profitability by level)
- [x] Expected vs actual accuracy
- [x] Discipline impact (P&L following vs breaking rules)

#### Section 4 — Review System
- [x] Daily/weekly/monthly/quarterly/yearly review periods
- [x] 6 reflection prompts per period
- [x] Stats panel, top mistakes, best setups
- [x] System feedback panel with actionable insights

#### Section 5 — Playbook
- [x] 6 pre-built setups (Breakout, Pullback, VWAP Reclaim, Reversal, EMA Bounce, Gap Fill)
- [x] Each with description, entry/stop/target rules, 3-condition checklist
- [x] Live P&L and win count per setup
- [x] Analyzer alignment (highest scores, ENTER verdict alignment)

#### Section 6 — Analyzer Integration
- [x] "Add to Journal" button in Trade Analyzer
- [x] Auto-fill ticker, contract, score, verdict, constraints, expected move, win probability
- [x] User only fills thesis, mood, notes

#### Section 7 — Data Handling
- [x] localStorage persistence (no backend required)
- [x] Persist all trades across refresh
- [x] No mock trades injected
- [x] Empty state when no trades

#### Section 8 — UI/UX
- [x] Terminal-style UI, monospace numbers, dark theme
- [x] High contrast text, dense but readable layout

#### Acceptance
- [x] All TradeVault features exist and work
- [x] Analyzer data captured in journal entries
- [x] No mock data injected
- [x] Trades persist across refresh
- [x] System distinguishes good vs bad trades beyond P&L
- [x] Journal integrates directly with Trade Analyzer
- [x] Performance by verdict, delta, DTE, IV
- [x] Actionable feedback (not just data)
- [x] Terminal-grade UI quality
- [x] No existing logic or APIs changed

#### Section 9 — Vitest Tests
- [x] 26 vitest tests covering: calculatePnL, computeDashboardMetrics, computeEquityCurve, computePlaybookStats, computeReviewStats, computeStreaks, computeSetupPerformance, computeRuleAdherence, computeMistakeFrequency, pnlBySession, computeSystemDiscipline
- [x] All 26 tests passing (91 total tests passing across project)

### Stage 18 — Execution Dashboard / Command Center

#### Section A — Core System
- [x] Unified dashboard combining Scanner, Analyzer, Journal, Watchlist, Alerts, Risk & Discipline
- [x] Scan → Analyze → Decide → Journal → Monitor workflow
- [x] No duplicate searching, no disconnected panels, no mock data

#### Section B — Shared State Architecture
- [x] Create shared dashboard state context (activeTicker, activeContract, analyzerResult, etc.)
- [x] Scanner selection updates activeTicker + activeContract
- [x] Analyzer reads activeTicker + activeContract
- [x] Right panel reads analyzerResult
- [x] Journal prefill reads analyzerResult
- [x] Watchlist reads/writes activeTicker + activeContract
- [x] Alerts read activeContract + analyzerResult
- [x] No panel maintains isolated duplicate state

#### Section C — Scanner Data Rule
- [x] Live API data only
- [x] Unsupported categories show "Scanner unavailable — live data source not configured."
- [x] Whale Flow shows "Whale Flow requires live options flow data source."

#### Section D — Chart Rules
- [x] Price chart (candles or line) using existing historical price API
- [x] Expected move overlay
- [x] Entry zone overlay
- [x] Invalidation level overlay
- [x] Fallback: "Chart unavailable — live candle data missing."

#### Section E — Command Bar
- [x] Structured commands only (no NLP)
- [x] analyze TICKER STRIKE EXPIRATION
- [x] refresh TICKER
- [x] watch TICKER
- [x] journal current
- [x] alert current
- [x] compare current
- [x] Fallback: "Command not recognized. Try: analyze AMD 270C 2026-05-08"

#### Section F — Journal Prefill
- [x] Auto-fill: ticker, option type, strike, expiration, premium, direction, entry timestamp
- [x] Auto-fill: analyzer score, verdict, win probability, EV, constraint summary, risk warnings
- [x] Auto-fill: data status, setup type (if available)
- [x] User completes: thesis, mood, notes, mistake category, exit data

#### Section G — Bottom Panel
- [x] Reads from journalEntries only
- [x] Empty state: "No journal data yet. Log your first trade to activate performance analytics."
- [x] No sample trades

#### Section H — Responsive Behavior
- [x] Desktop ≥1200px: Left 320px, Center flexible, Right 320px, Bottom full width
- [x] Tablet: side panels collapse to drawers
- [x] Mobile: vertical stack, tabs for Scanner/Journal/Alerts, verdict at top, no small fonts

#### Section I — Layout
- [x] Left panel: Scanner + Watchlist with Calls/Puts/Both filter, DTE/Delta/IV filters
- [x] Center panel: Status strip, verdict card, score/win%/EV, constraints, entry/exit, chart
- [x] Right panel (tabs): Contracts, Journal, Alerts, Watchlist
- [x] Bottom panel: P&L daily/weekly, win rate, equity curve, rule adherence, violations

#### Section J — Workflow
- [x] Scanner/search sets activeTicker
- [x] Contract selection sets activeContract
- [x] Analyzer runs automatically
- [x] analyzerResult updates globally
- [x] All panels update automatically
- [x] Journal/Alert/Watchlist/Compare without re-entry

#### Section K — Data Status
- [x] Always show: LIVE MODE / PARTIAL LIVE DATA / DATA STALE / API ERROR / NO API DATA AVAILABLE
- [x] Never fallback silently

#### Section L — Brand Update
- [x] Replace "Option Intelligence Hub" with "Trade Sentinel" in all user-facing text
- [x] Add slogan "Guard Your Capital" as secondary text
- [x] Apply to: page titles, headers, navigation, dashboard, analyzer, watchlist, journal, scanner, loading screens, empty states, footer
- [x] Do not rename routes unless required

#### Section M — Acceptance
- [x] Dashboard panels share one unified state layer
- [x] Scanner → Analyzer → Journal flow is seamless
- [x] No duplicate searching required
- [x] No mock data anywhere
- [x] Unsupported data shows honest empty states
- [x] Journal auto-fill works
- [x] Performance uses real journal data
- [x] Mobile layout is usable
- [x] Data status always visible
- [x] Branding fully updated to Trade Sentinel
- [x] No existing logic, APIs, or scoring changed
- [x] Vitest tests for shared state and command bar (22 tests passing)

### Alerts Page Redesign — Professional Options-Flow Feed

#### Section A — Feed Layout
- [x] Alerts page opens to automated alert feed (not manual search)
- [x] Each alert card shows: timestamp, ticker, contract badge, expiration/DTE, severity score
- [x] Contract badges use green accent for calls, red accent for puts
- [x] Middle row: execution side (ASK/BID/MID), size, premium traded, avg price, vol/OI, vol/OI multiple
- [x] Right side: signal strength bar/percentage badge, expand/collapse caret

#### Section B — Expanded Alert Card
- [x] Stats grid: ask vol, bid vol, mid vol, no-side vol, sweep%, block%, premium, IV, volume, OI, vol/OI multiple
- [x] Action buttons: Analyze, Watchlist, Alert, Journal, Ignore
- [x] Clicking Analyze opens selected contract directly in Trade Analyzer

#### Section C — Detail View
- [x] Header: ticker + contract, expiration, current premium, premium change, share/watch/alert icons
- [x] Chart: bid/ask/mid volume bars, average fill line, intraday time axis, price axis
- [x] Historical section: historical vol/OI, daily volume, OI, OI change, bid/ask balance
- [x] Footer actions: Analyze Contract, Add to Watchlist, Create Alert, Add to Journal

#### Section D — Sweeps and Blocks
- [x] Monitor and display both sweeps and block trades
- [x] Sweeps labeled SWEEP with higher urgency, show ask-side or bid-side
- [x] Blocks labeled BLOCK, institutional size, no directional assumption
- [x] Combined: SWEEP + BLOCK CONFIRMED FLOW
- [x] Unsupported data: "Sweep and block monitoring requires advanced options flow data."
- [x] Never fake sweep/block data

#### Section E — Signal Strength
- [x] Score based on: vol/OI multiple, premium size, aggressiveness, sweep/block, OTM distance, DTE, IV movement, event proximity, repeat flow
- [x] Display: percentage badge, colored bar, severity label (Info/Unusual/Aggressive/Whale/Critical)

#### Section F — Filters
- [x] Calls / Puts / Both
- [x] Severity filter
- [x] Alert type filter
- [x] Sweep filter
- [x] Block filter
- [x] Ticker filter
- [x] Watchlist only filter
- [x] Open trades only filter
- [x] Time range filter

#### Section G — Empty State & Design
- [x] Empty state: "No alerts yet. Add tickers to your Watchlist or connect live options-flow data to begin monitoring."
- [x] Dark terminal theme with rounded alert cards
- [x] Clear red/green call-put accents
- [x] Readable timestamps, strong ticker+contract hierarchy
- [x] Compact but readable stats, expandable rows

#### Section H — Acceptance
- [x] Alerts page opens to automated alert feed
- [x] Manual search is not the primary UI
- [x] Alert cards visually resemble professional options-flow feeds
- [x] Cards include ticker, contract, expiration, DTE, vol/OI, premium, severity, timestamp
- [x] Expanded cards show detailed stats and action buttons
- [x] Detail view shows volume/OI chart style and historical section
- [x] Sweeps and blocks both monitored when supported
- [x] Sweeps and blocks never faked
- [x] Unsupported flow data shows honest unavailable state
- [x] Analyze action opens selected contract directly in Trade Analyzer

### Landing Page — Trade Sentinel

#### Section A — Hero
- [x] Full-width hero with dashboard visual (realistic UI mockup, not stock photo)
- [x] Headline: decision intelligence positioning
- [x] Subheadline: risk-first, capital protection
- [x] Primary CTA button + secondary link
- [x] Dark theme with gold+blue accents

#### Section B — Value Strip
- [x] 3 outcome cards (decision intelligence, risk management, capital protection)
- [x] Clean icons, short descriptions
- [x] Horizontal layout on desktop, stack on mobile

#### Section C — Alerts System
- [x] Sweeps, blocks, unusual activity monitoring
- [x] Realistic alert feed UI mockup (not mock data — static illustration)
- [x] Signal strength, severity labels
- [x] Honest about data source requirements

#### Section D — Smart Money Flow
- [x] Volume, premium, direction tracking
- [x] Visual showing flow analysis concept
- [x] No exaggerated claims

#### Section E — Gamma + Structure
- [x] GEX levels, gamma walls, dealer positioning
- [x] Market structure visualization
- [x] Honest about what's available vs planned

#### Section F — Command Center
- [x] Scanner → Analyzer → Journal → Alerts workflow
- [x] 4-step visual flow
- [x] Shows unified dashboard concept

#### Section G — Discipline + Journal
- [x] Performance tracking, rule adherence
- [x] Trade quality rating system
- [x] Journal analytics preview

#### Section H — Final CTA
- [x] Strong closing CTA
- [x] Reinforce value proposition
- [x] Educational disclaimer

#### Section I — Design & Acceptance
- [x] Dark theme throughout
- [x] Gold + blue accent colors
- [x] Clean, premium, minimal noise
- [x] Product-focused visuals (not generic stock graphics)
- [x] No mock data, no exaggerated features
- [x] Realistic UI components
- [x] Not a copy of CheddarFlow
- [x] Responsive (mobile + tablet + desktop)

### Landing Page Color System Update

- [x] Background: #0A0F1C throughout
- [x] Primary accent: #2D7FF9 (buttons, CTAs, highlights)
- [x] Positive: #22C55E, Negative: #EF4444, Warning: #F59E0B
- [x] Text primary: #E6EDF6, Text secondary: #94A3B8
- [x] No heavy gradients, no neon, no glow effects
- [x] Subtle borders instead of glow
- [x] High contrast and readability
- [x] Cards structured and controlled
- [x] Terminal-style feel maintained
- [x] No black+gold, no CheddarFlow styling
- [x] Buttons use blue accent (#2D7FF9)
- [x] Page feels like professional trading terminal

### Footer Navigation System

#### Footer Component
- [x] Footer with Product section: Features, Pricing, Blog, Customers, Markets, Affiliates, Help Desk, Discord
- [x] Footer with Legal section: Refund Policy, Terms of Service, Privacy Policy
- [x] Footer visible on ALL pages (dashboard + landing + content pages)
- [x] All links route to real pages (no # placeholders)
- [x] Consistent spacing, typography, Trade Sentinel design system

#### Content Pages — Product
- [x] /features — Feature overview with structured sections
- [x] /pricing — Pricing tiers with clear value proposition
- [x] /blog — Blog listing with articles
- [x] /customers — Customer stories / use cases
- [x] /markets — Supported markets and instruments
- [x] /affiliates — Affiliate program details
- [x] /help-desk — Help center with FAQ and support
- [x] /discord — Discord community page with invite

#### Content Pages — Legal
- [x] /refund-policy — Refund policy with clear terms
- [x] /terms-of-service — Terms of service with structured sections
- [x] /privacy-policy — Privacy policy with data handling details

#### Acceptance
- [x] Footer visible on all pages
- [x] Every footer link routes correctly
- [x] All pages load with real content
- [x] No broken or empty pages
- [x] Design consistent across all pages (#0A0F1C bg, #2D7FF9 accent, #E6EDF6 text)
- [x] No existing functionality changed

### Typography & Readability Fix

#### Font System
- [x] Inter font loaded via Google Fonts CDN
- [x] font-family: 'Inter', system-ui, -apple-system, Segoe UI, Roboto, sans-serif

#### Type Scale
- [x] Hero headlines: 48px, weight 700, line-height 1.2
- [x] Section headers: 28-32px, weight 600
- [x] Subheadings: 18px, weight 400-500
- [x] Body text: 15-16px, weight 400, line-height 1.6-1.7
- [x] Nav/buttons: 14-16px, weight 500-600
- [x] Labels/tags: 12-13px, weight 500

#### Contrast Fix
- [x] Primary text uses #E6EDF6 for all important content
- [x] Secondary text #94A3B8 only for less important info
- [x] No low-contrast slate gray on main content
- [x] All key text clearly readable immediately

#### Spacing
- [x] Increased spacing between sections
- [x] Text blocks narrow (max readability width)
- [x] Consistent vertical rhythm
- [x] Clear separation between sections

#### Pages Updated
- [x] Landing page (hero, features, CTAs, footer)
- [x] ContentPageLayout (shared wrapper)
- [x] Features, Pricing, Blog, Customers, Markets, Affiliates, Help Desk, Discord
- [x] Refund Policy, Terms of Service, Privacy Policy

#### Acceptance
- [x] All text clearly readable on desktop and mobile
- [x] No important text uses low-contrast gray
- [x] Headlines, body, labels have clear hierarchy
- [x] Landing page feels clean and professional
- [x] Footer pages readable and structured
- [x] No dashboard or logic changes

### Landing Page Restructure — 11 Conversion-Focused Sections

#### Section 1: Hero (Pain-First)
- [x] Headline: "You Picked the Right Direction. The Contract Killed the Trade."
- [x] Subheadline about contract evaluation
- [x] Primary CTA: "See How It Works" (low intent)
- [x] Secondary CTA: "View Platform Features"
- [x] Trust line: "Built for traders who want discipline before execution."
- [x] Hero visual: REJECT verdict card (Score 62/100, Win Prob 38%, reasons, better alternative)

#### Section 2: The Real Problem
- [x] Headline: "Most Losing Trades Aren't Wrong. They're Poorly Structured."
- [x] Checklist: delta too low, breakeven too far, spread too wide, premium overpriced, liquidity too thin
- [x] Visual: "Why Trades Fail" checklist card

#### Section 3: Decision Engine
- [x] Headline: "A Verdict, Not Just Numbers."
- [x] Show 5 verdicts: ENTER, ENTER WITH CAUTION, WAIT, RESTRUCTURE, REJECT
- [x] Include score, probability, risks, alternatives
- [x] CTA: "Run a Sample Trade" (mid intent)

#### Section 4: Unusual Activity
- [x] Headline: "See What Matters — Not Just What Moves."
- [x] Show vol/OI anomalies, premium size, directional flow
- [x] Alert feed cards visual
- [x] CTA: "Explore Alerts"

#### Section 5: Risk-First Analyzer
- [x] Headline: "Know When to Say No."
- [x] Hard blockers, soft warnings, better alternatives

#### Section 6: Command Center
- [x] Headline: "One System. Complete Control."
- [x] Flow: Scan → Analyze → Decide → Journal → Improve

#### Section 7: Journal & Discipline
- [x] Headline: "Track Decisions. Not Just Outcomes."

#### Section 8: Real Scenario
- [x] Headline: "What This Prevents"
- [x] Bad contract → trade loses / System flags → trade avoided

#### Section 9: Pricing/Access
- [x] Headline: "Start Free. Upgrade When You're Ready."
- [x] CTA: "Start Free Trial"

#### Section 10: Trust/Compliance
- [x] Not financial advice, no guarantees, user responsible

#### Section 11: Final CTA (High Intent)
- [x] Headline: "Guard Your Capital Before You Enter the Trade."
- [x] CTA: "Start Using Trade Sentinel"

#### Footer Pages Verification
- [x] Verify all 11 footer pages have correct typography and readability
- [x] Consistent color system across all pages

#### Acceptance Criteria
- [x] Text clearly readable on all devices
- [x] No low-contrast text for important content
- [x] Strong visual hierarchy
- [x] Hero uses pain-first messaging
- [x] Verdict system includes real example
- [x] CTA flow escalates (low → mid → high intent)
- [x] Landing page feels professional and structured
- [x] Footer pages fully readable and consistent
- [x] No system logic changed
- [x] Page feels like a decision system, not a marketing gimmick

### Logo Integration — Consistent Placement Across Site

#### Logo Assets
- [x] Full horizontal logo uploaded to webdev storage
- [x] Icon-only shield uploaded to webdev storage
- [x] Favicon (ICO) created and uploaded

#### Landing Page Header
- [x] Logo top-left, 42-48px desktop, 34-38px mobile
- [x] Logo links to homepage /
- [x] Mobile: icon-only if horizontal too wide

#### Footer
- [x] Full or compact logo at top of footer
- [x] "Guard Your Capital" slogan under logo
- [x] Footer logo links to homepage /

#### Auth Pages
- [x] Logo centered at top of login/signup (auth uses Manus OAuth redirect, no custom login page)
- [x] Slogan "Guard Your Capital" underneath (N/A - Manus OAuth)

#### Dashboard Sidebar
- [x] Icon-only mark in sidebar top-left
- [x] Compact "Trade Sentinel" text beside if space allows
- [x] Dashboard UI stays clean and terminal-like

#### Legal / Footer Pages
- [x] Logo in page header/navigation only
- [x] No giant logo inside legal content

#### Favicon
- [x] Icon-only version as favicon
- [x] Set in index.html

#### Acceptance Criteria
- [x] Logo appears in landing page header
- [x] Logo appears in footer
- [x] Logo appears on auth pages (Manus OAuth — no custom auth page)
- [x] Icon-only mark appears in dashboard/sidebar
- [x] Logo links back to homepage
- [x] Mobile logo does not overlap buttons
- [x] Favicon uses icon-only version
- [x] Logo is never distorted, cropped, or recolored

### Landing Page Upgrade — Copy, Readability, Visuals, CTA Flow

#### Hero Rewrite
- [x] Headline: "You Picked the Right Direction. The Contract Killed the Trade."
- [x] Supporting line: "You were right about the move. The contract didn't give the trade a chance."
- [x] Subheadline: Trade Sentinel identifies bad contracts before they cost you money
- [x] Primary CTA (low intent): "See How It Works"
- [x] Secondary CTA: "View Platform Features"
- [x] Trust line: "Built on principles used by disciplined professional traders."
- [x] Hero visual: REJECT verdict card (Score 58/100, Win Prob 38%, sharp, high-contrast, readable)

#### Problem Section
- [x] Headline: "Most Losing Trades Aren't Wrong. They're Poorly Structured."
- [x] Bullets: delta too low, breakeven too far, spread too wide, premium overpriced, liquidity too thin
- [x] Tighten spacing and readability

#### Decision Engine
- [x] Headline: "A Verdict, Not Just Numbers."
- [x] Show 5 verdicts with score, probability, risks, suggested improvement
- [x] CTA (mid intent): "Run a Sample Trade"

#### Alert Section
- [x] Headline: "See What Matters — Not Just What Moves."
- [x] Add: "Not all activity matters. This highlights what might."
- [x] Show vol/OI anomalies, premium size, directional flow
- [x] Do NOT mention missing data

#### Risk Analyzer
- [x] Headline: "Know When to Say No."
- [x] Hard blockers, soft warnings, better alternatives

#### Command Center
- [x] Headline: "One System. Complete Control."
- [x] Flow: Scan → Analyze → Decide → Journal → Improve

#### Journal Section
- [x] Headline: "Track Decisions. Not Just Outcomes."
- [x] Reduce text density, keep bullet clarity

#### Scenario Section (replace abstract content)
- [x] Headline: "What This Prevents"
- [x] Without Trade Sentinel: buy low-delta, stock moves up, still lose money
- [x] With Trade Sentinel: flags low delta, unrealistic breakeven → REJECT → trade avoided

#### Pricing Section
- [x] Headline: "Start Free. Upgrade When You're Ready."
- [x] Free trial access, CTA: "Start Free Trial"

#### CTA Flow
- [x] Top: "See How It Works"
- [x] Middle: "Run a Sample Trade"
- [x] Bottom: "Start Free Trial"
- [x] Do NOT repeat same CTA everywhere

#### Final CTA
- [x] Headline: "Stop Letting Bad Contracts Kill Good Trades."
- [x] Subheadline: "Guard your capital before you enter."
- [x] CTA: "Start Using Trade Sentinel"

#### Readability Fix (apply everywhere)
- [x] Max text width: 600-700px
- [x] Break paragraphs every 2-3 lines
- [x] Increase vertical spacing ~20%
- [x] Primary text: #E6EDF6, Secondary: #94A3B8 (only minor text)
- [x] No low-contrast gray for important content

#### Visual System
- [x] All visuals sharp, high-res, clean, product-focused
- [x] Background: #0A0F1C, Accent: #2D7FF9, Green: #22C55E, Red: #EF4444
- [x] No stock photos, no generic illustrations, no neon gradients

#### Footer Pages Readability
- [x] Apply readability + spacing rules to all footer pages
- [x] Clean, structured, readable

#### Acceptance Criteria
- [x] Hero creates immediate emotional recognition
- [x] Verdict system clearly visible and believable
- [x] Scenario section replaces abstract content
- [x] CTA flow escalates correctly
- [x] Pricing anchor exists
- [x] Page is highly readable
- [x] Visuals are sharp and product-focused
- [x] No stock images used
- [x] Page feels like a decision system
- [x] No system logic was changed

### UI System Upgrade — Professional Readability, Spacing, Hierarchy

#### Color System
- [x] Background: #0A0F1C
- [x] Surface/Card: #101827
- [x] Surface Elevated: #121E33
- [x] Primary Accent: #2D7FF9
- [x] Positive: #22C55E, Negative: #EF4444, Warning: #F59E0B
- [x] Text Primary: #E6EDF6, Text Secondary: #94A3B8
- [x] Border: rgba(148, 163, 184, 0.18)
- [x] No black/gold, no heavy gradients, no neon effects

#### Typography Scale
- [x] Hero headline: 48-56px desktop, 36-42px tablet, 30-34px mobile, weight 700, line-height 1.1-1.2
- [x] Section headlines: 30-34px desktop, 26-30px tablet, 24-26px mobile, weight 600-700, line-height 1.2-1.3
- [x] Subheadings: 18-20px, weight 500, line-height 1.45
- [x] Body: 15-16px, weight 400, line-height 1.6-1.7
- [x] Buttons: 14-16px, weight 600
- [x] Labels/tags/data: 12-13px, weight 500-600
- [x] No important text smaller than 13px

#### Layout System
- [x] Max content width: 1180-1240px
- [x] Section vertical padding: 90-120px desktop, 56-72px mobile
- [x] Hero vertical padding: 100-140px
- [x] Card padding: 24-32px desktop, 18-22px mobile
- [x] Card border radius: 18-24px
- [x] Grid gap: 24-32px

#### Card System
- [x] Dark, structured, readable, high-contrast, premium
- [x] Subtle border, elevated surface, clean spacing
- [x] Clear title hierarchy, signal-colored accents
- [x] No overused shadows, no glow effects

#### Button System
- [x] Primary: bg #2D7FF9, text #FFFFFF, bold, rounded not pill
- [x] Secondary: transparent/dark surface, subtle blue-gray border, text #E6EDF6
- [x] CTA flow: See How It Works → Run a Sample Trade → Start Free Trial

#### Readability
- [x] Primary content uses #E6EDF6
- [x] Text blocks max 600-700px
- [x] Break long paragraphs
- [x] No dense text clusters

#### Footer Pages
- [x] Same design system applied to ContentPageLayout
- [x] Consistent spacing, typography, card styling

#### Acceptance Criteria
- [x] Landing page feels premium and trading-focused
- [x] Typography hierarchy feels sharp and professional
- [x] Visuals are product-based and clear
- [x] Text is easy to read on desktop and mobile
- [x] Does not look like CheddarFlow
- [x] No black/gold design direction
- [x] No system logic or app functionality changed
- [x] Footer pages follow same readability and design system

### Positioning Section Replacement — Risk-First Decision System

- [x] Remove current "Trade Sentinel is not an alert platform" section
- [x] Insert new positioning block after Decision Engine, before Alerts
- [x] Headline: "Trade Sentinel is not an alert platform."
- [x] Subheadline: "It is a risk-first decision system built to fix the decision before the trade."
- [x] Body: "It evaluates the structure of a trade before capital is exposed— so decisions are made with clarity, not urgency."
- [x] Close: "Built to prioritize:" + 3 principles (Risk clarity, Decision quality, Capital protection)
- [x] Analyzer visual pairing (ENTER WITH CAUTION verdict, Score 68/100, Win Prob 55%, risk indicators)
- [x] No feature lists, no marketing language, no extra bullets
- [x] No other sections altered
- [x] Placement correct: after Decision Engine, before Alerts

### Global UI Consistency Pass — Internal + Footer Pages

#### Design System Tokens
- [x] Background: #0A0F1C, Surface: #101827, Elevated: #121E33
- [x] Border: rgba(148,163,184,0.18)
- [x] Text Primary: #E6EDF6, Secondary: #94A3B8
- [x] Accent: #2D7FF9, Positive: #22C55E, Negative: #EF4444, Warning: #F59E0B
- [x] No black/gold, no heavy gradients, no glow effects

#### Typography
- [x] Headers: 28-34px / 600-700 weight
- [x] Subheaders: 18-20px / 500 weight
- [x] Body: 15-16px / 400 weight
- [x] Labels/Data: 12-13px / 500-600 weight
- [x] No small unreadable text, clear hierarchy

#### Card System
- [x] Card bg: #101827, border-radius: 18-22px, padding: 20-28px
- [x] Subtle border, no harsh shadows
- [x] Structured, readable, spaced, not cramped

#### Data Readability
- [x] Important values use PRIMARY text color
- [x] Secondary info uses #94A3B8
- [x] No low-contrast gray for key data
- [x] Increased spacing between rows

#### Alerts UI
- [x] Structured, clean, professional alert cards
- [x] Clear: ticker, contract, premium, volume/OI, signal strength, reason
- [x] No clutter, no compressed rows

#### Analyzer UI
- [x] Verdict large and obvious
- [x] Score, win probability, risk breakdown, suggested alternative clear
- [x] Hierarchy: Verdict → Score → Risks → Details

#### Dashboard Layout
- [x] Panels clearly separated
- [x] Spacing consistent
- [x] No overlapping content

#### Footer Pages
- [x] Same typography system
- [x] Same spacing rules
- [x] Max text width 600-700px
- [x] Small readable paragraph blocks

#### Spacing
- [x] Increase vertical spacing ~20%
- [x] Consistent gaps 24-32px
- [x] No crowded layouts

#### Buttons
- [x] Primary: #2D7FF9, white text, bold
- [x] Secondary: subtle border, no heavy fill

#### Acceptance Criteria
- [x] All pages feel visually consistent
- [x] Internal UI matches landing page quality
- [x] Data is easy to read instantly
- [x] Alerts and analyzer feel structured and clean
- [x] Footer pages are readable and professional
- [x] No clutter or cramped layouts
- [x] No low-contrast text used
- [x] No design drift between pages

### Header Dropdowns + Pricing Page + Refund Policy

#### Header Navigation Dropdowns
- [x] Nav items: Features, Solutions, Resources, Pricing, Start Trial, Login
- [x] Features dropdown: Trade Analyzer, Unusual Flow Alerts, Sweeps & Blocks, Risk Engine, Journal & Discipline, Command Center — with descriptions and icons
- [x] Solutions dropdown: For Retail Traders, For Active Options Traders, For Risk-First Traders, For Traders Building Discipline — with descriptions
- [x] Resources dropdown: Blog, Help Desk, Customer Reviews, Video Tutorials, Roadmap, Discord Community, Affiliate Program, Markets
- [x] Dropdown UI: dark elevated panel, rounded 18-22px, subtle border, 24px padding, 18-22px row gap
- [x] Icons beside each item
- [x] Hover state: subtle blue highlight (not gold)
- [x] Dropdowns close on outside click
- [x] Mobile: hamburger menu with collapsible groups

#### Pricing Page (/pricing)
- [x] Route registered at /pricing
- [x] Headline: "Choose the Plan That Helps You Trade With More Structure."
- [x] Subheadline about free start and upgrade
- [x] 3 tiers: Starter (Free Trial), Pro ($29/mo), Sentinel Elite (Coming Soon)
- [x] Starter includes: Limited Trade Analyzer, Basic contract review, Limited watchlist, Basic educational access, Footer/legal pages access
- [x] Pro includes: Full analyzer, verdict engine, risk score, probability+EV, better contract suggestions, alerts, watchlist, journal, performance tracking, rule adherence tracking — "Most Popular" badge
- [x] Sentinel Elite includes: Advanced flow, sweeps/blocks, gamma/structure, dark pool, advanced alerts, coaching insights, priority access — "Join Waitlist" CTA
- [x] Feature comparison table (10 rows x 3 columns)
- [x] "Why Trade Sentinel" section: "Not More Signals. Better Decisions."
- [x] FAQ section with 5 questions (financial advice, guarantees, flow data, cancel/refund, free trial)
- [x] FAQ #4 includes 7-day refund window
- [x] Primary CTA = blue, Most Popular badge = blue, no gold

#### Refund Policy Page Update
- [x] Match FAQ language exactly
- [x] 7-day refund window for new subscriptions
- [x] Eligibility criteria: within 7 days, original account holder, first-time only
- [x] After 7 days: non-refundable, access until end of billing period
- [x] Abuse prevention clause
- [x] Processing: original payment method, reasonable timeframe

#### Acceptance Criteria
- [x] Header has working dropdowns for Features, Solutions, Resources
- [x] Dropdowns feel premium, clean, readable
- [x] Pricing page exists at /pricing with 3 tiers
- [x] Feature comparison table present
- [x] FAQ present with cancel/refund info
- [x] Refund Policy page matches FAQ language
- [x] Mobile navigation works cleanly
- [x] No app logic changed

### Command Center Restructure — Decision Environment

#### Global: Remove STAGE Labels
- [x] Remove all "STAGE X — LIVE" labels from entire portal
- [x] Replace with status: ACTIVE, MONITORING, IDLE, WARNING, OFFLINE

#### Layout: Discover / Decide / Manage Grouping
- [x] DISCOVER: Market Scanner, Alerts Engine
- [x] DECIDE: Trade Analyzer (PRIMARY), Options Chain
- [x] MANAGE: Watchlist, Trade Journal

#### Top Strip
- [x] Replace tag strip with: Market Status, System Status, System Bias, Active Signals, Risk Environment, Suggested Action

#### Module Cards
- [x] Each card: Icon, Module Name, Status, Short description (max 2 lines)
- [x] Optional: Last Scan, Last Verdict, Risk Flags
- [x] Micro CTA: "Open Analyzer →", "View Alerts →", etc.
- [x] Entire card clickable with cursor pointer
- [x] Hover: border #2D7FF9, slight elevation, 150-200ms transition
- [x] Routes: Analyzer→/analyzer, Scanner→/scanner, Alerts→/alerts, Chain→/options-chain, Watchlist→/watchlist, Journal→/journal

#### Visual Hierarchy
- [x] Trade Analyzer card larger and visually emphasized
- [x] Shows Last Verdict, Score, Risk Flags

#### Bottom Section
- [x] Rename "FULL SYSTEM ARCHITECTURE" → "SYSTEM STATUS"
- [x] Show only: Decision Engine ON, Market Intelligence ON, Alert Engine ON, Probability Engine ON

#### Acceptance Criteria
- [x] No STAGE labels exist anywhere in the portal
- [x] Cards are clickable and route correctly
- [x] Clear Discover / Decide / Manage grouping
- [x] Analyzer is primary card with emphasis
- [x] Top strip shows system decision info
- [x] UI feels like a finished product
- [x] No app logic changed

### Trade Analyzer Results UI Redesign — Decision-First Layout

#### Top Panel
- [x] Verdict displayed large and prominent
- [x] Score, Win Probability, Expected Value, Risk Level visible
- [x] Suggested Action line added

#### Why Panel
- [x] Title: "Why This Verdict"
- [x] Bullet list, max 5 points, no paragraphs

#### Better Alternative Panel
- [x] Show improved strike, expiration, delta, breakeven, probability
- [x] If none: "No stronger alternative found"

#### Breakdown Cards (4 cards, max 4 lines each)
- [x] Structure: Delta, Breakeven, DTE
- [x] Liquidity: Spread, Volume, OI
- [x] Volatility: IV Rank, Premium
- [x] Risk: Position risk, Exposure

#### Blockers / Warnings
- [x] Hard Blockers separated (red)
- [x] Warnings separated (yellow)

#### Trade Plan
- [x] Entry, Stop, Take Profit, Time Exit
- [x] Placeholder message if unavailable

#### Bottom Action Bar
- [x] Add to Watchlist button
- [x] Reanalyze button
- [x] Save to Journal button

#### Cleanup
- [x] Remove duplicate data
- [x] Remove long paragraphs
- [x] Remove clutter

#### Acceptance Criteria
- [x] Verdict visible instantly
- [x] Why panel exists
- [x] Alternative panel exists
- [x] Breakdown cards exist
- [x] Blockers vs warnings separated
- [x] Trade plan present
- [x] No logic changed

### Alerts UI Redesign — Automated Monitoring Feed

#### Top Header & Filters
- [x] Header: "Alert Feed"
- [x] Filter tabs: All, Calls, Puts, Critical, Whale, Aggressive, Unusual, Info
- [x] Optional toggles: Watchlist Only, Open Trades Only

#### Alert Card Layout
- [x] Top: timestamp, ticker, contract, DTE, severity, signal strength
- [x] Middle: premium, volume/OI, execution (ask/bid/mid)
- [x] Reason: 1 short sentence
- [x] Right: strength bar, expand icon

#### Expanded Card
- [x] Grid: Ask volume, Bid volume, Sweep %, Block %, Premium, IV, Volume, OI
- [x] Buttons: Analyze, Watchlist, Alert, Journal, Ignore
- [x] Analyze opens contract directly in Analyzer

#### Detail View
- [x] Ticker + contract, premium + change, actions
- [x] Chart: volume bars, fill line (if available)
- [x] If unavailable: show message (no fake data)

#### Sweeps / Blocks
- [x] Show SWEEP (urgency) and BLOCK (size) when supported
- [x] If unsupported: show small notice

#### Empty State
- [x] "No alerts yet" message
- [x] Buttons: Open Watchlist, Configure Alerts

#### Style
- [x] Dark theme, clean cards, strong hierarchy, readable text

#### Acceptance Criteria
- [x] Alerts feel automated
- [x] Cards show key data clearly
- [x] Expanded cards work
- [x] Analyze opens correct contract
- [x] Empty state is useful
- [x] No fake data
- [x] No logic changed

### Global Typography, Contrast & Spacing Overhaul
#### Text Contrast
- [x] Primary text #E6EDF6 for all important content (headings, values, metrics, verdicts)
- [x] Secondary text #94A3B8 only for descriptions, helper text, minor labels
- [x] No low-contrast gray for important data
- [x] Remove opacity-reduced text (muted-foreground/70, /60, /50)
#### Font System
- [x] Inter, system-ui font stack applied globally
- [x] Headlines: 32-40px / 600-700
- [x] Section titles: 24-28px / 600
- [x] Data values: 18-22px / 500-600
- [x] Body text: 15-16px / 400
- [x] Labels: 12-13px / 500
- [x] Important numbers always larger than labels
#### Line Height & Spacing
- [x] Line height 1.5-1.7 for body text
- [x] 24-32px spacing between sections
- [x] Avoid tight or cramped layouts
#### Card Readability
- [x] Padding 20-28px on all cards
- [x] Clear separation between lines
- [x] Each row easily scannable
#### Data Hierarchy
- [x] Primary (largest) → Secondary (supporting) → Label (smallest)
- [x] New .text-data-value and .text-label utility classes
#### Visual Noise Reduction
- [x] Reduced excessive borders
- [x] Clean grouping with intentional spacing
#### Global Consistency
- [x] Applied across all pages, modules, dropdowns, tables, cards
- [x] No text-xs or text-[10px] or text-[11px] remaining in any page
#### Acceptance Criteria
- [x] No important text uses low-contrast gray
- [x] All key data is instantly readable
- [x] Visual hierarchy clear across all pages
- [x] Spacing feels open and structured
- [x] UI no longer requires effort to read
- [x] Consistency maintained across entire portal

### Preview Auth Fix & App Rename
#### Preview Authentication
- [x] Diagnose why mobile preview redirects to sign-in (platform-level auth gate, not app code)
- [x] Diagnose why desktop preview doesn't finish loading (2.3MB bundle + platform auth gate)
- [x] No app-level auth guard exists — DashboardLayout has no auth requirement
- [x] Production auth remains intact (protectedProcedure still guards watchlist mutations)
#### App Name Rename
- [x] Replace "Option Intelligence Hub" with "Trade Sentinel" in all visible UI
- [x] package.json name updated to trade-sentinel
- [x] VITE_APP_TITLE needs manual update in Management UI → Settings → General (BLOCKED: user action required)
#### Acceptance Criteria
- [x] Mobile preview opens Trade Sentinel portal (after platform auth)
- [x] Desktop preview loads successfully (after platform auth)
- [x] Preview mode does not get stuck on login (no app-level auth guard)
- [x] Auth system remains intact for production
- [x] "Option Intelligence Hub" no longer appears in app code UI
- [x] VITE_APP_TITLE platform setting needs manual update by user (BLOCKED: user action required)

### Data Persistence Fix — Watchlist & Journal

#### Audit Results
- [x] Watchlist: uses tRPC backend (DB) when authenticated, localStorage when not
- [x] Journal: localStorage ONLY (no backend persistence exists)
- [x] localStorage keys: oih_watchlist, oih_trade_journal

#### Backend Persistence for Journal
- [x] Add journal_trades table to drizzle/schema.ts
- [x] Add journal CRUD helpers to server/db.ts
- [x] Add journal tRPC router to server/routers.ts
- [x] Update TradeJournal.tsx to use backend when authenticated (localStorage fallback)
- [x] Run pnpm db:push to sync schema

#### Export / Import Controls
- [x] Add Export Data button to Watchlist page (downloads JSON)
- [x] Add Import Data button to Watchlist page (uploads JSON, merges, avoids duplicates)
- [x] Add Export Data button to Journal page (downloads JSON)
- [x] Add Import Data button to Journal page (uploads JSON, merges, avoids duplicates)
- [x] JSON includes: items, created dates, updated dates, schema version

#### Preview Data Warning
- [x] Show banner when using localStorage: "Preview data is stored locally and may reset if the preview URL changes. Export your data before major updates."
- [x] Only show when not authenticated (localStorage mode)

#### Acceptance Criteria
- [x] Watchlist and Journal storage method clearly identified
- [x] Watchlist and Journal no longer silently disappear without warning
- [x] Export/Import works for both Watchlist and Journal
- [x] Imported data merges safely and avoids duplicates
- [x] Preview/localStorage limitation clearly disclosed
- [x] No analyzer, alert, scoring, or API logic changed

### Market Scanner — Automatic Continuous Scanning

#### Scan Frequency
- [x] Refresh scanner results every 15–30 seconds when user is on Scanner page
- [x] Do not fully reload the page, only update changed/new results

#### Background Scanning
- [x] Refresh scanner data every 60–120 seconds when user is in app but not on Scanner page
- [x] Keep results ready for instant display when user returns

#### Tab Inactive Behavior
- [x] Pause or reduce scan frequency to 2–5 minutes when tab is inactive
- [x] Resume normal frequency when tab becomes active again

#### Update Behavior
- [x] Do NOT clear existing results on refresh
- [x] Merge new results into existing list
- [x] Update changed items in place
- [x] Preserve scroll position

#### New Signal Handling
- [x] Highlight newly detected opportunities
- [x] Show "New" badge or visual indicator
- [x] Optionally show subtle notification

#### Performance Rules
- [x] Do not overload API calls, respect current API limits
- [x] Use efficient polling strategy

#### Fallback
- [x] Show "Live scanning limited by current data source" if auto-scan cannot run
- [x] Maintain best available refresh behavior

#### Acceptance Criteria
- [x] Scanner updates automatically without user input
- [x] Results refresh every 15–30 seconds when active
- [x] Background scanning works at reduced frequency
- [x] Scanner does not flicker or reset on update
- [x] New opportunities are clearly highlighted
- [x] Scroll position is preserved
- [x] No API, analyzer, alerts, or scoring logic changed

### Market Scanner — Persistent Daily Opportunity Log Upgrade

#### Auto-Scan on Page Load
- [x] Start scanning immediately when user opens Market Scanner
- [x] Load initial results automatically without manual scan
- [x] Show "Scanning market…" loading state briefly
- [x] Remove manual scan dependency for auto-scan activation

#### Default Scanning Universe
- [x] Scan predefined high-volume tickers + major ETFs when no user input
- [x] Include watchlist tickers if available
- [x] Never leave scanner empty because no manual scan was run

#### Signal Persistence
- [x] Keep detected signals visible for entire trading day
- [x] Do NOT remove signals on refresh
- [x] Update signals in place, add new signals, preserve old signals
- [x] Merge updates into existing signals using unique key (ticker+contract+expiration)

#### Signal Status System
- [x] ACTIVE status — still receiving confirmation signals
- [x] DEVELOPING status — conditions still forming
- [x] STALE status — no longer progressing or no updates for a period
- [x] CLOSED status — strong reversal or disappearance of activity
- [x] Status update rules: strengthen→ACTIVE, forming→DEVELOPING, weaken→STALE, reversal→CLOSED
- [x] Never remove signals when status changes

#### Daily Reset
- [x] Clear previous day signals at start of new trading day
- [x] Do not mix prior-day signals into today's active list

#### Signal Metadata
- [x] Store: first detected timestamp, last updated timestamp, status, setup type, score, verdict, reason, ticker, contract, expiration

#### UI Display — Today's Opportunities
- [x] Title: "Today's Opportunities"
- [x] Each signal shows: ticker, setup type, score, verdict, status, first detected, last updated, reason
- [x] Sort options: most recent, highest score, strongest activity
- [x] Status filters: All, Active, Developing, Stale, Closed
- [x] Use cautious language (Likely active, Likely closed, No recent activity)

#### Empty State
- [x] Show: "Scanner feed unavailable. Connect supported live market data or add tickers to your Watchlist to begin scanning."

#### Performance + Duplicate Control
- [x] No duplicate signals (unique key: ticker+contract+expiration)
- [x] Merge updates into existing signals
- [x] Preserve scroll position
- [x] No flicker or reset on refresh

#### Acceptance Criteria
- [x] Scanner starts automatically on page load
- [x] No manual scan required to enable auto-scan
- [x] Scanner uses a default scanning universe
- [x] Scanner updates frequently without full reload
- [x] Signals do not disappear after detection
- [x] Scanner maintains a full intraday opportunity list
- [x] Signals update status instead of disappearing
- [x] Daily reset works correctly
- [x] No duplicate signals appear
- [x] UI clearly shows signal lifecycle
- [x] Manual scan is optional only
- [x] No APIs, analyzer logic, alerts logic, scoring, verdict logic, or data flow changed

### Trade Analyzer — Implementation Override Application (Controlled Test)

- [x] Apply IBM Plex Mono to all numerical values (scores, prices, percentages, deltas)
- [x] Apply correct type scale: 36px verdict, 28px section headers, 22px card titles, 20px data values, 16px body, 12px labels, 11px micro
- [x] Apply correct text colors: #E6EDF6 primary, #94A3B8 secondary, #6B7280 muted
- [x] Apply correct spacing: 24px card padding, 16px row gap, 32px section gap, 6px label gap
- [x] Ensure verdict panel uses 36px/700 for verdict text with signal color
- [x] Ensure data values are visually larger than their labels
- [x] Remove any glow effects or gradients
- [x] No layout structure changes
- [x] No logic or API changes
- [x] Build passes after changes
- [x] No other pages affected

### Trade Analyzer — Score vs Verdict Separation Upgrade

- [x] Verdict panel: verdict is largest/dominant element, score is secondary below
- [x] Add "Setup Quality: B (72)" and "Decision: REJECT" visual separation
- [x] Add one-line explanation below score (e.g. "Moderate quality but fails critical risk rule")
- [x] Add "Why This Verdict" section with max 5 bullet points
- [x] Add "Hard Blockers" section with red accent (only shown when blockers exist)
- [x] Add "Warnings" section with amber accent
- [x] Use Implementation Override typography and colors
- [x] Numbers use IBM Plex Mono
- [x] No logic, API, scoring, or verdict calculation changes
- [x] Build passes after changes
- [x] User can understand REJECT within 2 seconds

### Trade Analyzer — Contextual Hover Card Tooltips

- [x] Create reusable MetricTooltip component (hover on desktop, tap on mobile)
- [x] Define tooltip content for: Score, Win Probability, EV, Risk Level, Delta, Breakeven, DTE, Spread, R:R, Verdict, Hard Blockers, Warnings
- [x] Each tooltip: what it is, why it matters, good vs bad ranges
- [x] Design: bg #121E33, text #E6EDF6, padding 12-16px, rounded, max-width 260px
- [x] Apply tooltips to verdict panel metrics
- [x] Apply tooltips to breakdown card labels
- [x] Apply tooltips to Hard Blockers and Warnings headers
- [x] No layout structure changes
- [x] No logic, API, or scoring changes
- [x] Works on desktop (hover) and mobile (tap)
- [x] Build passes after changes

### Trade Analyzer — Score/Verdict Consistency Fix

- [x] Rename Best Contracts panel score label from generic "score" to "Contract Rank"
- [x] Main verdict panel clearly labeled as "Trade Score" (not generic "score")
- [x] Add tooltip to Best Contracts panel: "Best Contracts are ranked alternatives. Final verdict still depends on risk rules."
- [x] Update BEST badge logic: BEST (enter), BEST WITH CAUTION (enter caution), BEST ALTERNATIVE (restructure), BEST AVAILABLE — BLOCKED (reject)
- [x] No green BEST badge on rejected contracts
- [x] Selecting a Best Contract updates all main analyzer data (strike, expiration, verdict, score)
- [x] No stale state after contract switch
- [x] If BEST contract is rejected, show Contract Rank alongside Trade Score with clear blocker explanation
- [x] User can distinguish Contract Rank from Trade Score within 2 seconds
- [x] No API, scoring formula, or data flow changes
- [x] Build passes after changes
### Trade Analyzer — Entry/Exit Signals Text Clipping Fix
- [x] Remove text clipping (overflow hidden, text-overflow ellipsis, fixed width constraints) from Entry/Exit Signals section
- [x] Enable proper text wrapping (white-space normal, word-break break-word, overflow-wrap anywhere)
- [x] Expand explanation column width or allow flexible width
- [x] Proper alignment: condition label left, full explanation right (multi-line allowed)
- [x] Responsive: stack vertically on smaller screens
- [x] Line height 1.5-1.7, no clipped text anywhere
- [x] Keep checklist format, icons, hierarchy unchanged
- [x] No logic, API, scoring, or design structure changes

### Spec PATCH v1.2 — Threshold Override Corrections
- [x] Add DTE ≤ 1 hard blocker (Rule 13): DTE ≤ 1 AND 0DTE mode not selected → Critical → REJECT
- [x] Remove score-as-hard-blocker: Score < 40 must not trigger REJECT on its own (remove Rule 1 critical tier)
- [x] Add Best Contract UI tooltip: "Best relative contract. Still subject to full trade validation."
- [x] Add short-circuit REJECT rule to decision tree: once any condition produces REJECT, evaluation stops

### PATCH v1.2 Implementation
- [x] Score never directly triggers REJECT — remap score < 50 to RESTRUCTURE initial verdict
- [x] REJECT only from critical hard blockers — remove score-based REJECT path
- [x] Add Rule 13: DTE ≤ 1 AND 0DTE mode not selected → Critical → REJECT
- [x] 0DTE High-Risk Mode persistent warning display
- [x] Best Contract badge variants (BEST / BEST WITH CAUTION / BEST ALTERNATIVE / BEST AVAILABLE — BLOCKED)
- [x] Short-circuit REJECT: once REJECT triggered, stop evaluation and display primary blocker
- [x] Score and Verdict visually separate in UI
- [x] Acceptance: 70+ score not REJECT without hard blocker, best contract ≠ approved, 0DTE blocked without mode

### UI/Navigation Integration — Option Data & Market Data
- [x] Add DISCOVER section to Command Center with Option Data cards (0DTE Flow, Flow Alerts, Dark Pool Feed)
- [x] Add DISCOVER section to Command Center with Market Data cards (News, Economic Calendar, Earnings Calendar, Sector Flow)
- [x] Cards use existing card structure with icon, name, status, description, CTA
- [x] Cards route to existing pages/components (no duplicate pages)
- [x] Update Left Panel navigation with structured grouping: DISCOVER (Option Data + Market Data), DECIDE, MANAGE
- [x] Left Panel uses collapsible sections with consistent icons
- [x] All items map to existing pages/components
- [x] No duplicate pages, no logic changes, no API changes

### UI Routing & Market Data Page Fixes
- [x] Command Center cards open in new tab (target="_blank" rel="noopener noreferrer") — entire card + micro CTA
- [x] Left panel navigation remains same-tab (no changes)
- [x] News page shows market headlines by default without requiring ticker search
- [x] Earnings page shows upcoming earnings list by default without requiring ticker search
- [x] Economic Calendar preserved (already correct)
- [x] Cursor pointer on Command Center cards
- [x] No duplicate pages, no logic/API/data flow changes

### Data Pipeline, Routing & Status Fixes
- [x] Remove hardcoded AAPL fallback — confirmed scanner uses DEFAULT_SCAN_TICKERS (10 tickers), no AAPL-only fallback exists; rate limits on free Polygon plan cause limited results
- [x] Verify API responses — orchestrator now surfaces errors when market regime fails; scanner shows clear empty state with ticker count
- [x] Fix Economic Calendar — created /economic-calendar page with real Finnhub API data (7-day lookahead)
- [x] Fix Market Status — real-time calculation using US Eastern Time (OPEN/CLOSED/PRE-MARKET/AFTER-HOURS) in DashboardLayout + Home
- [x] Fix Sector Flow routing — created /sector-flow page with real Polygon ETF data (11 sector ETFs)
- [x] Remove synthetic orchestrator fallback (vix:18, breadth:1, putCallRatio:0.8) — now shows zeros + error message
- [x] Remove hardcoded sectorStrength from marketRegimeService — returns empty object
- [x] Validation: Scanner returns varied tickers when API responds; Economic Calendar shows real events; Sector Flow shows real ETF data

### Bug Fixes — Live Data/Scanner/Routing Issues (User Report)
- [x] Fix Market Status showing "Closed" when market is open — verified DashboardLayout MarketStatusIndicator uses real-time US Eastern Time
- [x] Fix Market Scanner only returning AAPL — added 2s delay between tickers + Finnhub fallback for quotes/candles
- [x] Fix 0DTE card only returning AAPL — same fix as scanner (rate limiting + Finnhub fallback)
- [x] Fix Option Chain Flow — added auto-refresh toggle (30s during market hours) with countdown display
- [x] Fix Economic Calendar showing "what is supported" — /economic-calendar page uses Finnhub API (7-day lookahead)
- [x] Fix Dark Pool Scanner — confirmed manual-input tool (no API calls, no rate limit issue)
- [x] Fix Sector Flow routing — /sector-flow page uses Finnhub-first data fetching (60 calls/min)
- [x] Fix Command Center cards — target="_blank" on all card links (verified in code)

### Data Pipeline Fixes — Finnhub Fallback & Rate Limiting
- [x] Add rateLimitedPolygonFetch() helper (1.2s delay between Polygon calls)
- [x] Add fetchFinnhubQuote() function (Finnhub quote fetcher)
- [x] Add fetchFinnhubCandles() function (Finnhub candle fetcher)
- [x] Add Finnhub fallback in getStockQuote() — tries Finnhub when Polygon fails
- [x] Add Finnhub standalone path in getStockQuote() — uses Finnhub when no Polygon/Tradier configured
- [x] Add Finnhub fallback in getHistoricalCandles() — tries Finnhub when Polygon fails
- [x] Add Finnhub standalone path in getHistoricalCandles() — uses Finnhub when no Polygon/Tradier configured
- [x] MarketScanner: 2-second delay between ticker scans to avoid rate limiting
- [x] SectorFlow: Finnhub-first data fetching (tries Finnhub before Polygon for all 11 ETFs)
- [x] OptionsChain: Auto-refresh toggle (30s interval during market hours, countdown display)
- [x] Dark Pool / Whale Flow: confirmed manual-input only (no API calls, no rate limit issue)

### Option Chain Flow — Live Feed Default View Fix
- [x] Create OptionFlow.tsx — live flow feed page showing multi-symbol options flow data immediately
- [x] Flow feed displays: ticker, contract (strike+type), expiration, premium, volume, OI, vol/OI ratio, execution side
- [x] Include sweep/block classification label where available
- [x] Search box is optional filter only (not required to see data)
- [x] Empty state: "No options flow detected. Connect supported data source or wait for activity."
- [x] Add /option-flow route in App.tsx
- [x] Update Command Center "Option Flow" card to route to /option-flow (new tab, target="_blank")
- [x] Update left panel "Option Flow" nav item to route to /option-flow (same tab)
- [x] Existing /options-chain page preserved (no changes)
- [x] No duplicate pages, no logic/API/scoring changes

### Polygon API Integration Fix — Option Flow
- [x] Verify VITE_POLYGON_API_KEY env var is correctly loaded and used (confirmed in apiConfig.ts)
- [x] Confirm upgraded key is active (20 rapid-fire calls all returned 200 from server-side test)
- [x] Fix request throttling — created shared polygonFetchService with serial queue + endpoint-specific delays (9s for /v2/aggs, 1s for /v3/reference)
- [x] Add response cache (5-min TTL in sessionStorage) to avoid redundant API calls across page navigations
- [x] Add API diagnostics display (DIAG button shows total calls, 429s, cache hits, avg response time, last success/error)
- [x] Improve error state — retry countdown with progress bar, last successful time, skipped symbols listed
- [x] No fallback to AAPL, no fake data — honest empty state only
- [x] Cancel duplicate in-flight requests (fetchingRef prevents concurrent scans, inFlightRequests deduplicates same-URL calls)
- [x] Unified all Polygon calls app-wide through shared service (marketDataService, optionsDataService, SectorFlow, marketRegimeService)
- [x] Progressive loading — results appear as each ticker completes, not after full scan

### Feed State Persistence & Module Isolation
- [x] Option Flow: persist loaded rows in sessionStorage via usePersistedFeed hook (key: feed_option_flow)
- [x] Option Flow: merge/dedupe new rows on refresh via feed.mergeItems() with deduplication by contract key
- [x] Option Flow: preserve filters (search, type, class) and last-updated timestamp across navigations
- [x] 0DTE Flow: independent state via useScannerStore (key: scanner_store) — sessionStorage-backed
- [x] Flow Alerts: independent state — uses empty array (no live API connected), shows honest empty state
- [x] Dark Pool Feed: independent state via sessionStorage (key: whale_flow_alerts) — persists scan results
- [x] Options Chain: independent state via sessionStorage (key: options_chain_state) — persists ticker, contracts, expirations
- [x] Stable unique keys for deduplication per module — each uses unique sessionStorage key
- [x] Auto-refresh appends/merges — feed.mergeItems() deduplicates by contract key
- [x] Manual refresh merges — same mergeItems logic, no wipe
- [x] Error state preserves existing data — shows non-blocking warning, keeps existing rows visible
- [x] Command Center cards open modules in new tabs (target="_blank" on motion.a)
- [x] Left Panel navigation remains same-tab (no target attribute on sidebar links)
- [x] No shared state causes one module to erase another — verified unique keys
- [x] No fake fallback data introduced — all modules show honest empty state when no data

### Polygon Performance Improvements & Dynamic Prioritization
- [x] Reduce REQUEST_DELAY_MS from 9000 to 300
- [x] Reduce AGGS_DELAY_MS from 9000 to 300
- [x] Reduce REF_DELAY_MS from 1000 to 200
- [x] Add CONCURRENT_LIMIT = 3 (parallel Polygon requests)
- [x] Remove static FLOW_TICKERS hardcoded list
- [x] Create dynamic scan prioritization service (watchlist → activity → movers → events → remaining)
- [x] Add scan mode selector UI: Fast (5), Standard (15), Full (watchlist/expanded)
- [x] Show "Dynamic scan universe unavailable" if no data — no silent AAPL fallback
- [x] Keep sessionStorage cache behavior
- [x] Keep progressive loading
- [x] First rows appear within a few seconds
- [x] Standard scan completes in under 30 seconds

### Reusable Content Page Template (Dropdown Pages)
- [x] Create FeaturePageTemplate component with 7 standard sections (Hero, What It Shows, How To Use, What To Look For, Common Mistakes, Example Use Case, CTA)
- [x] Template must be reusable — accepts content as props, no hardcoded text
- [x] Apply template to ONE page only: Option Flow content page
- [x] Option Flow content page has real, practical, actionable content (no placeholder/generic marketing)
- [x] Wire route for the Option Flow content page
- [x] No changes to existing pages, APIs, logic, scoring, verdicts, alerts, or data flow

### Dropdown Navigation Fix & Content Page Upgrades
- [x] Fix Features dropdown: all 6 items clickable with correct routes (desktop + mobile)
- [x] Fix Solutions dropdown: all 4 items clickable with correct routes (desktop + mobile)
- [x] Fix Resources dropdown: all 8 items clickable with correct routes (desktop + mobile)
- [x] Create/upgrade content pages for Features items: Trade Analyzer, Unusual Flow Alerts, Sweeps & Blocks, Risk Engine, Journal & Discipline, Command Center
- [x] Create/upgrade content pages for Solutions items: For Retail Traders, For Active Options Traders, For Risk-First Traders, For Traders Building Discipline
- [x] Create/upgrade content pages for Resources items: Blog, Help Desk, Customer Reviews, Video Tutorials, Roadmap, Discord Community, Affiliate Program, Markets
- [x] All pages use FeaturePageTemplate with real actionable content
- [x] No 404s from any dropdown item
- [x] Remove orphan /guide/option-flow route (merge into dropdown structure)
- [x] No changes to APIs, logic, scoring, verdicts, alerts, or data flow

### Content & Visual Upgrade for Dropdown Pages
- [x] Update FeaturePageTemplate to support visual example sections (images/diagrams)
- [x] Generate dark-theme product UI mockup visuals for each page category
- [x] Upgrade Features pages (6): specific content, real examples, product visuals
- [x] Upgrade Solutions pages (4): specific content, real examples, product visuals
- [x] Upgrade Resources pages (2): specific content, real examples, product visuals
- [x] Every page has at least one real-style trade example with ticker/contract/action/outcome
- [x] No placeholder images, no generic stock photos, no filler text
- [x] Visuals match dark-theme product UI
- [x] No changes to APIs, logic, scoring, verdicts, alerts, or data flow

### High-Resolution Visual Upgrade for Dropdown Pages
- [x] Generate high-res product UI images for Features pages (Trade Analyzer, Unusual Flow, Sweeps & Blocks, Risk Engine, Journal, Command Center)
- [x] Generate high-res product UI images for Solutions pages (Retail, Active, Risk-First, Discipline)
- [x] Generate high-res product UI images for Resources pages (Tutorials, Roadmap)
- [x] All images are retina quality (minimum 2x resolution)
- [x] Images placed at hero, how-to-use, and example sections (not just one visual per page)
- [x] No stock photos, no generic illustrations, no abstract backgrounds
- [x] Visuals match dark-theme product UI (#0A0F1C background, #2D7FF9 accent)
- [x] Images are sharp and readable on desktop and mobile
- [x] No changes to APIs, logic, scoring, verdicts, alerts, or data flow

### Layout Refactor — Match Landing Page
- [x] Audit landing page container width, spacing, card styles, typography
- [x] Refactor FeaturePageTemplate to use same container/max-width as landing page
- [x] Match section spacing (large vertical gaps between sections)
- [x] Match card styles (same bg, border, radius, shadow)
- [x] Match typography (same heading sizes, paragraph spacing)
- [x] Visual + text alignment matches landing page pattern
- [x] No new layout styles introduced — reuse landing page components
- [x] No changes to navigation, routes, content, APIs, or logic

### Layout Density Refinement — Cheddar Flow Style
- [x] Reduce vertical spacing between sections (py-16/20/28 → py-8/10/12)
- [x] Use two-column grid as primary layout (text left, visual right)
- [x] Group related content into unified blocks (reduce separate section count)
- [x] Compact card design (reduce padding, increase data density)
- [x] Align visuals directly next to supporting text
- [x] Remove unnecessary whitespace between sections
- [x] Maintain consistency across all 12 pages
- [x] No changes to landing page, navigation, routes, content meaning, APIs, or logic

### Typography Scale Upgrade
- [x] Hero headlines: 44-52px, bold (600-700), line-height 1.2
- [x] Section titles: 26-30px, semi-bold (600)
- [x] Important content (signals, steps): 18-20px, medium (500)
- [x] Body text: 16-18px, regular (400), line-height 1.4-1.6
- [x] Labels/meta: 13-14px, medium (500)
- [x] Data values (premium, volume, signals) stand out larger than surrounding text
- [x] Improved contrast for primary text
- [x] Consistent hierarchy across all 12 pages
- [x] No new pages, routes, or layout changes
- [x] All dropdown links continue to work

### Global Typography System
- [x] Define .text-hero, .text-section, .text-primary, .text-body, .text-meta utility classes in index.css
- [x] Remove all inline font-size/line-height/font-weight styles from ContentPageLayout
- [x] Remove all inline font-size/line-height/font-weight styles from FeaturePageTemplate
- [x] Replace hardcoded text-[Xpx] Tailwind classes with global typography classes
- [x] Verify no inline font overrides remain in shared components
- [x] All pages use consistent font scale from global system
- [x] No changes to layout, navigation, routes, APIs, or logic

### Premium Quality Pass — Content Pages
- [x] Rewrite FeaturePageTemplate for premium legibility (hero 44-52px, section 28-32px, body 18-20px, high contrast)
- [x] Visual-first layout: product visual near top of every page
- [x] Generate new high-resolution product UI visuals for all 12 pages
- [x] Rewrite all 12 pages with short, sharp, trader-focused copy (no filler, no marketing fluff)
- [x] Every page clearly answers: What is this? Why does it matter? What to look for? What action next?
- [x] Images are high-res, product-relevant, show realistic Trade Sentinel UI
- [x] Tight section grouping, no excessive whitespace
- [x] No routes, APIs, navigation, or logic changed

### Data-First Image Replacement (No Mockups, No Glow)
- [x] Replace TradeAnalyzerGuide images with flat data table showing verdict + constraint scores
- [x] Replace UnusualFlowGuide images with tightly-cropped flow table (ticker, premium, vol/OI, sweep/block)
- [x] Replace SweepsBlocksGuide images with order classification table (type, venue, urgency, size)
- [x] Replace RiskEngineGuide images with position sizing table and portfolio heat numbers
- [x] Replace JournalGuide images with trade log table (entry/exit, P&L, tags, grade)
- [x] Replace CommandCenterGuide images with multi-panel data view (flow feed, positions, regime)
- [x] Replace RetailTradersGuide images with unified workflow data panels
- [x] Replace ActiveTradersGuide images with multi-ticker monitoring table
- [x] Replace RiskFirstGuide images with risk metrics table (heat, drawdown, concentration)
- [x] Replace DisciplineGuide images with adherence score table and behavioral data
- [x] Replace TutorialsGuide images with lesson list table (title, duration, level, progress)
- [x] Replace RoadmapGuide images with feature status table (name, status, priority, ETA)
- [x] All images: no device mockups, no glow, no blur, no cinematic effects
- [x] All images: flat dark UI, sharp text, real contrast, tightly cropped
- [x] All images: consistent dark theme, same font scale, same table style
- [x] No routes, APIs, navigation, or logic changed

### Logo Visibility, Header Size, and Footer Features Card Fixes
- [x] Fix "TRADE" text contrast — new logo with white TRADE on dark bg
- [x] Increase header logo height to 48px desktop, 38px mobile
- [x] Make all Features page cards clickable with correct routes
- [x] Each card routes to correct existing feature page (no 404s)
- [x] No APIs, routes, or logic changed
- [x] Favicon updated to transparent-bg shield/helmet mark (matches brand)

### Blog Card Click Fix
- [x] Make blog cards clickable with pointer cursor
- [x] Route clicks to /blog (same page, scrolls to top)
- [x] No complex blog system, CMS, or dynamic content
- [x] No APIs or logic changed
