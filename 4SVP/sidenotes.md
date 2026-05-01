##  Mechanical Daily & Weekly Bias Dashboard


### 1. DAILY BIAS DETECTION RULES

**Step 1: Previous Day's RTH Volume Profile**
- Use the previous trading day's RTH (Regular Trading Hours / NY Session) volume profile.
- Extract: Value Area High (VAH), Value Area Low (VAL), Point of Control (POC).

**Step 2: Analyze Previous Day's Close vs Value Area**
- If Close > VAH → Daily Close Signal = BULLISH
- If Close < VAL → Daily Close Signal = BEARISH
- If VAL ≤ Close ≤ VAH → Daily Close Signal = NEUTRAL

**Step 3: Analyze Today's Open vs Previous Day's Value Area**
- Compare today's RTH open (9:30 AM) to the previous day's value area bounds (VAH/VAL).
- If Open > VAH → Daily Open Signal = BULLISH
- If Open < VAL → Daily Open Signal = BEARISH
- If VAL ≤ Open ≤ VAH → Daily Open Signal = NEUTRAL

**Daily Bias Resolution**
- If Daily Close Signal == Daily Open Signal → Daily Bias = that signal.
- If they differ → Daily Bias = NEUTRAL (conflicted).

---

### 2. WEEKLY BIAS DETECTION RULES

**Step 1: Previous Week's RTH Volume Profile**
- Use the previous calendar week's RTH volume profile (Monday-Friday NY session).
- Extract: Weekly VAH, Weekly VAL, Weekly POC.

**Step 2: Analyze Previous Week's Close vs Weekly Value Area**
- If Prev Week Close > Weekly VAH → Weekly Close Signal = BULLISH
- If Prev Week Close < Weekly VAL → Weekly Close Signal = BEARISH
- If Weekly VAL ≤ Prev Week Close ≤ Weekly VAH → Weekly Close Signal = NEUTRAL

**Step 3: Analyze Current Week's Open vs Previous Week's Value Area**
- Compare current week's first RTH open (Monday 9:30 AM) to previous week's value area bounds.
- If Current Week Open > Prev Week VAH → Weekly Open Signal = BULLISH
- If Current Week Open < Prev Week VAL → Weekly Open Signal = BEARISH
- If Prev Week VAL ≤ Current Week Open ≤ Prev Week VAH → Weekly Open Signal = NEUTRAL

**Weekly Bias Resolution**
- If Weekly Close Signal == Weekly Open Signal → Weekly Bias = that signal.
- If they differ → Weekly Bias = UNDETERMINED. Do NOT defer to monthly or higher timeframe. The weekly filter is invalid until both signals align.

---

### 3. FINAL BIAS RESOLUTION (Daily vs Weekly)

- If Daily Bias == Weekly Bias → Final Bias = Daily Bias.
- If Daily Bias != Weekly Bias:
  - If Weekly Bias is BULLISH → Final Bias = BULLISH (weekly overrides daily).
  - If Weekly Bias is BEARISH → Final Bias = BEARISH (weekly overrides daily).
  - If Weekly Bias is NEUTRAL → Final Bias = NEUTRAL (reduced conviction).
  - If Weekly Bias is UNDETERMINED → Final Bias = UNDETERMINED. No directional edge. Stand aside.

---

### 4. TABLE DASHBOARD SPECIFICATION

Draw a single compact table positioned at top-right of the chart.

**Table Structure:**

```
┌─────────────────────────────────────────────┐
│  D/W BIAS        [D]🟢BULL  [W]🟢BULL       │
│  ─────────────────────────────────────────  │
│  Prev Close  XXXX.XX  >  VAH  🟢BULL        │
│  Today Open  XXXX.XX  >  VAH  🟢BULL        │
│  ─────────────────────────────────────────  │
│  FINAL BIAS        ████████████  BULLISH    │
└─────────────────────────────────────────────┘
```