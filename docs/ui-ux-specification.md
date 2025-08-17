# UI/UX Specification for Options Strategy Bot Trader

## Executive Summary

This document defines the user interface and user experience design for the Options Strategy Bot Trader, addressing the critical issues identified in initial designs and establishing clear, intuitive interaction patterns.

## Design Philosophy

### Core Principles
1. **Clarity Over Features** - In critical moments, users need instant understanding
2. **Consistent Hierarchy** - Strict separation between system, strategy, and trade levels
3. **Safety First** - Dangerous actions are visually distinct and require confirmation
4. **Mobile Ready** - All critical functions accessible on mobile devices

### Key Problems Solved
- ✅ Clear visual hierarchy (no mixing of global/local controls)
- ✅ Understandable labels (no cryptic abbreviations)
- ✅ Obvious emergency controls (always same position)
- ✅ Simple mode switching per strategy (no complex global state)

## Information Architecture

```
LEVEL 1: SYSTEM HEADER
├── Application Title
├── TWS Connection Status
├── Settings Access [⚙️]
└── Emergency Stop [🔴]

LEVEL 2: STRATEGY CARDS
├── Strategy Name & Status
├── Mode Control (Live/Experiment)
├── Current P/L
├── Quick Actions
└── Active Trade Summary

LEVEL 3: TRADE DETAILS (On-Demand)
├── Entry/Exit Information
├── Progress Tracking
└── Technical Details
```

## Component Library

### Status Indicators
- `● ACTIVE` - Green dot + text (strategy running)
- `○ WAITING` - Gray dot + text (strategy idle)
- `⚠ WARNING` - Yellow dot + text (attention needed)
- `✕ ERROR` - Red dot + text (critical issue)

### Mode Indicators
- `💰` - LIVE mode (real money trading)
- `🧪` - EXPERIMENT mode (simulated trading)

### Action Buttons
- **Primary**: `[▶ Start]` `[⏸ Pause]` - Large, clear icons
- **Secondary**: `[👁 View]` `[📊 History]` - Smaller, subtle
- **Danger**: `[🔴 EMERGENCY STOP]` - Always red, isolated, top-right

### Information Display
```
Label: Value          ← Static information
P/L: +$125.50

Progress: X → Y       ← Goal tracking
$125 → $200 target
```

## Main Dashboard Layout - Kanban Board Architecture

### Core Concept: Strategy Instance Management
The dashboard uses a revolutionary Kanban-based approach that distinguishes between:
- **STRATEGY** (SPX Iron Condor, SPY Strangle) - The trading rule set
- **STRATEGY INSTANCE** (SPX#001, SPY#007) - Individual executions of that strategy

This enables tracking multiple parallel instances per strategy (e.g., SPY Strangle has 3+ concurrent 42 DTE positions).

### Desktop Kanban View
```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Options Trading Bot Dashboard                              [⚙️ Settings] [🔴 STOP ALL]                  │
│ TWS: ● Connected                                                                                                        │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                                         │
│ KANBAN BOARD                                     Portfolio P/L: +$1,450 💰 | +$780 🧪 | +$2,230 Total │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                                                                                                         │
├─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐                                                   │
│  NEXT   │ SEARCH  │ PLACE   │ SETUP   │ ACTIVE  │ CLOSED  │ ARCHIVE │                                                   │
│         │ OPTIONS │ ORDER   │ ORDER   │         │         │         │                                                   │
│         │         │         │  EXIT   │         │         │         │                                                   │
├─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤                                                   │
│         │         │         │         │         │         │         │ SPX IRON CONDOR                                   │
│SPX#003  │         │SPX#002  │         │SPX#001  │SPX#999  │SPX#990  │ P/L: +$680 💰 | +$340 🧪                         │
│✅ Ready │         │🔄 Doing │         │✅ Good  │✅ Done  │✅ Done  │ Mode: [LIVE] [EXPERIMENT]                         │
│Tom 9:32 │         │Placing  │         │+$125🟢  │+$200✓  │+$180✓  │                                                   │
│🧪 EXP   │         │💰LIVE   │         │2h left  │Profit   │45d old │                                                   │
│Market✓  │         │[🔴STOP] │         │[🔴STOP] │Target   │         │                                                   │
│         │         │         │         │         │         │         │                                                   │
│SPX#004  │         │         │         │         │SPX#998  │SPX#989  │                                                   │
│⏸️ Wait  │         │         │         │         │❌ Loss  │✅ Done  │                                                   │
│Fri 9:32 │         │         │         │         │-$150✗  │+$90✓   │                                                   │
│💰LIVE   │         │         │         │         │Stop Hit │Time Exit│                                                   │
│Mkt Close│         │         │         │         │         │         │                                                   │
├─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤                                                   │
│         │         │         │         │         │         │         │ SPY STRANGLE                                      │
│SPY#009  │SPY#008  │         │SPY#007  │SPY#006  │SPY#005  │SPY#001  │ P/L: +$770 💰 | +$440 🧪                         │
│✅ Ready │🔄 Doing │         │✅ Good  │🔄 Roll  │✅ Done  │✅ Done  │ Mode: [LIVE] [EXPERIMENT]                         │
│Fri 18h  │Delta15  │         │Setting  │Rolling  │+$400✓  │+$280✓  │                                                   │
│💰LIVE   │🧪 EXP   │         │OCA      │28 DTE   │Profit   │60d old │                                                   │
│Market✓  │         │         │💰LIVE   │💰LIVE   │Target   │         │                                                   │
│         │         │         │         │[🔴STOP] │         │         │                                                   │
│         │         │         │         │         │         │         │                                                   │
│SPY#010  │SPY#011  │         │         │SPY#004  │SPY#003  │SPY#002  │                                                   │
│⏸️ Wait  │❌ Error │         │         │✅ Good  │❌ Loss  │✅ Done  │                                                   │
│Next Fri │No Opts  │         │         │-$45🔴  │-$300✗  │+$150✓  │                                                   │
│🧪 EXP   │🧪 EXP   │         │         │35 DTE   │Stop Hit │Time Exit│                                                   │
│Market✓  │Retry?   │         │         │[🔴STOP] │         │         │                                                   │
└─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘                                                   │
│                                                                                                                         │
│ LEGEND: ✅Good 🔄Doing ⏸️Wait ❌Error | 💰Live 🧪Exp | 🟢Profit 🔴Loss                                                  │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Kanban Flow Columns
1. **NEXT** - Instance ready for execution (market open, TWS connected, schedule met)
2. **SEARCH OPTIONS** - Finding delta-appropriate strikes
3. **PLACE ORDER** - Submitting combo/BAG order to TWS
4. **SETUP ORDER EXIT** - Placing OCA exit orders (profit target, time close, stop loss)
5. **ACTIVE** - Position monitoring phase with real-time P/L
6. **CLOSED** - Position closed, final P/L calculated
7. **ARCHIVE** - Historical instances (30+ days old or manually archived)

### Strategy Swimlanes
- Each strategy (SPX Iron Condor, SPY Strangle) has its own horizontal lane
- Strategy-level controls and statistics displayed in lane header
- Instances flow left-to-right through status columns within their lane

### Instance Status Indicators
- **✅ Good/Ready/Done** - Healthy status, can proceed
- **🔄 Doing** - Work in progress  
- **⏸️ Wait** - Waiting for external condition (market, time)
- **❌ Error** - Problem requiring intervention (automatic retry in progress)
- **⚠️ Warning** - Attention needed but not critical

### Status Distribution by Column
- **NEXT Column**: ✅ READY, ⏸️ WAIT, ⚠️ WARNING only (no ❌ ERROR - problems still fixable)
- **Active Columns** (2-5): All status types including ❌ ERROR
- **CLOSED/ARCHIVE**: ✅ DONE, ❌ LOSS final states

### Instance Field Definitions

#### Core Fields (All Strategies)
- **Instance ID**: Format: `[STRATEGY]-[YYMMDD]-[###]` (e.g., SPXIC-240115-001)
- **Status Badge**: ✅ Good | 🔄 Doing | ⏸️ Wait | ❌ Error | ⚠️ Warning
- **Mode Badge**: 💰 LIVE | 🧪 EXPERIMENT
- **P/L Display**: 
  - Format: `+$125` | `-$125` | `$0` 
  - Card background color from ACTIVE column onward:
    - Green background: Profit (+$X)
    - Red background: Loss (-$X) 
    - Gray background: Break-even ($0)
- **Greeks Block**:
  - Format: `D:0.15 G:0.02 T:-45 V:12`
  - Delta, Gamma, Theta and Vega of the overall Option-Strategy of this trade
- **Time Display**:
  - DTE: Days to Expiry 
    - Current: "42 DTE" (single expiry for SPX IC and SPY Strangle)
    - Future: "7/35 DTE" (for calendar spreads - not yet implemented)
  - TTC: Time to Close (e.g., "2.5h", "45m", "5m")
  - **Scheduled time (Column 1 NEXT only)**: First content line in Ready/Waiting status displays execution time in user timezone with market time in parentheses:
    - **Today**: "Today 15:32 CET (9:32 ET)"
    - **Tomorrow**: "Tom 15:32 CET (9:32 ET)" 
    - **This week (2-6 days)**: "Mi 15:32 CET (9:32 ET)"
    - **Beyond 6 days**: "29.01.2025 15:32 CET (9:32 ET)" (locale date format)
  - **Remaining time (Column 1 NEXT only)**: Second content line in ✅ Ready and ⏸️ Wait status shows time until execution:
    - **Same day**: "in 2h 15m" | "in 45m" | "in 3m"
    - **Future days**: "in 1 day" | "in 5 days" | "in 12 days"

#### Column-Specific Display Context
- **SEARCH OPTIONS Column**:
  - Search progress: "Finding options..." | "Analyzing strikes..."
  - Error messages: "No options found" | "Retry 5/10"
  
- **ACTIVE Column**:
  - P/L displayed with card background color (green/red/gray)
  - Greeks block displayed (values change with market)
  - TTC countdown displayed
  - Roll status when applicable: "🔄 Rolling" (SPY Strangle only)
  
- **CLOSED Column**:
  - Final P/L with card background color (green/red/gray)
  - Exit reason: "Profit target hit" | "Stop loss hit" | "Time exit 11:30"

- **ARCHIVE Column**:
  - Final P/L (no background color, compact display)
  - Archive age: "31d ago" | "45d ago"

### Mode Control Architecture

#### Strategy-Level Mode Controls
Each strategy has independent mode setting:
- **[LIVE] [EXPERIMENT]** toggle in strategy swimlane header
- New instances inherit current strategy mode
- Mode can only be changed when NO instances are in ACTIVE column
- Mode change affects ALL future instances of that strategy

#### Instance-Level Controls
- **[🔴 STOP]** button appears only on instances in ACTIVE column
- STOP behavior depends on instance mode:
  - **Live Instance STOP**: Position closed → becomes EXPERIMENT instance in CLOSED
  - **Experiment Instance STOP**: Tracking stopped → moves to CLOSED
- STOP immediately moves instance from ACTIVE → CLOSED

#### Global Controls
- **[🔴 STOP ALL]**: Affects ALL Live instances across all strategies
- Converts Live instances to Experiment mode and moves to CLOSED
- Experiment instances remain unchanged (already safe)

#### Mode Switch Rules
- **Strategy Mode Switch**: Only allowed when strategy has NO ACTIVE instances
- **During Active Trading**: Mode buttons disabled with tooltip
- **Confirmation Required**: EXPERIMENT → LIVE requires confirmation dialog
- **Safety Default**: All new strategies start in EXPERIMENT mode

### Error Handling Philosophy
- Error instances remain in their column (not separate error lane)
- "Blocked" instances naturally create urgency ("I love when error cards clog the columns!")
- **UNIFIED ARCHIVING RULES:**
  - ✅ COMPLETED instances: Archive after 30 days (configurable) OR manually
  - ❌ FAILED instances: Manual archive immediately possible OR after 30 days automatic
  - 🔄 ACTIVE instances: NO archiving possible (trade running)
  - 🔄 DOING/⏸️ WAIT instances: [🔴 STOP] button available

### View Model Architecture

#### Three-Level View System
1. **Compact View** (Default in Kanban)
   - 4-5 lines of essential information
   - Instance ID, status, P/L, mode, primary action
   - Optimized for scanning multiple instances

2. **Expanded View** (Hover/Click on card)
   - 8-10 lines with additional details
   - Greeks, entry/exit prices, detailed timing
   - In-place expansion within Kanban column

3. **Detail View** (Full screen)
   - Complete trade information
   - Charts, full history, all Greeks over time
   - Desktop: Split-view (50% list, 50% detail)
   - Mobile: Full-screen modal

### Mobile View
```
┌─────────────────────┐
│ Bot Trader    [⚙️]  │
│ [🔴 EMERGENCY STOP] │
├─────────────────────┤
│ Total: +$580.50     │
├─────────────────────┤
│ SPX CONDOR          │
│ ● ACTIVE            │
│ [LIVE] 💰 +$125.50  │
│ [Pause] [Details]   │
├─────────────────────┤
│ SPY STRANGLE        │
│ ○ WAITING           │
│ [EXPERIMENT] 🧪 +$45│
│ [Start] [Details]   │
└─────────────────────┘
```

## Strategy Instance State Management

### Kanban-Based State Machine
With the Kanban architecture, the state machine operates at the instance level:

```
INSTANCE FLOW STATE MACHINE:
════════════════════════════

READY + Mode(LIVE/EXP) 
├─ Execution time reached → OPTIONS_SEARCH + Mode (automatic progression)
├─ Mode switch → ALLOWED (no active trading)
└─ Status: ✅Ready / ⏸️Wait / ❌Error

OPTIONS_SEARCH + Mode
├─ Options found → ORDER_PLACING + Mode  
├─ Mode switch → BLOCKED (instance in progress)
└─ Status: 🔄Doing / ❌Error

ORDER_PLACING + Mode
├─ Order filled → EXIT_SETUP + Mode
├─ Mode switch → BLOCKED
└─ Status: 🔄Doing / ❌Error

EXIT_SETUP + Mode  
├─ Exit orders set → ACTIVE + Mode
├─ Mode switch → BLOCKED
└─ Status: 🔄Doing / ❌Error

ACTIVE + Mode
├─ Exit triggered → CLOSED + Mode
├─ [🔴 STOP] → CLOSED + Mode  
├─ Mode switch → BLOCKED
└─ Status: ✅Good / ❌Error

CLOSED + Mode
├─ After 30 days → ARCHIVE + Mode
├─ Mode switch → N/A (completed)
└─ Status: ✅Done / ❌Loss
```

### Strategy-Level Mode Control Rules
- **Mode Switch Allowed**: Only when strategy has NO instances in columns 2-5 (OPTIONS_SEARCH through ACTIVE)
- **Mode Inheritance**: All new instances inherit current strategy mode
- **Active Instance Protection**: Cannot change strategy mode while instances are actively trading

### Visual Mode Switch Interactions

#### Strategy Mode Toggle (When Allowed)
```
SPX IRON CONDOR                           [🔴 STOP ALL]
Mode: [● LIVE] ○ EXPERIMENT    ← Clickable when no active instances
P/L: +$680 💰 | +$340 🧪
```

#### Strategy Mode Toggle (When Blocked)
```
SPY STRANGLE                              [🔴 STOP ALL]  
Mode: [● LIVE] ○ EXPERIMENT    ← Disabled, tooltip: "2 instances actively trading"
P/L: +$770 💰 | +$440 🧪
```

#### Instance STOP Behavior
```
ACTIVE Column:
┌─────────────────┐
│ SPX#001         │ 
│ ✅ Good         │
│ +$125🟢 💰LIVE │  ← Live instance
│ [🔴 STOP]       │  → Becomes EXPERIMENT in CLOSED
└─────────────────┘

┌─────────────────┐
│ SPY#007         │
│ ✅ Good         │ 
│ -$45🔴 🧪EXP    │  ← Experiment instance  
│ [🔴 STOP]       │  → Moves to CLOSED
└─────────────────┘
```

#### Global STOP ALL Behavior
```
[🔴 STOP ALL] → Converts ALL Live instances to Experiment + moves to CLOSED
                Experiment instances simply move to CLOSED
```

#### Confirmation Dialog (EXPERIMENT → LIVE)
```
┌─────────────────────────────────────────┐
│      💰 ENABLE REAL MONEY TRADING?      │
├─────────────────────────────────────────┤
│ Strategy: SPX 0DTE Iron Condor          │
│                                          │
│ ⚠️ This will use REAL MONEY!            │
│ Next instances will trade with real $   │
│                                          │
│ Active instances: 0                     │
│ Completed experiments: 12                │
│                                          │
│ [STAY SAFE]       [ENABLE LIVE TRADING] │
└─────────────────────────────────────────┘
```

## Settings Panel

### Access
Settings accessible via [⚙️] button in header

### Layout
```
┌──────────────────────────────────────────────────────────────┐
│ ⚙️ SETTINGS                                          [✕]    │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│ CONNECTION                                                    │
│ ├─ TWS Host: [127.0.0.1        ]                            │
│ ├─ TWS Port: [7496             ] (7496=Live, 7497=Paper)    │
│ └─ Auto-Reconnect: [✓]                                      │
│                                                               │
│ NOTIFICATIONS                                                 │
│ ├─ Discord Token: [••••••••••••] [Show]                     │
│ ├─ Alert Channel: [#trading-alerts    ▼]                    │
│ └─ Summary Time: [18:00 CET    ]                            │
│                                                               │
│ RISK MANAGEMENT                                              │
│ ├─ Position Size Mode: [○ Fixed] [● % of Capital]           │
│ ├─ Max % per Trade: [5%        ]                            │
│ ├─ Daily Loss Limit: [8%       ]                            │
│ └─ Consecutive Loss Pause: [3  ]                            │
│                                                               │
│ EXIT ORDER MANAGEMENT                                         │
│ ├─ Exit Retry Attempts: [30     ] (max retries)             │
│ ├─ Exit Retry Interval: [20 sec ] (between attempts)        │
│ ├─ Auto-Emergency After: [10 min] (calculated)              │
│ └─ Discord Alerts: [✓] Immediate [✓] Every minute           │
│                                                               │
│ TRADING WINDOWS                                              │
│ ├─ SPX IC Entry: [9:32 ET      ] (fixed)                    │
│ ├─ SPY Strangle Entry: [Friday 18:00 CET ▼]                 │
│ └─ Time Zone Display: [CET     ▼]                           │
│                                                               │
│ [Cancel]                                    [Save Settings]  │
└──────────────────────────────────────────────────────────────┘
```

## Emergency Stop

### Positioning
- Desktop: Top-right corner, always visible
- Mobile: Below header, full-width button

### Behavior
```
┌─────────────────────────────────────────┐
│        🔴 EMERGENCY STOP ACTIVATED      │
├─────────────────────────────────────────┤
│ Are you sure you want to:               │
│ • Cancel all pending orders             │
│ • Close all open positions at market    │
│ • Stop all trading for today            │
│                                          │
│ [CANCEL]          [YES, STOP EVERYTHING]│
└─────────────────────────────────────────┘
```

## Skip Instance Confirmation (Column 1)

### Trigger
- Clicking [🗃️ SKIP] button in Column 1 (NEXT)

### Behavior
```
┌─────────────────────────────────────────┐
│          🗃️ SKIP THIS INSTANCE?         │
├─────────────────────────────────────────┤
│ Instance: SPXIC-240115-001              │
│ Status: Ready to execute                 │
│                                          │
│ Are you sure you want to skip this      │
│ trading instance?                        │
│                                          │
│ • Instance will be archived              │
│ • No position will be opened             │
│ • Next scheduled instance continues      │
│                                          │
│ [CANCEL]                    [YES, SKIP]  │
└─────────────────────────────────────────┘
```

## Stop Instance Confirmation (Column 2)

### Trigger
- Clicking [🔴 STOP] button in Column 2 (SEARCH OPTIONS) during active work

### Behavior
```
┌─────────────────────────────────────────┐
│          🔴 STOP THIS INSTANCE?          │
├─────────────────────────────────────────┤
│ Instance: SPXIC-240115-001              │
│ Status: Searching options (Retry 5/10)   │
│                                          │
│ Are you sure you want to stop this      │
│ search operation?                        │
│                                          │
│ • Search will be cancelled               │
│ • Instance will be archived              │
│ • No position will be opened             │
│                                          │
│ [CANCEL]                    [YES, STOP]  │
└─────────────────────────────────────────┘
```

## Visual Design Guidelines

### Color Scheme
- **Background**: Light gray (#F5F5F5) or white
- **Cards**: White with subtle shadow
- **Live Mode**: Light red tint (#FFF5F5)
- **Experiment Mode**: Light blue tint (#F5F5FF)
- **Profit**: Green (#28A745)
- **Loss**: Red (#DC3545)
- **Warning**: Yellow (#FFC107)

### Typography
- **Headers**: System font, bold, 18px
- **Labels**: System font, regular, 14px
- **Values**: System font, medium, 16px
- **Buttons**: System font, medium, 14px

### Spacing
- **Card padding**: 16px
- **Element spacing**: 8px
- **Section spacing**: 24px

## User Journey Validations

### Journey 1: Morning Check
1. Open Dashboard → Immediately see all strategy statuses ✓
2. Check P/L → Large, prominent numbers ✓
3. See Next Actions → Clear timing for each strategy ✓

### Journey 2: Emergency Intervention
1. Alert on Phone → "Position at risk!"
2. Open Dashboard → Emergency Stop immediately visible ✓
3. One Click → Confirmation → Done ✓

### Journey 3: Spouse Monitoring
1. Opens Dashboard → Understands status immediately ✓
2. Sees Problem → Knows where Emergency Stop is ✓
3. Can Act → Without technical knowledge ✓

### Journey 4: Testing New Strategy
1. Add Strategy → Defaults to Experiment mode ✓
2. Run Tests → See results marked as 🧪 ✓
3. Ready for Live → Switch mode when WAITING ✓
4. Confirm → Clear warning about real money ✓

## Implementation Notes

### Technology Stack
- **Frontend Framework**: HTML + HTMX for real-time updates
- **CSS Framework**: Tailwind CSS or custom CSS with variables
- **Icons**: Emoji for universal support, or icon font
- **Charts**: Chart.js for performance visualization

### Responsive Breakpoints
- **Mobile**: < 768px
- **Tablet**: 768px - 1024px
- **Desktop**: > 1024px

### Update Frequency
- **Status Updates**: Every 2 seconds via HTMX
- **P/L Updates**: Real-time when positions active
- **Connection Status**: Every 30 seconds

### Accessibility
- **WCAG AA Compliance**: Proper contrast ratios
- **Keyboard Navigation**: All actions keyboard accessible
- **Screen Reader**: Semantic HTML with ARIA labels
- **Focus Indicators**: Clear visual focus states

## Testing Criteria

### 5-Second Test
Users should understand within 5 seconds:
- System status (connected/disconnected)
- Which strategies are active
- Whether real money is at risk
- Current profit/loss

### Panic Test
Users should find Emergency Stop within 2 seconds

### Spouse Test
Non-technical users should be able to:
- Understand system status
- Identify problems
- Execute emergency stop

## Definition of Ready/Done (DoR/DoD) Framework

### Kanban Column Rules

#### READY → OPTIONS SEARCH (DoR)
- ✅ Strategy enabled (mode selected)
- ✅ Current execution day (today is scheduled execution day)
- ✅ Market officially open (market hours for strategy's underlying)
- ✅ Execution time reached (strategy's scheduled execution time)
- ✅ TWS connection established
- ✅ Sufficient capital available
- ✅ No blocking errors from previous runs

#### OPTIONS SEARCH → ORDER PLACING (DoR)
- ✅ Target delta options found (±1.9 tolerance)
- ✅ Wing strikes calculated
- ✅ All 4 legs identified and priced
- ✅ Spread within acceptable range

#### ORDER PLACING → EXIT SETUP (DoR)
- ✅ Combo order 100% filled
- ✅ Entry price confirmed
- ✅ Position reflected in TWS

**Column 3 Definition of Done (DoD) - Exit Rules:**
- **SUCCESS Exit (Automatic)**: Instance automatically progresses to Column 4 when all DoR criteria met
- **FAILURE Exit (Manual)**: Failed instances remain in Column 3 with [🗃️ ARCHIVE] button until manually archived
  - "Order rejected" → Direct "Order failed" state (no retries)
  - "Insufficient margin" → Direct "Order failed" state (no retries)
  - "Order cancelled" → Direct "Order failed" state (no retries)  
  - "No TWS connection" → Retry up to 10 attempts with 20-second intervals, then "Order failed (10/10)" state
- **MANUAL STOP Exit**: [🔴 STOP] button moves instance directly to ARCHIVE with ⏸️ status
- **Auto-Archive**: Failed instances move to ARCHIVE with ❌ status after 30-day auto-archive or manual archiving
- **"Clog the Flow" Principle**: Failed instances intentionally remain visible to force user attention and manual resolution

#### EXIT SETUP → ACTIVE (DoR)
- ✅ Profit target order placed & confirmed
- ✅ Time-based exit order placed & confirmed
- ✅ Stop-loss order placed & confirmed
- ✅ OCA group validated

#### ACTIVE → CLOSED (DoR)
- ✅ One exit condition triggered
- ✅ Position fully closed
- ✅ Final P/L calculated

#### CLOSED → ARCHIVE (DoR)
- ✅ 30+ days since closure
- ✅ Statistics updated
- ✅ No pending reconciliation

## Compact View Specifications by Column

### Column 1: NEXT
```
┌─────────────────┐
🧪 SPXIC-240115-001 ✅
─────────────────
Today 15:32 CET (9:32 ET)
in 2h 15m
─────────────────
[🗃️ SKIP]
└─────────────────┘

┌─────────────────┐
💰 SPYST-240115-003 ⏸️
─────────────────
Tom 18:00 CET (12:00 ET)
in 1 day
─────────────────
[🗃️ SKIP]
└─────────────────┘

┌─────────────────┐
🧪 SPXIC-240115-002 ⚠️
─────────────────
Today 15:32 CET (9:32 ET)
TWS disconnected
─────────────────
[🗃️ SKIP]
└─────────────────┘

┌─────────────────┐
💰 SPXIC-240115-004 ⚠️
─────────────────
Today 15:32 CET (9:32 ET)
Insufficient capital
─────────────────
[🗃️ SKIP]
└─────────────────┘

┌─────────────────┐
💰 SPXIC-240115-006 ⏸️
─────────────────
Mi 15:32 CET (9:32 ET)
in 5 days
─────────────────
[🗃️ SKIP]
└─────────────────┘

┌─────────────────┐
🧪 SPYST-240115-007 ⏸️
─────────────────
29.01.2025 18:00 CET (12:00 ET)
in 12 days
─────────────────
[🗃️ SKIP]
└─────────────────┘
```

**NEXT Column Status Rules:**
- ✅ **READY**: All conditions met for immediate execution
  1. **Current execution day** (today is scheduled execution day)
  2. **Market open** (market hours for the strategy's underlying)
  3. **Before execution time** (not yet reached strategy's scheduled execution time)
  4. **All systems check passed** (TWS connected, sufficient capital, no blocking errors)
  - **Content lines**: Line 1 = execution time, Line 2 = remaining time ("in 2h 15m")
- ⏸️ **WAIT**: Waiting for execution conditions
  - Wrong day (not scheduled execution day)
  - Market closed (outside market hours)
  - System conditions met but timing not ready
  - **Content lines**: Line 1 = execution time, Line 2 = remaining time ("in 5 days")
- **AUTOMATIC PROGRESSION**: At execution time, instances automatically move from NEXT (Column 1) → SEARCH OPTIONS (Column 2). No instance remains in NEXT beyond its scheduled execution time.
- ⚠️ **WARNING**: Fixable problems with time remaining
  - TWS disconnected (system auto-retry active)
  - Insufficient capital (may become available)
  - **Content lines**: Line 1 = execution time, Line 2 = error message (no remaining time)
- **No ❌ ERROR**: Problems in NEXT are still fixable, errors only occur in active trading phases

**Auto-Status Updates:**
- ⚠️ WARNING automatically becomes ✅ READY when problem resolved AND timing conditions met
- ⏸️ WAIT automatically becomes ✅ READY when execution day + market open + before execution time + all checks passed

### Column 2: SEARCH OPTIONS

**Status States:**
- 🔄 **DOING** - Actively searching for options OR retry in progress after error
- ❌ **ERROR** - Search failed, waiting for next retry OR max retries reached

**Line-by-Line Display:**
1. Header: `[Mode Badge] [Instance ID] [Status Badge]`
2. Divider: `─────────────────`
3. Content Line 1: Progress/Error message
4. Content Line 2: Action detail/Retry counter
5. Divider: `─────────────────`
6. Action: `[🔴 STOP]` during active work OR `[🗃️ ARCHIVE]` after max retries

**Status Examples:**

```
SEARCH OPTIONS - Status: 🔄 DOING (Initial Search)
┌─────────────────┐
💰 SPXIC-240115-001 🔄
─────────────────
Finding delta 15...
Scanning strikes...
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: 🔄 DOING (Analyzing)
┌─────────────────┐
🧪 SPYST-240115-002 🔄
─────────────────
Analyzing spreads...
4 candidates found
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: 🔄 DOING (Validating)
┌─────────────────┐
💰 SPXIC-240115-003 🔄
─────────────────
Validating legs...
Call: 5900/5920
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: ❌ ERROR (No Market Data)
┌─────────────────┐
💰 SPXIC-240115-004 ❌
─────────────────
No market data
Retry 5/30
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: ❌ ERROR (No TWS Connection)
┌─────────────────┐
🧪 SPYST-240115-005 ❌
─────────────────
No TWS connection
Retry 8/10
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: 🔄 DOING (Reconnecting)
┌─────────────────┐
🧪 SPYST-240115-005 🔄
─────────────────
Reconnecting TWS...
Attempt 13/30
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: ❌ ERROR (No Options Found)
┌─────────────────┐
💰 SPXIC-240115-006 ❌
─────────────────
No options found
Retry 28/30
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: ❌ ERROR (Search Timeout)
┌─────────────────┐
🧪 SPYST-240115-007 ❌
─────────────────
Search timeout
Retry 3/30
─────────────────
[🔴 STOP]
└─────────────────┘

SEARCH OPTIONS - Status: ❌ ERROR (Max Retries - Final)
┌─────────────────┐
💰 SPXIC-240115-008 ❌
─────────────────
Search failed
Retry 10/10 (MAX)
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘
```

**Progression Rules:**
- **Success (🔄 → Column 3)**: Automatic when valid options found with acceptable delta (±1.9 tolerance)
- **Error & Retry Cycle**:
  - Error occurs → Status: ❌ ERROR (shows "Retry X/10")
  - Waits 20 seconds → Status changes to 🔄 DOING ("Retrying search...")
  - During retry → Back to searching logic
  - If retry fails → Back to ❌ ERROR with incremented counter
- **Button Logic**:
  - **[🔴 STOP]** - Shows during 🔄 DOING or ❌ ERROR (with retries remaining)
    - Requires confirmation dialog
    - Stops all activity → ARCHIVE with ⏸️ status and $ -
  - **[🗃️ ARCHIVE]** - Shows ONLY when ❌ ERROR with "Retry 10/10 (MAX)"
    - NO confirmation dialog (nothing left to stop)
    - Direct archiving with ❌ status and $ -
- **Max Retries Behavior**:
  - Instance **STAYS IN Column 2** permanently
  - "Clogs" the column until manually archived
  - Manual [🗃️ ARCHIVE] or automatic after 30 days
- **NO Automatic Column Movement**: Instances NEVER move automatically from Column 2 (except 30-day archive rule)

**Error Types (Realistic for Production):**
1. **"No market data"** - TWS connected but no current market data available
2. **"No TWS connection"** - Connection to TWS/IB Gateway lost
3. **"No options found"** - No strikes found within Delta 15 ±1.9 tolerance (rare)
4. **"Search timeout"** - Operation exceeded time limit
5. **"Search failed"** - Generic message after max retries (30/30)

### Column 3: PLACE ORDER

**Status States:**
- 🔄 **DOING** - Placing order OR retry in progress  
- ❌ **ERROR** - Order failed, retry in progress OR max retries reached

**Order Retry Mechanisms:**
- **System Error Retries**: 10 attempts with 20-second intervals (globally configurable retry count and intervals per error type) for connection/system failures 
- **Order Modify Retries**: Configurable per strategy (default: 5-second intervals for 1-minute maximum duration) for unfilled orders at mid-price
- Each retry fetches fresh mid-price from TWS

**🔄 DOING Display Logic:**
- Initial: "Placing order..." + "LMT: $X.XX"
- After Modify: "Waiting for fill..." + "LMT: $X.XX (Try Y/Z)" where Z = strategy-specific modify limit

**❌ ERROR Types by Mode:**
- **LIVE Mode**: "Order rejected", "Insufficient margin", "Order cancelled" (direct failure), "No TWS connection" (10 retries with 20-second intervals, globally configurable retry count and intervals per error type)
- **EXPERIMENT Mode**: Only "No TWS connection" (requires TWS for mid-price simulation, 10 retries with 20-second intervals)

**Button Logic:**
- **[🔴 STOP]** during retries (with confirmation)
- **[🗃️ ARCHIVE]** when "Order failed" state reached (no confirmation)

**Examples:**
```
PLACE ORDER - Status: 🔄 DOING (Initial Order)
┌─────────────────┐
💰 SPXIC-240115-001 🔄
─────────────────
Placing order...
LMT: $2.50
─────────────────
[🔴 STOP]
└─────────────────┘

PLACE ORDER - Status: 🔄 DOING (After Order Modify)
┌─────────────────┐
🧪 SPYST-240115-002 🔄
─────────────────
Waiting for fill...
LMT: $2.45 (Try 2/12)
─────────────────
[🔴 STOP]
└─────────────────┘

PLACE ORDER - Status: 🔄 DOING (Multiple Modifies)
┌─────────────────┐
💰 SPXIC-240115-003 🔄
─────────────────
Waiting for fill...
LMT: $2.38 (Try 5/12)
─────────────────
[🔴 STOP]
└─────────────────┘

PLACE ORDER - Status: ❌ ERROR (Order Rejected - LIVE only)
┌─────────────────┐
💰 SPXIC-240115-004 ❌
─────────────────
Order rejected
Order failed
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘

PLACE ORDER - Status: ❌ ERROR (Insufficient Margin - LIVE only)
┌─────────────────┐
💰 SPXIC-240115-005 ❌
─────────────────
Insufficient margin
Order failed
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘

PLACE ORDER - Status: ❌ ERROR (No TWS Connection - LIVE/EXP)
┌─────────────────┐
🧪 SPYST-240115-006 ❌
─────────────────
No TWS connection
Retry 3/10
─────────────────
[🔴 STOP]
└─────────────────┘

PLACE ORDER - Status: ❌ ERROR (Order Cancelled - LIVE only)
┌─────────────────┐
💰 SPXIC-240115-007 ❌
─────────────────
Order cancelled
Order failed
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘

PLACE ORDER - Status: ❌ ERROR (Max Retries - LIVE/EXP)
┌─────────────────┐
🧪 SPYST-240115-008 ❌
─────────────────
No TWS connection
Order failed (10/10)
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘
```

**Progression Rules:**
- **Success (🔄 → Column 4)**: Automatic when order 100% filled
- **Error Retry**: Cancel order, get fresh mid-price, resubmit with new price
- **Max Retries**: Instance stays in Column 3, requires manual [🗃️ ARCHIVE]

### Column 4: SETUP ORDER EXIT

**CRITICAL COLUMN**: Any failure here = UNPROTECTED position (unlimited risk)
- Discord CRITICAL alert on FIRST error (not after retries)
- All errors lead to UNPROTECTED state after max retries
- [🔴 CLOSE NOW] button replaces archive until position closed

**Status Examples:**
```
SETUP ORDER EXIT - Status: 🔄 DOING
┌─────────────────┐
💰 SPXIC-240115-008 🔄
─────────────────
Setting OCA...
Validating group...
─────────────────
[🔴 STOP]
└─────────────────┘

SETUP ORDER EXIT - Status: ❌ ERROR (Retrying)
┌─────────────────┐
🧪 SPYST-240115-009 ❌
─────────────────
No market data
Retry 8/30
─────────────────
[🔴 STOP]
└─────────────────┘

SETUP ORDER EXIT - Status: 🚨 UNPROTECTED (After Max Retries)
┌─────────────────┐
💰 SPXIC-240115-010 🚨
─────────────────
🚨 UNPROTECTED
No exit orders!
─────────────────
[🔴 CLOSE NOW]
└─────────────────┘
```

### Column 5: ACTIVE
```
┌─────────────────┐ (green bg)
💰 SPXIC-240115-010 ✅
─────────────────
+$125
2.5h left
─────────────────
[🔴 STOP]
└─────────────────┘

┌─────────────────┐ (red bg)
💰 SPYST-240115-011 🔄
─────────────────
-$85
Rolling...
─────────────────
[🔴 STOP]
└─────────────────┘

┌─────────────────┐ (green bg)
🧪 SPXIC-240115-012 ⚠️
─────────────────
+$180
Approaching stop
─────────────────
[🔴 STOP]
└─────────────────┘
```

### Column 6: CLOSED
```
┌─────────────────┐ (green bg)
💰 SPXIC-240115-013 ✅
─────────────────
+$200
Profit target hit
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘

┌─────────────────┐ (red bg)
🧪 SPYST-240115-014 ❌
─────────────────
-$300
Stop loss hit
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘

┌─────────────────┐ (green bg)
💰 SPXIC-240115-015 ✅
─────────────────
+$45
Time exit 11:30
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘

┌─────────────────┐ (gray bg)
🧪 SPYST-240115-016 ❌
─────────────────
$0
Exit order failed
─────────────────
[🗃️ ARCHIVE]
└─────────────────┘
```

### Column 7: ARCHIVE
```
┌─────────────────┐
💰 SPXIC-240114-001 ✅
─────────────────
+$180
31d ago
└─────────────────┘

┌─────────────────┐
🧪 SPYST-240113-002 ❌
─────────────────
-$150
45d ago
└─────────────────┘

┌─────────────────┐
💰 SPXIC-240115-003 ⏸️
─────────────────
$ -
Today
└─────────────────┘
```

## Button Availability Rules
- **[🗃️ SKIP]**: Only in Column 1 (NEXT)
  - Available for READY/WAIT/WARNING status
  - Shows confirmation dialog before execution
  - Skipped instance moves directly to ARCHIVE column with ⏸️ status and $ -
- **[🔴 STOP]**: 
  - Column 2: During 🔄 DOING or ❌ ERROR (with retries < 30)
  - Column 3: During 🔄 DOING or ❌ ERROR (with retries < 12)  
  - Columns 4-5: During active operations
  - Shows confirmation dialog
  - Stopped instance → ARCHIVE with ⏸️ status and $ -
- **[🗃️ ARCHIVE]**: 
  - Column 2: Only when ❌ ERROR with "Retry 10/10 (MAX)"
  - Column 3: Only when ❌ ERROR with "Retry 12/12 (MAX)"
  - Column 6 (CLOSED): For manual archiving of completed trades
  - NO confirmation dialog (direct archiving)
  - Archives with current status (❌ for failures, ✅ for completed)

## Open Design Questions (For Future Discussion)

### Statistics Placement
- **Decision**: Strategy statistics appear BEFORE the swimlane header
- **Format**: Compact stats bar with key metrics
- **Considerations**: Visual hierarchy, mobile responsiveness

### Card Content Design
- **Question**: What information appears on instance cards in compact vs detail view?
- **Compact**: Essential info only (instance ID, status, P/L, mode)
- **Detail**: Full trade information, Greeks, technical details

### Archive Management
- **Question**: How much content should Archive column show?
- **Options**: Limited recent items vs separate trade log view
- **Considerations**: Performance, usability, historical analysis needs

### Instance Lifecycle
- **Archive Rules**: 
  - **Manual trigger**: [🗃️ ARCHIVE] button for COMPLETED/FAILED instances
  - **Automatic**: After configurable days (default: 30) automatic archiving
  - **ACTIVE Protection**: Running trades NEVER archivable
  - **STOP Protection**: [🔴 STOP] available in DOING/WAIT/ACTIVE status

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-01-15 | Initial specification |
| 1.1 | 2024-01-15 | Removed global mode switch, added per-strategy control |
| 1.2 | 2024-01-15 | Refined state machine and visual indicators |
| 2.0 | 2024-01-15 | Revolutionary Kanban board architecture with instance management |

## Next Steps

1. Create HTML/CSS prototypes based on these specifications
2. Implement HTMX real-time updates
3. User testing with paper trading mode
4. Iterate based on feedback
5. Production deployment