# Market Structures Screener | Flux Charts

This Pine Script™ code is a TradingView indicator that identifies and displays market structures like Break of Structure (BOS), Change of Character (CHoCH), and an enhanced version, CHoCH+. It's designed to screen multiple tickers and timeframes for these patterns.

## Algorithm and Logic

The script's logic for identifying market structures can be broken down into the following steps:

### 1. Pivot Point Identification

The foundation of the market structure analysis is the identification of significant pivot highs and pivot lows. The script uses a function called `atrPivots` to achieve this. This function uses a combination of the built-in `ta.pivothigh` and `ta.pivotlow` functions along with an Average True Range (ATR) based condition. This allows the script to filter out minor price fluctuations and focus on more significant price swings.

### 2. Market Structure Calculation

The core of the logic resides in the `calculateMarketStructure` function. This function takes the identified pivot points and analyzes them to identify BOS, CHoCH, and CHoCH+ patterns.

Here's how it works:

*   **Iteration:** The function iterates through the list of detected pivot points.
*   **Cross Check:** For each pivot, it checks if the current price has "crossed" the pivot's price level. A custom function, `betterCross`, is used to determine the direction of the cross (bullish or bearish).
*   **Break of Structure (BOS):** A BOS is identified when the price breaks through a pivot point in the *same direction* as the prevailing market structure.
    *   **Bullish BOS:** If the current market structure is determined to be bullish (`currentStructure == 1`), and a bullish cross occurs (price breaks above a high pivot), it's identified as a bullish BOS.
    *   **Bearish BOS:** If the current market structure is bearish (`currentStructure == -1`), and a bearish cross occurs (price breaks below a low pivot), it's identified as a bearish BOS.
*   **Change of Character (CHoCH):** A CHoCH is identified when the price breaks through a pivot point in the *opposite direction* of the prevailing market structure, suggesting a potential trend reversal.
    *   **Bullish CHoCH:** If the current market structure is bearish, and a bullish cross occurs, it signals a bullish CHoCH.
    *   **Bearish CHoCH:** If the current market structure is bullish, and a bearish cross occurs, it signals a bearish CHoCH.
*   **CHoCH+:** This is an enhanced version of the CHoCH pattern, designed to identify stronger reversal signals. A CHoCH+ is confirmed when a CHoCH occurs, and the new pivot formed is "stronger" than the previous pivot in the same direction.
    *   **Bullish CHoCH+:** For a bullish CHoCH to be upgraded to a CHoCH+, the new low pivot that is formed must be *lower* than the previous low pivot.
    *   **Bearish CHoCH+:** For a bearish CHoCH to be upgraded to a CHoCH+, the new high pivot that is formed must be *higher* than the previous high pivot.

### 3. Data Handling and Display

The script is designed to be a screener, which means it can analyze multiple financial instruments and timeframes simultaneously.

*   **Custom Data Types:** The script uses custom data types (`labelPlot`, `linePlot`, `marketStructureInfo`) to efficiently store the information about the identified market structures, including their price levels, time of occurrence, and type (BOS, CHoCH, or CHoCH+).
*   **Dashboard Display:** A table is used to present the results in a clear and organized manner. The `handleDashboard` function is responsible for creating and updating this table, which shows the status of BOS, CHoCH, and CHoCH+ for each configured ticker and timeframe.
*   **Orchestration:** The `handleMarketStructures` function acts as the main orchestrator, calling the pivot identification and market structure calculation functions on each new bar of the chart.
