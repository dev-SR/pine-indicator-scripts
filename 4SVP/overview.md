 # System Understanding Document: 4 Session Volume Profiles

## 1. High-Level System Overview

**Purpose:** A multi-profile volume distribution overlay indicator for TradingView that renders volume profiles for five distinct market periods plus session analytics for two custom trading sessions.

**Domain:** Financial technical analysis / volume profile & market structure visualization.

**Application Type:** Pine Script v5 indicator (`indicator()`, `overlay = true`).

**Main Capabilities:**
- Render volume profiles (horizontal histograms) for: Current Session (CSVP), Previous Week (PWVP), Previous Day RTH (PDVP), Overnight (OVP), and Current Week (CWVP).
- Compute and display VAH (Value Area High), POC (Point of Control), VAL (Value Area Low), and High/Low levels for each profile.
- Provide session-specific analytics (VWAP, mean, trendline, max/min, R²) for two configurable sessions (NY and Overnight).
- Display a bias dashboard resolving daily vs weekly directional bias.
- Render daily dividers with day-of-week labels.
- Support lookback-based cleanup of historical session drawing objects.

---

## 2. Architecture Summary

**Platform:** TradingView Pine Script v5.

**Execution Model:** Event-driven per-bar execution with deferred rendering on `barstate.islast`.

**Structure:** Single-file procedural script organized into labeled blocks:
- Input Configuration (10+ groups)
- Helper Functions (LuxAlgo-style session analytics + volume profile computation)
- Session Detection & Data Collection
- Profile Computation & Caching
- Rendering (deferred to `barstate.islast`)
- Session Analytics Execution
- Daily Dividers
- Dashboards (Session + Bias)

**Resource Limits Declared:**
- `max_boxes_count = 2000`

---

## 3. Core Modules / Components

### 3.1 Volume Profile Input Configuration
- **Groups:** ⓪ Profile Colors, ① Previous Week (PWVP), ② Previous Day RTH (PDVP), ③ Overnight (OVP), ④ Current Session (CSVP), ⑤ Current Week (CWVP).
- **Per-profile controls:** Show Profile, Show Lines (VAH/POC/VAL), Show High/Low, Rows (2–500), Volume Type (Total/Delta), Value Area % (1–100).
- **Shared colors:** VAH (red), POC (gray), VAL (blue), High/Low (gray), Bar (white@80), Value Area Bar (white@50).

### 3.2 Session Analytics Input Configuration
- **Group ⑦ Session A (NY):** Enable, name, time (`0930-1600`), color, toggles for Range Box, Trendline, Mean, VWAP, Max/Min.
- **Group ⑧ Session B (Overnight):** Enable, name, time (`1700-0930`), color, same toggles.
- **Group ⑨ Ranges & Lookback:** Lookback Sessions (1–50, default 5), Range Transparency, Range Outline, Full Height Backgrounds, Range Label.
- **Group ⑩ Dividers:** Show Daily Dividers toggle.
- **Group ⑪ Session Dashboard:** Show Session Dashboard toggle.

### 3.3 Bias Dashboard Input Configuration
- **Group ⑥ Bias Dashboard:** Show Dashboard, Position (4 corners), Text Size (Tiny–Auto).
- **Group Display:** Level Line Width Scale (1–4), Show Level Labels, Label Size, Profile Width % of session span.
- **Group Timezone:** Preset (America/New_York, UTC, Exchange, Manual Offset), Manual UTC Offset.

### 3.4 LuxAlgo Session Helper Functions
| Function                                                                                          | Returns               | Behavior                                                                                                            |
| ------------------------------------------------------------------------------------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `ses_get_avg(active, start)`                                                                      | `float`               | Cumulative mean of `close`. Resets on `start`.                                                                      |
| `ses_get_linreg(active, start)`                                                                   | `[y1, y2, stdev, r2]` | Linear regression via least-squares on array of closes. Returns endpoints, stdev, R². Resets on `start`.            |
| `ses_get_vwap(active, start)`                                                                     | `[num, den]`          | Cumulative `close*volume` and `volume`. Resets on `start`.                                                          |
| `ses_get_range(active, start, name, css, draw_box, draw_lbl, full_h, trans, outl, bxs[], lbls[])` | `[max, min]`          | Creates/updates box and label. Supports full-height mode (`top=9999999, bot=0`). Pushes objects to tracking arrays. |
| `ses_set_line(active, start, y1, y2, css, lns[])`                                                 | `void`                | Creates/updates line object. Pushes to tracking array.                                                              |

### 3.5 Session Detection Module
- **Hardcoded session strings:** `rth_ses_str = "0930-1600"`, `ovn_ses_str = "1800-0930"`, `week_ses_str = "1800-1700:16"`.
- **Detection:** `math.sign(nz(time(tf, ses_str, tz)))` → `is_rth`, `is_ovn`, `is_week`.
- **Boundary flags:** `rth_start`, `ovn_start`, `week_start` (transition from 0 to 1).
- **Open tracking:** `current_rth_open`, `current_week_open` captured on session start.

### 3.6 Data Collection Module
- **Per-profile accumulators:** Arrays of `high`, `low`, `volume`, `buy_vol` (close >= open ? volume : 0), `close` (where applicable).
- **Snapshot mechanism:** On session start, live arrays are copied to snapshot arrays (`pw_sh`, `pd_sh`, etc.) for profile computation.
- **Time tracking:** `t1` (start time), `t2` (end time) per profile.
- **Dirty flags:** `cs_dirty`, `cw_dirty` to defer CSVP/CWVP computation to `barstate.islast`.

### 3.7 Profile Computation Engine (`compute_profile`)
- **Algorithm:** O(1) row-index math per bar (optimized from O(rows) scan).
- **Steps:**
  1. Determine `hi`/`lo` range, divide into N rows.
  2. For each bar, compute overlapping row indices and distribute volume proportionally.
  3. Find POC (max volume row).
  4. Expand Value Area from POC outward until cumulative volume reaches `va_pct`% of total.
- **Returns:** `[tops, bots, vols, poc_p, vah_p, val_p]`.

### 3.8 Rendering Engine (`barstate.islast`)
- **Pattern:** Delete all existing objects → recompute if dirty → redraw all profiles.
- **Profile rendering:** `draw_boxes()` creates horizontal bar boxes scaled by volume fraction.
- **Level rendering:** `draw_levels()` creates dotted lines for VAH/POC/VAL with labels.
- **High/Low rendering:** `draw_hl()` creates solid lines for session high/low with labels.
- **Label offset:** Prefix-based horizontal offset to prevent overlap (CS=10, OV=20, PD=40, PW=60, CW=80).

### 3.9 Session Analytics Execution
- **Detection:** `is_sa` (NY), `is_sb` (Overnight) via `time()` with configurable strings.
- **Computation:** Calls LuxAlgo helpers per bar.
- **Plotting:** VWAP, Max, Min via `plot(style = plot.style_linebr)`.
- **History cleanup:** `manage_boxes/lines/labels()` removes oldest objects when count exceeds lookback limit.

### 3.10 Daily Dividers
- **Trigger:** `dayofweek != dayofweek[1]`.
- **Visual:** Vertical dashed line + day-of-week label at top.
- **Week start:** Monday (`dayofweek == 2`) uses `size.small`; all others use `size.tiny`.

### 3.11 Session Dashboard
- **Trigger:** `barstate.islast` if `show_sd`.
- **Structure:** 4 columns × 3 rows table (Header + 2 session rows).
- **Columns:** Session, Status (Active/Inactive with green/red bg), Trend (R²), Volume.
- **No σ column:** Standard deviation is computed but not displayed.

### 3.12 Bias Dashboard
- **Trigger:** `barstate.islast` if `show_dashboard`.
- **Daily Bias:** Compares previous day's close and current RTH open against PDVP VAH/VAL.
- **Weekly Bias:** Compares previous week's close and current week open against PWVP VAH/VAL.
- **Resolution Logic:**
  - If daily == weekly → that bias.
  - If weekly is BULLISH/BEARISH → weekly overrides daily.
  - If weekly is NEUTRAL → NEUTRAL.
  - If weekly is UNDETERMINED → UNDETERMINED.
- **Visual:** 4–5 row table with color-coded signals, override messages, and final bias.

---

## 4. Data Flow

1. **Input Phase:** TradingView UI provides all configuration.
2. **Detection Phase:** Each bar, `time()` evaluates session membership → `is_rth`, `is_ovn`, `is_week`, `is_sa`, `is_sb`.
3. **Data Collection Phase:** Conditional array pushes accumulate OHLCV data for active profiles/sessions.
4. **Snapshot Phase:** On session boundaries, live data is copied to snapshot arrays for completed profiles.
5. **Profile Computation Phase:** Snapshot arrays fed into `compute_profile()` → cached row data + VAH/POC/VAL.
6. **Deferred Computation Phase:** On `barstate.islast`, CSVP/CWVP computed if dirty.
7. **Cleanup Phase:** On `barstate.islast`, all existing drawing objects deleted.
8. **Render Phase:** Cached data used to recreate all boxes, lines, labels.
9. **Dashboard Phase:** Final bias and session status computed and rendered.

---

## 5. State & Data Models

### 5.1 Profile Data Structures
- **Live arrays:** `pw_h/l/v/b/c`, `pd_h/l/v/b/c`, `ov_h/l/v/b`, `cs_h/l/v/b`, `cw_h/l/v/b` (current accumulating data).
- **Snapshot arrays:** `pw_sh/sl/sv/sb/sc`, `pd_sh/sl/sv/sb/sc`, `ov_sh/sl/sv/sb` (frozen at session end for computation).
- **Cached row data:** `pw_row_tops/bots/vols`, `pd_row_tops/bots/vols`, etc. (persisted for re-rendering).
- **Level caches:** `pw_vah/poc/val`, `pd_vah/poc/val`, etc.
- **Time bounds:** `pw_t1/t2`, `pd_t1/t2`, `ov_t1/t2`, `cs_t1`, `cs_draw_t2`.

### 5.2 Drawing Object Tracking Arrays
- `pw_boxes/lines/lbls`, `pd_boxes/lines/lbls`, `ov_boxes/lines/lbls`, `cs_boxes/lines/lbls`, `cw_boxes/lines/lbls`.
- `sa_bxs/lns/lbls`, `sb_bxs/lns/lbls` (for session analytics).

### 5.3 Session Analytics State
- `sa_avg`, `sa_y1/y2/std/r2`, `sa_vwap_num/den`, `sa_max/min`.
- `sb_avg`, `sb_y1/y2/std/r2`, `sb_vwap_num/den`, `sb_max/min`.

### 5.4 Bias State
- `current_rth_open`, `current_week_open`.
- `d_bias`, `w_bias`, `final_bias`, `weekly_overrides`.

### 5.5 Dashboard State
- `sd_table`, `dash_table` (persistent `var table` objects, recreated each `barstate.islast`).

---

## 6. Business Logic Summary

### 6.1 Volume Profile Logic
- **Row distribution:** Volume distributed proportionally based on bar's overlap with each price row.
- **Delta volume:** `math.abs(buy_vol - (volume - buy_vol))` when `vol_type == "Delta"`.
- **Value Area expansion:** Greedy outward expansion from POC, preferring the adjacent row with higher volume.

### 6.2 Bias Resolution Rules
| Daily    | Weekly          | Final        | Override? |
| -------- | --------------- | ------------ | --------- |
| BULLISH  | BULLISH         | BULLISH      | No        |
| BEARISH  | BEARISH         | BEARISH      | No        |
| Any      | BULLISH/BEARISH | Weekly       | Yes       |
| Any      | NEUTRAL         | NEUTRAL      | Yes       |
| Any      | UNDETERMINED    | UNDETERMINED | Yes       |
| Mismatch | —               | NEUTRAL      | No        |

### 6.3 Lookback Management
- On session start (`sa_start`, `sb_start`), oldest objects are removed from tracking arrays if count exceeds limit.
- Limits: `lookback_ses` for boxes/labels, `lookback_ses * 2` for lines (each session can have trendline + mean).

### 6.4 Full Height Backgrounds
- When `rng_full == true`, box top/bottom set to extreme values (`9999999` / `0`) instead of session high/low.

### 6.5 Dirty Flag Pattern (CSVP/CWVP)
- `cs_dirty` set to `true` when new CSVP data arrives.
- Computation deferred to `barstate.islast` to avoid O(bars²) complexity.
- Flag cleared after computation.

---

## 7. External Dependencies

**Platform:** TradingView Pine Script v5 runtime.

**Data Dependencies:**
- OHLCV: `open`, `high`, `low`, `close`, `volume`
- Time functions: `time()`, `dayofweek`, `bar_index`, `time`
- Chart info: `syminfo.timezone`, `syminfo.mintick`
- Built-in: `array.*`, `box.*`, `line.*`, `label.*`, `table.*`, `math.*`, `str.*`

**No External APIs/Network:** Fully sandboxed.

---

## 8. Assumptions

- **Session overlap:** NY (`0930-1600`) and Overnight (`1700-0930`) may overlap by design or by user modification. The code does not prevent this.
- **Timezone consistency:** The hardcoded `rth_ses_str`/`ovn_ses_str` for profile detection use the same `tz` as configurable sessions, but `week_ses_str = "1800-1700:16"` is hardcoded to Sunday 6pm–Friday 5pm (16 hours offset implied by `:16` suffix for Friday close).
- **R² computation:** Uses standard least-squares on an array of closes. The array grows with each bar in session, which may become expensive for very long sessions.
- **Label positioning:** `get_label_offset()` uses fixed pixel offsets that may collide on high-DPI or very zoomed charts.
- **Full height boxes:** Uses `9999999` and `0` as extreme price values. Assumes no instrument trades outside this range.
- **Volume availability:** All profile and VWAP calculations assume `volume` is available. No fallback for tick/volume-less instruments.
- **Array performance:** `compute_profile` uses integer math for row indexing, but still iterates overlapping rows per bar. Worst-case performance on high-row-count, high-bar-count sessions is not bounded.
- **Dashboard recreation:** Both dashboards are deleted and recreated every `barstate.islast`, which is acceptable for Pine Script but may cause flicker on rapid updates.

---

## 9. Missing Context

- **No alert functionality:** No `alertcondition()` calls present.
- **No strategy mode:** This is purely an indicator; no `strategy()` or order logic.
- **Incomplete file:** The script ends with `// ───────────────────────────────────────` and `// END`, but no plot or final statement. The last visible code is the bias dashboard block.
- **No error handling:** No explicit `na` guards around array access in some loops (e.g., `array.get` without bounds checks in helper functions).
- **No documentation on `:16` suffix:** `week_ses_str = "1800-1700:16"` uses a Pine Script session string extension for Friday close at 17:00, but this syntax is not standard and may be platform-specific.
- **Missing `csvp_session` logic for OVP:** The CSVP session selector has `"Overnight (6:00pm)"` but the actual detection uses `is_ovn` which is based on `1800-0930`, not `1700-0930` (the configurable Overnight session uses `1700-0930`). This is a potential inconsistency.