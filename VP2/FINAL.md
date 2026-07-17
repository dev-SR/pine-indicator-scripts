
## 3. Notation, Session Universe, and Data Prerequisites

## 3.1 The session set

All structure in this framework is built from profiles defined over a fixed universe of sessions:

$$
\mathcal{S} = \{\ PW,\\ PAs^{(d)},\\ PLs^{(d)}, \\ PNYs^{(d)},\\ PONs^{(d)}, \\ DEV,\ \ JOINT\ \}
$$

where:

- **$PAs^{(d)}, PLs^{(d)}, PNYs^{(d)}, PONs^{(d)}$** — all are completed sessions (Prior Asia, Prior London, Prior NY, Prior Overnight) of any given dataset (d-1th asian session, d-1th london session, d-1th ny session, d-1th overnight session for a given day d) including the current session if it is completed, if the current session is not completed then it will be the developing session $DEV$.
- **DEV** — Developing Session. The session that is currently in progress. It is not a fixed session type like Asia, London, or NY. Instead, it represents whichever session is currently forming (Asia if it's before 7:00 AM EST, London if it's 7:00 AM - 12:00 PM EST, NY if it's 12:00 PM - 5:00 PM EST, or Overnight if it's 5:00 PM - 7:00 AM EST).
- **PW** — Prior Week. The most recently completed calendar week, treated as a single structural reference.
- **$JOINT^{(d)}$** — It is an equal-weight mixture of all the profiles of a given day $d$ in the set $\mathcal{S} \setminus \{JOINT, DEV\}$ which are completed on that day $d$.

For forex/markets, a common EST reference standard is:

| Session               | EST Time (12h)            | EST Time (24h)    |
| --------------------- | ------------------------- | ----------------- |
| Asia Session          | 5:00 PM – 4:00 AM EST     | 17:00 – 04:00     |
| London Session        | 3:00 AM – 12:00 PM EST    | 03:00 – 12:00     |
| New York Session      | 8:00 AM – 5:00 PM EST     | 08:00 – 17:00     |
| **Overnight Session** | **5:00 PM – 8:00 AM EST** | **17:00 – 08:00** |



### 3.2 The five profile lenses

Every session $s \in \mathcal{S}$ at offset $d$ is not one profile but five, all built from the same underlying trades and bars:

$$
\mathbf{P}_s^{(d)} = \left\{\ f_s^{tot,(d)},\ \ f_s^{buy,(d)},\ \ f_s^{sell,(d)},\ \ \Delta_s^{(d)},\ \ \mathcal{T}_s^{(d)}\ \right\}
$$

| Lens                                              | Meaning                                                                                                |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| $f_s^{tot,(d)}$                                   | Total volume-at-price. The standard volume profile.                                                    |
| $f_s^{buy,(d)}$                                   | Aggressor-buy volume-at-price.                                                                         |
| $f_s^{sell,(d)}$                                  | Aggressor-sell volume-at-price.                                                                        |
| $\Delta_s^{(d)} = f_s^{buy,(d)} - f_s^{sell,(d)}$ | Net delta-at-price. Positive = net aggressive buying at that price; negative = net aggressive selling. |
| $\mathcal{T}_s^{(d)}$                             | Time-at-price (TPO/Market Profile). Counts periods spent at a price, independent of volume.            |


### 3.3 Profile primitives

Every lens exposes a shared set of primitives:

$$
POC_s^{(d)},\quad VAH_s^{(d)},\quad VAL_s^{(d)},\quad \mathcal{H}_s^{(d)} = \{HVN_1, HVN_2, \dots\},\quad \mathcal{L}_s^{(d)} = \{LVN_1, LVN_2, \dots\}
$$




### 3.5 Thin-session handling

A session instance is flagged **THIN** if it has fewer than 20 bars, or its total volume is below 5% of $PW$'s total volume. Thin sessions are not discarded — they are down-weighted everywhere they would otherwise participate: a 0.40× multiplier on any confirmation weight in the Location Quality Engine (§15), and the same 0.40× multiplier on any session-state classification in the Session Sequence Engine (§11). A session that barely traded should not be allowed to produce a confident reading anywhere in the framework.

### 3.6 A general note on notation

Throughout this document:

- Lowercase $x$ or $p$ denotes a price.
- $P_t$ denotes current price at time $t$.
- A subscript $h$ (as in $Z_h$, $w_h$) denotes "a historical zone/profile," used when a formula compares a live quantity against a fixed reference.
- A subscript $k$ (as in $Z_k$) denotes "the candidate zone under evaluation."
- $ATR$ denotes a rolling average true range, used throughout as the universal distance-normalizer so that thresholds are comparable across instruments and volatility regimes.
- A model subscript ($struct$, $vol$, $flow$, $seq$) on $\text{Bias}$ (e.g. $\text{Bias}_{struct}$) denotes the directional output of one of the four models in Section 13, before the consensus vote combines them. Unsubscripted $\text{Bias}$ always refers to the post-vote, combined figure.
- Every parameter introduced in the body of the document is collected in Appendix B with its default value.

## 5. Profile Construction Pipeline

### 5.1 Bin construction

Each session's price range is divided into price bins. The bin width is dynamically computed as:

$$
w_{bin} = \max\left(\text{tick}, \frac{p_{high} - p_{low}}{N}\right), \qquad N = 200
$$

where $N = 200$ (defined by `NUM_PROFILE_ROWS` in configuration). The actual number of bins $n$ is determined by:

$$
n = \max\left(1, \left\lceil \frac{p_{high} - p_{low}}{w_{bin}} \right\rceil\right)
$$

The bin centers are located at $c_j = p_{low} + (j + 0.5) \cdot w_{bin}$ for $j \in [0, n - 1]$.

**Volume and Weight Allocation:**
For each bar $i$ with price range $[lo_i, hi_i]$, the start and end bin indices are calculated:

$$
j_{0,i} = \text{clip}\left(\left\lfloor \frac{lo_i - p_{low}}{w_{bin}} \right\rfloor, 0, n - 1\right), \qquad j_{1,i} = \text{clip}\left(\left\lfloor \frac{hi_i - p_{low}}{w_{bin}} \right\rfloor, 0, n - 1\right)
$$

The number of bins touched by the bar is $n_i = j_{1,i} - j_{0,i} + 1$. The bar's weight $W_i$ is distributed uniformly across all bins in the range $[j_{0,i}, j_{1,i}]$. That is, each touched bin receives a weight contribution of:

$$
\delta_{w} = \frac{W_i}{n_i}
$$

where:
- $W_i = V_i$ (the traded volume) for standard volume profiles (`VolumeProfileEngine`).
- $W_i = 1.0$ (representing one time unit) for TPO profiles (`TPOProfileEngine`).

### 5.2 Buy/sell allocation and normalization

For volume profiles, if explicit aggressor-side columns `buy_volume` and `sell_volume` are available, they are used to distribute buy and sell volume across the touched bins. If they are not available, a close-location ratio $\alpha_i \in [0, 1]$ is used as a fallback:

$$
\alpha_i = \text{clip}\left(\frac{\text{close}_i - lo_i}{hi_i - lo_i + 10^{-9}}, 0.0, 1.0\right)
$$

The buy and sell volumes for bar $i$ are then allocated as:

$$
V_i^{buy} = \alpha_i \cdot V_i, \qquad V_i^{sell} = (1 - \alpha_i) \cdot V_i
$$

These quantities are also distributed uniformly across the touched bins, ensuring that at the bin level, the additive identity $f_S^{buy}(c_j) + f_S^{sell}(c_j) = f_S^{tot}(c_j)$ holds exactly.

### 5.3 Density estimation

Rather than working with the raw binned histogram directly, every profile is smoothed into a continuous density via kernel density estimation, so that zone extraction and confluence scoring are not sensitive to arbitrary bin-edge placement.

**Bandwidth.** The bandwidth is set dynamically based on the volume-weighted standard deviation of price in the session:

$$
h_S = \max(\kappa_h \cdot \sigma_{V,S},\, \text{tick}), \qquad \kappa_h = 0.04
$$

where:

$$
\sigma_{V,S} = \sqrt{\sum_{b} w_b (c_b - \mu_{w})^2}, \qquad \mu_{w} = \sum_{b} w_b c_b
$$

with $w_b = \frac{\text{vol}_b}{V_{tot,S}}$ the volume share of bin $b$ (which sums to $1$ across all bins).

**Analytic weighted KDE.** The total volume density at any price $x$ is evaluated on a grid of $n_{eval} = 600$ points spanning from $p_{min} - 0.08R$ to $p_{max} + 0.08R$ (where $R = p_{max} - p_{min}$):

$$
\hat{f}_S(x) = \sum_{b} \frac{w_b}{\sqrt{2\pi}\,h_S} \exp\!\left(-\frac{(x - c_b)^2}{2 h_S^2}\right)
$$

Because the weights sum to 1, $\hat{f}_S(x)$ is a proper probability density function.

**Buy/Sell sub-profile KDEs.** Sub-profile densities are evaluated on the same grid using the same bandwidth $h_S$, but normalized using the total volume $V_{tot,S}$ as the divisor:

$$
\hat{f}_S^{buy}(x) = \text{KDE}(\text{buy\_vols})(x) \cdot \frac{V_{tot,S}^{buy}}{V_{tot,S}}, \qquad \hat{f}_S^{sell}(x) = \text{KDE}(\text{sell\_vols})(x) \cdot \frac{V_{tot,S}^{sell}}{V_{tot,S}}
$$

This maintains the exact additive identity in the density domain: $\hat{f}_S^{buy}(x) + \hat{f}_S^{sell}(x) = \hat{f}_S(x)$.

**CDF.** The cumulative distribution function $\hat{F}_S(x)$ is computed on the grid via numeric integration and normalized to exactly $1.0$ at the upper boundary:

$$
\hat{F}_S(x) = \frac{\int_{-\infty}^{x} \hat{f}_S(u)\,du}{\int_{-\infty}^{\infty} \hat{f}_S(u)\,du}
$$

**Continuous Z-Score Field.** Used for identifying high and low volume nodes:

$$
\mathfrak{z}_S(x) = \frac{\hat{f}_S(x) - \mu_{f,S}}{\sigma_{f,S}}
$$

where $\mu_{f,S}$ and $\sigma_{f,S}$ are the mean and standard deviation of the continuous density function values evaluated across the $n_{eval}$ grid points.

**Peak-Normalized Density.** Used for attraction and confluence weighting:

$$
\hat{z}_S(x) = \frac{\hat{f}_S(x)}{\max_{u} \hat{f}_S(u)} \in [0, 1]
$$

### 5.4 Dynamic key-level zones

A **PriceZone** is the tradeable object representing a key price level or area: $Z = [lo, hi]$. Every extracted zone is represented by a `PriceZone` structure containing:
- `center`: The density-weighted centroid of the zone or the exact price level.
- `low`, `high`: The price boundaries of the zone.
- `zone_type`: Identifier (e.g., "POC", "VAL", "VAH", "HVN", "LVN").
- `label`: Label describing the zone.
- `z_score`: Peak z-score associated with the zone.
- `source`: The source session label (e.g., "PAs0", "DEV").

The half-width $w$ of the zone is determined dynamically based on the zone type, as detailed in Section 6.

---

## 6. Key Levels and Zone Extraction

*[Phase 0.]*

### 6.1 Point of Control and Value Area

- **Point of Control (POC)**: The level $POC_S$ is the center of the bin that contains the absolute maximum volume:

  $$
  POC_S = \arg\max_{c_b} f_S^{tot}(c_b)
  $$

- **Value Area (VAL/VAH)**: The value area is built in the binned volume domain. Bins are sorted in descending order of volume. We accumulate volumes from these sorted bins until the total volume reaches or exceeds the target proportion $70\%$ (configured via `va_pct`):

  $$
  \sum_{b \in \mathcal{V}} \text{vol}_b \geq 0.70 \cdot V_{tot,S}
  $$

  The lowest and highest bin centers in the accumulated set $\mathcal{V}$ are defined as the value area boundaries:

  $$
  VAL_S = \min_{b \in \mathcal{V}} c_b, \qquad VAH_S = \max_{b \in \mathcal{V}} c_b
  $$

These levels are converted into two types of zones:

1. **Fixed-width key levels**:
   `poc`, `val`, and `vah` are represented with a fixed half-width equal to half the bin width:

   $$
   w = \frac{w_{bin}}{2} \implies Z = [\text{Level} - w, \text{Level} + w]
   $$

2. **Dynamically-sized zones**:
   - **POC Zone**:
     - Sized by checking if the price $POC_S$ falls within any of the computed HVN zones.
     - If it does, the POC zone inherits the boundaries $[lo, hi]$ of that HVN zone.
     - If it does not, it falls back to a width based on the KDE bandwidth: $w = \max(h_S, 2 \cdot \text{tick})$, giving the zone $[POC_S - w, POC_S + w]$.
   - **VAL and VAH Zones**:
     - Sized from the local curvature (derivative) of the continuous KDE density $\hat{f}_S$:

       $$
       w_{VAL/VAH} = \text{clip}\left(\alpha_{VA} \cdot \frac{f_0}{|g| + \epsilon},\, w_{\min},\, w_{\max}\right)
       $$

       where:
       - $f_0$ is the density value at the level price: $f_0 = \hat{f}_S(VAL_S)$ or $\hat{f}_S(VAH_S)$.
       - $g$ is the local backward (VAL) or forward (VAH) gradient of density on the evaluation grid:

         $$
         g_{VAL} = \max\left(0, \frac{\hat{f}_S(VAL_S) - \hat{f}_S(VAL_S - dx)}{dx}\right)
         $$

         $$
         g_{VAH} = \max\left(0, \frac{\hat{f}_S(VAH_S) - \hat{f}_S(VAH_S + dx)}{dx}\right)
         $$

       - $\alpha_{VA} = 0.50$ (from configuration).
       - $\epsilon = 10^{-3} \cdot \max_{u} \hat{f}_S(u)$ (prevents division by zero).
       - $R$ is the session's overall high-to-low range.
       - Clamping limits: $w_{\min} = \max(2 \cdot \text{tick},\, 0.005 \cdot R)$, $w_{\max} = 0.04 \cdot R$.
       The resulting zone is $[\text{Level} - w,\, \text{Level} + w]$.

### 6.2 High- and low-volume nodes

HVN and LVN zones are extracted by thresholding the z-score field $\mathfrak{z}_S(x)$ on the continuous evaluation grid $x_j$ ($j = 0, \dots, n_{eval}-1$) with uniform step $dx = x_1 - x_0$:

- **HVN Candidates**: $\mathfrak{z}_S(x_j) \geq \zeta_{HVN} = 0.80$
- **LVN Candidates**: $\mathfrak{z}_S(x_j) \leq \zeta_{LVN} = -0.50$

**Index-Clustering Pipeline** (replaces the old forward-scan segment bridge):

1. **Collect qualifying indices**:

   $$
   \mathcal{I}_{HVN} = \{ j \mid \mathfrak{z}_S(x_j) \geq \zeta_{HVN} \},\qquad
   \mathcal{I}_{LVN} = \{ j \mid \mathfrak{z}_S(x_j) \leq \zeta_{LVN} \}
   $$

2. **Convert price-space gap to index-space gap**:

   The merge-gap limit $G = 0.020 \cdot R$ (2\% of the session range) is converted to a maximum index separation:

   $$
   \Delta j_{\max} = \left\lceil \frac{G}{dx} \right\rceil
                      = \left\lceil \frac{0.020 \cdot R}{dx} \right\rceil
   $$

   Two candidate indices $j_a, j_b$ belong to the same cluster iff all intermediate gaps are $\leq \Delta j_{\max}$, which is exactly equivalent to $|x_{j_a} - x_{j_b}| \leq G$ in price space.

3. **Cluster by index gap**:

   Split $\mathcal{I}$ at every position where the consecutive difference exceeds $\Delta j_{\max}$:

   $$
   \text{split at } k \iff \mathcal{I}[k] - \mathcal{I}[k-1] > \Delta j_{\max}
   $$

   This produces a set of contiguous index clusters $\mathcal{K}_1, \mathcal{K}_2, \dots$.

4. **Centroid and Padding**:

   For each cluster $\mathcal{K} = \{k_1, \dots, k_m\}$:

   $$
   c = \frac{\sum_{k \in \mathcal{K}} \hat{f}_S(x_k) \cdot x_k}
              {\sum_{k \in \mathcal{K}} \hat{f}_S(x_k)},\qquad
   lo = \max(x_0,\; x_{k_1} - P),\qquad
   hi = \min(x_{n_{eval}-1},\; x_{k_m} + P)
   $$

   where $P = 0.025 \cdot R$ (2.5\% of the session range).
   The peak z-score $z_{pk}$ is the maximum (for HVN) or minimum (for LVN) z-score within the cluster.

5. **Overlapping Merges**:

   Sort candidate zones by their low boundary. If any adjacent zones overlap ($lo_{\text{next}} \le hi_{\text{prev}}$), they are merged:

   - Boundaries: $lo_{\text{merged}} = \min(lo_{\text{prev}}, lo_{\text{next}})$,
     $hi_{\text{merged}} = \max(hi_{\text{prev}}, hi_{\text{next}})$.
   - Center: density-weighted average of their centers:

     $$
     c_{\text{merged}} = \frac{c_{\text{prev}} \cdot \hat{f}(c_{\text{prev}}) +
                               c_{\text{next}} \cdot \hat{f}(c_{\text{next}})}
                              {\hat{f}(c_{\text{prev}}) + \hat{f}(c_{\text{next}})}
     $$

   - Z-score: the z-score with the largest absolute value:

     $$
     z_{\text{merged}} = \text{sign}(z) \cdot \max(|z_{\text{prev}}|, |z_{\text{next}}|)
     $$

6. **Sorting and Capping**:

   Sort the merged zones in descending order of absolute z-score $|z_{pk}|$. Retain at most the top $6$ HVN zones and top $4$ LVN zones.

- **HVN (high-volume node):** an area of genuine acceptance. Once revisited, it tends to act as support or resistance because a large resting inventory of participants transacted there and will defend or add to their positions on a return.
- **LVN (low-volume node):** an area of rejection or rapid price expansion. Because the auction spent little time or volume there, it offers little resistance to price passing through quickly — an "air pocket." LVNs are the volume-profile-native counterpart of an ICT fair value gap.

#### Why index-clustering?

The old forward-scan bridge (scanning the grid left-to-right and bridging non-qualifying gaps smaller than $G$) and the new index-clustering approach are **mathematically equivalent** on a uniform grid, because:

$$
|x_{j_a} - x_{j_b}| \leq G \quad\Longleftrightarrow\quad |j_a - j_b| \leq \Delta j_{\max}
$$

The index-clustering implementation is preferred because:
- It is **vectorized** (uses `np.where` + `np.split` instead of a Python loop).
- It eliminates the complex nested while-loop that handled the forward bridge.
- It produces **identical zone centers** (within floating-point noise) while being significantly faster and easier to maintain.

### 6.3 Buy/sell profiles

In the codebase, buy-dominant and sell-dominant zones are not extracted via separate z-score clustering. Instead, `buy_density` and `sell_density` are computed in the density estimation pipeline (Section 5.3) for volume profiles, allowing direct visualization and downstream analysis of aggressor flows relative to total volume density.
*(Note: Delta pivot level calculation and delta-based zones are part of the theoretical specification but are not implemented in the current compute layer).*

### The JOINT mixture

The $JOINT$ profile is constructed by averaging the density functions of a list of completed profiles $\mathcal{S}'$ (excluding $DEV$ and $PW$) on a shared evaluation grid $x$ consisting of 1200 points spanning from the global minimum evaluation point to the global maximum evaluation point:

$$
\hat{f}_{JOINT}^{tot}(x) = \frac{1}{|\mathcal{S}'|} \sum_{s \in \mathcal{S}'} \hat{f}_s^{tot}(x)
$$

with out-of-bounds interpolation padded with zeros. The buy/sell sub-densities are averaged in the same way:

$$
\hat{f}_{JOINT}^{buy}(x) = \frac{1}{|\mathcal{S}'|} \sum_{s \in \mathcal{S}'} \hat{f}_s^{buy}(x), \qquad \hat{f}_{JOINT}^{sell}(x) = \frac{1}{|\mathcal{S}'|} \sum_{s \in \mathcal{S}'} \hat{f}_s^{sell}(x)
$$

- $POC_{JOINT}$ is the grid point $x_j$ that maximizes $\hat{f}_{JOINT}^{tot}(x)$.
- $VAL_{JOINT}$ and $VAH_{JOINT}$ are extracted as the 15th and 85th percentiles of the cumulative distribution function (CDF) of the joint profile, defining a symmetric 70\% value area:

  $$
  VAL_{JOINT} = \hat{F}_{JOINT}^{-1}(0.15), \qquad VAH_{JOINT} = \hat{F}_{JOINT}^{-1}(0.85)
  $$

Because $JOINT$ pools volume across sessions of very different width, it is used only as a reference plane — a way of asking "does the whole recent history agree," never as a tradeable zone in its own right, and it never appears as a candlestick-anchored series (it has no single time axis).

### The TPO Profile

The TPO lens $\mathcal{T}_S$ answers where the market dwelt. In the codebase, this is implemented not via 30-minute letter brackets, but as a continuous density function built through the exact same fixed-count binning and Gaussian KDE pipeline as the volume profile.
- The only difference is the bin weighting: each bar contributes a constant weight of $1.0$ (representing one time unit) distributed uniformly across all bins its high/low range touches.
- The TPO lens has no buy/sell subprofiles.
- Resulting levels and zones include $tPOC_S$ (most-visited price by time), $tVAL_S$, $tVAH_S$, and time-based HVN/LVN zones.
