# TRY FX Forecast Calibration Monitor

A client-side tool that calibrates USD/TRY and EUR/TRY 12-month forecast paths. It blends survey-based expectations, historical bias correction, realized-trend extrapolation, and a discrete-shock overlay, with automated checks that catch calculation drift.

[Open the live app](https://can-koksal.github.io/emfx-calibration-monitor/)

Independent research prototype. Not affiliated with any bank or financial institution. Not a trading recommendation.

---

## What it does

Given a raw expected FX path (1M, 2M, 3M, 5M, 6M, 12M) for a currency pair, the tool produces a blended forecast using four components:

| Component | What it captures |
|---|---|
| Raw expectation | The unadjusted survey or forecast input as entered |
| Bias-adjusted | Raw expectation corrected using the average historical forecast error from a user-supplied database of past forecast-vs-actual cases |
| Trend-adjusted | An extrapolation of the currency's own realized depreciation trend, annualized from 1, 3, and 6-month trailing spot moves |
| Shock-adjusted | Raw expectation plus a probability-weighted overlay for discrete policy or political shocks, sized from historical shock episodes |

The four components combine through user-adjustable weights (auto-normalized to 100%) into one final blended forecast.

## Why it exists

TCMB's Survey of Market Participants only publishes two data points: the current-year-end expectation and the 12-month-ahead expectation. It never publishes the intermediate horizons. This tool fills that gap with disclosed, labeled methods (linear interpolation, cross-rate derivation) instead of pretending intermediate forecasts exist. It also tries to answer a harder question: does correcting for historical survey bias actually improve forecast accuracy, and by how much.

## Validation

The bias-adjustment component was tested with leave-one-out cross-validation across 9 non-shock historical windows from June 2022 to June 2025. TCMB survey data was cross-checked against Turkish financial press, and realized rates were verified against daily market closes.

- Raw expectation alone: 4.38% average absolute forecast error
- Bias-adjusted, out-of-sample: 4.06% average absolute forecast error
- Net improvement: 0.32 percentage points

This is a modest, real, honestly measured edge, not a dramatic one. The correction helps in stable-regime periods (2024-2025) and underperforms around regime discontinuities, like the 2022 pre-orthodoxy crisis period. That is exactly why a separate shock-scenario overlay exists instead of folding tail risk into the same smooth correction. One historical episode, the mid-2023 policy pivot embedded in the December 2022 window, currently anchors the shock component. This is a real limitation of the available sample, not a design flaw, and it is disclosed as such in the app's Methodology tab.

The trend and shock components have not been validated with the same leave-one-out rigor as the bias component. Trend requires trailing spot data from before each historical T0, which has not been sourced for all windows yet. Shock validation needs more than the one historical shock episode currently available.

## Data sources

- TCMB Survey of Market Participants (Piyasa Katilimcilari Anketi): year-end and 12-month USD/TRY expectations, cross-checked against NTV, Bigpara, Investing.com Turkiye, and Akbank's investor-relations mirror of the official TCMB report
- Historical spot and close data: poundsterlinglive.com daily historical series, using the nearest trading day when an exact date fell on a weekend
- EUR/TRY expectations are cross-rate derived: TCMB's USD/TRY expectation multiplied by an EUR/USD assumption, since TCMB does not survey EUR/TRY directly

## Known limitations

- TCMB does not publish sub-12-month forecasts. All 1M to 6M expected values are linear interpolations anchored to two real data points, clearly labeled as such
- The historical case database currently spans 10 T0 windows from 2022 to 2025, a modest sample for a systematic bias estimate
- Only one flagged shock episode exists in the current dataset, which limits how well the shock component itself can be validated
- Bias and trend for EUR/TRY are deliberately redirected to reuse USD/TRY's values. Testing showed EUR/TRY's own historical bias was almost entirely explained by EUR/USD path drift, not independent lira-forecasting error

## Running it

No build step, no dependencies. Download index.html and open it in any modern browser. All state saves to browser localStorage. Use "Clear Saved Data" to reset, or "Load Demo Cases" to reload the built-in historical dataset.

## Tech notes

Single-file vanilla HTML, CSS, and JS. Includes a built-in regression check that runs on every render and flags known failure patterns, like EUR/TRY bias or trend silently diverging from USD/TRY, or the blended forecast collapsing to the raw input. A build marker at the top of the page identifies which version is currently running.

## License

MIT, see LICENSE.
