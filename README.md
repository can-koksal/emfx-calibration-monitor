# TRY FX Forecast Calibration Monitor

A client-side tool that calibrates USD/TRY and EUR/TRY 12-month forecast paths by blending survey-based expectations, historical bias correction, realized-trend extrapolation, and a discrete-shock overlay with automated regression checks to guard against calculation drift.

**https://can-koksal.github.io/emfx-calibration-monitor/(#)** 
Independent research prototype. Not affiliated with any bank or financial institution. Not a trading recommendation.

---

## What it does

Given a "raw" expected FX path (1M/2M/3M/5M/6M/12M) for a currency pair, the tool produces a **blended forecast** using four components:

| Component | What it captures |
|---|---|
| **Raw expectation** | The unadjusted survey/forecast input as entered |
| **Bias-adjusted** | Raw expectation corrected using the average historical forecast error from a user-supplied database of past forecast-vs-actual cases |
| **Trend-adjusted** | An extrapolation of the currency's own realized depreciation trend (annualized from 1/3/6-month trailing spot moves) |
| **Shock-adjusted** | Raw expectation plus a probability-weighted overlay for discrete policy/political shocks, sized from historical shock episodes |

The four components are combined via user-adjustable weights (auto-normalized to 100%) into a single **Final Blended Forecast**.

## Why it exists

Turkish Central Bank (TCMB) surveys only ever publish two data points: current-year-end expectation and 12-month-ahead expectation never the intermediate horizons. This tool fills that gap with disclosed, labeled methodologies (linear interpolation, cross-rate derivation) rather than pretending intermediate forecasts exist, and separately tries to answer a harder question: does correcting for historical survey bias actually improve forecast accuracy, and by how much?

## Validation

The bias-adjustment component was tested via **leave-one-out cross-validation** across 9 non-shock historical windows spanning June 2022–June 2025 (TCMB survey data cross-checked against Turkish financial press; realized rates verified against daily market closes):

- Raw expectation alone: **4.38%** average absolute forecast error
- Bias-adjusted (out-of-sample): **4.06%** average absolute forecast error
- **Net improvement: +0.32 percentage points**

This is a modest, real, honestly-measured edge not a dramatic one. The correction helps meaningfully in stable-regime periods (2024–2025) and actively underperforms around regime discontinuities (the 2022 pre-orthodoxy crisis period), which is precisely why a **separate shock-scenario overlay** exists rather than folding tail risk into the same smooth correction. A single historical episode (the mid-2023 policy pivot, embedded in the December 2022 window) currently anchors the shock component; this is a real limitation of the available sample, not a design flaw, and is disclosed as such in the app's Methodology tab.

The trend and shock components have not yet been validated with the same leave-one-out rigor as the bias component trend requires trailing spot data from before each historical T0 (not yet sourced for all windows), and shock validation requires more than the single historical shock episode currently available.

## Data sources

- **TCMB Survey of Market Participants** (Piyasa Katılımcıları Anketi) year-end and 12-month USD/TRY expectations, cross-checked against NTV, Bigpara, Investing.com Türkiye, and Akbank's investor-relations mirror of the official TCMB report
- **Historical spot/close data** poundsterlinglive.com daily historical series, nearest-trading-day convention applied when an exact date fell on a weekend
- EUR/TRY expectations are cross-rate derived (TCMB USD/TRY expectation × EUR/USD assumption) since TCMB does not survey EUR/TRY directly

## Known limitations

- TCMB does not publish sub-12-month forecasts; all 1M–6M "expected" values are linear interpolations anchored to two real data points, clearly labeled as such
- The historical case database currently spans 10 T0 windows (2022–2025) a modest sample for a systematic bias estimate
- Only one flagged shock episode exists in the current dataset, limiting the shock component's own validation
- Bias/trend for EUR/TRY are deliberately redirected to reuse USD/TRY's values, based on testing that showed EUR/TRY's own historical bias was almost entirely explained by EUR/USD path drift rather than independent lira-forecasting error

## Running it

No build step, no dependencies. Download `index.html` and open it in any modern browser. All state persists to browser `localStorage`; use "Clear Saved Data" to reset, or "Load Demo Cases" to reload the built-in historical dataset.

## Tech notes

Single-file vanilla HTML/CSS/JS. Includes a built-in regression sanity check (`regressionSanityCheck()`) that runs on every render and flags known failure patterns (e.g., EUR/TRY bias or trend silently diverging from USD/TRY, or the blended forecast collapsing to the raw input). A visible build marker at the top of the page identifies which version is currently running.

## License

MIT see [LICENSE](LICENSE).
