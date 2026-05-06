# Unified Volume Profile + CVD Trade Decision Indicator
## Complete Code-Ready Implementation Plan — v2.0

---

## Table of Contents

- [Unified Volume Profile + CVD Trade Decision Indicator](#unified-volume-profile--cvd-trade-decision-indicator)
  - [Complete Code-Ready Implementation Plan — v2.0](#complete-code-ready-implementation-plan--v20)
  - [Table of Contents](#table-of-contents)
  - [1. File Structure](#1-file-structure)
  - [2. Indicator Header \& Limits](#2-indicator-header--limits)
  - [3. Input Groups — Complete Specification](#3-input-groups--complete-specification)
    - [GROUP: Profile Settings](#group-profile-settings)
    - [GROUP: Session Definitions](#group-session-definitions)
    - [GROUP: CVD Settings](#group-cvd-settings)
    - [GROUP: SPC Settings](#group-spc-settings)
    - [GROUP: Bias Settings](#group-bias-settings)
    - [GROUP: IB Settings](#group-ib-settings)
    - [GROUP: Previous Sessions — NY RTH](#group-previous-sessions--ny-rth)
    - [GROUP: Previous Sessions — Overnight](#group-previous-sessions--overnight)
    - [GROUP: Previous Sessions — Weekly](#group-previous-sessions--weekly)
    - [GROUP: Session Boxes](#group-session-boxes)
    - [GROUP: Dashboard](#group-dashboard)
    - [GROUP: Colors](#group-colors)
  - [4. UDT Definitions](#4-udt-definitions)
    - [4.1 `ProfileState` — Extended from prior script](#41-profilestate--extended-from-prior-script)
    - [4.2 `SessionBox` — unchanged from prior script](#42-sessionbox--unchanged-from-prior-script)
    - [4.3 `InitialBalance`](#43-initialbalance)
    - [4.4 `CvdSignalState`](#44-cvdsignalstate)
    - [4.5 `SpcSetupState`](#45-spcsetupstate)
    - [4.6 `CompositeScore`](#46-compositescore)
  - [5. Module 1 — Session Time Utilities](#5-module-1--session-time-utilities)
  - [6. Module 2 — CVD Engine (Session-Reset + All-Time)](#6-module-2--cvd-engine-session-reset--all-time)
  - [7. Module 3 — Volume Profile Engine](#7-module-3--volume-profile-engine)
    - [7a. `calcProfile()` — Reusable Calculation Function](#7a-calcprofile--reusable-calculation-function)
    - [7b. LVN / HVN Classification](#7b-lvn--hvn-classification)
    - [7c. Profile Shape Classification](#7c-profile-shape-classification)
    - [7d. Poor High / Poor Low Detection](#7d-poor-high--poor-low-detection)
    - [7e. Session Management \& State Transitions](#7e-session-management--state-transitions)
    - [7f. POC Migration Tracker (within developing session)](#7f-poc-migration-tracker-within-developing-session)
  - [8. Module 4 — Initial Balance Engine](#8-module-4--initial-balance-engine)
  - [9. Module 5 — Opening Gap Analysis](#9-module-5--opening-gap-analysis)
  - [10. Module 6 — Bias Engine](#10-module-6--bias-engine)
    - [6a. 3-Step Checklist](#6a-3-step-checklist)
    - [6b. VA Acceptance / Rejection Logic](#6b-va-acceptance--rejection-logic)
    - [6c. Macro HTF Filter](#6c-macro-htf-filter)
    - [6d. Composite Daily Bias](#6d-composite-daily-bias)
  - [11. Module 7 — CVD Signal Engine](#11-module-7--cvd-signal-engine)
    - [7a. Signal State Variables](#7a-signal-state-variables)
    - [7b. Signal Firing Logic](#7b-signal-firing-logic)
    - [7c. Signal Invalidation](#7c-signal-invalidation)
  - [12. Module 8 — SPC Setup Detector](#12-module-8--spc-setup-detector)
  - [13. Module 9 — Composite Scorer](#13-module-9--composite-scorer)
    - [9a. Point Allocation Table](#9a-point-allocation-table)
    - [9b. Veto Conditions](#9b-veto-conditions)
    - [9c. Composite Score Calculation](#9c-composite-score-calculation)
  - [14. Module 10 — Session Statistics](#14-module-10--session-statistics)
  - [15. Module 11 — Drawing Engine](#15-module-11--drawing-engine)
    - [11a. Profile Polylines (helper functions)](#11a-profile-polylines-helper-functions)
    - [11b. Extended Level Lines](#11b-extended-level-lines)
    - [11c. Naked POC Lines](#11c-naked-poc-lines)
    - [11d. LVN / HVN Lines](#11d-lvn--hvn-lines)
    - [11e. Session Boxes](#11e-session-boxes)
    - [11f. IB Box \& Extension Marks](#11f-ib-box--extension-marks)
    - [11g. SPC Entry Triangles](#11g-spc-entry-triangles)
    - [11h. CVD Signal Diamonds (overlay)](#11h-cvd-signal-diamonds-overlay)
  - [16. Module 12 — Dashboard Renderer](#16-module-12--dashboard-renderer)
  - [17. Module 13 — Alert Conditions](#17-module-13--alert-conditions)
  - [18. Known Pine v5 Constraints \& Workarounds](#18-known-pine-v5-constraints--workarounds)
  - [19. Performance Budget](#19-performance-budget)
  - [20. Known Bugs Fixed from Prior Scripts](#20-known-bugs-fixed-from-prior-scripts)
    - [Bug 1 — `isEndSes` uses `bar_index` instead of `state.endIndex`](#bug-1--isendses-uses-bar_index-instead-of-stateendindex)
    - [Bug 2 — VA Polyline with `lowerRow == upperRow`](#bug-2--va-polyline-with-lowerrow--upperrow)
    - [Bug 3 — `isNewSession` double-fires when `sessInput == "None"`](#bug-3--isnewsession-double-fires-when-sessinput--none)
    - [Bug 4 — Session box accumulation (no expiry)](#bug-4--session-box-accumulation-no-expiry)
    - [Bug 5 — CVD `lastPhBar` anchor advances on non-signal pivots](#bug-5--cvd-lastphbar-anchor-advances-on-non-signal-pivots)
    - [Bug 6 — Labels on `bar_index + labelOffset` misplace on historical scroll](#bug-6--labels-on-bar_index--labeloffset-misplace-on-historical-scroll)
  - [21. Implementation Phases](#21-implementation-phases)
    - [Phase 1 — Scaffold \& Core Profile Engine](#phase-1--scaffold--core-profile-engine)
    - [Phase 2 — Profile Enrichment](#phase-2--profile-enrichment)
    - [Phase 3 — CVD Engine \& Signals](#phase-3--cvd-engine--signals)
    - [Phase 4 — Bias \& Opening Context](#phase-4--bias--opening-context)
    - [Phase 5 — SPC Detector](#phase-5--spc-detector)
    - [Phase 6 — Composite Scorer \& Dashboard](#phase-6--composite-scorer--dashboard)
    - [Phase 7 — Optimization \& Polish](#phase-7--optimization--polish)
  - [22. Companion Pane Script Spec](#22-companion-pane-script-spec)

---

## 1. File Structure

```
unified_vp_cvd.pine          ← Main overlay indicator (this plan)
cvd_pane_companion.pine      ← Lightweight CVD waveform pane (separate visual)
```

The companion pane script shares all CVD input variables and renders:
- Session-reset CVD line + all-time CVD line
- CVD MA + fill
- Pivot dots
- No dashboard, no labels

Both files use identical CVD settings so they stay visually synchronized.

---

## 2. Indicator Header & Limits

```pine
//@version=5
indicator(
    title              = "Unified VP + CVD Trade Decision",
    shorttitle         = "VP·CVD",
    overlay            = true,
    max_lines_count    = 500,
    max_labels_count   = 500,
    max_boxes_count    = 100,
    max_polylines_count = 100
)
```

**Why these limits:**
- `max_lines_count = 500` — extended POC/VAH/VAL lines + LVN/HVN lines + IB levels + SL/T1/T2 lines
- `max_labels_count = 500` — level price labels + SPC/CVD signal labels + IB labels
- `max_boxes_count = 100` — session boxes (NY, ON, LN, AS) + IB box (max ~20 live at once)
- `max_polylines_count = 100` — profile polylines (developing + completed, full + VA)

---

## 3. Input Groups — Complete Specification

### GROUP: Profile Settings
```pine
grp_prof = "Profile Settings"
i_sess         = input.string("NY RTH",  "Session",          options=["Daily","NY RTH","Overnight","London","Asia","None"], group=grp_prof)
i_rows         = input.int(500,          "Rows",             minval=100, maxval=1000, group=grp_prof)
// NOTE: Default 500, NOT 1000. 1000 rows × every bar = primary timeout source.
i_vaPct        = input.float(68.0,       "Value Area %",     minval=1, maxval=99, group=grp_prof)
i_profWidth    = input.float(30.0,       "Profile Width %",  minval=5, maxval=100, group=grp_prof)
i_showN        = input.int(3,            "Show Last N Completed Profiles", minval=1, maxval=10, group=grp_prof)
i_hvnMult      = input.float(1.5,        "HVN Threshold (× mean vol)", minval=1.0, group=grp_prof)
i_lvnMult      = input.float(0.5,        "LVN Threshold (× mean vol)", minval=0.1, maxval=0.99, group=grp_prof)
i_maxLvnLines  = input.int(8,            "Max LVN Lines to Draw", minval=0, maxval=20, group=grp_prof)
```

### GROUP: Session Definitions
```pine
grp_sess = "Session Definitions"
i_nyStr   = input.string("0930-1600", "NY RTH Session",   group=grp_sess)
i_onStr   = input.string("1600-0930", "Overnight Session", group=grp_sess)
i_lnStr   = input.string("0300-1130", "London Session",   group=grp_sess)
i_asStr   = input.string("1800-0300", "Asia Session",     group=grp_sess)
i_tz      = input.string("America/New_York", "Timezone",  group=grp_sess)
i_ibMins  = input.int(60,             "Initial Balance Duration (minutes)", minval=5, maxval=240, group=grp_sess)
```

### GROUP: CVD Settings
```pine
grp_cvd = "CVD Settings"
i_lbL          = input.int(5,   "Left Bars",                minval=1, group=grp_cvd)
i_lbR          = input.int(3,   "Right Bars",               minval=1, group=grp_cvd)
i_minDist      = input.int(5,   "Min Pivot Distance (bars)", minval=1, group=grp_cvd)
i_maxDist      = input.int(60,  "Max Pivot Distance (bars)", minval=1, group=grp_cvd)
i_maLen        = input.int(20,  "CVD MA Length",            minval=1, group=grp_cvd)
i_stalenessBars = input.int(50, "Signal Staleness (bars)",  minval=10, group=grp_cvd)
i_onlyAtLevel  = input.bool(true, "Only Show CVD Signals At Profile Levels", group=grp_cvd)
i_proxRows     = input.int(3,   "Level Proximity (rows)",   minval=1, group=grp_cvd)
```

### GROUP: SPC Settings
```pine
grp_spc = "SPC Settings"
i_showSpc      = input.bool(true,  "Show SPC Setups",       group=grp_spc)
i_spcProxRows  = input.int(2,      "Sweep Proximity (rows)", minval=1, group=grp_spc)
i_spcSlTicks   = input.int(4,      "SL Ticks Beyond Level", minval=1, group=grp_spc)
i_spcAtrSl     = input.bool(true,  "Use ATR-Based SL (overrides tick SL)", group=grp_spc)
i_spcAtrLen    = input.int(14,     "ATR Length for SL",     minval=1, group=grp_spc)
i_spcAtrMult   = input.float(0.5,  "ATR Multiplier for SL", minval=0.1, group=grp_spc)
```

### GROUP: Bias Settings
```pine
grp_bias = "Bias Settings"
i_htfTf        = input.string("D",    "HTF Timeframe for Macro Bias", options=["D","W","M"], group=grp_bias)
i_htfEmaLen    = input.int(20,        "HTF EMA Length",  minval=1, group=grp_bias)
i_weeklyVeto   = input.bool(true,     "Weekly Bias Veto (blocks counter-trend signals)", group=grp_bias)
i_biasBackground = input.bool(false,  "Show Bias Background Tint", group=grp_bias)
i_vaAcceptBars = input.int(2,         "VA Acceptance: Bars Outside VA", minval=1, group=grp_bias)
```

### GROUP: IB Settings
```pine
grp_ib = "Initial Balance"
i_showIbBox    = input.bool(true,  "Show IB Box",              group=grp_ib)
i_showIbExt    = input.bool(true,  "Show IB Extension Levels", group=grp_ib)
i_ibBoxColor   = input.color(color.new(color.yellow, 88), "IB Box Color", group=grp_ib)
i_ibExtColor   = input.color(color.yellow, "IB Extension Color", group=grp_ib)
```

### GROUP: Previous Sessions — NY RTH
```pine
grp_ny = "Previous NY RTH"
i_showNyProf   = input.bool(true,  "Show Previous NY RTH", group=grp_ny)
i_nyProfColor  = input.color(color.new(color.blue, 70),   "Profile Color", group=grp_ny)
i_nyVaColor    = input.color(color.new(color.blue, 30),   "VA Color",      group=grp_ny)
i_nyPocColor   = input.color(color.orange,                "POC Color",     group=grp_ny)
i_nyVahColor   = input.color(color.red,                   "VAH Color",     group=grp_ny)
i_nyValColor   = input.color(color.green,                 "VAL Color",     group=grp_ny)
i_nyRows       = input.int(500, "Rows", group=grp_ny)
i_nyVaPct      = input.float(68.0, "VA %", group=grp_ny)
i_nyWidth      = input.float(30.0, "Width %", group=grp_ny)
i_nyLabelOff   = input.int(40, "Label Offset (bars)", group=grp_ny)
```

### GROUP: Previous Sessions — Overnight
```pine
grp_on = "Previous Overnight"
// Same fields as NY RTH, prefixed with i_on*
// i_showOnProf, i_onProfColor, i_onVaColor, i_onPocColor, i_onVahColor, i_onValColor
// i_onRows, i_onVaPct, i_onWidth, i_onLabelOff
```

### GROUP: Previous Sessions — Weekly
```pine
grp_wk = "Previous Week"
// Same fields as NY RTH, prefixed with i_wk*
// i_showWkProf, i_wkProfColor, i_wkVaColor, i_wkPocColor, i_wkVahColor, i_wkValColor
// i_wkRows, i_wkVaPct, i_wkWidth, i_wkLabelOff
```

### GROUP: Session Boxes
```pine
grp_boxes = "Session Boxes"
i_showNyBox = input.bool(true, "NY Box", group=grp_boxes)
i_showOnBox = input.bool(true, "Overnight Box", group=grp_boxes)
i_showLnBox = input.bool(false, "London Box", group=grp_boxes)
i_showAsBox = input.bool(false, "Asia Box", group=grp_boxes)
// Color inputs for each box (bgcolor + border)
```

### GROUP: Dashboard
```pine
grp_dash = "Dashboard"
i_showDash    = input.bool(true, "Show Dashboard", group=grp_dash)
i_dashPos     = input.string("Top Right", "Position", options=["Top Right","Top Left","Bottom Right","Bottom Left"], group=grp_dash)
i_showRR      = input.bool(true, "Show R:R Calculation", group=grp_dash)
i_showEntZone = input.bool(true, "Show Entry Zone", group=grp_dash)
i_minRR       = input.float(1.5, "Minimum R:R to flag trade", minval=0.5, group=grp_dash)
i_staleThresh = input.int(50,    "Signal staleness threshold (bars)", group=grp_dash)
```

### GROUP: Colors
```pine
grp_col = "Colors"
i_profColor   = input.color(color.new(#747474, 0), "Developing Profile Color", group=grp_col)
i_vaColor     = input.color(color.new(#ffffff, 18), "Developing VA Color", group=grp_col)
i_pocColor    = input.color(color.orange, "POC Color", group=grp_col)
i_vahColor    = input.color(color.red, "VAH Color", group=grp_col)
i_valColor    = input.color(color.green, "VAL Color", group=grp_col)
i_hlColor     = input.color(color.gray, "High/Low Color", group=grp_col)
i_nakedPocCol = input.color(color.fuchsia, "Naked POC Color", group=grp_col)
i_lvnColor    = input.color(color.new(color.gray, 40), "LVN Line Color", group=grp_col)
i_hvnColor    = input.color(color.new(color.white, 50), "HVN Line Color", group=grp_col)
i_wkPocColor  = input.color(color.purple, "Weekly POC Color", group=grp_col)
i_cvdBullAbs  = input.color(color.new(color.green, 0), "CVD Bull Abs Color", group=grp_col)
i_cvdBearAbs  = input.color(color.new(color.red, 0), "CVD Bear Abs Color", group=grp_col)
i_cvdBullExh  = input.color(color.new(color.green, 50), "CVD Bull Exh Color", group=grp_col)
i_cvdBearExh  = input.color(color.new(color.red, 50), "CVD Bear Exh Color", group=grp_col)
```

---

## 4. UDT Definitions

### 4.1 `ProfileState` — Extended from prior script

```pine
type ProfileState
    // Identity
    string  name            // "NY", "ON", "WK"
    bool    show

    // Colors (passed at construction)
    color   profColor
    color   vaColor
    color   pocColor
    color   vahColor
    color   valColor
    int     rowsInput
    float   vaPct
    float   widthPct
    int     labelOffset

    // Accumulation arrays (current session in progress)
    float[] highs
    float[] lows
    float[] vols
    int     startIndex
    int     endIndex        // last bar of the IN-SESSION accumulation
    int     endIndexFinal   // set at isEndSes moment, = endIndex (one bar prior to detection)

    // Completed session levels
    float   prevPoc
    float   prevVah
    float   prevVal
    float   prevHigh
    float   prevLow
    float   prevClose       // close of the final bar of the prior session
    float   prevProfileShape  // 0=D, 1=P, -1=b, 2=B (double distribution)

    // Drawing objects (completed session)
    polyline fullPoly
    polyline vaPoly
    line     pocLine        // extend.right
    line     vahLine        // extend.right
    line     valLine        // extend.right
    label    pocLabel
    label    vahLabel
    label    valLabel

    // Naked POC
    bool    pocNaked
    int     pocNakedSince
    line    nakedPocLine    // distinct style vs visited POC
    label   nakedPocLabel

    // LVN / HVN
    float[] lvnPrices       // midpoint prices of LVN rows
    float[] hvnPrices       // midpoint prices of HVN rows
    line[]  lvnLines
    line[]  hvnLines

    // Poor High / Poor Low
    bool    prevPoorHigh
    bool    prevPoorLow
    line    poorHighLine
    line    poorLowLine

    // POC Migration (within current developing session)
    float   pocMaPrev       // prior POC price for migration comparison
    bool    pocMigratingUp
    bool    pocMigratingDown

    // Session statistics
    float[] sessionRanges   // last 10 completed session ranges
    float   avgSessionRange
    float   avgVaWidth
```

### 4.2 `SessionBox` — unchanged from prior script

```pine
type SessionBox
    box     b
    float   h
    float   l
    int     startIdx
```

### 4.3 `InitialBalance`

```pine
type InitialBalance
    bool    set             // true once IB period has completed
    float   ibHigh
    float   ibLow
    float   ibRange
    int     startBar
    int     endBar
    bool    extendedUp      // price has broken above ibHigh after IB closed
    bool    extendedDown    // price has broken below ibLow after IB closed
    box     ibBox
    line    ibHighLine
    line    ibLowLine
    line    ibExtUpLine
    line    ibExtDownLine
    label   ibHighLabel
    label   ibLowLabel
```

### 4.4 `CvdSignalState`

```pine
type CvdSignalState
    string  lastSig         // "BullAbs" | "BullExh" | "BearAbs" | "BearExh" | "None"
    int     lastSigBar
    float   lastSigPrice    // price at the CVD pivot bar
    float   lastSigCvd
    int     lastTier        // 1 | 2 | 3
    bool    atLevel         // was the signal within proximity of a key level
    bool    valid           // false = stale or price-invalidated
    color   sigColor
    string  sigText
    string  actionText
    string  qualityText
    bool    isBull
    bool    isAbs
```

### 4.5 `SpcSetupState`

```pine
type SpcSetupState
    bool    pendingLong
    bool    pendingShort
    float   sweptLevel
    string  sweptLevelName  // "PD VAL" | "PD POC" | "PD VAH" | "WK POC" etc.
    int     sweepBar
    float   sweepHigh       // high of the sweep bar (for SL reference)
    float   sweepLow        // low of the sweep bar
    bool    atLvn           // was the swept level near an LVN
```

### 4.6 `CompositeScore`

```pine
type CompositeScore
    int     bullPts
    int     bearPts
    string  direction       // "LONG" | "SHORT" | "WAIT"
    string  confidence      // "HIGH" | "MEDIUM" | "LOW"
    float   entryPrice
    float   slPrice
    float   t1Price         // nearest POC or LVN/HVN
    float   t2Price         // next VA boundary
    float   rrRatio
    bool    rrValid         // rrRatio >= i_minRR
    string  vetoReason      // non-empty if a veto suppressed a signal
```

---

## 5. Module 1 — Session Time Utilities

```pine
// ── Session strings ──────────────────────────────────────────────────────────
f_sesStr() =>
    switch i_sess
        "NY RTH"    => i_nyStr
        "Overnight" => i_onStr
        "London"    => i_lnStr
        "Asia"      => i_asStr
        => "0000-0000"

bool isDaily    = i_sess == "Daily"
bool isNone     = i_sess == "None"
string mainSess = f_sesStr()

// Main session detection
bool inMainSess  = isNone ? false : isDaily ? true : not na(time(timeframe.period, mainSess, i_tz))
bool isNewSess   = isNone ? false : isDaily ? ta.change(time("D")) != 0 : (inMainSess and not inMainSess[1])
bool isEndSess   = isNone ? false : isDaily ? false : (not inMainSess and inMainSess[1])

// Per-session flags used by ProfileState instances
bool inNY    = not na(time(timeframe.period, i_nyStr, i_tz))
bool isNewNY = inNY and not inNY[1]
bool isEndNY = not inNY and inNY[1]

bool inON    = not na(time(timeframe.period, i_onStr, i_tz))
bool isNewON = inON and not inON[1]
bool isEndON = not inON and inON[1]

bool inWK    = true
bool isNewWK = ta.change(time("W")) != 0
bool isEndWK = isNewWK   // week boundary = end of prior week

// IB window: first i_ibMins minutes of NY RTH
// time("D") resets at midnight; NY RTH opens at 09:30
// Track IB separately using a bar counter from isNewNY
var int ibBarCount = 0
if isNewNY
    ibBarCount := 0
if inNY and not isNewNY
    ibBarCount += 1

// IB bars = i_ibMins / timeframe.in_seconds(timeframe.period) * 60
float tfMins = timeframe.in_seconds(timeframe.period) / 60.0
int   ibBars = math.ceil(i_ibMins / tfMins)
bool  inIB   = inNY and ibBarCount < ibBars
bool  ibJustClosed = inNY and ibBarCount == ibBars  // first bar AFTER IB period
```

---

## 6. Module 2 — CVD Engine (Session-Reset + All-Time)

```pine
// ── OHLCV delta approximation ────────────────────────────────────────────────
float rng      = high - low
float bullPow  = rng == 0 ? 0.0 : (close - low)
float bearPow  = rng == 0 ? 0.0 : (high - close)
float totalPwr = bullPow + bearPow
float bullVol  = totalPwr == 0 ? volume * 0.5 : volume * (bullPow / totalPwr)
float bearVol  = totalPwr == 0 ? volume * 0.5 : volume * (bearPow / totalPwr)
float barDelta = bullVol - bearVol

// ── All-time CVD (cumulative, never resets) ───────────────────────────────────
float cvdAll = ta.cum(barDelta)

// ── Session CVD (resets at main session open) ─────────────────────────────────
// This is the PRIMARY signal engine input. Session-scoped delta is more
// sensitive to within-session order flow shifts than all-time CVD.
var float cvdSess = 0.0
if isNewSess
    cvdSess := 0.0
cvdSess += barDelta

float cvdMa = ta.sma(cvdSess, i_maLen)

// ── CVD state ─────────────────────────────────────────────────────────────────
string cvdFlow = cvdSess > cvdMa ? "Bullish" : "Bearish"
float  cvdVsMaPct = math.abs(cvdMa) > 0 ? (cvdSess - cvdMa) / math.abs(cvdMa) * 100 : 0.0

// ── Session CVD extremes (context for exhaustion assessment) ─────────────────
var float sessMaxCvd = 0.0
var float sessMinCvd = 0.0
if isNewSess
    sessMaxCvd := cvdSess
    sessMinCvd := cvdSess
if inMainSess
    sessMaxCvd := math.max(sessMaxCvd, cvdSess)
    sessMinCvd := math.min(sessMinCvd, cvdSess)

// sessMaxCvd / sessMinCvd used in tier boosting when signal fires near session extremes

// ── Pivot detection (operates on session CVD) ─────────────────────────────────
float cvdPH = ta.pivothigh(cvdSess, i_lbL, i_lbR)
float cvdPL = ta.pivotlow (cvdSess, i_lbL, i_lbR)

// ── Normalization windows ─────────────────────────────────────────────────────
float cvdRange = math.max(ta.highest(cvdSess, i_maxDist) - ta.lowest(cvdSess, i_maxDist), 1.0)
float prcRange = math.max(ta.highest(high,    i_maxDist) - ta.lowest(low,     i_maxDist), 0.0001)
```

---

## 7. Module 3 — Volume Profile Engine

### 7a. `calcProfile()` — Reusable Calculation Function

```pine
// Returns a tuple of all profile analytics.
// Called once when a session ends and once per bar for the developing profile.
// NOTE: Pine functions cannot return arrays directly; rowVols is mutated in place.
// Caller must pass a pre-allocated float[] of size `rows` filled with 0.0.

calcProfile(
    float[] highs,
    float[] lows,
    float[] vols,
    int     rows,
    float   vaPct,
    float[] rowVols   // mutated in place — must be pre-allocated to `rows` size with 0.0
) =>
    float dHigh    = array.max(highs)
    float dLow     = array.min(lows)
    float rowSize  = (dHigh - dLow) / rows
    float totalVol = 0.0

    if rowSize <= 0
        // Degenerate session (all bars same price) — return safe defaults
        => [0.0, 0.0, 0.0, dHigh, dLow, rowSize, 0.0, 0, 0, 0, 0.0]

    // Distribute volume into rows
    int n = array.size(highs)
    for i = 0 to math.min(n - 1, 5000)   // loop guard: max 5000 bars
        float h = array.get(highs, i)
        float l = array.get(lows,  i)
        float v = array.get(vols,  i)
        int   startRow = math.max(0, math.floor((l - dLow) / rowSize))
        int   endRow   = math.min(rows - 1, math.floor((h - dLow) / rowSize))
        int   rowsSpanned = endRow - startRow + 1
        float volPerRow   = rowsSpanned > 0 ? v / rowsSpanned : 0.0
        for j = startRow to endRow
            array.set(rowVols, j, array.get(rowVols, j) + volPerRow)
            totalVol += volPerRow

    // POC
    float maxVol = 0.0
    int   pocRow = 0
    for j = 0 to rows - 1
        float v = array.get(rowVols, j)
        if v > maxVol
            maxVol := v
            pocRow := j

    // Value Area expansion
    float vaTarget   = totalVol * (vaPct / 100.0)
    float currentVaVol = array.get(rowVols, pocRow)
    int   upperRow   = pocRow
    int   lowerRow   = pocRow

    while currentVaVol < vaTarget and (upperRow < rows - 1 or lowerRow > 0)
        float upVol  = upperRow < rows - 1 ? array.get(rowVols, upperRow + 1) : -1.0
        float dnVol  = lowerRow > 0        ? array.get(rowVols, lowerRow - 1) : -1.0
        if upVol >= dnVol and upVol != -1.0
            upperRow      += 1
            currentVaVol  += upVol
        else if dnVol != -1.0
            lowerRow      -= 1
            currentVaVol  += dnVol

    float pocPrice = dLow + (pocRow  * rowSize) + (rowSize / 2)
    float vahPrice = dLow + (upperRow * rowSize) + (rowSize / 2)
    float valPrice = dLow + (lowerRow * rowSize) + (rowSize / 2)

    [pocPrice, vahPrice, valPrice, dHigh, dLow, rowSize, maxVol, pocRow, upperRow, lowerRow, totalVol]
```

**Important:** `rowVols` is passed by reference (array mutation). Always `array.clear()` and `array.fill(0.0)` before calling.

---

### 7b. LVN / HVN Classification

```pine
// Called after calcProfile() returns rowVols, totalVol, and rows.
// Populates state.lvnPrices and state.hvnPrices.

f_classifyNodes(float[] rowVols, int rows, float rowSize, float dLow, float totalVol, float hvnMult, float lvnMult, float[] lvnOut, float[] hvnOut) =>
    array.clear(lvnOut)
    array.clear(hvnOut)

    float meanVol = rows > 0 ? totalVol / rows : 0.0
    if meanVol <= 0
        => na

    float hvnThresh = meanVol * hvnMult
    float lvnThresh = meanVol * lvnMult

    for j = 0 to rows - 1
        float vol   = array.get(rowVols, j)
        float price = dLow + (j * rowSize) + (rowSize / 2)
        if vol >= hvnThresh
            array.push(hvnOut, price)
        else if vol <= lvnThresh
            array.push(lvnOut, price)
```

**Usage note:** After classification, sort `lvnOut` and `hvnOut` (insertion sort — Pine has no native array sort).

```pine
// Insertion sort ascending (for LVN/HVN price arrays)
f_sortAsc(float[] arr) =>
    int n = array.size(arr)
    for i = 1 to n - 1
        float key = array.get(arr, i)
        int   j   = i - 1
        while j >= 0 and array.get(arr, j) > key
            array.set(arr, j + 1, array.get(arr, j))
            j -= 1
        array.set(arr, j + 1, key)
```

---

### 7c. Profile Shape Classification

```pine
// Returns: 1=P (bullish), -1=b (bearish), 0=D (balanced), 2=B (double dist.)
f_profileShape(int pocRow, int rows, float[] rowVols, float totalVol) =>
    float pocRelPos = rows > 0 ? pocRow / rows : 0.5

    // Check for double distribution (B-profile):
    // Two separate clusters of high-volume rows separated by a low-volume gap in the middle
    // Heuristic: find highest-vol row in upper half and lower half,
    // check if there is an LVN gap between them
    int   midRow    = rows / 2
    float upperMax  = 0.0
    float lowerMax  = 0.0
    int   upperMaxRow = 0
    int   lowerMaxRow = 0
    float meanVol   = totalVol / rows

    for j = 0 to midRow - 1
        float v = array.get(rowVols, j)
        if v > lowerMax
            lowerMax    := v
            lowerMaxRow := j

    for j = midRow to rows - 1
        float v = array.get(rowVols, j)
        if v > upperMax
            upperMax    := v
            upperMaxRow := j

    // Check for valley between the two peaks
    bool hasValley = false
    if upperMaxRow > lowerMaxRow + 2
        for j = lowerMaxRow + 1 to upperMaxRow - 1
            if array.get(rowVols, j) < meanVol * 0.4
                hasValley := true
                break

    int shape = 0
    if hasValley and lowerMax > meanVol * 1.3 and upperMax > meanVol * 1.3
        shape := 2   // B-profile
    else if pocRelPos > 0.65
        shape := 1   // P-profile
    else if pocRelPos < 0.35
        shape := -1  // b-profile
    // else shape = 0 (D-profile)

    shape
```

---

### 7d. Poor High / Poor Low Detection

```pine
// Count bars within tickThreshold of session high/low.
// A "poor" extreme means the auction ran out of time without completing —
// price is likely to return.

f_detectPoorExtremes(float[] highs, float[] lows, float dHigh, float dLow) =>
    float tickThresh = syminfo.mintick * 4
    int   barsNearHigh = 0
    int   barsNearLow  = 0
    int   n = array.size(highs)

    for i = 0 to math.min(n - 1, 5000)
        if math.abs(array.get(highs, i) - dHigh) <= tickThresh
            barsNearHigh += 1
        if math.abs(array.get(lows, i) - dLow) <= tickThresh
            barsNearLow += 1

    [barsNearHigh >= 3, barsNearLow >= 3]
```

Store `[state.prevPoorHigh, state.prevPoorLow]` in `ProfileState` after each session ends.

---

### 7e. Session Management & State Transitions

The `processSession` method on `ProfileState`. This is where all session lifecycle events are handled.

```pine
method processSession(ProfileState state, bool inSes, bool isNewSes, bool isEndSes) =>

    // ── STEP 1: Session end — finalize and draw ───────────────────────────────
    if isEndSes and array.size(state.highs) > 0
        // BUG FIX from v1: use state.endIndex (last inSes bar), NOT bar_index.
        // At isEndSes, bar_index is the FIRST bar of the next session.
        int finalEnd = state.endIndex

        // Alloc rowVols
        float[] rowVols = array.new_float(state.rowsInput, 0.0)

        // Calculate
        [poc, vah, val, dHigh, dLow, rowSize, maxVol, pocRow, upperRow, lowerRow, totalVol] =
            calcProfile(state.highs, state.lows, state.vols, state.rowsInput, state.vaPct, rowVols)

        if rowSize > 0
            // Store completed levels
            state.prevPoc   := poc
            state.prevVah   := vah
            state.prevVal   := val
            state.prevHigh  := dHigh
            state.prevLow   := dLow
            state.prevClose := close[1]  // close of the last bar IN the session

            // Profile shape
            state.prevProfileShape := f_profileShape(pocRow, state.rowsInput, rowVols, totalVol)

            // LVN / HVN
            array.clear(state.lvnPrices)
            array.clear(state.hvnPrices)
            f_classifyNodes(rowVols, state.rowsInput, rowSize, dLow, totalVol, i_hvnMult, i_lvnMult, state.lvnPrices, state.hvnPrices)
            f_sortAsc(state.lvnPrices)
            f_sortAsc(state.hvnPrices)

            // Poor High / Poor Low
            [pH, pL] = f_detectPoorExtremes(state.highs, state.lows, dHigh, dLow)
            state.prevPoorHigh := pH
            state.prevPoorLow  := pL

            // Naked POC — new session's POC is naked until price touches it
            state.pocNaked      := true
            state.pocNakedSince := bar_index

            // Session range statistics (rolling last 10)
            array.push(state.sessionRanges, dHigh - dLow)
            if array.size(state.sessionRanges) > 10
                array.shift(state.sessionRanges)
            state.avgSessionRange := array.avg(state.sessionRanges)
            state.avgVaWidth      := state.avgVaWidth * 0.8 + (vah - val) * 0.2  // EMA approximation

            // Delete old drawing objects
            f_deleteProfileDrawings(state)   // helper — deletes all polylines/lines/labels on state

            // Draw profile polylines
            int dayLen       = finalEnd - state.startIndex + 1
            float maxWidBars = dayLen * (state.widthPct / 100.0)

            state.fullPoly := f_drawFullPoly(state.startIndex, dLow, dHigh, state.rowsInput, rowVols, maxVol, rowSize, maxWidBars, state.profColor)
            state.vaPoly   := f_drawVaPoly(state.startIndex, dLow, upperRow, lowerRow, vah, val, state.rowsInput, rowVols, maxVol, rowSize, maxWidBars, state.vaColor)

            // Extended level lines (extend.right so they follow price forward)
            state.pocLine  := line.new(state.startIndex, poc, finalEnd, poc, color=state.pocColor, width=2, extend=extend.right)
            state.vahLine  := line.new(state.startIndex, vah, finalEnd, vah, color=state.vahColor, width=1, style=line.style_dashed, extend=extend.right)
            state.valLine  := line.new(state.startIndex, val, finalEnd, val, color=state.valColor, width=1, style=line.style_dashed, extend=extend.right)

            // Labels (placed at finalEnd + labelOffset, refreshed each bar)
            f_refreshLabels(state, poc, vah, val, bar_index)

            // LVN lines (capped at i_maxLvnLines)
            f_drawLvnLines(state, dLow, dHigh, state.startIndex, finalEnd)

    // ── STEP 2: Session start — reset state ──────────────────────────────────
    if isNewSes
        state.startIndex := bar_index
        array.clear(state.highs)
        array.clear(state.lows)
        array.clear(state.vols)
        state.pocMaPrev      := na
        state.pocMigratingUp := false
        state.pocMigratingDown := false

    // ── STEP 3: Accumulate current bar ───────────────────────────────────────
    if inSes
        array.push(state.highs, high)
        array.push(state.lows,  low)
        array.push(state.vols,  volume)
        state.endIndex := bar_index  // always the LAST inSes bar

    // ── STEP 4: Naked POC check (every bar) ──────────────────────────────────
    if state.pocNaked and not na(state.prevPoc)
        if low <= state.prevPoc and high >= state.prevPoc
            state.pocNaked := false
            // Redraw POC line in "visited" style
            if not na(state.nakedPocLine)
                line.delete(state.nakedPocLine)
                state.nakedPocLine := na
            // Update existing pocLine style to visited (solid, thinner)
            if not na(state.pocLine)
                line.set_style(state.pocLine, line.style_solid)
                line.set_width(state.pocLine, 1)

    // ── STEP 5: Naked POC line — create / update ─────────────────────────────
    if state.pocNaked and not na(state.prevPoc)
        if na(state.nakedPocLine)
            state.nakedPocLine := line.new(state.pocNakedSince, state.prevPoc, bar_index, state.prevPoc, color=i_nakedPocCol, width=2, style=line.style_dotted, extend=extend.right)
        // No need to update — extend.right handles it

    // ── STEP 6: Label refresh (every bar) ────────────────────────────────────
    if not na(state.prevPoc)
        f_refreshLabels(state, state.prevPoc, state.prevVah, state.prevVal, bar_index)
```

---

### 7f. POC Migration Tracker (within developing session)

Called inside the developing profile calculation block (runs every bar when `inSes`):

```pine
// After computing currPocPrice for the developing profile:
if not na(state.pocMaPrev)
    float pocDelta = currPocPrice - state.pocMaPrev
    float migThresh = currRowSize * 2   // must move > 2 rows to be considered migration
    state.pocMigratingUp   := pocDelta > migThresh
    state.pocMigratingDown := pocDelta < -migThresh

state.pocMaPrev := currPocPrice

// Used in bias engine and dashboard:
// Migrating POC = trending market → follow trend, don't fade
// Stationary POC = rotational market → fade extremes back to POC
```

---

## 8. Module 4 — Initial Balance Engine

```pine
var InitialBalance ib = InitialBalance.new(false, na, na, na, 0, 0, false, false, na, na, na, na, na, na, na)

if isNewNY
    // Reset IB state
    ib.set      := false
    ib.ibHigh   := high
    ib.ibLow    := low
    ib.startBar := bar_index
    ib.endBar   := na
    ib.extendedUp   := false
    ib.extendedDown := false

    // Delete old IB drawings
    if not na(ib.ibBox)
        box.delete(ib.ibBox)
        ib.ibBox := na
    if not na(ib.ibHighLine)
        line.delete(ib.ibHighLine)
        line.delete(ib.ibLowLine)
        ib.ibHighLine := na
        ib.ibLowLine  := na
    // Delete extension lines / labels similarly

if inIB and not isNewNY
    // Expand IB range
    ib.ibHigh := math.max(ib.ibHigh, high)
    ib.ibLow  := math.min(ib.ibLow,  low)

    // Draw / update IB box
    if i_showIbBox
        if na(ib.ibBox)
            ib.ibBox := box.new(ib.startBar, ib.ibHigh, bar_index, ib.ibLow, border_color=i_ibExtColor, bgcolor=i_ibBoxColor)
        else
            box.set_top(ib.ibBox, ib.ibHigh)
            box.set_bottom(ib.ibBox, ib.ibLow)
            box.set_right(ib.ibBox, bar_index)

if ibJustClosed
    ib.set     := true
    ib.endBar  := bar_index
    ib.ibRange := ib.ibHigh - ib.ibLow

    if i_showIbExt
        // Draw IB high/low as extend.right lines
        ib.ibHighLine := line.new(ib.startBar, ib.ibHigh, bar_index, ib.ibHigh, color=i_ibExtColor, style=line.style_dashed, extend=extend.right)
        ib.ibLowLine  := line.new(ib.startBar, ib.ibLow,  bar_index, ib.ibLow,  color=i_ibExtColor, style=line.style_dashed, extend=extend.right)
        ib.ibHighLabel := label.new(bar_index, ib.ibHigh, "IB High: " + str.tostring(ib.ibHigh, format.mintick), textcolor=i_ibExtColor, color=color.new(i_ibExtColor, 90), style=label.style_label_left, size=size.small)
        ib.ibLowLabel  := label.new(bar_index, ib.ibLow,  "IB Low: "  + str.tostring(ib.ibLow,  format.mintick), textcolor=i_ibExtColor, color=color.new(i_ibExtColor, 90), style=label.style_label_left, size=size.small)

// IB Extension detection (after IB is set)
if ib.set and inNY
    if not ib.extendedUp and close > ib.ibHigh
        ib.extendedUp := true
        if i_showIbExt
            label.new(bar_index, high, "▲ IB EXT", color=color.new(i_ibExtColor, 70), textcolor=color.white, style=label.style_label_down, size=size.small)

    if not ib.extendedDown and close < ib.ibLow
        ib.extendedDown := true
        if i_showIbExt
            label.new(bar_index, low, "▼ IB EXT", color=color.new(i_ibExtColor, 70), textcolor=color.white, style=label.style_label_up, size=size.small)

// Expose for scorer:
bool ibAlignsBull = ib.set and ib.extendedUp    // IB extension upward
bool ibAlignsBear = ib.set and ib.extendedDown   // IB extension downward
float ibNarrowFactor = not na(nyState.avgSessionRange) and nyState.avgSessionRange > 0 ? ib.ibRange / nyState.avgSessionRange : na
bool ibNarrow = not na(ibNarrowFactor) and ibNarrowFactor < 0.5   // narrow IB = breakout potential
```

---

## 9. Module 5 — Opening Gap Analysis

```pine
// Runs on the first bar of NY RTH (isNewNY)
var float gapSize     = na
var bool  gapUp       = false
var bool  gapDown     = false
var bool  gapIntoVA   = false    // opened inside prior VA (mean reversion target = POC)
var bool  gapOutOfVA  = false    // opened outside prior VA (acceptance / continuation)
var bool  gapFilled   = false
var float prevSessClose = na

// Capture close of last NY session bar at isEndNY
if isEndNY
    prevSessClose := close

if isNewNY and not na(prevSessClose) and not na(nyState.prevVah)
    gapSize    := open - prevSessClose
    gapUp      := gapSize > nyState.prevVah - nyState.prevVal   // gap > prior VA width
    gapDown    := gapSize < -(nyState.prevVah - nyState.prevVal)

    // Opened inside prior VA = gap-into-VA (mean reversion to POC expected)
    gapIntoVA  := open >= nyState.prevVal and open <= nyState.prevVah
    // Opened outside prior VA in gap direction
    gapOutOfVA := (gapSize > 0 and open > nyState.prevVah) or
                  (gapSize < 0 and open < nyState.prevVal)

    gapFilled  := false   // reset fill tracker

// Gap fill tracker
if (gapUp or gapDown) and not gapFilled and not na(prevSessClose)
    if (gapUp and low <= prevSessClose) or (gapDown and high >= prevSessClose)
        gapFilled := true

// Scoring contribution (see Module 9):
// gapIntoVA  → +2 to bias TOWARD POC (mean reversion)
// gapOutOfVA → +1 in direction of gap (continuation)
// gapFilled  → gap is no longer a directional driver
```

---

## 10. Module 6 — Bias Engine

### 6a. 3-Step Checklist

```pine
// Computed once per session open bar (isNewNY)
// All "prev" references come from nyState.prev* fields.

var string closeBias   = "Neutral"
var string openBias    = "Neutral"
var string weeklyBias  = "Neutral"
var string dailyBias   = "Neutral"

if isNewNY
    // Step 1: Yesterday's close vs yesterday's VA
    if not na(nyState.prevClose) and not na(nyState.prevVah)
        closeBias := nyState.prevClose > nyState.prevVah ? "Bullish" :
                     nyState.prevClose < nyState.prevVal ? "Bearish" : "Neutral"

    // Step 2: Today's open vs yesterday's VA
    if not na(nyState.prevVah)
        openBias := open > nyState.prevVah ? "Bullish" :
                    open < nyState.prevVal ? "Bearish" : "Neutral"

    // Step 3: Weekly VA
    if not na(wkState.prevVah)
        weeklyBias := open > wkState.prevVah ? "Bullish" :
                      open < wkState.prevVal ? "Bearish" : "Neutral"

    // Composite
    if closeBias == openBias
        dailyBias := closeBias
    else if weeklyBias != "Neutral"
        dailyBias := weeklyBias     // weekly resolves conflict
    else
        dailyBias := "Neutral"      // genuine conflict = wait

// Profile shape vote
string shapeBias = "Neutral"
if nyState.prevProfileShape == 1
    shapeBias := "Bullish"    // P-profile = buyers in control
else if nyState.prevProfileShape == -1
    shapeBias := "Bearish"    // b-profile = sellers in control
// B or D = neutral
```

### 6b. VA Acceptance / Rejection Logic

```pine
// These update every bar (not just session open)
var int barsAboveVah = 0
var int barsBelowVal = 0

if not na(nyState.prevVah) and not na(nyState.prevVal)
    if close > nyState.prevVah
        barsAboveVah += 1
    else
        barsAboveVah := 0

    if close < nyState.prevVal
        barsBelowVal += 1
    else
        barsBelowVal := 0

bool vaAcceptAbove = barsAboveVah >= i_vaAcceptBars   // bullish continuation
bool vaAcceptBelow = barsBelowVal >= i_vaAcceptBars   // bearish continuation
bool vaRejectAbove = close < nyState.prevVah and close[1] > nyState.prevVah  // failed breakout → fade
bool vaRejectBelow = close > nyState.prevVal and close[1] < nyState.prevVal  // failed breakdown → fade

// Price position string for dashboard
string pricePosition = "—"
if not na(nyState.prevVah)
    pricePosition := close > nyState.prevVah ? "Above VAH (Premium)" :
                     close < nyState.prevVal ? "Below VAL (Discount)" :
                     close > nyState.prevPoc ? "Inside VA — Above POC" : "Inside VA — Below POC"
```

### 6c. Macro HTF Filter

```pine
// Uses request.security with lookahead=off to avoid future leak
// barmerge.gaps_off ensures we always get a value even on non-trading days

float htfClose = request.security(syminfo.tickerid, i_htfTf, close[1],   lookahead=barmerge.lookahead_off, gaps=barmerge.gaps_off)
float htfEma   = request.security(syminfo.tickerid, i_htfTf, ta.ema(close, i_htfEmaLen)[1], lookahead=barmerge.lookahead_off, gaps=barmerge.gaps_off)

string macroBias = not na(htfClose) and not na(htfEma) ?
                   (htfClose > htfEma ? "Bullish" : "Bearish") : "Neutral"
```

### 6d. Composite Daily Bias

```pine
// Persists across bars (updated on session transitions)
var string compositeBias = "Neutral"
var int    biasStrength  = 0   // 0=weak, 1=medium, 2=strong

if isNewNY
    int bullBiasVotes = 0
    int bearBiasVotes = 0

    if closeBias == "Bullish"  then bullBiasVotes += 1 else if closeBias == "Bearish"  then bearBiasVotes += 1
    if openBias  == "Bullish"  then bullBiasVotes += 1 else if openBias  == "Bearish"  then bearBiasVotes += 1
    if weeklyBias == "Bullish" then bullBiasVotes += 2 else if weeklyBias == "Bearish" then bearBiasVotes += 2
    if macroBias == "Bullish"  then bullBiasVotes += 2 else if macroBias == "Bearish"  then bearBiasVotes += 2
    if shapeBias == "Bullish"  then bullBiasVotes += 1 else if shapeBias == "Bearish"  then bearBiasVotes += 1

    compositeBias := bullBiasVotes > bearBiasVotes ? "Bullish" :
                     bearBiasVotes > bullBiasVotes ? "Bearish" : "Neutral"
    biasStrength  := math.abs(bullBiasVotes - bearBiasVotes)
```

---

## 11. Module 7 — CVD Signal Engine

### 7a. Signal State Variables

```pine
// Pivot reference anchors (only advance under strict conditions — see 7b)
var int   lastPhBar   = na
var float lastPhPrice = na
var float lastPhCvd   = na

var int   lastPlBar   = na
var float lastPlPrice = na
var float lastPlCvd   = na

// Signal state object
var CvdSignalState cvdState = CvdSignalState.new("None", na, na, na, 0, false, false, color.gray, "—", "—", "—", na, na)
```

### 7b. Signal Firing Logic

```pine
// ── Helper: is price near a key level? ───────────────────────────────────────
f_nearKeyLevel(float price, float rowSize) =>
    float thresh = rowSize * i_proxRows
    bool  near   = false
    string nearName = "—"

    // Check all key levels from all active ProfileState instances
    float[] levels     = array.new_float()
    string[] levelNames = array.new_string()

    // NY levels
    if not na(nyState.prevPoc)  => array.push(levels, nyState.prevPoc),  array.push(levelNames, "PD POC")
    if not na(nyState.prevVah)  => array.push(levels, nyState.prevVah),  array.push(levelNames, "PD VAH")
    if not na(nyState.prevVal)  => array.push(levels, nyState.prevVal),  array.push(levelNames, "PD VAL")
    // ON levels
    if not na(onState.prevPoc)  => array.push(levels, onState.prevPoc),  array.push(levelNames, "PON POC")
    // WK levels
    if not na(wkState.prevPoc)  => array.push(levels, wkState.prevPoc),  array.push(levelNames, "WK POC")
    if not na(wkState.prevVah)  => array.push(levels, wkState.prevVah),  array.push(levelNames, "WK VAH")
    if not na(wkState.prevVal)  => array.push(levels, wkState.prevVal),  array.push(levelNames, "WK VAL")
    // IB levels
    if ib.set and not na(ib.ibHigh) => array.push(levels, ib.ibHigh), array.push(levelNames, "IB High")
    if ib.set and not na(ib.ibLow)  => array.push(levels, ib.ibLow),  array.push(levelNames, "IB Low")
    // LVN levels (from nyState)
    for i = 0 to math.min(array.size(nyState.lvnPrices) - 1, 10)
        array.push(levels, array.get(nyState.lvnPrices, i))
        array.push(levelNames, "LVN")

    for i = 0 to array.size(levels) - 1
        if math.abs(price - array.get(levels, i)) <= thresh
            near     := true
            nearName := array.get(levelNames, i)
            break

    [near, nearName]

// ── Proximity boost calculation ───────────────────────────────────────────────
f_proximityBoost(float price, float rowSize) =>
    float boost = 0.0
    float thresh = rowSize * i_proxRows

    float[] checkLevels = array.new_float()
    if not na(nyState.prevPoc) => array.push(checkLevels, nyState.prevPoc)
    if not na(nyState.prevVah) => array.push(checkLevels, nyState.prevVah)
    if not na(nyState.prevVal) => array.push(checkLevels, nyState.prevVal)
    if not na(wkState.prevPoc) => array.push(checkLevels, wkState.prevPoc)
    for i = 0 to math.min(array.size(nyState.lvnPrices) - 1, 5)
        array.push(checkLevels, array.get(nyState.lvnPrices, i))

    for i = 0 to array.size(checkLevels) - 1
        if math.abs(price - array.get(checkLevels, i)) <= thresh
            boost += 0.15

    // Additional boost if signal fires at a naked POC
    if nyState.pocNaked and not na(nyState.prevPoc)
        if math.abs(price - nyState.prevPoc) <= thresh
            boost += 0.20

    math.min(boost, 0.40)   // cap boost at +0.40
```

```pine
// ── Bearish signals (CVD pivot highs) ────────────────────────────────────────
if not na(cvdPH)
    int   curBar   = bar_index - i_lbR
    float curPrice = high[i_lbR]
    float curCvd   = cvdSess[i_lbR]

    // CRITICAL FIX: pivot reference only advances when dist >= minDist
    if na(lastPhBar) or (curBar - lastPhBar) >= i_minDist
        if not na(lastPhBar) and (curBar - lastPhBar) <= i_maxDist
            float rowSizeRef = not na(nyState.prevPoc) ? (nyState.prevHigh - nyState.prevLow) / nyState.rowsInput : syminfo.mintick * 10

            float pDiff = math.abs(curPrice - lastPhPrice) / prcRange
            float cDiff = math.abs(curCvd   - lastPhCvd)  / cvdRange
            float proxBoost = f_proximityBoost(curPrice, rowSizeRef)
            float score = pDiff + cDiff + proxBoost
            int   tier  = score >= 0.50 ? 3 : score >= 0.25 ? 2 : 1
            int   lw    = tier == 3 ? 3 : tier == 2 ? 2 : 1

            bool isExh = curPrice > lastPhPrice and curCvd < lastPhCvd   // Price HH, CVD LH
            bool isAbs = curPrice < lastPhPrice and curCvd > lastPhCvd   // Price LH, CVD HH

            [near, nearName] = f_nearKeyLevel(curPrice, rowSizeRef)
            bool shouldDraw = not i_onlyAtLevel or near

            if isExh and shouldDraw
                // [draw line, label, from-dot — same as Script B v6]
                cvdState.lastSig      := "BearExh"
                cvdState.lastSigBar   := curBar
                cvdState.lastSigPrice := curPrice
                cvdState.lastTier     := tier
                cvdState.atLevel      := near
                cvdState.valid        := true
                cvdState.isBull       := false
                cvdState.isAbs        := false
                cvdState.sigColor     := i_cvdBearExh
                cvdState.sigText      := "Exhausted Buyers"
                cvdState.actionText   := "⚠ Tighten longs / Sniper short"
                cvdState.qualityText  := tier==3 ? "★★★ Buyers stalling" : tier==2 ? "★★☆ Momentum fading" : "★☆☆ Marginal"
                alert("Bear Exhaustion | Price HH | CVD LH" + (near ? " | At " + nearName : ""), alert.freq_once_per_bar_close)

            else if isAbs and shouldDraw
                // [draw line, label, from-dot]
                cvdState.lastSig      := "BearAbs"
                cvdState.lastSigBar   := curBar
                cvdState.lastSigPrice := curPrice
                cvdState.lastTier     := tier
                cvdState.atLevel      := near
                cvdState.valid        := true
                cvdState.isBull       := false
                cvdState.isAbs        := true
                cvdState.sigColor     := i_cvdBearAbs
                cvdState.sigText      := "Buying Pressure Absorbed"
                cvdState.actionText   := "✅ SHORT — Sell-limit wall active"
                cvdState.qualityText  := tier==3 ? "★★★ Strong wall" : tier==2 ? "★★☆ Clear absorption" : "★☆☆ Weak — confirm"
                alert("Bear Absorption | Price LH | CVD HH" + (near ? " | At " + nearName : ""), alert.freq_once_per_bar_close)

        // Advance anchor — always, regardless of signal fire
        // (This is correct behavior: if no signal, the pivot is still the new reference.
        //  The v6 concern was about MICRO pivots advancing the anchor. That is now
        //  handled by the minDist gate above.)
        lastPhBar   := curBar
        lastPhPrice := curPrice
        lastPhCvd   := curCvd

// ── Bullish signals (CVD pivot lows) — mirror of above with isExh/isAbs inverted ──
// [same structure, curPrice = low[i_lbR], uses lastPlBar/lastPlPrice/lastPlCvd]
```

### 7c. Signal Invalidation

```pine
// Run every bar — invalidates stale or price-broken signals

if cvdState.valid and not na(cvdState.lastSigBar)
    bool staleness = (bar_index - cvdState.lastSigBar) > i_stalenessBars

    bool priceBreak = false
    if cvdState.lastSig == "BearAbs" or cvdState.lastSig == "BearExh"
        // Invalidate if price CLOSES ABOVE the signal's curPrice
        priceBreak := close > cvdState.lastSigPrice
    else if cvdState.lastSig == "BullAbs" or cvdState.lastSig == "BullExh"
        // Invalidate if price CLOSES BELOW the signal's curPrice
        priceBreak := close < cvdState.lastSigPrice

    if staleness or priceBreak
        cvdState.valid := false
```

---

## 12. Module 8 — SPC Setup Detector

The Sweep → Profile Level → Confirmation state machine. SPC = Sweep, Profile, Confirm.

```pine
var SpcSetupState spc = SpcSetupState.new(false, false, na, "—", na, na, na, false)

// ── All key levels to check for sweep ────────────────────────────────────────
// Build sweep level array once per bar
float[] sweepLevels     = array.new_float()
string[] sweepLevelNames = array.new_string()

if not na(nyState.prevVah) => array.push(sweepLevels, nyState.prevVah), array.push(sweepLevelNames, "PD VAH")
if not na(nyState.prevVal) => array.push(sweepLevels, nyState.prevVal), array.push(sweepLevelNames, "PD VAL")
if not na(nyState.prevPoc) => array.push(sweepLevels, nyState.prevPoc), array.push(sweepLevelNames, "PD POC")
if not na(onState.prevVah) => array.push(sweepLevels, onState.prevVah), array.push(sweepLevelNames, "PON VAH")
if not na(onState.prevVal) => array.push(sweepLevels, onState.prevVal), array.push(sweepLevelNames, "PON VAL")
if not na(wkState.prevVah) => array.push(sweepLevels, wkState.prevVah), array.push(sweepLevelNames, "WK VAH")
if not na(wkState.prevVal) => array.push(sweepLevels, wkState.prevVal), array.push(sweepLevelNames, "WK VAL")
if not na(wkState.prevPoc) => array.push(sweepLevels, wkState.prevPoc), array.push(sweepLevelNames, "WK POC")
if ib.set and not na(ib.ibHigh) => array.push(sweepLevels, ib.ibHigh), array.push(sweepLevelNames, "IB High")
if ib.set and not na(ib.ibLow)  => array.push(sweepLevels, ib.ibLow),  array.push(sweepLevelNames, "IB Low")

float rowSz = not na(nyState.prevHigh) and nyState.rowsInput > 0 ? (nyState.prevHigh - nyState.prevLow) / nyState.rowsInput : syminfo.mintick * 5

// ── Reset pending states on session open ─────────────────────────────────────
if isNewNY
    spc.pendingLong  := false
    spc.pendingShort := false
    spc.sweptLevel   := na

// ── Sweep detection ───────────────────────────────────────────────────────────
if not spc.pendingLong and not spc.pendingShort
    for i = 0 to array.size(sweepLevels) - 1
        float lvl  = array.get(sweepLevels, i)
        string nm  = array.get(sweepLevelNames, i)

        bool wickAbove = high > lvl and close < lvl   // wick above, close below
        bool wickBelow = low  < lvl and close > lvl   // wick below, close above

        // Check if swept level is near an LVN (higher probability rejection)
        bool atLvn = false
        for k = 0 to math.min(array.size(nyState.lvnPrices) - 1, 10)
            if math.abs(lvl - array.get(nyState.lvnPrices, k)) <= rowSz * i_spcProxRows
                atLvn := true
                break

        if wickAbove
            // Price spiked above the level and came back — potential SHORT setup
            spc.pendingShort    := true
            spc.sweptLevel      := lvl
            spc.sweptLevelName  := nm
            spc.sweepBar        := bar_index
            spc.sweepHigh       := high
            spc.sweepLow        := close   // close is already below level
            spc.atLvn           := atLvn
            break

        else if wickBelow
            spc.pendingLong     := true
            spc.sweptLevel      := lvl
            spc.sweptLevelName  := nm
            spc.sweepBar        := bar_index
            spc.sweepHigh       := close
            spc.sweepLow        := low
            spc.atLvn           := atLvn
            break

// ── Confirmation check (close back through swept level) ───────────────────────
bool spcLongTrigger  = false
bool spcShortTrigger = false

if spc.pendingLong and not na(spc.sweptLevel) and bar_index > spc.sweepBar
    if close > spc.sweptLevel
        spcLongTrigger   := true
        spc.pendingLong  := false
    else if (bar_index - spc.sweepBar) > 5
        // Stale — sweep not confirmed in 5 bars = invalidate
        spc.pendingLong  := false

if spc.pendingShort and not na(spc.sweptLevel) and bar_index > spc.sweepBar
    if close < spc.sweptLevel
        spcShortTrigger  := true
        spc.pendingShort := false
    else if (bar_index - spc.sweepBar) > 5
        spc.pendingShort := false

// ── SL calculation (ATR or fixed ticks) ──────────────────────────────────────
float atr = ta.atr(i_spcAtrLen)
float slDistLong  = i_spcAtrSl ? atr * i_spcAtrMult : syminfo.mintick * i_spcSlTicks
float slDistShort = i_spcAtrSl ? atr * i_spcAtrMult : syminfo.mintick * i_spcSlTicks

float spcLongSl  = spcLongTrigger  ? spc.sweepLow  - slDistLong  : na
float spcShortSl = spcShortTrigger ? spc.sweepHigh + slDistShort : na

// ── Draw SPC signal triangles ──────────────────────────────────────────────
if i_showSpc
    if spcLongTrigger
        label.new(bar_index, low, "▲ SPC\n" + spc.sweptLevelName + (spc.atLvn ? " LVN" : ""),
             color=color.new(color.lime, 60), textcolor=color.white,
             style=label.style_label_up, size=size.small)
        line.new(bar_index, spcLongSl, bar_index + 3, spcLongSl,
             color=color.red, style=line.style_dotted, width=1)
        alert("SPC Long | Swept " + spc.sweptLevelName + (spc.atLvn ? " | At LVN" : ""), alert.freq_once_per_bar_close)

    if spcShortTrigger
        label.new(bar_index, high, "▼ SPC\n" + spc.sweptLevelName + (spc.atLvn ? " LVN" : ""),
             color=color.new(color.red, 60), textcolor=color.white,
             style=label.style_label_down, size=size.small)
        line.new(bar_index, spcShortSl, bar_index + 3, spcShortSl,
             color=color.red, style=line.style_dotted, width=1)
        alert("SPC Short | Swept " + spc.sweptLevelName + (spc.atLvn ? " | At LVN" : ""), alert.freq_once_per_bar_close)
```

---

## 13. Module 9 — Composite Scorer

### 9a. Point Allocation Table

| Signal / Condition                         | Bull Points                | Bear Points   | Notes                               |
| ------------------------------------------ | -------------------------- | ------------- | ----------------------------------- |
| `closeBias == "Bullish"`                   | +1                         | —             | Step 1                              |
| `closeBias == "Bearish"`                   | —                          | +1            | Step 1                              |
| `openBias == "Bullish"`                    | +1                         | —             | Step 2                              |
| `openBias == "Bearish"`                    | —                          | +1            | Step 2                              |
| `weeklyBias == "Bullish"`                  | +2                         | —             | Weekly override                     |
| `weeklyBias == "Bearish"`                  | —                          | +2            | Weekly override                     |
| `macroBias == "Bullish"`                   | +2                         | —             | HTF EMA                             |
| `macroBias == "Bearish"`                   | —                          | +2            | HTF EMA                             |
| `shapeBias == "Bullish"` (P-profile)       | +1                         | —             | Distribution shape                  |
| `shapeBias == "Bearish"` (b-profile)       | —                          | +1            | Distribution shape                  |
| `vaAcceptAbove` (close above VAH ≥ N bars) | +2                         | —             | Acceptance                          |
| `vaAcceptBelow` (close below VAL ≥ N bars) | —                          | +2            | Acceptance                          |
| `vaRejectAbove` (failed breakout)          | —                          | +2            | Rejection                           |
| `vaRejectBelow` (failed breakdown)         | +2                         | —             | Rejection                           |
| `nyState.pocNaked and prevPoc < close`     | —                          | +1            | Naked POC above = resistance/target |
| `nyState.pocNaked and prevPoc > close`     | +1                         | —             | Naked POC below = support/target    |
| CVD BullAbs valid                          | `+(tier + 1)`              | —             | Absorption = conviction             |
| CVD BearAbs valid                          | —                          | `+(tier + 1)` | Absorption = conviction             |
| CVD BullExh valid                          | `+tier`                    | —             | Exhaustion = weaker                 |
| CVD BearExh valid                          | —                          | `+tier`       | Exhaustion = weaker                 |
| `ibAlignsBull` (IB extension up)           | +2                         | —             | Strong directional                  |
| `ibAlignsBear` (IB extension down)         | —                          | +2            | Strong directional                  |
| `ibNarrow and ibAlignsBull`                | +1                         | —             | Narrow IB breakout                  |
| `ibNarrow and ibAlignsBear`                | —                          | +1            | Narrow IB breakout                  |
| `gapIntoVA`                                | Mean reversion bias to POC | —             | Special: adds +2 toward POC         |
| `gapOutOfVA and gapUp`                     | +1                         | —             | Gap continuation                    |
| `gapOutOfVA and gapDown`                   | —                          | +1            | Gap continuation                    |
| `nyState.pocMigratingUp`                   | +1                         | —             | POC migration = trending            |
| `nyState.pocMigratingDown`                 | —                          | +1            | POC migration = trending            |
| `spcLongTrigger`                           | +4                         | —             | Strongest single signal             |
| `spcShortTrigger`                          | —                          | +4            | Strongest single signal             |

### 9b. Veto Conditions

```pine
// Vetoes suppress HIGH CONFIDENCE signals only. MEDIUM/LOW still shows.
// This prevents mechanical trades that fight strong macro context.

bool vetoBull = false
bool vetoBear = false
string vetoReason = ""

if i_weeklyVeto
    // Weekly veto: if weekly bias is strongly bearish, no LONG signals
    if weeklyBias == "Bearish" and (not na(wkState.prevVah)) and close < wkState.prevVal
        vetoBull := true
        vetoReason := "WK Bias Bearish + Price Below WK VAL"

    if weeklyBias == "Bullish" and (not na(wkState.prevVal)) and close > wkState.prevVah
        vetoBear := true
        vetoReason := "WK Bias Bullish + Price Above WK VAH"

// Macro veto: if HTF EMA strongly opposed
if macroBias == "Bearish" and cvdFlow == "Bearish" and compositeBias == "Bearish"
    vetoBull := true
    vetoReason := vetoReason == "" ? "Triple Bearish Alignment — No Longs" : vetoReason

if macroBias == "Bullish" and cvdFlow == "Bullish" and compositeBias == "Bullish"
    vetoBear := true
    vetoReason := vetoReason == "" ? "Triple Bullish Alignment — No Shorts" : vetoReason
```

### 9c. Composite Score Calculation

```pine
var CompositeScore score = CompositeScore.new(0, 0, "WAIT", "LOW", na, na, na, na, 0.0, false, "")

// Reset points each bar
score.bullPts := 0
score.bearPts := 0

// Apply all votes (from table above)
if closeBias  == "Bullish" then score.bullPts += 1 else if closeBias  == "Bearish" then score.bearPts += 1
if openBias   == "Bullish" then score.bullPts += 1 else if openBias   == "Bearish" then score.bearPts += 1
if weeklyBias == "Bullish" then score.bullPts += 2 else if weeklyBias == "Bearish" then score.bearPts += 2
if macroBias  == "Bullish" then score.bullPts += 2 else if macroBias  == "Bearish" then score.bearPts += 2
if shapeBias  == "Bullish" then score.bullPts += 1 else if shapeBias  == "Bearish" then score.bearPts += 1
if vaAcceptAbove  then score.bullPts += 2
if vaAcceptBelow  then score.bearPts += 2
if vaRejectAbove  then score.bearPts += 2
if vaRejectBelow  then score.bullPts += 2
if nyState.pocNaked and not na(nyState.prevPoc)
    if nyState.prevPoc < close then score.bearPts += 1 else score.bullPts += 1
if cvdState.valid
    if   cvdState.lastSig == "BullAbs" then score.bullPts += (cvdState.lastTier + 1)
    elif cvdState.lastSig == "BearAbs" then score.bearPts += (cvdState.lastTier + 1)
    elif cvdState.lastSig == "BullExh" then score.bullPts += cvdState.lastTier
    elif cvdState.lastSig == "BearExh" then score.bearPts += cvdState.lastTier
if ibAlignsBull then score.bullPts += 2
if ibAlignsBear then score.bearPts += 2
if ibNarrow and ibAlignsBull then score.bullPts += 1
if ibNarrow and ibAlignsBear then score.bearPts += 1
if gapOutOfVA and gapUp   then score.bullPts += 1
if gapOutOfVA and gapDown then score.bearPts += 1
if nyState.pocMigratingUp   then score.bullPts += 1
if nyState.pocMigratingDown then score.bearPts += 1
if spcLongTrigger  then score.bullPts += 4
if spcShortTrigger then score.bearPts += 4

// Direction (must win by > 2 points)
score.direction := score.bullPts > score.bearPts + 2 ? "LONG"  :
                   score.bearPts > score.bullPts + 2 ? "SHORT" : "WAIT"

// Confidence
int maxPts = math.max(score.bullPts, score.bearPts)
score.confidence := maxPts >= 10 ? "HIGH" : maxPts >= 6 ? "MEDIUM" : "LOW"

// Apply vetoes (suppress HIGH only)
if score.confidence == "HIGH"
    if score.direction == "LONG"  and vetoBull then score.direction := "WAIT", score.vetoReason := vetoReason
    if score.direction == "SHORT" and vetoBear then score.direction := "WAIT", score.vetoReason := vetoReason

// R:R Calculation
if score.direction == "LONG" and not na(nyState.prevVal)
    score.entryPrice := close
    score.slPrice    := not na(spcLongSl) ? spcLongSl : nyState.prevVal - (atr * 0.25)
    score.t1Price    := nyState.prevPoc
    score.t2Price    := nyState.prevVah
    float risk = math.abs(score.entryPrice - score.slPrice)
    float reward = math.abs(score.t1Price - score.entryPrice)
    score.rrRatio := risk > 0 ? reward / risk : 0.0
    score.rrValid := score.rrRatio >= i_minRR

else if score.direction == "SHORT" and not na(nyState.prevVah)
    score.entryPrice := close
    score.slPrice    := not na(spcShortSl) ? spcShortSl : nyState.prevVah + (atr * 0.25)
    score.t1Price    := nyState.prevPoc
    score.t2Price    := nyState.prevVal
    float risk = math.abs(score.slPrice - score.entryPrice)
    float reward = math.abs(score.entryPrice - score.t1Price)
    score.rrRatio := risk > 0 ? reward / risk : 0.0
    score.rrValid := score.rrRatio >= i_minRR
```

---

## 14. Module 10 — Session Statistics

```pine
// Session range history is maintained inside ProfileState.sessionRanges (float[10])
// These are computed at isEndNY / isEndON / isEndWK inside processSession.

// For the dashboard, compute "current range vs average" at any bar:
float currSessionRange = not na(nyState.prevHigh) ?
    (array.size(nyState.highs) > 0 ? array.max(nyState.highs) - array.min(nyState.lows) : na) : na

string rangeBias = na(currSessionRange) or na(nyState.avgSessionRange) or nyState.avgSessionRange == 0 ? "—" :
    currSessionRange > nyState.avgSessionRange * 1.25 ? "Wide (trending)" :
    currSessionRange < nyState.avgSessionRange * 0.75 ? "Narrow (breakout watch)" : "Normal"

float rangePct = not na(currSessionRange) and not na(nyState.avgSessionRange) and nyState.avgSessionRange > 0 ?
    currSessionRange / nyState.avgSessionRange * 100 : na

// IB range context (ibNarrowFactor computed in Module 4)
string ibContext = na(ibNarrowFactor) ? "—" :
    ibNarrowFactor < 0.40 ? "Very Narrow IB — Breakout Likely" :
    ibNarrowFactor < 0.65 ? "Narrow IB" :
    ibNarrowFactor > 1.25 ? "Wide IB — Range Day" : "Normal IB"
```

---

## 15. Module 11 — Drawing Engine

### 11a. Profile Polylines (helper functions)

```pine
// These helpers are called by processSession and the developing profile block.

f_drawFullPoly(int startIdx, float dLow, float dHigh, int rows, float[] rowVols, float maxVol, float rowSize, float maxWidBars, color col) =>
    chart.point[] pts = array.new<chart.point>()
    array.push(pts, chart.point.from_index(startIdx, dLow))
    for j = 0 to rows - 1
        float price  = dLow + (j * rowSize)
        float vol    = array.get(rowVols, j)
        int   offset = math.round((vol / maxVol) * maxWidBars)
        array.push(pts, chart.point.from_index(startIdx + offset, price))
    array.push(pts, chart.point.from_index(startIdx, dHigh))
    array.push(pts, chart.point.from_index(startIdx, dLow))
    polyline.new(pts, line_color=col, closed=true)

f_drawVaPoly(int startIdx, float dLow, int upperRow, int lowerRow, float vah, float val, int rows, float[] rowVols, float maxVol, float rowSize, float maxWidBars, color col) =>
    // Guard: VA must span at least 2 rows
    if upperRow < lowerRow + 1
        => na

    chart.point[] pts = array.new<chart.point>()
    array.push(pts, chart.point.from_index(startIdx, val))
    for j = lowerRow to upperRow
        float price  = dLow + (j * rowSize)
        float vol    = array.get(rowVols, j)
        int   offset = math.round((vol / maxVol) * maxWidBars)
        array.push(pts, chart.point.from_index(startIdx + offset, price))
    array.push(pts, chart.point.from_index(startIdx, vah))
    array.push(pts, chart.point.from_index(startIdx, val))
    polyline.new(pts, line_color=col, closed=true, fill_color=color.new(col, 85))
```

### 11b. Extended Level Lines

All completed session POC/VAH/VAL lines use `extend=extend.right`. They are created inside `processSession` at `isEndSes`. They are **not** deleted every bar; they persist until the next `isEndSes` for that session type deletes and redraws them.

### 11c. Naked POC Lines

```pine
// Style difference between naked (unvisited) and visited POC:
// Naked  = dotted, width=2, color=i_nakedPocCol (fuchsia)
// Visited = solid, width=1, color=state.pocColor (same as normal POC)

// The line is created inside processSession at isEndSes.
// It is updated to "visited" style (width=1, style=solid) when the naked flag clears.
// A separate nakedPocLine object holds the fuchsia dotted line;
// it is deleted when the POC is visited and the normal pocLine styling takes over.
```

### 11d. LVN / HVN Lines

```pine
f_drawLvnLines(ProfileState state, float dLow, float dHigh, int startIdx, int endIdx) =>
    // Delete old LVN/HVN lines on state
    for i = 0 to array.size(state.lvnLines) - 1
        line.delete(array.get(state.lvnLines, i))
    array.clear(state.lvnLines)
    for i = 0 to array.size(state.hvnLines) - 1
        line.delete(array.get(state.hvnLines, i))
    array.clear(state.hvnLines)

    // Draw new — capped at i_maxLvnLines
    int lvnCount = 0
    for i = 0 to array.size(state.lvnPrices) - 1
        if lvnCount >= i_maxLvnLines
            break
        float p = array.get(state.lvnPrices, i)
        // Only draw LVNs within profile price range
        if p >= dLow and p <= dHigh
            line l = line.new(startIdx, p, endIdx, p, color=i_lvnColor, style=line.style_dotted, width=1, extend=extend.right)
            array.push(state.lvnLines, l)
            lvnCount += 1

    // HVN lines (cap at same limit)
    int hvnCount = 0
    for i = 0 to array.size(state.hvnPrices) - 1
        if hvnCount >= i_maxLvnLines
            break
        float p = array.get(state.hvnPrices, i)
        if p >= dLow and p <= dHigh
            line l = line.new(startIdx, p, endIdx, p, color=i_hvnColor, style=line.style_dotted, width=1, extend=extend.right)
            array.push(state.hvnLines, l)
            hvnCount += 1
```

### 11e. Session Boxes

Unchanged from prior script. `SessionBox` UDT + `updateBox()` method. Four instances: NY, ON, LN, AS.

### 11f. IB Box & Extension Marks

Handled in Module 4 inline (box created on `isNewNY`, updated each `inIB` bar, frozen at `ibJustClosed`).

### 11g. SPC Entry Triangles

Drawn inline in Module 8 at `spcLongTrigger` / `spcShortTrigger`. Includes SL dotted line.

### 11h. CVD Signal Diamonds (overlay)

When a CVD signal fires (`isExh` or `isAbs`) in Module 7, draw a **small shape on the price pane** (not the CVD pane) to allow visual reference without needing the companion pane:

```pine
// At signal fire, add a price-pane marker at the pivot price
// Bear signals: label at high[i_lbR], style=label.style_triangledown
// Bull signals: label at low[i_lbR], style=label.style_triangleup
// This is IN ADDITION to the CVD-pane line/label in the companion script.
// Text = "" (empty), just a colored shape marker.
// Size = size.tiny to avoid clutter.
```

---

## 16. Module 12 — Dashboard Renderer

```pine
// Position map
f_dashPos() =>
    switch i_dashPos
        "Top Left"     => position.top_left
        "Bottom Right" => position.bottom_right
        "Bottom Left"  => position.bottom_left
        => position.top_right

if i_showDash and barstate.islast
    var table dash = table.new(f_dashPos(), 2, 28,
        bgcolor      = color.new(#08080f, 8),
        border_width = 1,
        border_color = color.new(color.gray, 55),
        frame_width  = 1,
        frame_color  = color.new(color.gray, 35))

    // Utility: set a row quickly
    // f_row(dash, row, label, value, labelCol, valueCol)

    // ── ROW 0: Header ────────────────────────────────────────────────────────
    table.cell(dash, 0, 0, "⚡  VP·CVD  DECISION", text_color=color.white, text_size=size.normal, text_halign=text.align_center, bgcolor=color.new(#1a1a3e, 0))
    table.merge_cells(dash, 0, 0, 1, 0)

    // ── SECTION: SESSION ─────────────────────────────────────────────────────
    // Row 1: Section header
    table.cell(dash, 0, 1, "── SESSION ──", text_color=color.new(color.gray, 20), text_size=size.small, text_halign=text.align_center, bgcolor=color.new(#1a1a3e, 0))
    table.merge_cells(dash, 0, 1, 1, 1)

    // Row 2: Active session name + range context
    table.cell(dash, 0, 2, "Range",      text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 2, rangeBias,    text_color=color.white, text_size=size.small)

    // Row 3: Current range vs average
    string rangeStr = not na(rangePct) ? str.tostring(math.round(rangePct)) + "% of avg" : "—"
    table.cell(dash, 0, 3, "vs Avg",     text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 3, rangeStr,     text_color=color.white, text_size=size.small)

    // Row 4: IB context
    table.cell(dash, 0, 4, "IB",         text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 4, ibContext,    text_color=i_ibExtColor, text_size=size.small)

    // ── SECTION: BIAS ────────────────────────────────────────────────────────
    table.cell(dash, 0, 5, "── BIAS ──", text_color=color.new(color.gray, 20), text_size=size.small, text_halign=text.align_center, bgcolor=color.new(#1a1a3e, 0))
    table.merge_cells(dash, 0, 5, 1, 5)

    // Row 6–10: Bias rows with colored indicators
    // Macro / Weekly / Close / Open / Composite
    // f_biasColor(str) => str == "Bullish" ? color.lime : str == "Bearish" ? color.red : color.gray

    // Row 6
    table.cell(dash, 0, 6, "Macro ("  + i_htfTf + ")", text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 6, (macroBias == "Bullish" ? "🟢 " : macroBias == "Bearish" ? "🔴 " : "🟡 ") + macroBias, text_color=f_biasColor(macroBias), text_size=size.small)

    // Row 7: Weekly
    table.cell(dash, 0, 7, "Weekly",   text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 7, (weeklyBias == "Bullish" ? "🟢 " : weeklyBias == "Bearish" ? "🔴 " : "🟡 ") + weeklyBias, text_color=f_biasColor(weeklyBias), text_size=size.small)

    // Row 8: Prev Close vs VA
    table.cell(dash, 0, 8, "PD Close",  text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 8, (closeBias == "Bullish" ? "🟢 " : closeBias == "Bearish" ? "🔴 " : "🟡 ") + closeBias, text_color=f_biasColor(closeBias), text_size=size.small)

    // Row 9: Open vs VA
    table.cell(dash, 0, 9, "Open",      text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 9, (openBias == "Bullish" ? "🟢 " : openBias == "Bearish" ? "🔴 " : "🟡 ") + openBias, text_color=f_biasColor(openBias), text_size=size.small)

    // Row 10: Profile shape
    string shapeStr = nyState.prevProfileShape == 1 ? "P-profile (Bullish)" :
                      nyState.prevProfileShape == -1 ? "b-profile (Bearish)" :
                      nyState.prevProfileShape == 2  ? "B-profile (Contested)" : "D-profile (Neutral)"
    table.cell(dash, 0, 10, "Shape",    text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 10, shapeStr,   text_color=f_biasColor(shapeBias), text_size=size.small)

    // Row 11: Composite bias (bold / highlighted)
    table.cell(dash, 0, 11, "COMPOSITE", text_color=color.white, text_size=size.small)
    string compStr = compositeBias == "Bullish" ? "🟢 BULLISH" : compositeBias == "Bearish" ? "🔴 BEARISH" : "🟡 NEUTRAL"
    table.cell(dash, 1, 11, compStr,    text_color=f_biasColor(compositeBias), text_size=size.normal, bgcolor=color.new(#1a1a3e, 0))

    // ── SECTION: POSITION ────────────────────────────────────────────────────
    table.cell(dash, 0, 12, "── POSITION ──", text_color=color.new(color.gray, 20), text_size=size.small, text_halign=text.align_center, bgcolor=color.new(#1a1a3e, 0))
    table.merge_cells(dash, 0, 12, 1, 12)

    // Row 13: Price vs PD VA
    table.cell(dash, 0, 13, "vs PD VA",  text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 13, pricePosition, text_color=color.white, text_size=size.small)

    // Row 14: VA acceptance / rejection
    string vaStateStr = vaAcceptAbove ? "✓ Accepted Above VAH" : vaAcceptBelow ? "✓ Accepted Below VAL" :
                        vaRejectAbove ? "✗ Rejected at VAH" : vaRejectBelow ? "✗ Rejected at VAL" : "—"
    color vaStateCol = (vaAcceptAbove or vaAcceptBelow) ? color.lime : (vaRejectAbove or vaRejectBelow) ? color.orange : color.gray
    table.cell(dash, 0, 14, "VA State",  text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 14, vaStateStr,  text_color=vaStateCol,  text_size=size.small)

    // Row 15: Naked POC
    string nakStr = nyState.pocNaked and not na(nyState.prevPoc) ?
                    str.tostring(nyState.prevPoc, format.mintick) + " (" + str.tostring(bar_index - nyState.pocNakedSince) + " bars)" : "None"
    color nakCol  = nyState.pocNaked ? i_nakedPocCol : color.gray
    table.cell(dash, 0, 15, "Naked POC", text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 15, nakStr,      text_color=nakCol,      text_size=size.small)

    // Row 16: POC Migration
    string pocMigStr = nyState.pocMigratingUp ? "▲ Trending Up" : nyState.pocMigratingDown ? "▼ Trending Down" : "◆ Stationary"
    color pocMigCol  = nyState.pocMigratingUp ? color.lime : nyState.pocMigratingDown ? color.red : color.gray
    table.cell(dash, 0, 16, "POC Drift",  text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 16, pocMigStr,    text_color=pocMigCol,   text_size=size.small)

    // ── SECTION: CVD ─────────────────────────────────────────────────────────
    table.cell(dash, 0, 17, "── CVD ──", text_color=color.new(color.gray, 20), text_size=size.small, text_halign=text.align_center, bgcolor=color.new(#1a1a3e, 0))
    table.merge_cells(dash, 0, 17, 1, 17)

    // Row 18: Flow + vs MA
    string cvdFlowStr = cvdFlow == "Bullish" ? "▲ Bullish" : "▼ Bearish"
    table.cell(dash, 0, 18, "CVD Flow",  text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 18, cvdFlowStr,  text_color=cvdFlow == "Bullish" ? color.teal : color.red, text_size=size.small)

    string vsMaStr = (cvdVsMaPct >= 0 ? "+" : "") + str.tostring(math.round(cvdVsMaPct, 1)) + "%"
    table.cell(dash, 0, 19, "vs MA",     text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 19, vsMaStr,     text_color=cvdVsMaPct >= 0 ? color.teal : color.red, text_size=size.small)

    // Row 20: Last signal + tier
    table.cell(dash, 0, 20, "Signal",    text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 20, cvdState.sigText, text_color=cvdState.sigColor, text_size=size.small)

    string sigTierStr = cvdState.lastTier == 3 ? "★★★" : cvdState.lastTier == 2 ? "★★☆" : "★☆☆"
    table.cell(dash, 0, 21, "Tier",      text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 21, na(cvdState.lastTier) ? "—" : sigTierStr, text_color=cvdState.sigColor, text_size=size.small)

    // Row 22: Age + validity
    string ageStr   = na(cvdState.lastSigBar) ? "—" : str.tostring(bar_index - cvdState.lastSigBar) + " bars ago"
    string validStr = cvdState.valid ? "✓ Active" : "✗ Invalidated"
    color  validCol = cvdState.valid ? color.lime : color.red
    table.cell(dash, 0, 22, "Age / Valid", text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 22, ageStr + " " + validStr, text_color=validCol, text_size=size.small)

    // Row 23: At-level indicator
    string atLvlStr = cvdState.atLevel ? "✓ At Profile Level" : "✗ No Level Proximity"
    table.cell(dash, 0, 23, "Level",     text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 23, atLvlStr,    text_color=cvdState.atLevel ? color.lime : color.orange, text_size=size.small)

    // ── SECTION: SPC ─────────────────────────────────────────────────────────
    table.cell(dash, 0, 24, "── SPC ──", text_color=color.new(color.gray, 20), text_size=size.small, text_halign=text.align_center, bgcolor=color.new(#1a1a3e, 0))
    table.merge_cells(dash, 0, 24, 1, 24)

    string spcStr  = spc.pendingLong  ? "↑ Long Pending — " + spc.sweptLevelName :
                     spc.pendingShort ? "↓ Short Pending — " + spc.sweptLevelName : "No Active Setup"
    color spcCol   = spc.pendingLong ? color.lime : spc.pendingShort ? color.red : color.gray
    table.cell(dash, 0, 25, "Setup",     text_color=color.gray, text_size=size.small)
    table.cell(dash, 1, 25, spcStr,      text_color=spcCol,      text_size=size.small)

    // ── SECTION: TRADE DECISION ───────────────────────────────────────────────
    table.cell(dash, 0, 26, "── TRADE DECISION ──", text_color=color.new(color.gray, 20), text_size=size.small, text_halign=text.align_center, bgcolor=color.new(#1a1a3e, 0))
    table.merge_cells(dash, 0, 26, 1, 26)

    // Row 27: Direction + confidence + R:R
    string dirStr   = score.direction == "LONG"  ? "✅ LONG" :
                      score.direction == "SHORT" ? "🔴 SHORT" : "⏸ WAIT"
    color  dirCol   = score.direction == "LONG"  ? color.lime :
                      score.direction == "SHORT" ? color.red : color.gray
    string confStr2 = score.confidence + " (" + str.tostring(math.max(score.bullPts, score.bearPts)) + " pts)"

    // If vetoed, show reason instead of direction
    string finalDirStr = score.vetoReason != "" ? "⛔ " + score.vetoReason : dirStr + " — " + confStr2
    table.cell(dash, 0, 27, "Decision",  text_color=color.white, text_size=size.small)
    table.cell(dash, 1, 27, finalDirStr, text_color=score.vetoReason != "" ? color.orange : dirCol, text_size=size.small)

    // Optional: Entry zone / SL / T1 / T2 / R:R rows (shown if i_showRR and i_showEntZone)
    // These would extend the table with rows 28-32 for full entry plan display
```

---

## 17. Module 13 — Alert Conditions

```pine
// All alert conditions use alert.freq_once_per_bar_close to prevent duplicates
// Inline alerts are already placed in Modules 7 and 8.
// The following alertcondition() declarations allow TradingView alert dialog setup:

alertcondition(score.direction == "LONG"  and score.confidence == "HIGH" and score.rrValid,   title="HIGH CONF LONG",   message="VP·CVD: High Confidence LONG | {{ticker}} {{close}}")
alertcondition(score.direction == "SHORT" and score.confidence == "HIGH" and score.rrValid,   title="HIGH CONF SHORT",  message="VP·CVD: High Confidence SHORT | {{ticker}} {{close}}")
alertcondition(spcLongTrigger,                                                                  title="SPC LONG",         message="VP·CVD: SPC Long Trigger | Swept {{close}}")
alertcondition(spcShortTrigger,                                                                 title="SPC SHORT",        message="VP·CVD: SPC Short Trigger | Swept {{close}}")
alertcondition(nyState.pocNaked and math.abs(close - nyState.prevPoc) <= (nyState.prevHigh - nyState.prevLow) / nyState.rowsInput * 3, title="NAKED POC APPROACH", message="VP·CVD: Price Approaching Naked POC {{close}}")
alertcondition(cvdState.valid and cvdState.lastSig == "BullAbs" and cvdState.atLevel,          title="CVD BULL ABS AT LEVEL", message="VP·CVD: Bull Absorption at Profile Level")
alertcondition(cvdState.valid and cvdState.lastSig == "BearAbs" and cvdState.atLevel,          title="CVD BEAR ABS AT LEVEL", message="VP·CVD: Bear Absorption at Profile Level")
alertcondition(ibAlignsBull,                                                                    title="IB EXTENSION UP",   message="VP·CVD: IB Extended Up — Directional Day Possible")
alertcondition(ibAlignsBear,                                                                    title="IB EXTENSION DOWN",  message="VP·CVD: IB Extended Down — Directional Day Possible")
alertcondition(ta.change(compositeBias) != 0,                                                  title="BIAS FLIP",         message="VP·CVD: Composite Bias changed to " + compositeBias)
```

---

## 18. Known Pine v5 Constraints & Workarounds

| Constraint                                    | Workaround                                                                                                                                                   |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| No cross-indicator real-time data             | Recalculate CVD inline in overlay; keep companion pane for visuals only                                                                                      |
| `polyline` immutable — cannot update          | Store as `var`, delete and recreate each bar for developing profile                                                                                          |
| `request.security()` adds 1-bar delay         | Only used for HTF bias (daily/weekly EMA) where 1-bar delay is acceptable                                                                                    |
| `max_polylines_count = 100`                   | Cap completed profiles at 3 per session type; developing + VA = 2 active at a time                                                                           |
| Arrays have no native sort                    | Use `f_sortAsc()` insertion sort for LVN/HVN price ordering                                                                                                  |
| UDT fields cannot be arrays of UDTs           | Use parallel arrays indexed inside each ProfileState                                                                                                         |
| Functions cannot return UDTs                  | Use method pattern (mutate UDT in place) or tuple return                                                                                                     |
| `barstate.isconfirmed` vs `barstate.islast`   | All session transitions use `barstate.isconfirmed` (or check `not barstate.isrealtime`) to avoid false resets on live bars. Dashboard uses `barstate.islast` |
| `ta.pivothigh()` returns value `lbR` bars ago | All pivot references offset by `[i_lbR]` — never use `bar_index` directly at a pivot, always `bar_index - i_lbR`                                             |
| Session detection on 1D chart                 | Wrap all session-string detection in a timeframe guard: if timeframe is >= "D", skip intraday session logic and fall back to Daily mode                      |

---

## 19. Performance Budget

| Component                  | Estimated Iterations per Bar     | Notes                          |
| -------------------------- | -------------------------------- | ------------------------------ |
| `calcProfile()` developing | `rows × bars_in_session / 2` avg | Use 500 rows default, not 1000 |
| LVN/HVN classification     | `rows` (500)                     | Once per `isEndSes` only       |
| Profile shape              | `rows / 2` (250)                 | Once per `isEndSes` only       |
| Poor High/Low              | `bars_in_session`                | Once per `isEndSes` only       |
| CVD calculation            | O(1) per bar                     | Trivial                        |
| Pivot detection            | O(1) per bar                     | Built-in function              |
| SPC level scan             | `~10 levels`                     | O(1) per bar                   |
| LVN proximity scan         | `i_maxLvnLines` max              | O(1) per bar                   |
| Insertion sort             | `array.size` per sort            | Called once per `isEndSes`     |

**Loop guards:** Every `for i = 0 to array.size(arr) - 1` loop in `calcProfile()` must be wrapped:
```pine
for i = 0 to math.min(array.size(highs) - 1, 5000)
```
The `5000` ceiling prevents runaway loops on weekly Overnight sessions with many accumulated bars.

**Polyline budget:**
- 1 developing full poly
- 1 developing VA poly
- 3 completed NY RTH (× 2 = 6)
- 1 completed weekly (× 2 = 2)
- 1 completed overnight (× 2 = 2)
- **Total = 13 polylines** (well within the 100 limit)

---

## 20. Known Bugs Fixed from Prior Scripts

### Bug 1 — `isEndSes` uses `bar_index` instead of `state.endIndex`

**Problem:** When `isEndSes` fires, `bar_index` is the first bar of the *next* session. `drawCompletedProfile()` used `bar_index` as `endIdx`, making profiles 1 bar too wide.

**Fix:** Store `state.endIndex := bar_index` on every `inSes` bar (last value = last bar of the session). Use `state.endIndex` as `endIdx` in all drawing calls. Never use `bar_index` at `isEndSes` for right edge.

### Bug 2 — VA Polyline with `lowerRow == upperRow`

**Problem:** When a session has very few bars (e.g., late session, partial data), the VA expansion may result in `upperRow == lowerRow`. The polyline loop produces a degenerate 4-point shape that may render incorrectly.

**Fix:** Guard in `f_drawVaPoly()`:
```pine
if upperRow < lowerRow + 1
    => na
```

### Bug 3 — `isNewSession` double-fires when `sessInput == "None"`

**Problem:** The switch-case fallthrough in `getSessionString()` returns `"0000-0000"`, which evaluates `not na(time(...))` on some timeframes/symbols as non-null, causing spurious `isNewSession` fires.

**Fix:** Explicitly gate all session detection on `not isNone`:
```pine
bool isNewSess = isNone ? false : ...
bool isEndSess = isNone ? false : ...
```

### Bug 4 — Session box accumulation (no expiry)

**Problem:** `SessionBox.updateBox()` creates a new box on every `isNewSes` but never deletes boxes from prior sessions. With `max_boxes_count = 500`, this silently caps without error.

**Fix:** At `isNewSes`, delete the existing box before creating a new one:
```pine
if not na(sb.b)
    box.delete(sb.b)
    sb.b := na
```

### Bug 5 — CVD `lastPhBar` anchor advances on non-signal pivots

**Problem (from Script B v5):** The pivot reference advances even when no signal fires, meaning the *next* valid divergence compares against an intermediate no-signal pivot instead of the prior signal.

**Fix in Module 7:** Anchor still advances (this is correct — we want the most recent pivot as reference, not the last signal pivot). The v6 fix (minDist gate) already addresses the micro-pivot noise problem. The anchor advance behavior is intentional.

### Bug 6 — Labels on `bar_index + labelOffset` misplace on historical scroll

**Problem:** Labels are placed at `bar_index + labelOffset` on every bar in the refresh loop, but `label.set_x()` calls on historical bars push labels into the future where they may be off-screen.

**Fix:** Only refresh label X position if `bar_index + labelOffset <= last_bar_index + 100` (i.e., we're not so far in the past that labels would be meaningless):
```pine
if not na(state.pocLabel) and barstate.islast
    label.set_x(state.pocLabel, bar_index + state.labelOffset)
```
Restrict label position updates to `barstate.islast` to avoid per-bar label repositioning cost.

---

## 21. Implementation Phases

### Phase 1 — Scaffold & Core Profile Engine
- [ ] Indicator header, all input groups, all UDT definitions
- [ ] Session time utilities (Module 1)
- [ ] `calcProfile()` single reusable function (Module 3a)
- [ ] `f_classifyNodes()` LVN/HVN (Module 3b)
- [ ] `f_sortAsc()` insertion sort
- [ ] `ProfileState` UDT with all new fields
- [ ] `processSession()` method (all 6 steps in Module 3e)
- [ ] Three ProfileState instances: `nyState`, `onState`, `wkState`
- [ ] Drawing helpers: `f_drawFullPoly()`, `f_drawVaPoly()`, `f_drawLvnLines()`
- [ ] Session boxes (unchanged from prior script)
- **Test:** Load on ES1! 5m. Verify profiles draw correctly, LVN lines appear, session transitions clean up properly.

### Phase 2 — Profile Enrichment
- [ ] Profile shape classification `f_profileShape()` (Module 3c)
- [ ] Poor High/Poor Low detection `f_detectPoorExtremes()` (Module 3d)
- [ ] Naked POC tracker (Module 3f — inside `processSession`)
- [ ] POC migration tracker (Module 3g)
- [ ] Session range statistics (Module 10)
- [ ] IB engine (Module 4)
- **Test:** Verify naked POC lines draw/clear correctly. Verify IB box appears and freezes after i_ibMins. Check IB extension labels fire on breakout.

### Phase 3 — CVD Engine & Signals
- [ ] Session-reset CVD + all-time CVD (Module 2)
- [ ] CVD MA, flow string, vs-MA pct
- [ ] CVD pivot detection (operates on session CVD)
- [ ] `f_nearKeyLevel()` and `f_proximityBoost()` helpers (Module 7b)
- [ ] All 4 signal patterns (Module 7b)
- [ ] `CvdSignalState` UDT and state updates
- [ ] Signal invalidation (Module 7c)
- [ ] CVD signal price-pane diamond shapes (Module 11h)
- **Test:** Verify Bear Abs fires at PD VAH, Bull Abs fires at PD VAL. Verify `i_onlyAtLevel` suppresses signals in empty space. Verify staleness invalidation clears the dashboard.

### Phase 4 — Bias & Opening Context
- [ ] 3-step checklist (Module 6a)
- [ ] VA acceptance/rejection (Module 6b)
- [ ] Macro HTF filter (Module 6c)
- [ ] Composite daily bias (Module 6d)
- [ ] Opening gap analysis (Module 5)
- **Test:** On a trending day, verify weekly bias and macro bias align and composite shows correctly. On a rotational day, verify neutral/conflict resolution.

### Phase 5 — SPC Detector
- [ ] `SpcSetupState` UDT
- [ ] Sweep detection across all key levels (Module 8)
- [ ] LVN proximity check for swept level
- [ ] Confirmation state machine (Module 8)
- [ ] ATR-based SL calculation
- [ ] SPC triangle drawing + SL line
- [ ] Alert conditions for SPC
- **Test:** Manually verify SPC fires on a clear sweep-and-return at PD VAL. Test stale-sweep invalidation (no confirmation within 5 bars).

### Phase 6 — Composite Scorer & Dashboard
- [ ] Full point allocation logic (Module 9a)
- [ ] Veto conditions (Module 9b)
- [ ] R:R calculation (Module 9c)
- [ ] `CompositeScore` UDT
- [ ] Full dashboard renderer (Module 12) — all 28 rows
- [ ] All `alertcondition()` declarations (Module 13)
- **Test:** Verify HIGH CONF LONG only fires when R:R ≥ i_minRR. Verify veto message appears when weekly bias opposes direction. Verify dashboard refreshes every bar on live chart.

### Phase 7 — Optimization & Polish
- [ ] Reduce all `rowsInput` defaults to 500
- [ ] Add `math.min(..., 5000)` loop guards on all array loops
- [ ] Audit all `var` array declarations — none should grow unboundedly
- [ ] Label management: restrict `label.set_x()` updates to `barstate.islast`
- [ ] Session box deletion on `isNewSes` (Bug 4 fix)
- [ ] Test on: ES1!, NQ1!, EURUSD across 1m, 5m, 15m, 1H
- [ ] Test on: Daily chart (verify timeframe guard suppresses intraday session logic)
- [ ] Performance profiling on 1m chart for execution time warning
- [ ] Final cleanup — remove all debug labels/plots

---

## 22. Companion Pane Script Spec

**File:** `cvd_pane_companion.pine`

```pine
//@version=5
indicator("CVD Pane — VP·CVD Companion", overlay=false, format=format.volume, max_lines_count=500, max_labels_count=500)

// ── INPUTS (must match main script exactly) ───────────────────────────────────
// Copy: i_lbL, i_lbR, i_minDist, i_maxDist, i_maLen, i_nyStr, i_tz
// Copy: i_sess (for session reset)
// Copy: CVD color inputs

// ── CVD ENGINE ────────────────────────────────────────────────────────────────
// Identical calculation to Module 2 (both session and all-time CVD)
// Session CVD = yellow line (primary)
// All-time CVD = gray line (secondary, optional toggle)

// ── PLOTS ─────────────────────────────────────────────────────────────────────
// p_cvd_sess = plot(cvdSess, "Session CVD", color=col_cvd, linewidth=2)
// p_cvd_ma   = plot(cvdMa,   "CVD MA",      color=col_cvd_ma, linewidth=1)
// fill(p_cvd_sess, p_cvd_ma, color= cvdSess > cvdMa ? color.new(color.teal, 70) : color.new(color.red, 70))
// hline(0, color=color.new(color.gray, 65), linestyle=hline.style_dotted)

// ── PIVOT DOTS ────────────────────────────────────────────────────────────────
// Same pivot detection as Module 2 — circles at pivot high/low bars
// plot(showPivDots and not na(cvdPH) ? cvdSess[i_lbR] : na, offset=-i_lbR, ...)
// plot(showPivDots and not na(cvdPL) ? cvdSess[i_lbR] : na, offset=-i_lbR, ...)

// ── CVD SIGNAL LINES (visual only — no state tracking) ───────────────────────
// Minimal version of Module 7 signal drawing:
// Lines from (lastPhBar, lastPhCvd) to (curBar, curCvd)
// Labels at curBar with signal name
// No dashboard. No at-level filter (draws all pivots for visual reference).

// ── NO DASHBOARD ─────────────────────────────────────────────────────────────
// All decision logic lives in the main overlay script.
```

**Purpose:** Keeps the CVD waveform on a separate pane for clean visual inspection of raw delta flow, without duplicating the complex state management. The companion pane is purely visual and contains no signal logic.

---

*End of plan. Begin with Phase 1. Each phase should be independently testable before proceeding to the next.*