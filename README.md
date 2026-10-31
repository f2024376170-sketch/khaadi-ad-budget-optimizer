# Khaadi Digital Advertising Budget Optimizer

**Course:** Big Data Analytics (Project 2)
**Student:** Arooj Fatima Malik (f2024376170)
**Related work:** Project 1, Pakistani Social Media Content Detective: Khaadi (https://github.com/f2024376170-sketch/pakistan-brand-sentiment-khaadi)

## 1. Business Question
A company spends money on Facebook, Instagram, Google, TikTok and YouTube. Management asks: **"Where should we spend the next Rs. 1 million?"**

We built a **Marketing Budget Recommendation Engine** for Khaadi that answers this from historical campaign performance, and we compared **Pandas, MapReduce and Spark** on growing data sizes.

## 2. Data (simulated)
**All data in this project is simulated. It is not real Khaadi campaign data.** A Python generator creates records with the 12 required columns:

`date, campaign, platform, impressions, clicks, spend, conversions, revenue, age, gender, city, device`

- 5 platforms, 9 Khaadi-style campaigns (Lawn Collection, Eid Collection, Winter Collection, Summer Sale, Mid-Season Sale, Independence Day Sale, Black Friday, Bridal and Festive, Khaadi Home), 8 Pakistani cities, 3 device types and 5 age groups
- Each platform has its own assumed CTR, CPM, conversion rate and average order value. Campaign, city, device, age and weekday multipliers and random noise are applied on top
- Records were generated at four sizes: **10,000, 100,000, 1,000,000 and 10,000,000**
- The repository contains the 10,000 and 100,000 record files. The larger files (about 100 MB and 1 GB) are not uploaded, but the notebook regenerates them

## 3. Metrics
| Metric | Formula |
|---|---|
| CTR | clicks / impressions |
| CPC | spend / clicks |
| CPA | spend / conversions |
| Conversion rate | conversions / clicks |
| ROAS | revenue / spend |
| Net revenue | revenue - ad spend |

Group-level metrics are computed from **totals** (sum of clicks, spend and so on, then the ratio), not as averages of row-level ratios.

## 4. Results on 100,000 records
**Overall:** CTR 1.67%, CPC Rs. 27.12, CPA Rs. 766.18, conversion rate 3.54%, **ROAS 9.27**.

| Platform | Spend (Rs. M) | Revenue (Rs. M) | CTR % | CPC (Rs.) | CPA (Rs.) | Conv. rate % | ROAS |
|---|---|---|---|---|---|---|---|
| Instagram | 437.83 | 6,007.16 | 1.59 | 26.53 | 579.56 | 4.58 | **13.72** |
| Google | 547.01 | 4,614.29 | 2.51 | 32.04 | 844.99 | 3.79 | 8.44 |
| Facebook | 318.63 | 2,530.56 | 1.36 | 28.05 | 833.92 | 3.36 | 7.94 |
| TikTok | 126.88 | 889.95 | 1.94 | 13.51 | 694.70 | 1.94 | 7.01 |
| YouTube | 130.27 | 425.12 | 0.79 | 40.49 | 1,878.27 | 2.16 | 3.26 |

- **Instagram** has the best ROAS because it combines the highest conversion rate (4.58%) with the lowest CPA (Rs. 579.56) and a high order value.
- **Google** has the highest CTR but costlier clicks, so its ROAS is lower than Instagram's.
- **TikTok** has the cheapest clicks (Rs. 13.51) but a low conversion rate and order value.
- **YouTube** is the weakest platform: lowest CTR, highest CPA and a ROAS of 3.26.

**Other breakdowns (ROAS):**
- **Campaigns:** Eid Collection 12.78, Lawn Collection 11.79, Black Friday 10.94, Summer Sale 10.26, down to Khaadi Home at 4.20 (the weakest).
- **Cities:** Islamabad 10.41, Lahore 9.85, Karachi 9.40, down to Quetta at 7.15.
- **Devices:** Desktop 11.21, Tablet 9.42, Mobile 8.89. Mobile has the lowest ROAS even though it carries most of the traffic.
- Charts for platform, spend versus revenue share, CPA, city, weekday and monthly trend are in the notebook.

## 5. Big Data Comparison: Pandas vs MapReduce vs Spark
The same task was run with each tool: total impressions, clicks, spend, conversions and revenue per platform. All three return identical results (verified on the 10,000 record file). Times in seconds, one run each:

| Records | Pandas | MapReduce | Spark |
|---|---|---|---|
| 10,000 | **0.03** | 0.07 | 1.76 |
| 100,000 | **0.41** | 1.32 | 1.36 |
| 1,000,000 | **1.42** | 4.43 | 5.02 |
| 10,000,000 | **16.04** | 50.10 | 27.99 |

- **Pandas** was fastest at every size on this single machine.
- At 10,000,000 records **Spark was about 1.8x faster than MapReduce**, and Pandas was about 1.7x faster than Spark.
- When the data grew 1,000x (10,000 to 10,000,000 records), Spark's time grew only about 16x, while MapReduce grew roughly 700x and Pandas roughly 500x (small timings are rounded, so these multiples are approximate). Spark's fixed start-up cost dominates small data, but its time rises slowly. This suggests it would overtake Pandas at larger sizes or on a cluster, but we did not test that.
- **Implementations:** Pandas uses `read_csv` and `groupby`. MapReduce is a pure-Python implementation of the map, combine, shuffle and reduce phases, processing the file in chunks. Spark uses PySpark `groupBy` in local mode.

## 6. Marketing Budget Recommendation Engine
**Method**
1. Historical ROAS per platform is computed from the 1,000,000 record file.
2. Diminishing returns are modelled with a concave response curve: revenue = ROAS x baseline spend x (new spend / baseline spend)^0.5.
3. An optimizer (SLSQP) finds the split that maximizes expected revenue, with each platform limited to **5% minimum and 40% maximum** of the budget.
4. **Link with Project 1:** in Project 1, Khaadi YouTube comments were 42% positive and 22% negative. This raises YouTube's ROAS slightly (3.25 to 3.41, about +5%).

**Answer: where should the next Rs. 10 lakh go?**

| Platform | Current share | Recommended share | Budget (Rs.) |
|---|---|---|---|
| Instagram | 27.8% | 40% | 400,000 |
| Google | 35.5% | 33% | 330,000 |
| Facebook | 20.2% | 17% | 170,000 |
| TikTok | 8.0% | 5% | 50,000 |
| YouTube | 8.5% | 5% | 50,000 |

**Expected revenue from Rs. 10 lakh (model-based):**

| Strategy | Expected revenue (Rs.) |
|---|---|
| Equal split (20% each) | 8,401,487 |
| Current split | 9,249,506 |
| **Recommended split** | **9,582,185** |

The recommended split earns about **14.1% more than an equal split** and **3.6% more than the current split**. The main move is shifting money to Instagram, which reaches the 40% cap, from Google, Facebook, TikTok and YouTube.

**Sensitivity check:** the diminishing-returns exponent is an assumption, so we tested it.

| Platform | Exponent 0.3 | Exponent 0.5 | Exponent 0.7 |
|---|---|---|---|
| Instagram | 40.0% | 40.0% | 40.0% |
| Google | 32.3% | 33.0% | 34.0% |
| Facebook | 17.0% | 16.8% | 16.0% |
| TikTok | 5.7% | 5.3% | 5.0% |
| YouTube | 5.0% | 5.0% | 5.0% |

No platform's share changes by more than 1.7 percentage points, so the recommendation is stable. **Without the sentiment adjustment the split is identical**, because YouTube is already at the 5% minimum. The Project 1 link is built into the engine, but it does not change the final split on this data.

## 7. Limitations
- **The data is simulated.** Platform differences (for example Instagram being best) come from the assumptions inside the generator. This project demonstrates the pipeline and method, not real market facts. Real campaign exports should be used before spending real money.
- The 40% cap and 5% floor are business-style rules chosen by us. Instagram sits exactly at the cap, so a higher cap would move more money to it.
- The diminishing-returns exponent is an assumption, not estimated from data (see the sensitivity table).
- The benchmark ran on one Colab machine with one run per size, so times will vary. Spark was not tested on a cluster, and MapReduce is a Python implementation, not Hadoop.
- Net revenue means revenue minus ad spend, not accounting profit.
- Summary tables use 100,000 records and the recommendation uses 1,000,000 records, so small differences between them are expected.
- Absolute rupee values (and a ROAS above 9) are not realistic because simulated impression counts are large. Compare platforms by ratios, not raw totals.

## 8. Files
- `khaadi_ad_budget_optimizer.ipynb`: full notebook (generator, metrics, charts, benchmark, recommendation engine)
- `ads_10000.csv`, `ads_100000.csv`: sample simulated datasets
- `platform_kpi.csv`, `campaign_kpi.csv`: KPI tables
- `benchmark_results.csv`: Pandas vs MapReduce vs Spark timings
- `budget_recommendation.csv`: recommended budget split

## 9. Tech Stack
Python, Google Colab, pandas, NumPy, SciPy, PySpark, matplotlib

## 10. 
- Arooj Fatima Malik (f2024376170)
- Course: Big Data Analytics
