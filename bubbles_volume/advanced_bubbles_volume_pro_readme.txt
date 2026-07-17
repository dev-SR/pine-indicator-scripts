Here's the upgraded Pine Script with gradient colors, bull/bear absorption, bull/bear exhaustion, delta bias, climax detection, and more:

**Gradient Color System** — A 5-stop heatmap option (aqua → green → yellow → orange → deep red) runs across the normalized volume spectrum. In Delta mode, bubbles grade from pale lime into vivid green (bullish close bias) or pale salmon into vivid red (bearish close bias), with grey for neutral bars.

**Absorption Detection** — Fires when a bar has high volume, a tiny body relative to its range (configurable `body/range` ratio, default 20%), and balanced wicks. These appear as bias labels: green **`↑A`** below the bar for bull absorption and red **`↓A`** above the bar for bear absorption.

**Exhaustion Detection** — Two variants:
- **Bull Exhaustion (`↑X`)** — high-volume bar with a dominant lower wick (sellers rejected at the low). Colored green.
- **Bear Exhaustion (`↓X`)** — high-volume bar with a dominant upper wick (buyers rejected at the high). Colored red. The wick/range threshold is configurable (default 60%).

**Volume Climax** — Bars exceeding 4σ (configurable) get the huge bubble + raw volume label + optional **background color flash** (green for bull, red for bear). These are the "something big just happened" bars.

**Volume Delta Bias** — Estimated buy/sell pressure using `(close − low) / (high − low)`. No tick data needed — works on any timeframe.

**Color Priority** — Exhaustion → Absorption → Heatmap → Delta → Base gradient, so the most important signal always wins color.
