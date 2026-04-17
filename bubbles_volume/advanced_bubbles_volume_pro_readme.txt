Here's the fully upgraded Pine Script with gradient colors, absorption, exhaustion, delta bias, climax detection, and more:Now let me create a visual reference card for the indicator:Here's what's new in the upgraded indicator:

**Gradient Color System** — A proper 5-stop multi-color gradient (aqua → green → yellow → orange → deep red) runs across the full normalized volume spectrum. In Delta mode, bubbles grade from pale lime into vivid green (bullish close bias) or pale salmon into vivid red (bearish close bias), with grey for neutral doji-like bars.

**Absorption Detection** — Fires when a bar has high volume *and* a tiny body relative to its range (configurable `body/range` ratio, default 20%). These appear as **"A"** labels above the bar, colored in blue — classic institutional absorption of supply or demand. The body ratio threshold is fully adjustable.

**Exhaustion Detection** — Two variants:
- **Bull Exhaustion (`↑X`)** — high-volume up-candle with a massive upper wick (close rejected at the high). Potential reversal down.
- **Bear Exhaustion (`↓X`)** — high-volume down-candle with a massive lower wick (close rejected at the low). Potential reversal up. The wick/range threshold is configurable (default 60%).

**Volume Climax** — Bars exceeding 4σ (configurable) get the huge bubble + raw volume label + optional **background color flash** (green for bull, red for bear). These are the "something big just happened" bars.

**Volume Delta Bias** — Estimated buy/sell pressure using `(close − low) / (high − low)`. No tick data needed — works on any timeframe.

**Color Priority** — Absorption → Exhaustion → HeatMap → Delta → Base gradient, so the most important signal always wins color.
