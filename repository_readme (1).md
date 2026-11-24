# 🏏 SupeRR Selector Season 2: Rajasthan Royals IPL 2027 Auction Strategy

**An Empirical Bayes & Pre-Delivery Leverage Engine for Evaluating Clutch T20 Performance**

---

## 📌 Executive Summary & Problem Statement

In T20 franchise cricket, traditional evaluation metrics—such as gross batting strike rates and conventional bowling averages—are systematically misleading for auction scouting. They treat a boundary hit during a low-stress powerplay identically to a boundary hit during a last-over chase with eight wickets down.

**Our Solution:** Built for the **Rajasthan Royals SupeRR Selector Season 2** analytics challenge, this repository implements the **P.U.L.S.E. (Pressure Utility & Leverage Statistical Evaluation)** framework. Auditing **192,675+ delivery records (2022–2026)** across IPL and elite T20I circuits, our pipeline isolates true high-leverage deliveries, removes small-sample noise via Empirical Bayes shrinkage, and identifies the exact Top 10 Batters and Top 10 Bowlers built to win high-stakes matches at Sawai Mansingh Stadium.

---

## 🏗️ End-to-End Pipeline Architecture

```text
[Raw Delivery Logs (192k+ Balls)] 
       │
       ▼
[1. Data Hygiene & Schema Unmasking] ──► Fixes extras1_runs ID trap & excludes run-outs from bowler wickets
       │
       ▼
[2. Pre-Delivery Pressure Index (PI)] ──► Continuous Sigmoid Model (PI ∈ [0, 1]) isolating top 10.4% stress balls
       │
       ▼
[3. Context-Adjusted Net Impact]      ──► Benchmarks every ball against (Innings × Over × Wicket-Bucket) baselines
       │
       ▼
[4. Empirical Bayes & LCB Shrinkage]  ──► Smooths small samples (K=110/120) & penalizes game-to-game variance
       │
       ▼
[5. Composite PRI Score (0–100)]      ──► Final Ranked Top 10 Batters & Top 10 Bowlers + Squad Fit Blueprint
```

---

## ⚙️ Mathematical Methodology & Formulas

### 1. Continuous Pre-Delivery Pressure Index

Rather than relying on arbitrary over cutoffs (e.g., `over >= 16`), we compute a continuous pre-delivery Pressure Index ($PI \in [0, 1]$) before every ball is bowled.

**Second Innings (Chasing Pressure):**

$$
PI_{\text{inn2}} = \left[ 0.48 \cdot \sigma\left(0.42 \cdot (\text{RRR} - \text{RR}_{\text{par}})\right) + 0.32 \cdot \left(\frac{W_{\text{lost}}}{9}\right)^{1.25} + 0.20 \cdot e^{-\frac{\text{BallsRem}}{42}} \right] \cdot \mathcal{C}_{\text{alive}}
$$

* **Required Run Rate Stress:** $\sigma(z) = \frac{1}{1 + e^{-z}}$ maps the deficit between Required Run Rate ($\text{RRR}$) and venue-phase par scoring rate ($\text{RR}_{\text{par}}$).
* **Resource Depletion:** $\left(\frac{W_{\text{lost}}}{9}\right)^{1.25}$ captures the non-linear acceleration of panic as wickets fall.
* **Contest Closeness Filter ($\mathcal{C}_{\text{alive}}$):** Dampens mathematically dead matches ($\text{RRR} > 18.5$) so uncompetitive blowouts do not distort clutch metrics.

**First Innings (Setting / Rebuilding Pressure):**

$$
PI_{\text{inn1}} = 0.45 \cdot \sigma\left(0.95 \cdot (W_{\text{lost}} - \mathbb{E}[W])\right) + 0.25 \cdot \sigma\left(0.08 \cdot (\mathbb{E}[\text{Runs}] - \text{Runs}_{\text{actual}})\right) + 0.30 \cdot \sigma\left(0.55 \cdot (\text{OverID} - 15)\right)
$$

* **Empirical Validation:** Deliveries flagged in the High-Pressure zone ($PI \ge 0.65$, representing the top 10.4% most leveraged balls) exhibit a **74.2% relative surge in dismissal probability** (rising from 5.03% to 8.76%) alongside tighter scoring rates.

---

### 2. Context-Adjusted Net Impact ($xR, xW$)

To ensure players are not rewarded for flat pitches or weak bowling, every ball is evaluated against non-parametric empirical expectations ($xR$ and $xW$) derived from its exact match state bucket: $\text{State} = (\text{Innings}, \text{OverID}, \text{Wickets-Lost Bucket})$.

**Batting Net Impact per Ball:**

$$
\text{Bat Impact}_i = w_{\text{comp}} \cdot \left[ (R_i - xR_{\text{bat}, i}) - 18.0 \cdot (D_i - xW_{\text{bat}, i}) \right]
$$

**Bowling Net Impact per Ball:**

$$
\text{Bowl Impact}_i = w_{\text{comp}} \cdot \left[ (xR_{\text{bowl}, i} - R_{\text{total}, i}) + (18.0 \cdot m_{\text{order}}) \cdot (W_i - xW_{\text{bowl}, i}) \right]
$$

* **Wicket Valuation:** Calibrated at $18.0$ runs per wicket, with a top-order multiplier $m_{\text{order}} = 1.15$ for dismissing top-6 batters.
* **Competition Weight ($w_{\text{comp}}$):** IPL deliveries weighted at $1.00$; T20I deliveries weighted at $0.90$.

---

### 3. Empirical Bayes Shrinkage & Pressure Reliability Index (PRI)

To eliminate small-sample outliers (e.g., a player scoring 30 runs off 10 lucky balls), raw high-pressure impact is shrunk toward the global prior mean $\mu_{\text{prior}}$:

$$
\text{Shrunk Impact} = \left(\frac{n}{n + K}\right) \cdot \text{Raw Impact} + \left(\frac{K}{n + K}\right) \cdot \mu_{\text{prior}}
$$

* **Smoothing Constants:** $K = 110$ balls for batters and $K = 120$ balls for bowlers.
* **Lower Confidence Bound (LCB):** Penalizes erratic match-to-match variance using standard error across high-pressure appearances:

$$
\text{LCB} = \text{Shrunk Impact} - 0.30 \cdot \text{SE}_{\text{match}}
$$

* **Final Composite PRI Score (Scaled 0–100):**

$$
\text{PRI} = 0.50 \cdot (\text{Execution}) + 0.25 \cdot (\text{Pressure Lift}) + 0.25 \cdot (\text{Match Consistency and LCB})
$$

---

## 🏆 Final Ranked Shortlists & Strategic Solution

By filtering for hard sample minimums ($\ge 100$ HP balls for batters, $\ge 120$ HP balls for bowlers, and $\ge 10$ HP matches), our model solves Rajasthan Royals' core auction bottlenecks: **finisher reliability**, **domestic middle-order depth**, and **middle-over spin control at Sawai Mansingh Stadium**.

### Top 10 Batters Under Pressure

| Rank | Batter | HP Matches | HP Balls | HP Strike Rate | Win-Par Consistency | PRI Score | Strategic Squad Role |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **1** | **Tristan Stubbs** | 16 | 167 | 201.8 | 87.5% | **100.00** | Elite Overseas Death Finisher |
| **2** | **Tim David** | 28 | 238 | 185.3 | 78.6% | **92.84** | High-Volume Boundary Hitter |
| **3** | **Ayush Badoni** | 13 | 115 | 178.3 | 76.9% | **89.45** | Domestic Middle-Order Anchor |
| **4** | **Namboori Tilak Varma** | 17 | 131 | 201.5 | 64.7% | **86.44** | Left-Handed Collapse Absorber |
| **5** | **Jaker Ali Anik** | 11 | 141 | 186.5 | 63.6% | **85.07** | Lower-Order Pace Destroyer |
| **6** | **Ravindra Jadeja** | 26 | 231 | 154.5 | 73.1% | **83.51** | Proven Clutch All-Rounder (+25.6 SR Lift) |
| **7** | **Rinku Singh** | 20 | 187 | 166.8 | 65.0% | **82.60** | Specialist Chase Closer |
| **8** | **Shashank Singh** | 16 | 129 | 158.1 | 75.0% | **82.33** | High-Consistency Indian Finisher |
| **9** | **Romario Shepherd** | 35 | 295 | 151.2 | 68.6% | **79.52** | Heavy-Ball Death Power-Hitter |
| **10** | **Ashutosh Sharma** | 20 | 156 | 201.9 | 55.0% | **77.73** | High-Ceiling Impact Sub |

### Top 10 Bowlers Under Pressure

| Rank | Bowler | HP Matches | HP Balls | HP Economy | Win-Par Consistency | PRI Score | Strategic Squad Role |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **1** | **Rashid Khan** | 46 | 350 | 6.82 | 60.9% | **99.42** | Premier All-Phase Wicket-Taker |
| **2** | **Kuldeep Yadav** | 28 | 219 | 5.86 | 42.9% | **88.47** | Middle-Over Choke & Strike Spinner |
| **3** | **Wanindu Hasaranga** | 29 | 207 | 8.00 | 51.7% | **86.22** | Jaipur Large-Boundary Leg-Spinner |
| **4** | **Varun Chakravarthy** | 40 | 262 | 6.92 | 60.0% | **82.63** | High-Deception Mystery Spinner |
| **5** | **Jason Holder** | 36 | 230 | 8.17 | 55.6% | **82.61** | Slower-Bouncer Death Specialist |
| **6** | **Shaheen Afridi** | 24 | 167 | 6.54 | 54.2% | **79.35** | Left-Arm Yorker & Powerplay Threat |
| **7** | **Adam Zampa** | 21 | 144 | 7.42 | 57.1% | **75.39** | Overseas Wrist-Spin Backup |
| **8** | **Yuzvendra Chahal** | 30 | 201 | 7.01 | 50.0% | **73.62** | Proven Jaipur Death-Overs Leggie |
| **9** | **Jasprit Bumrah** | 33 | 233 | 6.41 | 45.5% | **65.01** | Gold-Standard Run Suppressor |
| **10** | **Sunil Narine** | 18 | 130 | 5.12 | 50.0% | **64.48** | Ultra-Economical Defensive Control |

---

## 📊 Repository Contents & Visual Outputs

* `notebooks/pressure_ranking_engine.ipynb`: Complete Python implementation of the P.U.L.S.E. pipeline.
* `SupeRR_Selector_Outputs/01_top10_rankings_pri.png`: Composite Pressure Reliability Index (PRI) bar rankings and match win-par consistency rates.
* `SupeRR_Selector_Outputs/02_baseline_vs_pressure_quadrant.png`: Normal-state vs. High-pressure net impact per ball quadrant scatter plot.
* `SupeRR_Selector_Outputs/*.csv`: Aggregated Top 25 rankings, pressure validation metrics, and tactical stroke/delivery DNA tables.

*(Note: Raw competition CSV datasets are strictly excluded in compliance with Rajasthan Royals & Unstop data confidentiality guidelines.)*