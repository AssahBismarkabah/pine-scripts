The logic for the proprietary ATS Trend Indicator combines the two distinct inefficiency models—the **3-Candle Separation (Fair Value Gap)** and the **Grouped Candle Separation (Cluster Lows/Highs)**—into a single, dynamic line that serves as the ultimate trailing stop for the directional bias.

Since the original code is proprietary, you can implement your own version using the following logic, which prioritizes the tighter, more recent institutional footprint.

-----

## Algorithmic Logic for a Combined Inefficiency Trend Tracker

This logic defines a single variable, `TrendLine`, which is constantly updated to the most conservative (widest) or most recent (tightest) valid support/resistance level, depending on the current directional bias.

### **1. Define Core Concepts**

| Concept | Bullish Logic (Support/Demand) | Bearish Logic (Resistance/Supply) |
| :--- | :--- | :--- |
| **3-Candle Inefficiency (FVG)** | Bullish FVG exists if $\text{Low}[C3] > \text{High}[C1]$, where $C1, C2, C3$ are three consecutive candles. The level is $\text{High}[C1]$. | Bearish FVG exists if $\text{High}[C3] < \text{Low}[C1]$. The level is $\text{Low}[C1]$. |
| **Group Inefficiency (Cluster)** | Last **Valid Pivot Low** during the impulsive move. This is the low of the shallowest retracement that occurred before the final acceleration. (Requires a lookback window, e.g., 5-10 bars). | Last **Valid Pivot High** during the impulsive move. |
| **Trend Direction** | `TrendIsUp` (Price closes above the previous `TrendLine`). | `TrendIsDown` (Price closes below the previous `TrendLine`). |

\<hr\>

### **2. Pseudo-Code Implementation**

The core loop for the indicator on each new bar should follow this order:

#### **A. Initial Trend Line Calculation**

1.  **Start:** Initialize `CurrentTrendLine` to a default value (e.g., the 20-period Exponential Moving Average, or the previous close).

2.  **Determine Primary Bias:**

      * **If Bullish FVG is detected:**
          * Set `FVG_Support` to the FVG level ($\text{High}[C1]$).
          * Set `TrendDirection` = **Up**.
      * **If Bearish FVG is detected:**
          * Set `FVG_Resistance` to the FVG level ($\text{Low}[C1]$).
          * **Set `TrendDirection` = Down.**

#### **B. Trailing and Prioritization (The Key Logic)**

This is where the Group Inefficiency is integrated to allow the line to trail aggressively.

1.  **If `TrendDirection` is Up (Long Bias):**

      * **Find New Cluster Low:** Calculate the **highest valid pivot low** (`ClusterSupport`) formed within the last N bars (e.g., $N=8$). This captures the last shallow pullback of the "grouped" order flow.
      * **Update `TrendLine`:** The indicator must choose the **tightest** support that is still valid.
          * If a new `FVG_Support` is available, **OR** if the `ClusterSupport` is **higher** than the current `TrendLine`:
            $$
            $$$$\\text{TrendLine} = \\text{MAX}(\\text{Previous TrendLine}, \\text{FVG\_Support}, \\text{ClusterSupport})
            $$
            $$$$
            $$
          * *Logic: The line only moves up, following the highest recent valid support (either the hard FVG line or the tighter Cluster line).*

2.  **If `TrendDirection` is Down (Short Bias):**

      * **Find New Cluster High:** Calculate the **lowest valid pivot high** (`ClusterResistance`) formed within the last N bars.
      * **Update `TrendLine`:** The indicator must choose the **tightest** resistance that is still valid.
          * If a new `FVG_Resistance` is available, **OR** if the `ClusterResistance` is **lower** than the current `TrendLine`:
            $$
            $$$$\\text{TrendLine} = \\text{MIN}(\\text{Previous TrendLine}, \\text{FVG\_Resistance}, \\text{ClusterResistance})
            $$
            $$$$
            $$
          * *Logic: The line only moves down, following the lowest recent valid resistance (either the hard FVG line or the tighter Cluster line).*

#### **C. Invalidation (Trend Flip)**

1.  **Check for Break:**
      * **If `TrendDirection` is Up** AND $\text{Close} < \text{TrendLine}$:
          * **Flip:** Set `TrendDirection` = **Down**.
          * Set the new `TrendLine` to the current high, or the new FVG resistance.
      * **If `TrendDirection` is Down** AND $\text{Close} > \text{TrendLine}$:
          * **Flip:** Set `TrendDirection` = **Up**.
          * Set the new `TrendLine` to the current low, or the new FVG support.

This structure allows the **3-Candle FVG** to establish the primary market inefficiency, while the **Grouped Candle logic** is used to trail the price aggressively, resulting in a single, smooth, dynamic line that represents the evolving line of institutional integrity.

-----

For a deeper dive into the logic of algorithmic trading systems and their components, you can watch this video:
[Automate Fair Value Gap Detection in Python for Algorithmic Trading](https://www.youtube.com/watch?v=cjDgibEkJ_M)
This video explains how to automate the detection of Fair Value Gaps, which forms the foundation of the 3-Candle Separation logic.