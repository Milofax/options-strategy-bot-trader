# SPX 0DTE Iron Condor Trading Bot Product Requirements Document (PRD)

## Goals and Background Context

### Goals
- Create multi-strategy trading bot platform with unified dashboard (not TWS replication)
- Phase 1: Automate daily 0DTE Iron Condor trading on SPX with 9:32 ET execution
- Phase 2: Add weekly 45 DTE SPY Strangle with rolling capability
- Implement robust exit management with configurable profit targets and time-based closes
- Enforce comprehensive risk controls including 300% stop-loss and kill-switch capability
- Enable parallel strategy testing through Experiment Mode without capital risk
- Provide strategy-centric monitoring with status tracking, P/L calculation, and dual time tracking (contract expiry vs strategy exit)
- Track performance metrics per strategy with detailed logs and future parameter tuning
- Ensure system resilience with automatic state recovery and collision prevention
- Enable "set-and-forget" operation suitable for vacation periods
- Create extensible architecture for additional strategies (Flyogonal, etc.)

### Background Context
The idea for this project was to automate a proven options trading strategy that has demonstrated an 83.7% win rate over 300+ backtested trades: the "Daily 0DTE SPX Iron Condor". The current manual process requires daily attention at specific times, making it incompatible with travel or other commitments. By automating the strategy through Interactive Brokers' TWS API (via TWS or IB Gateway), we eliminate human error, ensure consistent execution, and free the trader from daily operational tasks while maintaining the strategy's edge. The system is designed with safety-first principles, ensuring that automation enhances rather than endangers the proven strategy's performance. Critical constraints include single trade per day limits, US market hours operation only, strict adherence to the proven strategy without autonomous optimization, and not overriding but intelligently accepting manual changes made in TWS by the user while a strategy order is currently running.

**Strategy Validation & Correlation Considerations:** Both SPX Iron Condor (Phase 1) and SPY Strangle (Phase 2) strategies have been extensively backtested over 3+ year periods, demonstrating consistent performance across various market conditions. While SPX and SPY correlation during stress events is acknowledged, the proven historical performance and individual strategy risk controls (300% stop-losses, time-based exits) provide adequate risk management for the current implementation. Portfolio-level correlation analysis and VIX-based regime detection are designated for future enhancement phases but are not required for initial deployment given the robust individual strategy performance and comprehensive risk controls already in place.

### Change Log
| Date | Version | Description | Author |
|------|---------|------------|--------|
| 2024-01-14 | 1.0 | Initial PRD creation | John (PM) |
| 2024-01-14 | 1.1 | Added risk management and experiment mode goals | John (PM) |
| 2025-01-18 | 2.0 | MAJOR: Added Meta-Strategy Model for complete UI/Logic separation | John (PM) |
| 2025-01-18 | 2.1 | MAJOR: Introduced 9-line Kanban card structure (universal UI abstraction) | John (PM) |
| 2025-01-19 | 2.2 | Standardized emergency close terminology to CLOSE MKT | John (PM) |
| 2025-01-19 | 2.3 | Clarified archiving rules: manual/30-day auto-archive flow for closed trades | John (PM) |
| 2025-01-19 | 2.4 | Added OCA cancellation logic: only cancel if OCA group exists | John (PM) |
| 2025-01-24 | 2.5 | Added Epic 7: Complete Experiment Mode with Strategy Testing & Validation | John (PM) |
| 2025-01-24 | 2.6 | Revised SPY Strangle to use Delta 16, added Meta-Strategy references to Epics, reduced redundancies | John (PM) |
| 2025-01-24 | 2.7 | Added comprehensive performance metrics for strategies and portfolio (CAGR, Drawdown, MAR, etc.) | John (PM) |

## Requirements

### Functional Requirements

- **FR1:** Execute Iron Condor trades daily at 9:32 ET (2 minutes after market open) on SPX 0DTE options
- **FR2:** Select short strikes targeting Delta 15 (±1.9 tolerance, preferring higher delta when equidistant)
- **FR3:** Place long strikes exactly 20 points away from selected short strikes
- **FR4:** Submit Iron Condor as single combo/BAG order to ensure atomic execution
- **FR5:** Create three-order OCA group for exits: 25% profit target (LMT), 11:30 ET close (MKT), 300% stop-loss (STP). **CRITICAL**: Discord CRITICAL alert on FIRST exit order failure (not after retries). On technical failure: automatic retry with configurable attempts (default: 30 attempts over 10 minutes). If all retries fail: Position enters UNPROTECTED state (no exit orders active), emergency market close required. Discord alerts sent immediately on first error and continuously during retry period with current P/L and emergency close option
- **FR6:** Support two trading modes: "Live" (real orders to TWS) and "Experiment" (hypothetical tracking only)
- **FR7:** Maintain persistent state to recover from crashes and prevent duplicate trades
- **FR8:** Send notifications via Discord for trade execution, errors, and critical events
- **FR9:** Provide web dashboard showing real-time status, P/L, and trade history
- **FR10:** Track and store all trade data including entry/exit prices, times, and reasons. Include user-editable notes field per trade for personal insights and commentary. Archive instances: Closed instances can be manually archived OR automatically after configurable days (default: 30 days). Active/running trades cannot be archived. After emergency market close, instances move to closed state and follow standard archiving rules
- **FR11:** Implement kill-switch accessible via Discord and web interface with confirmation dialog
- **FR12:** Respect US market calendar, skipping weekends and market holidays
- **FR13:** Order execution with retry mechanism: Submit order as LMT at current mid-price, cancel and retry with updated mid-price at configurable intervals (default: every 5 seconds) until configurable timeout (default: 1 minute total). After timeout or max retries, mark instance as failed - enable manual archiving. **Skip function**: Available for scheduled instances with mandatory safety confirmation. **Stop function**: Available during active operations with confirmation. Stopped/skipped instances archive with SKIPPED status and null P/L, failed instances with ERROR status and null P/L
- **FR14:** **Position Sizing with Risk Management:**
  - **Fixed Contract Mode:** User sets number of contracts (default: 1)
  - **Dynamic Position Sizing Mode:** 
    - Base calculation: Min(% Net Liq per Trade, Available within Buying Power limit)
      - % Net Liq per Trade (default: 3%)
      - % Overall Buying Power Usage limit (default: 50%)
    - VIX-based adjustments (VIX value max 5 seconds old):
      - VIX > threshold X (default: 30): Reduce position by 50% (minimum 1 contract)
      - VIX > threshold Y (default: 50): Skip trade entirely
    - Pre-trade validation in Column 1 (NEXT): Display warnings for:
      - Insufficient Net Liquidity for requested position size
      - Insufficient Buying Power/Margin
      - VIX threshold conditions
    - Execution validation in Column 2 (SEARCH OPTIONS): Block trade with error for:
      - Cannot meet minimum position size (1 contract) due to capital constraints
      - VIX > threshold Y
      - Margin requirements cannot be met
- **FR15:** Export trade history in CSV format for external analysis
- **FR16:** Execute SPY Strangle trades weekly at configurable time (default Monday 11:45 ET / 18:45 CET, 2.25 hours after market open) using 45 DTE options. Skip entry if not a trading day. Skip entry if mandatory 21 DTE close date (Thursday, 24 calendar days forward) falls on non-trading day or after December 31st to prevent year-end tax complications. Record skip reason for audit trail (Phase 2)
- **FR17:** Select SPY Strangle strikes targeting Delta 16 (±1.9 tolerance, preferring higher absolute delta when equidistant - same selection logic as SPX Iron Condor). Initial delta configurable in settings (default: 16). Submit Strangle as single combo/BAG order to ensure atomic execution (Phase 2)
- **FR18:** [FUTURE ENHANCEMENT - Not in MVP] Implement defensive rolling: when strike tested (ATM/ITM) at ≤28 DTE, roll untested side to target Delta 30 (±1.9 tolerance, preferring higher absolute delta when equidistant). Maximum one roll per position (First-Touch-Rule). Continuous monitoring during market hours. Send standard notification when any leg becomes ATM/ITM at ≤28 DTE. Send escalated/prominent notification for any subsequent touches after First-Touch-Rule has been applied
- **FR19:** Enforce mandatory close at 21 DTE for SPY Strangle regardless of P/L or roll status, executed on Thursday at 11:30 ET / 18:30 CET (2 hours after market open) on normal trading days, adjusted for half trading days (Phase 2)
- **FR20:** Track dual time remaining: options expiry time (DTE) AND strategy exit time (TTC - Time to Close) for all positions. Store time values as days with decimal precision
- **FR21:** Provide streamlined installation and setup process via Homebrew with private tap distribution, web-based configuration wizard, and automatic browser launch for initial setup
- **FR22:** Instance ID format: [STRATEGY_ABBREV]-[YYMMDD]-[###] where STRATEGY_ABBREV is max 6 characters (e.g., SPXIC-240115-001). Each strategy must define its unique abbreviation
- **FR23:** Unified retry configuration: **Entry LMT orders**: Use executionWindow (default: 60 seconds) with mid-price updates, 12 attempts distributed across window. **All errors and failures**: Standard error retry with 30 attempts at 5-second intervals (2.5 minutes total). Emergency close uses same error retry configuration. ExecutionWindow only applies to entry orders; exit orders and market orders use error retry on failure
- **FR24:** TWS state reconciliation: Poll TWS every 10 seconds during retry operations to detect manual interventions (position closed, OCA manually set). Adapt system state accordingly and notify via Discord
- **FR25:** Orphaned order detection: After position closure, scan TWS for any open orders matching the closed trade's strike prices and expiration dates. This includes both OCA group orders and standalone orders that may have been manually created. Alert via Discord CRITICAL when orphaned orders are detected, requiring manual cleanup in TWS before archiving
- **FR26:** **VIX Data Integration:** Real-time VIX value retrieval via TWS API for position sizing decisions
  - Request VIX index value (symbol: VIX) with max 5-second cache
  - Architecture decision by Architect: Direct API call vs cached approach
  - Required for Dynamic Position Sizing Mode
  - Fallback: If VIX unavailable, use Fixed Contract Mode with warning

### Non-Functional Requirements

- **NFR1:** System must maintain connection to IB Gateway with automatic reconnection on disconnect
- **NFR2:** All sensitive credentials must be encrypted and stored in environment variables
- **NFR3:** System must run continuously in containerized environment (Podman) on Mac Mini server
- **NFR4:** Response time for dashboard updates must be under 2 seconds
- **NFR5:** System must log all actions with timestamps for audit trail
- **NFR6:** Database must handle minimum 5 years of trade history without performance degradation
- **NFR7:** System must send critical alerts within 10 seconds of event occurrence
- **NFR8:** Architecture must support addition of new strategies without modifying core engine
- **NFR9:** System must handle IB Gateway daily restart gracefully at any configured time
- **NFR10:** Web interface must be mobile-responsive for iPhone access
- **NFR11:** Store and process times in user-configurable timezone (default CET) and market time (ET). System must maintain both user timezone and market timezone for all time-based operations.

## Meta-Strategy Model

The Meta-Strategy Model is the core abstraction that separates trading logic from UI representation, enabling infinite strategy variations through configuration alone.

### Core Components

```yaml
strategy:
  id: unique_strategy_id  # e.g., "spx_ic_0dte"
  name: Human-readable name  # e.g., "SPX Iron Condor 0DTE"
  abbreviation: Max 6 chars  # e.g., "SPXIC" (for instance IDs)
  
  schedule:
    type: daily | weekly
    marketOpenOffset: integer  # Minutes after market open
    daysOfWeek: [1-7]  # For weekly: 1=Monday, 5=Friday
    
  underlying:
    symbol: SPX | SPY  # Ticker symbol
    
  entry:
    type: ironCondor | strangle
    dte: integer  # Days to expiration (0, 45, etc.)
    strikes:
      method: delta  # Currently only delta-based selection
      parameters:
        shortDelta: integer  # Target delta (15 for IC, 16 for strangle)
        tolerance: 1.9  # Delta selection tolerance
        wingWidth: integer  # For IC only: distance to long strikes
    quantity: integer  # Or use dynamic position sizing
    
  exit:
    orders:  # Array of exit orders (OCA group)
      - type: profitTarget
        trigger: percentage
        value: integer  # Profit percentage (25%, 50%)
        orderType: LMT
      - type: stopLoss
        trigger: percentage
        value: integer  # Loss percentage (200%, 300%)
        orderType: STP
      - type: timeExit
        trigger: marketOpenOffset
        value: integer  # Minutes after open
        orderType: MKT
      - type: dteBased  # For SPY strangle
        trigger: dteWithOffset
        value: integer  # DTE threshold (21)
        offsetValue: integer  # Minutes after open on that day
        orderType: MKT
  rolling:  # Optional, for SPY strangle
    enabled: boolean
    trigger: strike_tested  # When ATM/ITM at ≤28 DTE
    maxRolls: 1  # First-Touch-Rule
    targetDelta: integer  # New delta target (30)
    tolerance: 1.9
    # Monitored continuously with position data updates
```

**Note:** Position sizing (fixed vs dynamic) is handled at the system level, not per-strategy configuration.

### Key Principles

1. **Complete Separation**: Trading logic (Meta-Strategy) never knows about UI representation
2. **Configuration-Driven**: New strategies added via YAML, not code
3. **State Independence**: Each instance maintains its own state through Trading State Model
4. **UI Abstraction**: All strategies render identically in the UI (9-line card structure)

## Trading State Model

Universal state machine for all strategy instances:

```
SCHEDULED → SEARCHING → ORDERING → PROTECTING → ACTIVE → CLOSED → ARCHIVED
```

### State Definitions

- **SCHEDULED**: Waiting for execution time
- **SEARCHING**: Finding option contracts
- **ORDERING**: Placing and filling orders
- **PROTECTING**: Setting up exit orders (OCA group)
- **ACTIVE**: Position live, monitoring P/L
- **CLOSED**: Position closed, P/L finalized
- **ARCHIVED**: Historical record

### Sub-States

Each primary state can have sub-states:
- READY, WAIT, WARNING (for SCHEDULED)
- DOING, ERROR (for action states)
- RUNNING, WARNING, ROLLING, ERROR (for ACTIVE)
- DONE, ERROR, SKIPPED (for terminal states)

## Dashboard Architecture

### Revolutionary Kanban-Based Approach

The dashboard uses a Kanban board metaphor that maps directly to the Trading State Model:

```
┌─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐
│  NEXT   │ SEARCH  │ PLACE   │ SETUP   │ ACTIVE  │ CLOSED  │ ARCHIVE │
│         │ OPTIONS │ ORDER   │ EXIT    │         │         │         │
├─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
│  Cards  │  Cards  │  Cards  │  Cards  │  Cards  │  Cards  │  Cards  │
└─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘
```

### Column Mapping
1. **NEXT** → SCHEDULED state
2. **SEARCH OPTIONS** → SEARCHING state
3. **PLACE ORDER** → ORDERING state
4. **SETUP EXIT** → PROTECTING state
5. **ACTIVE** → ACTIVE state
6. **CLOSED** → CLOSED state
7. **ARCHIVE** → ARCHIVED state

### Universal 9-Line Card Structure

EVERY card in EVERY column follows this exact structure:

```
Line 1: [Mode Badge] [Instance ID] [Status Badge]
Line 2: ─────────────────
Line 3: [Content based on column/state]
Line 4: [Content based on column/state]
Line 5: [Empty or content]
Line 6: [Empty or content]
Line 7: [Empty or content]
Line 8: ─────────────────
Line 9: [Primary Action Button if applicable]
```

This rigid structure ensures:
- Consistent visual hierarchy
- Predictable information placement
- Clean abstraction from strategy specifics
- Mobile-responsive design

## Strategy Configuration Examples

### SPX 0DTE Iron Condor (Phase 1)

```yaml
strategy:
  id: spx_ic_0dte
  name: SPX Iron Condor 0DTE
  abbreviation: SPXIC
  
  schedule:
    type: daily
    marketOpenOffset: 2  # Minutes after market open (configurable, default: 2)
    # Results in 09:32 ET when market opens at 09:30 ET
    
  underlying:
    symbol: SPX
    
  entry:
    type: ironCondor
    dte: 0
    strikes:
      method: delta
      parameters:
        shortDelta: 15  # Configurable (default: 15)
        tolerance: 1.9
        wingWidth: 20
    quantity: 1  # Or dynamic based on position sizing
    
  exit:
    orders:
      - type: profitTarget
        trigger: percentage
        value: 25  # Configurable (default: 25%)
        orderType: LMT
      - type: timeExit
        trigger: marketOpenOffset
        value: 120  # Minutes after market open (configurable, default: 120)
        # Results in 11:30 ET when market opens at 09:30 ET
        orderType: MKT
      - type: stopLoss
        trigger: percentage
        value: 300  # Configurable (default: 300%)
        orderType: STP
```

### SPY Weekly Strangle (Phase 2)

```yaml
strategy:
  id: spy_strangle_45dte
  name: SPY Strangle Weekly
  abbreviation: SPYST
  
  schedule:
    type: weekly
    marketOpenOffset: 135  # Minutes after market open (configurable, default: 135)
    # Results in 11:45 ET when market opens at 09:30 ET
    daysOfWeek: [1]  # Monday
    
  underlying:
    symbol: SPY
    
  entry:
    type: strangle
    dte: 45
    strikes:
      method: delta
      parameters:
        shortDelta: 16  # Configurable (default: 16)
        tolerance: 1.9  # Same logic as SPX IC
    quantity: 1  # Or dynamic based on position sizing
    
  exit:
    orders:
      - type: profitTarget
        trigger: percentage
        value: 50  # Configurable (default: 50%)
        orderType: LMT
      - type: stopLoss
        trigger: percentage  
        value: 200  # Configurable (default: 200%)
        orderType: STP
      - type: dteBased
        trigger: dteWithOffset
        value: 21  # Mandatory close at 21 DTE
        offsetValue: 120  # Minutes after market open (configurable, default: 120)
        # Results in 11:30 ET on Thursday when market opens at 09:30 ET
        orderType: MKT
        
  # Rolling disabled for initial implementation
  # Future enhancement: Add defensive rolling (FR18)
```

## Technical Architecture Overview

### System Components

- **Core Engine**: Python-based monolith in Podman container
- **TWS Integration**: ib_async for reliable IB Gateway connection
- **Web Framework**: FastAPI for REST API and WebSocket support
- **Frontend**: HTMX + Server-Sent Events for real-time updates
- **Database**: SQLite for persistence with encrypted secrets
- **Notifications**: Discord.py for alerts and remote control
- **Deployment**: Mac Mini server with Podman orchestration

### Key Design Decisions

- **Modular Monolith**: Single deployable unit with clear service boundaries
- **Strategy Registry**: Dynamic loading of strategy configurations
- **State Machine**: Centralized state management with event-driven transitions
- **Retry Patterns**: Unified retry logic for orders and errors
- **Market Hours**: Dynamic fetching and caching from IB API
- **OCA Management**: Atomic creation of exit order groups
- **Position Reconciliation**: Startup sync between database and TWS
- **Emergency Controls**: Kill-switch and manual intervention handling

### Technology Stack

- **Backend**: Python 3.11+, ib_async v2.0.1, FastAPI
- **Frontend**: HTMX, Tailwind CSS, DaisyUI (Aqua theme)
- **Database**: SQLite with SQLAlchemy ORM
- **Messaging**: Discord.py for notifications
- **Container**: Podman for deployment
- **Monitoring**: Built-in health checks and status endpoints
- **Configuration (Hybrid Approach):** 
  - YAML files for non-sensitive trading parameters (deltas, timing, position sizing)
  - Encrypted SQLite table for secrets (Discord token, IB credentials, API keys)
  - Master encryption key stored in macOS Keychain
  - Python cryptography library (Fernet) for symmetric encryption

### Order Execution Framework Note for Architect
All strategies use unified order execution pattern: mid-price calculation from IB quotes, LMT order submission, and configurable retry logic with updated pricing on each attempt. Implement generic order execution engine with per-strategy configuration parameters: retry intervals (default: 5 seconds), timeout duration (default: 1 minute), and max attempts (default: 12). Strategy-specific execution windows (SPX IC: critical 9:32-9:33 ET timing, SPY Strangle: relaxed Friday evening timing) handled through different timeout configurations. All strategies use combo/BAG orders for atomic execution with ib_async bracketOrder() helper function and oneCancelsAll() method for OCA group management. **OCA Implementation Details**: Use ib.bracketOrder('BUY', quantity, limitPrice, takeProfitPrice, stopLossPrice) to create three-order structure, then place each order individually. For manual control, use oneCancelsAll([profit_order, stop_order, time_order], ocaGroup="strategy_id", ocaType=1) for custom OCA groups. This unified approach reduces code duplication while maintaining strategy-specific flexibility.

### Operational Edge Case Handling Requirements for Architect
The system must handle critical operational failures gracefully to prevent capital loss and ensure reliable automated trading. Key requirements:

**System Recovery & State Management:**
- Implement position reconciliation on startup by comparing database state with TWS actual positions
- Handle bot crash scenarios during critical windows (e.g., 9:31-9:35 ET for SPX strategy) with missed window detection and trader alerts
- Ensure database transaction consistency during order execution - treat entry order + exit orders as atomic unit
- Implement graceful degradation during network outages with position status caching and recovery procedures

**Market Disruption Handling:**
- Handle trading halts on underlying securities by queuing exit orders until market reopens
- Manage circuit breaker scenarios with extended exit windows when markets are paused
- Account for early assignment risks on American-style options (SPY) with position status monitoring
- Adapt execution times for half-trading days while maintaining strategy integrity

**Connection & TWS Integration:**
- Implement robust TWS connection management with automatic reconnection and session restoration
- Handle TWS daily restart gracefully during market hours without losing active orders or positions
- Implement API rate limiting and backoff strategies to prevent connection throttling
- Ensure order status verification after connection restore to prevent duplicate or missed orders

**Error Classification & Response:**
- Define clear error severity levels: INFO (recoverable), WARNING (requires attention), CRITICAL (immediate action required), FATAL (system halt required)
- Implement escalating notification system with configurable channels (Discord, email, SMS)
- Ensure all fatal errors trigger immediate trader notification with system status and recommended actions
- Log all operational errors with sufficient context for post-incident analysis and system improvement

**Fail-Safe Mechanisms:**
- Default to safe mode on ambiguous system state - prefer no action over potentially harmful action
- Implement emergency position closure mechanisms when system integrity is compromised
- Ensure kill-switch functionality remains accessible even during system degradation
- Maintain audit trail of all system decisions and error responses for regulatory and debugging purposes

### Market Hours Integration
The system will dynamically fetch and cache market hours from IB API to handle RTH-relative timing, DST transitions, and market holidays.

## Epic List - Value-Driven Increments

### NEW APPROACH: Each Epic Delivers Immediate User Value
**Philosophy:** Every Epic must deliver a working, testable increment that provides concrete benefit to the trader. No more technical-only epics that require multiple phases before seeing value.

### Epic 1: One-Click Daily Trade Execution with Meta-Strategy Foundation
**Goal:** Implement the Meta-Strategy Model architecture and deliver immediate value through automated SPX Iron Condor execution.

**User Value:** 
- Saves 16+ manual clicks in TWS
- Eliminates strike selection errors
- Ensures OCA orders are properly configured
- Trade can still be managed in TWS after setup

**Deliverables:**
- **Meta-Strategy Model Implementation** (see Meta-Strategy Model section)
  - Abstract strategy engine with configuration-driven behavior
  - SPX Iron Condor configuration (see "SPX 0DTE Iron Condor" config)
- Minimal web UI with "Schedule Trade" button
- Configurable parameters matching SPX IC strategy config:
  - Entry Offset: [2] min after open → "09:32 ET (15:32 CET)"
  - Exit Offset: [120] min after open → "11:30 ET (17:30 CET)"
  - Profit Target (default: 25%)
  - Stop Loss (default: 300%)
  - Position Size (default: 1 contract)
  - Short Delta (default: 15, tolerance: ±1.9)
  - **UI displays calculated times in both ET and user timezone**
- **Execution Logging with Error Handling:**
  - Real-time execution log with timestamps (YY-MM-DD HH:MM:SS format)
  - Clear error messages for all failure scenarios:
    - No valid strikes found at target delta
    - TWS connection failures
    - Order rejection reasons
    - Insufficient buying power
    - Market circuit breaker active
  - Server-Sent Events (SSE) for live updates during execution
  - Log retention: Current day only (cleared at midnight)
  - All errors logged with actionable descriptions
- TWS connection and order execution per strategy config
- Podman deployment ready
- Playwright E2E test suite

**Success Criteria:**
- [ ] Meta-Strategy Model correctly processes SPX IC configuration
- [ ] 5 successful trades executed matching strategy parameters
- [ ] Runs stable in Podman container
- [ ] State survives container restart
- [ ] TWS connection works from container
- [ ] Strike selection follows Delta 15 ± 1.9 rule from config
- [ ] OCA group creation matches exit order specifications
- [ ] Playwright tests validate strategy execution
- [ ] Architecture ready for additional strategies

### Epic 2: Minimal Kanban with Full Automation
**Goal:** Implement the Dashboard Architecture with Trading State Model visualization and enable full automation.

**User Value:**
- Full "set and forget" automation
- Visual trade lifecycle tracking
- Skip functionality for vacation days
- Emergency stop capability

**Deliverables:**
- **Kanban Implementation** (see Dashboard Architecture section):
  - 7 columns mapping to Trading State Model
  - 9-line card structure per specification
- **Daily automation** per strategy schedule config
- **Control buttons:** Skip, Archive, Stop
- **Status badges** per Trading State Model sub-states
- Real-time updates via Server-Sent Events
- In-memory state only (persistence in Epic 3)

**Success Criteria:**
- [ ] Trading State Model transitions work correctly
- [ ] 9-line card structure displays properly
- [ ] Daily automation follows strategy schedule
- [ ] Control buttons function as specified
- [ ] 5 consecutive days of automated trading
- [ ] Playwright tests validate Kanban interactions

### Epic 3: Professional UI, Analytics & Persistence
**Goal:** Add professional UI polish, analytics, and full data persistence.

**User Value:**
- Professional Aqua-style interface
- Performance metrics and analytics
- Trade history survives restarts
- Mobile-optimized access

**Deliverables:**
- **Aqua UI Polish:** Gradients, glass morphism, animations, dark mode
- **Swimlane Architecture:** Strategy rows with statistics bars
- **Performance Metrics für Einzelstrategien:**
  - P/L ($): Absoluter Gewinn/Verlust in Dollar
  - CAGR (%): Compound Annual Growth Rate (annualisierte Rendite)
  - Max. Drawdown (%): Maximaler Wertverlust vom Höchststand
  - MAR Ratio: CAGR / Max. Drawdown (Risk-adjusted Return)
  - Win Percentage (%): Prozentsatz profitabler Trades
  - Avg. Minutes in Trade: Durchschnittliche Haltedauer in Minuten
  - # Trades (int): Gesamtzahl der Trades
  - # Winners (int): Anzahl profitabler Trades
  - # Losers (int): Anzahl verlustbringender Trades
  - Daily/Weekly/Monthly Breakdown
- **Portfolio-Metriken (aggregiert über alle aktiven Strategien):**
  - Enthaltene Strategien: Liste der laufenden Strategien
  - P/L ($): Gesamt-P/L aller Strategien
  - CAGR (%): Portfolio-weite annualisierte Rendite
  - Max. Drawdown (%): Portfolio-weiter maximaler Drawdown
  - MAR Ratio: Portfolio CAGR / Portfolio Max. Drawdown
- **Trade Log:** Sortable/filterable table with CSV export (FR15)
- **SQLite Persistence:** Database schema, state recovery, backups
- **Settings Modal:** Categorized settings, validation, save confirmation

**Success Criteria:**
- [ ] UI matches Aqua design mockups
- [ ] All data persists across restarts
- [ ] CSV export works correctly
- [ ] P/L ($) zeigt absoluten Gewinn/Verlust korrekt
- [ ] CAGR (%) berechnet annualisierte Rendite korrekt
- [ ] Max. Drawdown (%) trackt maximalen Wertverlust vom Höchststand
- [ ] MAR Ratio berechnet sich korrekt als CAGR / Max. Drawdown
- [ ] Win Percentage (%) berechnet korrekt (winners/total trades * 100)
- [ ] Avg. Minutes in Trade zeigt durchschnittliche Haltedauer
- [ ] # Trades, Winners, Losers werden korrekt gezählt
- [ ] Portfolio-Metriken aggregieren korrekt über alle aktiven Strategien
- [ ] Daily/Weekly/Monthly Breakdowns funktionieren
- [ ] Mobile layout works on iPhone
- [ ] Settings modal validates inputs
- [ ] Delete functionality removes trades properly
- [ ] Backup system runs daily
- [ ] Database handles 5 years of data
- [ ] Trade states survive restart
- [ ] Metrics recalculate from DB on startup
- [ ] Performance: Metric updates <100ms
- [ ] Playwright tests for UI interactions

### Epic 4: Setup Wizard & Localization
**Goal:** Provide user-friendly initial setup and proper localization, making the system accessible and correctly displaying times in user's timezone.

**User Value:**
- Guided setup process instead of manual config editing
- See times in your local timezone (not just ET)
- Offset-based configuration with real-time preview
- German language support for European users
- TWS connection testing before going live
- Automatic browser launch on first run

**Deliverables:**
- **Web-based Setup Wizard (4 screens):**
  - Screen 1: Language selection (English/German)
  - Screen 2: Timezone selection (for display purposes)
  - Screen 3: TWS connection config and test
  - Screen 4: Discord integration (optional)
- **Settings Modal (post-setup):**
  - Trading parameters with offset-to-time display:
    - Entry Offset input: [___] minutes after open → "09:32 ET (15:32 CET)"
    - Exit Offset input: [___] minutes after open → "11:30 ET (17:30 CET)"
    - Real-time calculation showing both ET and user timezone
  - All strategy-specific parameters configurable here
- **Localization Features:**
  - User timezone selection and display
  - Dual time display: "9:32 ET (15:32 CET)"
  - German translation for all UI elements
  - Locale-specific number/date formatting
- **Connection Testing:**
  - Test TWS connection button
  - Verify market data subscriptions
  - Check account permissions
  - Display account balance
- **First-Run Experience:**
  - Auto-detect first run
  - Launch wizard automatically
  - Save settings to encrypted storage
  - Auto-open dashboard after setup
- **Homebrew Installation:**
  - Private tap for easy installation
  - Automatic dependency management
  - Launch scripts included
  - Update mechanism

**Success Criteria:**
- [ ] Wizard completes in <2 minutes
- [ ] TWS connection test works
- [ ] Settings persist correctly
- [ ] Browser auto-launches
- [ ] Homebrew install works on macOS
- [ ] German translation complete
- [ ] Times show in user timezone
- [ ] Dual time display works
- [ ] Can skip Discord setup
- [ ] Wizard accessible from settings
- [ ] All times display in user timezone
- [ ] German translation complete
- [ ] Locale-specific formatting works
- [ ] Can re-run wizard from settings

### Epic 5: Minimal Discord Integration
**Goal:** Implement critical Discord notifications and kill-switch per requirements.

**User Value:**
- Critical alerts for errors
- Emergency stop from anywhere
- Peace of mind through notifications

**Deliverables:**
- Discord Notifications (FR8): Trade execution, errors, critical events
- Exit Order Failure Alerts (FR5): Immediate alert, retry updates, UNPROTECTED state
- Orphaned Order Detection alerts (FR25)
- Kill Switch command (FR11): !kill with confirmation
- TWS State Change notifications (FR24)

**Explicitly NOT Included:** Status commands, skip/pause, summaries, P/L queries

**Success Criteria:**
- [ ] Discord bot connects and authenticates
- [ ] Critical alerts fire within 10 seconds of events
- [ ] Kill-switch works with confirmation
- [ ] Exit order failure alerts work as specified
- [ ] Orphaned order detection functional
- [ ] 5 days of reliable notifications
- [ ] Zero false positives in critical alerts

### Epic 6: Dynamic Position Sizing with Risk Management
**Goal:** Implement intelligent position sizing with VIX-based risk controls.

**User Value:**
- Automatic sizing based on account metrics
- VIX-based volatility protection
- Protection against over-leveraging
- Switch between Fixed/Dynamic modes

**Deliverables:**
- **Dynamic Position Sizing (FR14):**
  - % of Net Liquidity calculation (default: 3%)
  - Buying power limit (default: 50%)
  - Real-time TWS account metrics
- **VIX Integration (FR26):**
  - Real-time VIX retrieval (5-second cache)
  - Position reduction: VIX > 30 (halve), VIX > 50 (skip)
  - Configurable thresholds
- **Pre-Trade Validation:** Warnings in Column 1, blocks in Column 2
- **Settings:** Mode toggle, threshold configuration, size preview
- **Fallback Logic:** VIX unavailable → Fixed mode, insufficient capital → block

**Success Criteria:**
- [ ] Dynamic sizing calculates correctly based on account metrics
- [ ] VIX data retrieves within 5 seconds
- [ ] Position adjustments display clear warnings before execution
- [ ] Settings changes take effect immediately
- [ ] 10 successful trades with dynamic sizing
- [ ] VIX > 30 condition correctly halves position
- [ ] VIX > 50 condition correctly blocks trade
- [ ] Buying power limit never exceeded
- [ ] Smooth switching between Fixed and Dynamic modes

### Epic 7: Complete Experiment Mode with Strategy Testing & Validation
**Goal:** Enable safe, parallel strategy testing without capital risk while providing comprehensive validation tools and advanced error recovery mechanisms, allowing traders to test new parameters and strategies before going live.

**User Value:**
- Test new strategies and parameters without risking real money
- Run experiments parallel to live trading for A/B testing
- Validate strategy performance before deployment
- Advanced error recovery ensures trades are never left unprotected
- Complete confidence through comprehensive testing capabilities

**Deliverables:**
- **Full Experiment Mode Implementation (FR6):**
  - Separate EXPERIMENT swimlane below LIVE trading
  - Hypothetical order tracking without TWS submission
  - Simulated fills using mid-price at execution time
  - Parallel P/L tracking with live prices
  - Clear visual distinction (🧪 badge vs 💰 badge)
  - Independent scheduling from LIVE mode
- **Advanced TWS State Reconciliation (FR24):**
  - Poll TWS every 10 seconds during operations
  - Detect manual interventions automatically
  - Adapt system state to manual changes
  - Discord notifications for state mismatches
  - Automatic recovery from disconnections
- **Orphaned Order Management (FR25):**
  - Automatic scan after position closure
  - Detection of orders matching closed trade parameters
  - Cleanup assistant with one-click resolution
  - Prevention of accidental order duplication
  - Clear warnings before archiving
- **Experiment Analytics & Comparison:**
  - Side-by-side P/L comparison (LIVE vs EXPERIMENT)
  - Performance metrics per mode
  - Strategy parameter impact analysis
  - Export experiment results to CSV
  - Promotion path: EXPERIMENT → LIVE
- **User Notes & Insights (FR10):**
  - Editable notes field per trade instance
  - Searchable notes history
  - Tag system for categorization
  - Insights panel showing patterns
  - Notes preserved in archives
- **Advanced Error Recovery (Edge Cases):**
  - Automatic position reconciliation on startup
  - Missed window detection with alerts
  - Circuit breaker handling with queue system
  - Network outage recovery procedures
  - Database transaction rollback safety

**Success Criteria:**
- [ ] Experiment mode runs completely independent of LIVE
- [ ] 20 successful experiment trades tracked accurately
- [ ] Manual TWS changes detected within 10 seconds
- [ ] Orphaned orders detected and cleaned up properly
- [ ] State reconciliation works after container restart
- [ ] Notes searchable across all trades
- [ ] Experiment results exportable to CSV
- [ ] Side-by-side performance comparison works
- [ ] Recovery from all defined edge cases
- [ ] Zero data loss during system failures
- [ ] Playwright tests cover experiment workflows

**Architecture Notes:**
- Experiment mode uses same Meta-Strategy Model
- Mock TWS client for experiment order simulation
- Separate database tables for experiment vs live trades
- Unified dashboard with mode filtering
- State machine supports both modes independently

### Epic 8: SPY Weekly Strangle Strategy (Phase 2) - WITHOUT Rolling
**Goal:** Add the second proven strategy using the established Meta-Strategy Model infrastructure, initially WITHOUT defensive rolling for risk management.

**User Value:**
- Diversification through weekly 45 DTE strategy
- Automated entry and exit management
- Tax-aware year-end handling
- Simpler implementation without rolling complexity

**Deliverables:**
- **SPY Strangle Configuration Implementation** (see "SPY Weekly Strangle" config)
  - Delta 16 strike selection (configurable)
  - Profit Target (default: 50%, configurable)
  - Stop Loss (default: 200%, configurable)
  - Entry Time (default: Monday 11:45 ET, configurable)
- Weekly scheduling per strategy config (FR16)
- Mandatory 21 DTE close (FR19) per exit config
- Dual time tracking (DTE/TTC) implementation (FR20)
- Year-end skip logic per FR16
- New swimlane in dashboard for SPY strategy
- **EXPLICITLY EXCLUDED:** Defensive rolling (FR18) - reserved for future enhancement

**Success Criteria:**
- [ ] Strategy config correctly processed by Meta-Strategy Model
- [ ] Delta 16 strikes selected per configuration rules
- [ ] All configurable parameters work (delta, TP%, SL%, times)
- [ ] 21 DTE mandatory close enforced
- [ ] Year-end skip logic prevents tax complications
- [ ] System handles tested strikes WITHOUT rolling (stop loss only)

### Epic 9: [TO BE DEFINED]
Will be determined based on Epic 7 & 8 outcomes

**Note:** Future epics will be defined iteratively based on actual usage and feedback from previous epics. This ensures we build what's actually needed rather than what we think might be needed.


## Checklist Results Report

### PRD Validation Summary
The Product Requirements Document has been validated against the PM checklist with the following results:

**Overall Completeness: 99.5%**
**Readiness Status: READY FOR ARCHITECTURE**

All 9 major requirement categories passed validation:
- Problem Definition & Context: PASS
- MVP Scope Definition: PASS  
- User Experience Requirements: PASS
- Functional Requirements: PASS
- Non-Functional Requirements: PASS
- Epic & Story Structure: PASS
- Technical Guidance: PASS
- Cross-Functional Requirements: PASS
- Clarity & Communication: PASS

### Key Strengths
- Clear problem statement with quantified success metrics (83.7% win rate)
- Well-structured epic progression with safety-first approach
- Comprehensive technical decisions with clear rationale
- Excellent acceptance criteria - specific and testable
- Strong risk mitigation (kill switch, experiment mode, validation process)

### No Critical Issues Identified
The PRD is comprehensive, well-structured, and ready for the architecture phase.

## Next Steps

### UX Expert Prompt
Please review this PRD and create detailed UI/UX specifications for the SPX 0DTE Iron Condor Trading Bot, focusing on the Apple Aqua-inspired dashboard design, mobile responsiveness, and intuitive status visualization that matches the provided mockups.

### Architect Prompt

**Context:** You are a senior software architect tasked with implementing a multi-strategy options trading bot based on this PRD. The system must be production-ready, fault-tolerant, and maintainable.

**Your Task:** Create a comprehensive technical architecture document that includes:

**1. System Architecture:**
- Modular monolith design within Podman container
- Service layer architecture (MarketHoursService, OrderExecutionService, StrategyManager, etc.)
- Data flow between components
- State management and persistence strategy

**2. Core Services Implementation:**
- **MarketHoursService**: See Technical Reference for complete implementation requirements
- **OrderExecutionService**: Unified retry logic, mid-price calculation, execution window handling
- **StrategyManager**: Abstract base class, strategy registry, independent scheduling
- **PositionMonitor**: Real-time P/L, Greeks calculation, warning thresholds
- **NotificationService**: Discord integration, alert prioritization, rate limiting

**3. Technology Stack Details:**
- **Python 3.11+**: Type hints, async/await patterns, dataclasses
- **ib_async v2.0.1**: Connection management, order placement, market data streaming
- **FastAPI**: REST endpoints, WebSocket support, dependency injection
- **HTMX + SSE**: Real-time updates without JavaScript complexity
- **Tailwind CSS + DaisyUI**: Responsive design, dark mode support
- **SQLite**: Schema design, transaction management, migration strategy
- **Discord.py**: Bot commands, embed formatting, interactive components

**4. Critical Implementation Details:**
- Startup sequence: TWS connection → Contract validation → Market hours fetch → Strategy initialization
- Graceful degradation during market data loss
- Order state reconciliation after reconnection
- Atomic transaction handling for entry + exit orders
- Emergency close implementation with retry logic

**5. Error Handling Strategy:**
- Error classification (INFO, WARNING, CRITICAL, FATAL)
- Retry mechanisms (mid-price retry vs error retry)
- Circuit breaker patterns for API rate limiting
- Fallback behaviors for missing market data

**6. Testing Approach:**
- Unit tests for strategy logic and calculations
- Integration tests with IB Gateway test account
- Experiment mode for parallel strategy validation
- Performance benchmarking for order execution

**7. Deployment & Operations:**
- Podman container configuration
- Environment variable management
- Health check endpoints
- Monitoring and alerting setup
- Backup and recovery procedures

**Key Constraints:**
- Must handle TWS daily restart gracefully
- Support concurrent strategies without collision
- Maintain sub-2-second dashboard update latency
- Ensure zero data loss during crashes
- Respect IB API rate limits

**Deliverables Expected:**
1. Component diagram with service interactions
2. Database schema with relationships
3. API endpoint specifications
4. Sequence diagrams for critical flows (order execution, emergency close)
5. Error handling decision tree
6. Deployment architecture diagram

Please use Context7 MCP to research best practices for ib_async, FastAPI, HTMX, and DaisyUI implementation patterns. Focus on production-ready, maintainable code that can be extended for future strategies.

## Future Enhancement Roadmap

### Defensive Rolling for SPY Strangle (Post-Epic 8)
**Priority:** High - Important risk management feature
**Complexity:** High - Requires careful implementation and testing

**Implementation Requirements:**
- Continuous monitoring of strike positions vs underlying
- Automatic roll execution when strike becomes ATM/ITM at ≤28 DTE
- Roll only the untested side to Delta 30
- First-Touch-Rule: Maximum one roll per position
- Clear notifications and audit trail
- Experiment mode testing before live deployment

**Why Deferred:**
- Adds significant complexity to initial SPY Strangle implementation
- Requires extensive testing to ensure correct execution
- Better to establish stable base strategy first
- Can be added incrementally once base strategy proven

### Discord Command Extensions (Post-Epic 5)
**After the minimal Discord integration is proven stable, consider adding:**

**Status & Information Commands:**
- `!status` - Current position status and P/L
- `!today` - Today's scheduled and executed trades
- `!history [days]` - Recent trade history
- `!metrics` - Performance metrics summary

**Control Commands:**
- `!skip [date]` - Skip specific trading day
- `!pause` - Pause automation temporarily
- `!resume` - Resume automation
- `!settings` - View current settings

**Advanced Features:**
- Daily summary at market close
- Weekly performance reports
- Custom alert thresholds
- Multi-user support with permissions
- Voice channel alerts for critical events

**Rationale:** Start minimal (Epic 5), expand based on actual usage patterns.

### Advanced Greeks Trading Integration (Future Version)
**Research Status:** Completed via Archon MCP analysis  
**Implementation Priority:** Post-MVP (Version 2.0+)  
**Complexity:** High - requires advanced options theory and real-time data processing

#### Identified Enhancement Opportunities
Based on comprehensive research including Interactive Brokers API capabilities and modern Greeks-trading methodologies:

**Real-Time Greeks Monitoring:**
- Live Greeks calculation via `tickOptionComputation` callbacks (Delta, Gamma, Theta, Vega, Rho)
- Portfolio-level Greeks aggregation across all active positions
- Greeks-based risk warnings and position size recommendations

**Dynamic Risk Management:**
- Delta-hedging opportunities identification for enhanced P/L
- Gamma risk alerts during high-volatility periods
- Theta decay optimization for exit timing
- Vega exposure monitoring for volatility regime changes

**Strategic Enhancements:**
- VIX regime-based strategy parameter adjustment
- Gamma scalping opportunity detection
- Cross-strategy Greeks hedging (SPX Condor + SPY Strangle correlation management)