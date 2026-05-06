# CVD Absorption / Exhaustion Indicator - Technical Overview

This document provides a technical explanation of the architecture, divergence logic, and logic improvements implemented in the CVD (Cumulative Volume Delta) Absorption/Exhaustion script.

## Core Purpose
The indicator identifies **Absorption** (hidden divergences where volume acts against price) and **Exhaustion** (regular divergences where volume fails to confirm price action). It uses the interaction between Price and CVD at CVD Pivot points to identify potential reversals and continuations.

## Identifying Divergences correctly

One of the common issues in script-based divergence detection is relying entirely on oscillator (CVD) pivots or entirely on price pivots. The behavior and utility of each pattern dictates which pivot source provides the most accurate and actionable signal.

### 1. Absorptions (Continuation)

Absorption occurs when aggressive market participants are hammering into a passive limit-order wall. The footprint of this is extreme CVD activity without a corresponding extreme in price. 

- **Bearish Absorption**: CVD makes a Higher High (HH), indicating strong aggressive buying, but Price makes a Lower High (LH) or fails to break the high. Buyers are being absorbed by a passive sell-limit wall.
- **Bullish Absorption**: CVD makes a Lower Low (LL), indicating strong aggressive selling, but Price makes a Higher Low (HL) or fails to break the low. Sellers are being absorbed by a passive buy-limit wall.

### 2. Exhaustions (Reversal Warning)

Exhaustion occurs when price continues to trend and makes new extremes, but the driving volume/order-flow is no longer supporting the move.

- **Bearish Exhaustion**: Price makes a Higher High (HH), but CVD makes a Lower High (LH) or fails to make a new high. The buyers are exhausted.
- **Bullish Exhaustion**: Price makes a Lower Low (LL), but CVD makes a Higher Low (HL) or fails to make a new low. The sellers are exhausted.

**Implementation Strategy**: Both Absorption and Exhaustion strictly utilize **CVD Pivots** (`ta.pivothigh(cvd)` and `ta.pivotlow(cvd)`). By anchoring to the CVD peaks/troughs, the indicator perfectly aligns its analysis with the exact moments of peak order-flow pressure, and evaluates the price response at those precise bars.

## Structural Improvements

### 1. Precise Pivot State Engine

The script stores the immediately preceding valid CVD pivot (`lastPhCvd`, `lastPhPrice`, `lastPhBar`) and evaluates divergence strictly against it. This prevents the "fan effect" (multiple lines drawing from the same starting point) and ensures that each divergence line strictly connects adjacent major structure points, maintaining clean chart visualizations. It enforces a configurable minimum distance (`minDist`) between pivots to filter out market noise.

### 2. Strength Normalization & Tiers

To ensure signals work robustly across different instruments and timeframes, the absolute divergence magnitudes are normalized against a rolling range (configurable via `maxDist`, defaulting to 60 bars):

- `prcRange = math.max(ta.highest(high, maxDist) - ta.lowest(low, maxDist), 0.0001)`
- `cvdRange = math.max(ta.highest(cvd, maxDist) - ta.lowest(cvd, maxDist), 1.0)`

The divergence score is formulated as the sum of these percentage differences:
`Score = (Price Difference / prcRange) + (CVD Difference / cvdRange)`

Based on the combined score, signals are classified into three tiers:

- **Tier 1 (★☆☆)**: Marginal or weak divergence. Score `< 0.25`
- **Tier 2 (★★☆)**: Clear divergence. Score `>= 0.25`
- **Tier 3 (★★★)**: Extremely strong divergence. Score `>= 0.50`

### 3. Rendering Enhancements

Because lines are drawn on the CVD oscillator panel, the Y-coordinates of the `line.new()` functions strictly use `lastPhCvd`/`lastPlCvd` and `curCvd`. The endpoints correctly hook onto the CVD values. A start-of-line marker (`●`) explicitly identifies the left anchor pivot.

Visuals are designed for maximum clarity and configurability:

- **Customizable Colors**: Users can independently define colors for the CVD line, CVD Moving Average, and all four signal types (Bull/Bear Exhaustion and Absorption) via the script settings.
- **Explicit Interpretations**: Instead of cryptic single letters, the chart labels directly state the actionable interpretation: `"Exhausted Sellers"`, `"Selling Pressure Absorbed"`, `"Exhausted Buyers"`, and `"Buying Pressure Absorbed"`.
- **Label Placement**: Labels are intelligently placed pointing up from the trough (for Bullish signals) and pointing down from the peak (for Bearish signals).
- **Trade Decision Dashboard**: A comprehensive, real-time table display that surfaces the latest flow bias, signal interpretation, trend alignment, and immediate trade action guidelines using highly legible typography sizes.
