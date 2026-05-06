CVD Absorption/Exhaustion Indicator
CVD Absorption/Exhaustion Indicator – Explanation This indicator identifies trading opportunities by analyzing the relationship between price action and Cumulative Volume Delta (CVD) at key pivot points. It implements a professional trading framework that distinguishes between tradeable continuation signals (Absorption) and potential reversal warnings (Exhaustion). Part 1: Foundation – CVD Calculation The indicator starts by calculating Cumulative Volume Delta using the Bull & Bear Balance formula: Volume Pressure Calculation Bull Power: Measures buying pressure based on candlestick characteristics Bear Power: Measures selling pressure using the same methodology Volume Split: Each bar's volume is proportionally divided between bull and bear pressure Delta: bullVolume - bearVolume (net buying vs selling per bar) CVD: Running total (ta.cum(delta)) that shows cumulative market order flow On the chart: Yellow line = raw CVD. White line = optional SMA (20-period default). Fill color = teal when CVD > MA (bullish flow), red when below (bearish flow). Part 2: Signal Logic – Pivot Detection The indicator identifies pivot points on the CVD line (not price) using lookback parameters: Left Bars (lbL=1): Minimum bars to left to form pivot Right Bars (lbR=2): Bars to right to confirm pivot (also creates offset) Range Validation Pivot signals only trigger if the distance between consecutive pivots is between 5-60 bars (adjustable). This filters out noise and ensures meaningful divergence patterns. Part 3: Trading Framework The core innovation is distinguishing two signal types at each pivot: 🟢 BULLISH SIGNALS (at CVD Pivot Lows) Table Copy Signal Type Price Action CVD Action Trading Action Color Exhaustion Lower Low (LL) Higher Low (HL) AVOID - Reversal warning Transparent Gray Absorption Higher Low (HL) Lower Low (LL) TRADE - Continuation likely Solid Green 🔴 BEARISH SIGNALS (at CVD Pivot Highs) Table Copy Signal Type Price Action CVD Action Trading Action Color Exhaustion Higher High (HH) Lower High (LH) AVOID - Reversal warning Transparent Gray Absorption Lower High (LH) Higher High (HH) TRADE - Continuation likely Solid Red Part 4: Visualization Mechanism The indicator uses precision plotting for clarity: Pivot Lines: Thin vertical lines appear exactly at the pivot bar using offset=-lbR (shifts plot back to correct location) Conditional Coloring: Lines are transparent (noneColor) unless a valid signal exists Minimal Labels: Single letters "E" (Exhaustion) or "A" (Absorption) in tiny size to avoid chart clutter Direction: Labels appear above the line for bullish signals, below for bearish signals Part 5: How to Read the Chart Signal Quality Hierarchy Solid Green/Red lines with "A" = Primary trade signals (Absorption/Continuation) Transparent Gray lines with "E" = Warning signals (Exhaustion/Reversal) - use for context or exit planning No lines at pivots = No valid pattern - ignore Timeframe Usage Best on: 5-minute to 1-hour charts (as per PDF) Multi-timeframe: Use the dropdown in settings to analyze higher timeframe signals while trading lower timeframe Practical Workflow Wait for solid color "A" signal in the direction of the trend Confirm with price action (e.g., support/resistance break) Use "E" signals as profit targets or trend exhaustion warnings Never trade Exhaustion signals alone – they indicate potential reversals, not entries Alert System Four distinct alerts fire on bar close with clear messages: Exhaustion Bullish: "Price:LL, CVD:HL (Reversal)" Absorption Bullish: "Price:HL, CVD:LL (Continuation)" Exhaustion Bearish: "Price:HH, CVD:LH (Reversal)" Absorption Bearish: "Price:LH, CVD:HH (Continuation)"
by Fundorin1337
Updated Apr 17

9
3
CVD Absorption/Exhaustion Indicator
CVD Absorption/Exhaustion Indicator – Explanation This indicator identifies trading opportunities by analyzing the relationship between price action and Cumulative Volume Delta (CVD) at key pivot points. It implements a professional trading framework that distinguishes between tradeable continuation signals (Absorption) and potential reversal warnings (Exhaustion). Part 1: Foundation – CVD Calculation The indicator starts by calculating Cumulative Volume Delta using the Bull & Bear Balance formula: Volume Pressure Calculation Bull Power: Measures buying pressure based on candlestick characteristics Bear Power: Measures selling pressure using the same methodology Volume Split: Each bar's volume is proportionally divided between bull and bear pressure Delta: bullVolume - bearVolume (net buying vs selling per bar) CVD: Running total (ta.cum(delta)) that shows cumulative market order flow On the chart: Yellow line = raw CVD. White line = optional SMA (20-period default). Fill color = teal when CVD > MA (bullish flow), red when below (bearish flow). Part 2: Signal Logic – Pivot Detection The indicator identifies pivot points on the CVD line (not price) using lookback parameters: Left Bars (lbL=1): Minimum bars to left to form pivot Right Bars (lbR=2): Bars to right to confirm pivot (also creates offset) Range Validation Pivot signals only trigger if the distance between consecutive pivots is between 5-60 bars (adjustable). This filters out noise and ensures meaningful divergence patterns. Part 3: Trading Framework – PDF Logic The core innovation is distinguishing two signal types at each pivot: 🟢 BULLISH SIGNALS (at CVD Pivot Lows) Table Copy Signal Type Price Action CVD Action Trading Action Color Exhaustion Lower Low (LL) Higher Low (HL) AVOID - Reversal warning Transparent Gray Absorption Higher Low (HL) Lower Low (LL) TRADE - Continuation likely Solid Green 🔴 BEARISH SIGNALS (at CVD Pivot Highs) Table Copy Signal Type Price Action CVD Action Trading Action Color Exhaustion Higher High (HH) Lower High (LH) AVOID - Reversal warning Transparent Gray Absorption Lower High (LH) Higher High (HH) TRADE - Continuation likely Solid Red Part 4: Visualization Mechanism The indicator uses precision plotting for clarity: Pivot Lines: Thin vertical lines appear exactly at the pivot bar using offset=-lbR (shifts plot back to correct location) Conditional Coloring: Lines are transparent (noneColor) unless a valid signal exists Minimal Labels: Single letters "E" (Exhaustion) or "A" (Absorption) in tiny size to avoid chart clutter Direction: Labels appear above the line for bullish signals, below for bearish signals Part 5: How to Read the Chart Signal Quality Hierarchy Solid Green/Red lines with "A" = Primary trade signals (Absorption/Continuation) Transparent Gray lines with "E" = Warning signals (Exhaustion/Reversal) - use for context or exit planning No lines at pivots = No valid pattern - ignore Timeframe Usage Best on: 5-minute to 1-hour charts (as per PDF) Multi-timeframe: Use the dropdown in settings to analyze higher timeframe signals while trading lower timeframe Practical Workflow Wait for solid color "A" signal in the direction of the trend Confirm with price action (e.g., support/resistance break) Use "E" signals as profit targets or trend exhaustion warnings Never trade Exhaustion signals alone – they indicate potential reversals, not entries Alert System Four distinct alerts fire on bar close with clear messages: Exhaustion Bullish: "Price:LL, CVD:HL (Reversal)" Absorption Bullish: "Price:HL, CVD:LL (Continuation)" Exhaustion Bearish: "Price:HH, CVD:LH (Reversal)" Absorption Bearish: "Price:LH, CVD:HH (Continuation)"
by Fundorin1337
Updated Apr 17

4
6
Comment

Close ad


Close
Updated Apr 17
CVD Absorption/Exhaustion Indicator

Remove from favorites

Use on chart

9
3


2 472
Nov 16, 2025
CVD Absorption/Exhaustion Indicator – Explanation

This indicator identifies trading opportunities by analyzing the relationship between price action and Cumulative Volume Delta (CVD) at key pivot points. It implements a professional trading framework that distinguishes between tradeable continuation signals (Absorption) and potential reversal warnings (Exhaustion).

Part 1: Foundation – CVD Calculation

The indicator starts by calculating Cumulative Volume Delta using the Bull & Bear Balance formula:

Volume Pressure Calculation

Bull Power: Measures buying pressure based on candlestick characteristics
Bear Power: Measures selling pressure using the same methodology
Volume Split: Each bar's volume is proportionally divided between bull and bear pressure
Delta: bullVolume - bearVolume (net buying vs selling per bar)
CVD: Running total (ta.cum(delta)) that shows cumulative market order flow
On the chart: Yellow line = raw CVD. White line = optional SMA (20-period default). Fill color = teal when CVD > MA (bullish flow), red when below (bearish flow).

Part 2: Signal Logic – Pivot Detection

The indicator identifies pivot points on the CVD line (not price) using lookback parameters:
Left Bars (lbL=1): Minimum bars to left to form pivot
Right Bars (lbR=2): Bars to right to confirm pivot (also creates offset)

Range Validation
Pivot signals only trigger if the distance between consecutive pivots is between 5-60 bars (adjustable). This filters out noise and ensures meaningful divergence patterns.

Part 3: Trading Framework

The core innovation is distinguishing two signal types at each pivot:

🟢 BULLISH SIGNALS (at CVD Pivot Lows)
Table
Copy
Signal Type Price Action CVD Action Trading Action Color
Exhaustion Lower Low (LL) Higher Low (HL) AVOID - Reversal warning Transparent Gray
Absorption Higher Low (HL) Lower Low (LL) TRADE - Continuation likely Solid Green

🔴 BEARISH SIGNALS (at CVD Pivot Highs)
Table
Copy
Signal Type Price Action CVD Action Trading Action Color
Exhaustion Higher High (HH) Lower High (LH) AVOID - Reversal warning Transparent Gray
Absorption Lower High (LH) Higher High (HH) TRADE - Continuation likely Solid Red

Part 4: Visualization Mechanism

The indicator uses precision plotting for clarity:
Pivot Lines: Thin vertical lines appear exactly at the pivot bar using offset=-lbR (shifts plot back to correct location)
Conditional Coloring: Lines are transparent (noneColor) unless a valid signal exists
Minimal Labels: Single letters "E" (Exhaustion) or "A" (Absorption) in tiny size to avoid chart clutter
Direction: Labels appear above the line for bullish signals, below for bearish signals

Part 5: How to Read the Chart

Signal Quality Hierarchy
Solid Green/Red lines with "A" = Primary trade signals (Absorption/Continuation)
Transparent Gray lines with "E" = Warning signals (Exhaustion/Reversal) - use for context or exit planning
No lines at pivots = No valid pattern - ignore
Timeframe Usage
Best on: 5-minute to 1-hour charts (as per PDF)
Multi-timeframe: Use the dropdown in settings to analyze higher timeframe signals while trading lower timeframe
Practical Workflow
Wait for solid color "A" signal in the direction of the trend
Confirm with price action (e.g., support/resistance break)
Use "E" signals as profit targets or trend exhaustion warnings
Never trade Exhaustion signals alone – they indicate potential reversals, not entries
Alert System
Four distinct alerts fire on bar close with clear messages:
Exhaustion Bullish: "Price:LL, CVD:HL (Reversal)"
Absorption Bullish: "Price:HL, CVD:LL (Continuation)"
Exhaustion Bearish: "Price:HH, CVD:LH (Reversal)"
Absorption Bearish: "Price:LH, CVD:HH (Continuation)"
Nov 16, 2025
Release Notes
CVD Absorption/Exhaustion Indicator – Explanation

This indicator identifies trading opportunities by analyzing the relationship between price action and Cumulative Volume Delta (CVD) at key pivot points. It implements a professional trading framework that distinguishes between tradeable continuation signals (Absorption) and potential reversal warnings (Exhaustion).

Part 1: Foundation – CVD Calculation

The indicator starts by calculating Cumulative Volume Delta using the Bull & Bear Balance formula:

Volume Pressure Calculation

Bull Power: Measures buying pressure based on candlestick characteristics
Bear Power: Measures selling pressure using the same methodology
Volume Split: Each bar's volume is proportionally divided between bull and bear pressure
Delta: bullVolume - bearVolume (net buying vs selling per bar)
CVD: Running total (ta.cum(delta)) that shows cumulative market order flow
On the chart: Yellow line = raw CVD. White line = optional SMA (20-period default). Fill color = teal when CVD > MA (bullish flow), red when below (bearish flow).

Part 2: Signal Logic – Pivot Detection

The indicator identifies pivot points on the CVD line (not price) using lookback parameters:
Left Bars (lbL=1): Minimum bars to left to form pivot
Right Bars (lbR=2): Bars to right to confirm pivot (also creates offset)

Range Validation
Pivot signals only trigger if the distance between consecutive pivots is between 5-60 bars (adjustable). This filters out noise and ensures meaningful divergence patterns.

Part 3: Trading Framework

The core innovation is distinguishing two signal types at each pivot:

🟢 BULLISH SIGNALS (at CVD Pivot Lows)
Table
Copy
Signal Type Price Action CVD Action Trading Action Color
Exhaustion Lower Low (LL) Higher Low (HL) AVOID - Reversal warning Transparent Gray
Absorption Higher Low (HL) Lower Low (LL) TRADE - Continuation likely Solid Green

🔴 BEARISH SIGNALS (at CVD Pivot Highs)
Table
Copy
Signal Type Price Action CVD Action Trading Action Color
Exhaustion Higher High (HH) Lower High (LH) AVOID - Reversal warning Transparent Gray
Absorption Lower High (LH) Higher High (HH) TRADE - Continuation likely Solid Red

Part 4: Visualization Mechanism

The indicator uses precision plotting for clarity:
Pivot Lines: Thin vertical lines appear exactly at the pivot bar using offset=-lbR (shifts plot back to correct location)
Conditional Coloring: Lines are transparent (noneColor) unless a valid signal exists
Minimal Labels: Single letters "E" (Exhaustion) or "A" (Absorption) in tiny size to avoid chart clutter
Direction: Labels appear above the line for bullish signals, below for bearish signals

Part 5: How to Read the Chart

Signal Quality Hierarchy
Solid Green/Red lines with "A" = Primary trade signals (Absorption/Continuation)
Transparent Gray lines with "E" = Warning signals (Exhaustion/Reversal) - use for context or exit planning
No lines at pivots = No valid pattern - ignore
Timeframe Usage
Best on: 5-minute to 1-hour charts (as per PDF)
Multi-timeframe: Use the dropdown in settings to analyze higher timeframe signals while trading lower timeframe
Practical Workflow
Wait for solid color "A" signal in the direction of the trend
Confirm with price action (e.g., support/resistance break)
Use "E" signals as profit targets or trend exhaustion warnings
Never trade Exhaustion signals alone – they indicate potential reversals, not entries
Alert System
Four distinct alerts fire on bar close with clear messages:
Exhaustion Bullish: "Price:LL, CVD:HL (Reversal)"
Absorption Bullish: "Price:HL, CVD:LL (Continuation)"
Exhaustion Bearish: "Price:HH, CVD:LH (Reversal)"
Absorption Bearish: "Price:LH, CVD:HH (Continuation)"
Apr 17
Release Notes
CVD Absorption & Exhaustion (Precise Order Flow)
The CVD Absorption & Exhaustion indicator is a sophisticated order flow tool designed to reveal the "hidden" battle between aggressive and passive market participants. Unlike standard volume indicators, this tool tracks the Cumulative Volume Delta (CVD) and compares it against price action to identify where market moves are losing steam or being intentionally blocked by large orders.

🟢 What it Tracks
This indicator calculates the intra-bar delta by analyzing where the price closes relative to its range (high/low). It then builds a cumulative line to show the net buying or selling pressure over time. By identifying pivots in this data, it detects two critical market phenomena:

1. Absorption (The "Brick Wall")
Absorption occurs when aggressive market participants (market orders) are being "filled" by a large passive participant (limit orders).

Bullish Absorption: Price makes a Higher Low, but CVD makes a Lower Low. This suggests aggressive sellers are slamming the market, but price isn't dropping because a "Big Player" is absorbing all the selling.

Bearish Absorption: Price makes a Lower High, but CVD makes a Higher High. Aggressive buyers are active, but price isn't rising because sell limit orders are blocking the move.

2. Exhaustion (The "Tired Trend")
Exhaustion occurs when a trend continues on price, but the aggressive participation is fading.

Bullish Exhaustion: Price makes a Lower Low, but CVD makes a Higher Low. Sellers are no longer as aggressive as they were at the previous low—the move is "running out of gas."

Bearish Exhaustion: Price makes a Higher High, but CVD makes a Lower High. Buyers are losing interest, and a reversal may be imminent.

🛠️ Key Features
Precise Plotting: Signals and labels are anchored directly to the CVD value and time-corrected (using lbR offsets) to ensure the visual lines match the exact point of the divergence.

Dynamic Trend Ribbon: A smoothed Moving Average (MA) fill provides an immediate visual of the overall delta trend (Teal for Bullish, Red for Bearish).

Clean Scaling: Built-in logic prevents "scale squishing," ensuring the CVD line remains readable even on assets with high price values like Bitcoin.

Tradeable Logic: High-conviction signals (Absorption) are highlighted in solid colors (Green/Red), while lower-conviction/reversal warning signals (Exhaustion) are highlighted in Gray.

💡 How to Trade with it
Context is King: Use the CVD MA to determine the overall bias. Look for Bullish Absorption when the CVD is above its MA for high-probability continuation entries.

The Counter-Trend Warning: If you are long and see a Bearish Exhaustion (EXH) label, it’s a signal that buyers are thinning out—consider tightening your stop-loss or taking profits.

The Reversal Entry: Look for Exhaustion at major support or resistance levels for precise "sniper" entries against the exhausted trend.
Protected script

Insort:

| **Market Side** | **Price Action** | **CVD Action** | **Interpretation**        |
| --------------- | ---------------- | -------------- | ------------------------- |
| 🟢 Bullish       | Lower Low        | Higher Low     | Exhausted Sellers         |
| 🟢 Bullish       | Higher Low       | Lower Low      | Selling Pressure Absorbed |
| 🔴 Bearish       | Higher High      | Lower High     | Exhausted Buyers          |
| 🔴 Bearish       | Lower High       | Higher High    | Buying Pressure Absorbed  |
