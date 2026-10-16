# Memory Demand Forecast & Accuracy

An end-to-end demand-planning analysis for the memory semiconductor industry: forecasting US semiconductor production, backtesting forecast accuracy the way demand planners measure it, and breaking down Kioxia's revenue into price and volume to show what actually drives the memory cycle.

**[Open the interactive Tableau dashboard](https://public.tableau.com/views/MemoryDemandForecastAccuracyDashboard_/Dashboard1)** · [Open the notebook in Colab](https://colab.research.google.com/github/sabrinachu-47/memory-demand-forecast/blob/main/notebooks/memory_demand_forecast.ipynb)

[![Dashboard preview: click to open the interactive version](images/dashboard.png)](https://public.tableau.com/views/MemoryDemandForecastAccuracyDashboard_/Dashboard1)

## The question

Demand planners need to know two things about any forecast: **how accurate it is**, and **whether it is consistently too high or too low**. This project asks:

1. How accurate are standard forecasting methods for semiconductor demand, and how quickly does accuracy fall off the further ahead we forecast?
2. Do forecasts lean in one direction (bias), and when?
3. In memory, is revenue growth driven by selling more (volume) or by charging more (price)?

## Data

- **US semiconductor production index**, monthly, 2010 to August 2026, from the Federal Reserve (FRED series [IPG3344S](https://fred.stlouisfed.org/series/IPG3344S), seasonally adjusted, and [IPG3344N](https://fred.stlouisfed.org/series/IPG3344N), not seasonally adjusted).
- **Kioxia quarterly results**, FY2020 Q1 to FY2026 Q1 (25 quarters of revenue), plus quarter-over-quarter bit growth and average selling price (ASP) for 13 quarters, compiled from Kioxia's [investor relations](https://www.kioxia-holdings.com/en-jp/ir.html) earnings materials. Each figure's source is recorded in `data/kioxia_quarterly.csv`.

## Method

1. **Explore:** year-over-year growth to reveal cycles, and average month-over-month change by calendar month to reveal seasonality.
2. **Baseline forecasts:** naive, seasonal naive, and seasonal naive with growth, tested on a 24-month holdout (Sep 2024 to Aug 2026).
3. **Exponential smoothing (ETS):** additive, multiplicative, and damped-trend versions, tested on the same holdout.
4. **Rolling backtest:** re-forecast every month from August 2022 onward, refitting each time on the data available then, and forecasting 1 to 12 months ahead. This produced **1,530 scored forecasts**, evaluated with **WMAPE** (total absolute error ÷ total actual) and **bias** (total signed error ÷ total actual).
5. **Price/volume split:** converted Kioxia's disclosed ranges (e.g., "low-teens % increase") into numbers, built a volume index from bit growth, and derived price per bit as revenue ÷ volume. Validated the result against Kioxia's own stated ASP changes.

## Key findings

- **Accuracy drops sharply with forecast distance.** The best model (ETS with additive seasonality) had a WMAPE of **1.8% one month ahead and 5.2% twelve months ahead**, nearly three times higher. Overall WMAPE across all horizons was 3.4%.
- **Forecasts ran consistently low, and more so further out.** Bias was near zero one month ahead (−0.4%) but reached **−4.2% at twelve months**, because production growth accelerated in 2025–26 faster than the models expected.
- **Bias changes direction at turning points.** Twelve-month-ahead forecasts **over-forecast in 2023** (made before the memory downturn) and **under-forecast from 2024 onward** (made before the upswing). Long-range forecasts lag the cycle in both directions.
- **The best model depends on the horizon.** ETS was most accurate for 1 to 9 months ahead. A simpler seasonal-naive-with-growth model caught up at 10 to 12 months and had the smallest bias at almost every horizon.
- **Kioxia's revenue growth was mostly price, not volume.** From Jan–Mar 2023 to Apr–Jun 2026, revenue grew **7.2x**, price per bit **4.6x**, and volume only **1.6x**. Roughly three-quarters of the growth came from price. In the two most recent quarters, volume fell while revenue tripled, the signature of a supply shortage.
- **Memory cycles show up in prices, not volumes.** The US production index rose a steady 38% over the same period and showed no sign of the boom, which is why cycles look muted in volume data.
- **Production has strong quarter-end seasonality.** Output jumps in the last month of each quarter (March, June, September, December) and drops in the first month.

## What it means for demand planning

- **Judge forecasts by horizon, not by a single number.** A 3% overall error hides the difference between near-term accuracy (good enough for production scheduling) and long-range accuracy (much weaker, but that is where wafer-start and capacity decisions are made).
- **Track bias alongside accuracy.** Consistent under-forecasting during an upswing means too few wafer starts and customers on allocation; over-forecasting into a downturn means excess inventory losing value as prices fall.
- **Use different models for different jobs.** A responsive model suits near-term planning; a steadier one may suit longer-range planning.
- **Plan volume and price separately.** Volume-driven growth calls for more supply. Price-driven growth signals demand exceeding supply and a need to allocate limited product.
- **Compare quarters or year-over-year, not month-to-month.** Quarter-end loading makes single-month swings misleading.

## Limitations and next steps

- The production index covers all US semiconductors, not memory alone, and is a national aggregate. Product- and customer-level demand is far noisier and would show much higher forecast error.
- Kioxia's bit growth and ASP are disclosed as ranges, so the price/volume split is approximate. ASP is reported in yen through FY2024 and in US dollars from FY2025, so currency moves affect comparisons (most visibly in FY25 Q1).
- The backtest covers August 2022 onward; long-horizon results rest on fewer forecasts than short-horizon ones.
- **Next steps with real company data:** forecast at the product and customer level, measure **forecast value added** (whether planner adjustments improve on the statistical forecast), forecast price and volume separately, and add leading indicators such as customer inventory levels and contract prices.

## Repository contents

- `notebooks/` – Python analysis (Google Colab), with notes for each step
- `data/` – input and output data files
- `tableau/` – Tableau workbook (.twbx)
- `images/` – dashboard preview

## Tools

Python (pandas, NumPy, statsmodels, matplotlib), Google Colab, Tableau Public
