# TradeDesk IN — Autonomous Multi-Agent Paper Trading System

An autonomous, multi-agent Indian equity and F&O day-trading paper system built natively for Android with Jetpack Compose and Material 3. Seven specialized agents coordinate all eleven trading skill engines in sequence, combining deterministic NSE/BSE math and a hard Risk Gatekeeper veto with hybrid Gemini reasoning via Firebase AI Logic.

## User Review & Critical Decisions

> [!IMPORTANT] All four product decisions below were confirmed during requirements alignment and govern the architecture and default behavior of the application.

- **Confirmed Decision 1 — Hybrid Agent Execution**: All seven specialist agents combine deterministic Kotlin implementations of the 11 trading skills with structured Gemini reasoning via Firebase AI Logic (with App Check attestation). Risk gates and financial arithmetic remain 100% deterministic.
- **Confirmed Decision 2 — Market Data Source Default**: The system launches in **Offline Replay** mode backed by bundled October 19–23, 2026 NSE, Yahoo Finance, XBRL, and SEBI PIT fixtures, with a live toggle to switch to **Live Public Feeds** (Yahoo Finance, cookie-warmed NSE public endpoints, and Google News India RSS).
- **Confirmed Decision 3 — Execution Cadence & Paper Auto-Booking**: Supports both a manual **"Run Cycle"** trigger and an optional **"Auto Mode (5m)"** timer. When Auto Mode is active and the Risk Agent outputs `PASS`, the paper trade and protective alerts are automatically booked into the local paper portfolio (`paper = true` enforced unconditionally).
- **Confirmed Decision 4 — Navigation & Home Screen Placement**: A dedicated **Agents Hub** serves as the home tab. To adhere strictly to Material 3's 3–5 bottom navigation destination limit without label truncation on compact phones, the bottom `NavigationBar` (and tablet `NavigationRail`) hosts five primary tabs (**Agents**, **Scanner**, **Trade Flow**, **Portfolio**, **Intel**), while **Desk Settings** opens as a dedicated full screen from a persistent top-bar settings action.

---

## 1. Overview & Core Concept

- **What It Does**: Transforms the eleven modular day-trading workflows—market scanning, corporate news filtering, XBRL financial analysis, SEBI insider/bulk-block deal tracking, intraday ORB entry timing, exit planning, 10,000-replay Monte Carlo simulation, position budgeting, 14-point risk gating, paper broker ledgering, and alert monitoring—into both interactive drill-down workspaces and a coordinated 7-agent autonomous pipeline.
- **Target Audience / Persona**: Active Indian retail equity and F&O paper traders who want disciplined, audit-logged intraday trade discovery, strict pre-trade risk enforcement, and transparent step-by-step reasoning across NSE/BSE market rules.
- **Key Value**: Eliminates undisciplined entries and hidden cost leakage by requiring every autonomous trade idea to survive news/fundamental screening, historical probability validation, Zerodha Oct 2026 charge/tick-grid checks, and an unbypassable Risk Agent veto before a paper fill is ever recorded.

---

## 2. User Experience & Visual Design

- **Key User Flows**:
  1. **Autonomous Agent Cycle (Home — Agents Hub)**: User sees the 7-agent pipeline status strip, Paper Portfolio summary KPIs (Cash, Realized/Unrealized P&L, Daily Trades), **Offline Replay / Live Feeds** badge, a primary **"Run Cycle"** button, and an **"Auto Mode (5m)"** switch with countdown timer. Triggering a cycle animates each agent stage sequentially as structured JSON state flows forward, appending live conversational cards showing why candidates were selected, filtered, Monte Carlo-tested, vetoed by Risk, or paper-booked with armed alerts.
  2. **Skill Drill-Down Workspaces (Tabs 2–5)**:
     - **Scanner**: Switch between ORB, Daily Setups, and Custom Watchlist modes; inspect Wilder RSI(14), F&O/CAS tags, and jump any hit straight into Trade Flow.
     - **Trade Flow**: Six-step pipeline (`Entry` -> `Exit Plan` -> `Time Machine` -> `Sizing` -> `Risk Gate` -> `Broker Confirm`) featuring an interactive intraday OHLCV chart, 2,000-path Monte Carlo percentile fan chart, 24-bin return histogram, and the explicit `"yes"` paper confirmation gate for manual orders.
     - **Portfolio & Alerts**: Live FIFO open positions with one-tap Flatten, equal/volatility-weighted basket allocator, paper ledger, and rule-based alert manager (`--from-plan` rung generator + instant evaluation).
     - **Intel Hub**: Unified lookup for NSE announcements & Google News (`!` importance tags), Ind-AS & Banking XBRL quarterly results ("What changed" highlights + bonus/split EPS normalization), and SEBI PIT lag / bulk-block deal churn & followed-investor matching.
  3. **Desk Configuration (Settings)**: Manage starting paper budget, risk caps, entry time cutoffs (`09:30`–`14:45` IST), product (`MIS`/`CNC`), data source mode, replay timestamp, and inspect the 2026 NSE holiday calendar.

- **Visual Identity & Theme**:
  - *Aesthetic Direction*: Obsidian Institutional Terminal — high-contrast dark surfaces tailored for numeric clarity, crisp tabular alignment, and unmistakable semantic status differentiation.
  - *Color Tokens*: Deep slate-obsidian canvas (`background` / `surface` / elevated containers), Bull Emerald (`primary`) for gains and `PASS` verdicts, Stop Crimson (`error`) for stop levels and `BLOCK` vetoes, Amber Saffron (`secondary`) for risk warnings, late PIT tags, and `!` announcements, and Sky Cyan (`tertiary`) for Time Machine charts and active AI reasoning steps.
  - *Typography & Hierarchy*: `Space Grotesk` for screen titles and hero KPI numerals, `Plus Jakarta Sans` for crisp UI body copy and agent explanations, and `JetBrains Mono` for all prices (`₹`), quantities, NSE timestamps, percentages, and expandable JSON payloads (`sp`-scaled for accessibility).
  - *Component Styling & Spatial Rhythm*: Edge-to-edge layout respecting status and navigation bar insets, 8dp spacing grid, 48dp minimum interactive touch targets, M3 cards with subtle borders and tonal elevation, and collapsible raw JSON inspection chips inside agent chat cards.

- **Interactive Feedback & Motion**:
  - Sequential agent progress pulsing as each step transitions (`Pending` -> `Running` -> `Passed` / `Vetoed`).
  - Smooth `AnimatedVisibility` reveals for expandable JSON payloads and filter drawers.
  - Global Material 3 `SnackbarHost` surfacing cycle summaries, paper fill confirmations, and transparent AI/network error notices without hiding real failures.

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1 — Hybrid Reasoning + Deterministic Risk/Math Gate**:
  - *Chosen Approach*: Gemini (`gemini-3.8-flash` via Firebase AI Logic with structured JSON schemas) synthesizes candidate priority, event/news flags, and human-readable agent dialogue, while **all** pricing, NSE tick snapping, Zerodha taxes, Monte Carlo simulation, and the 14-point Risk Gate run deterministically in Kotlin.
  - *Why*: Eliminates LLM arithmetic hallucinations on Indian market tick sizes (`0.01`/`0.05`/`0.10`), CAS vs non-CAS square-off timings, and hard loss caps while delivering rich multi-agent reasoning.
  - *Alternatives Considered*: Pure LLM prompting for calculations (unsafe for risk enforcement) or purely silent scripts (lacks multi-agent conversational explainability).

- **Decision 2 — In-App 5-Minute Foreground Auto Loop with Instant Fallback**:
  - *Chosen Approach*: Auto Mode runs inside a ViewModel-scoped coroutine loop while active, ticking a visible 5-minute countdown and catching network/AI exceptions to fall back gracefully to deterministic Kotlin reasoning if cloud AI is temporarily unreachable (while logging a clear status note).
  - *Why*: Guarantees immediate, live interactive preview behavior without fragile background battery-killer restrictions, and ensures offline replay mode never deadlocks if offline.
  - *Alternatives Considered*: Android `WorkManager` 15-minute minimum periodic background jobs (too coarse for a 5-minute intraday cycle and invisible during active testing).

- **Decision 3 — Firebase Provisioning & App Check Security**:
  - *Chosen Approach*: Provision Firebase AI Logic (`ai_logic`) via AI Studio backend setup and wire App Check token initialization synchronously in activity startup before UI or ViewModels initialize.
  - *Why*: Required for zero-key security and prevents HTTP 403 attestation errors on Gemini inference calls.

---

## 4. Technical Architecture & Data Strategy *(Technical Reference)*

- **Architecture & Component Diagram**:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                     Jetpack Compose UI (Material 3)                     │
│  [Agents Hub (Home)] [Scanner] [Trade Flow] [Portfolio] [Intel] [Setup] │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │ StateFlow / Events
┌───────────────────────────────────▼─────────────────────────────────────┐
│                          Main ViewModel                                 │
│  • Manual "Run Cycle" + 5m Auto-Mode Countdown Loop                     │
│  • Exposes Clock, Pipeline Progress, Live Agent Log, & Skill States     │
└───────────┬─────────────────────────────────────────┬───────────────────┘
            │                                         │
┌───────────▼───────────────────────────┐ ┌───────────▼───────────────────┐
│     7-Agent Orchestrator Engine       │ │   Hybrid Gemini Advisor       │
│ 1. Orchestrator ──▶ 2. Scanner        │ │  • Firebase AI Logic          │
│ 3. News/Analyst ──▶ 4. Entry          │ │    (gemini-3.8-flash +        │
│ 5. Exit/TimeMach──▶ 6. Risk (HARD     │ │     Schema.obj JSON output)   │
│    VETO GATE)   ──▶ 7. Alerts/Paper   │ │  • 60s timeout + App Check    │
└───────────┬───────────────────────────┘ └───────────┬───────────────────┘
            │ Passes & mutates Shared JSON State      │
┌───────────▼─────────────────────────────────────────▼───────────────────┐
│               11 Deterministic Kotlin Skill Engines                     │
│  Scanner & Entry │ Exit & TimeMachine │ Budget, Risk & Paper │ Intel    │
└───────────┬─────────────────────────────────────────┬───────────────────┘
            │                                         │
┌───────────▼───────────────────┐ ┌───────────────────▼───────────────────┐
│   Room Local Persistence      │ │       Market Data Source              │
│  • Desk Config & Portfolio    │ │  • Offline Replay (Oct 2026 Fixtures) │
│  • Broker Fills & Alert Rules │ │  • Live Public Feeds (Yahoo/NSE/RSS)  │
│  • Agent Cycle Logs & JSON    │ │                                       │
└───────────────────────────────┘ └───────────────────────────────────────┘
```

- **Data Model & State**:
  - **Shared Pipeline JSON State (`AgentCycleState`)**: Serializable payload passed sequentially across the 7 agents holding `cycleId`, `timestampIst`, `dataMode`, `watchlistCandidates`, `filteredSymbol`, `newsAnalystSummary`, `entryCheck`, `tradePlan`, `timeMachineStats`, `orderProposal`, `riskVerdict`, `alertsArmed`, `paperFillSummary`, and `haltedReason`.
  - **Agent Conversation Entries (`AgentLogMessage`)**: Stores agent role, invoked skill tag, verdict badge (`INFO` / `SELECTED` / `FILTERED` / `PASS` / `VETO_BLOCK` / `PAPER_FILLED` / `AI_FALLBACK`), concise explanation, and serialized JSON snapshot for interactive expansion.
  - **Persistent Room Entities**: `DeskStateEntity` (config & portfolio snapshot), `BrokerRecordEntity` (immutable paper fill ledger with 16-char SHA-256 fingerprint), `AlertRuleEntity`, `FollowedInvestorEntity`, and `AgentCycleRecordEntity` (persisted cycle history & chat messages).

- **Interactive Component & State Mapping**:
  - **"Run Cycle" Button / Auto Mode Switch**: Starts sequential coroutine execution through Agents 1 -> 7, updating an active-agent progress Flow in real time. Each stage updates the shared `AgentCycleState` JSON and appends an `AgentLogMessage` card to the live log.
  - **Risk Agent Hard Veto Gate**: Inspects the 14 deterministic risk rules against current IST time/holiday/budget/exposure/R:R. On any failure, marks Risk Agent `VETO_BLOCK`, clears order submission, prevents Agent 7 from executing, and appends a clear crimson veto log entry explaining every failed check.
  - **Expandable JSON State Chips**: Clicking a log card toggles visibility of the exact JSON state passed out by that agent.
  - **Candidate Shortcut Actions**: Tapping an approved or vetoed symbol on an agent card navigates directly into **Trade Flow** or **Intel** pre-loaded with that symbol for manual inspection or parameter override.
  - **Data Mode & Settings Controls**: Toggling between Offline Replay and Live Public Feeds updates persistent config, refreshes the NSE clock/holiday banner, and forces the next cycle to pull from the selected source.

- **Key Sequences & Integrations**:
  - **Firebase Provisioning & App Check Sequence**: Extract debug keystore fingerprints -> request user consent via AI Studio Firebase setup UI -> verify acceptance & provision `ai_logic` -> configure App Check synchronously on activity launch with `commit()` debug persistence before Compose UI mounts.
  - **Verification Gate**: All 11 deterministic domain engines and the 7-agent orchestrator (including Risk Veto halting and FIFO paper accounting) will be verified via JVM/Robolectric unit tests and full APK compilation before completion.
