# Report corrections — "Proactive Bitrate Adaptation for Resilient Cloud Gaming over Unstable 5G Networks"

Each item gives the page (printed page number), the current text, and the replacement.
Every number below was checked against the dataset with `CloudGaming_Full_Analysis.ipynb`.

---

## 1. Fig. 4 is a driving trace, not a walking trace — page 9

`run_num` restarts for each mobility mode, and `run_num == 1` exists only for **driving (clockwise)** — all 230 rows.

- **Figure:** replace the image with `figures/fig4_throughput_run1_driving.png` (same data, title corrected).
- **Caption — current:** "Raw 5G downlink throughput measured over time for a single walking scenario trace from the LUMOS dataset, …"
- **Caption — replace with:** "Raw 5G downlink throughput measured over time for a single driving scenario trace (Run 1, clockwise) from the LUMOS dataset, illustrating the volatile and unpredictable nature of commercial mmWave 5G signals."

## 2. Dataset location — page 9, Section 3.3.1

The dataset's own README says Lumos5G v1.0 covers only the "Loop" area. The other two sites belong to the full Lumos5G paper, not to this CSV.

- **Current:** "…collected from Verizon's mmWave 5G network in three locations in Minneapolis, Minnesota: a four-way traffic intersection, a mall area inside Minneapolis-St. Paul International Airport, and a 1,300-meter driving loop near U.S. Bank Stadium [10]."
- **Replace with:** "…collected from Verizon's mmWave 5G network along the "Loop" area, a 1,300-meter loop near U.S. Bank Stadium in downtown Minneapolis, Minnesota, covering roads, railroad crossings, restaurants, coffee shops and outdoor parks [13]."

## 3. The dataset is cited as the wrong paper — pages 2, 8, 9, 20

Reference [10] is "Proactive SFC Provisioning with Forecast-Driven DRL in Data Centers". The Lumos5G paper is not in the reference list.

- **Add to References (page 20):**
  > [13] A. Narayanan, E. Ramadan, R. Mehta, X. Hu, Q. Liu, R. A. K. Fezeu, U. K. Dayalan, S. Verma, P. Ji, T. Li, F. Qian, and Z.-L. Zhang, "Lumos5G: Mapping and Predicting Commercial mmWave 5G Throughput," in *Proc. ACM Internet Measurement Conference (IMC '20)*, 2020, pp. 176–193, doi: 10.1145/3419394.3423629.
- **Change `[10]` → `[13]`** where it refers to the dataset:
  - p. 2, Objective 1: "…across various mobility patterns [10]." → "[13]"
  - p. 8, Section 3.1: "…in a live commercial 5G deployment [10]." → "[13]"
  - p. 9, Section 3.3.1: covered by item 2 above.

## 4. Section 3.3.4 contradicts Table V — page 11

- **Current:** "The 1440p tier accounted for 40.5% of test predictions …"
- **Replace with:** "The 1440p tier accounted for 40.3% of test predictions …"
- **Current:** "The two lowest tiers were triggered in fewer than 1% of predictions."
- **Replace with:** "The 480p tier was selected for 11.0% of predictions, while the lowest 360p tier was selected only three times out of 13,622."

## 5. Handover count and method — pages 7, 10, 17, 18

The count used `tower_id.diff() != 0`, which also flags row 0 (the first row of the file) as a handover. Without it: **1,370 events — 881 walking, 489 driving**. The throughput values (386.99 → 301.75 Mbps, drop 85.23 Mbps) are correct and reproduce exactly.

- **Change "1,371" → "1,370"** in: Abstract (p. 7), Section 3.3.1 (p. 10), Section 4.5 (p. 17), Section 5.1 (p. 18).
- **Change "490 occurred during driving" → "489"** in Section 3.3.1 (p. 10) and Section 4.5 (p. 17).
- **Add the method** after "…from a pre-handover mean of 386.99 Mbps to a post-handover mean of 301.75 Mbps." (p. 10):
  > "Pre-handover throughput is the mean over the five seconds before the tower change, and post-handover throughput is the mean over the handover second and the following five seconds."

## 6. LSTM numbers come from two different training runs — pages 7, 14, 15, 18

**The problem.** Table II lists two models: 20 epochs (MSE 0.007694) and 50 epochs (MSE 0.007447). Section 4.1 says later analyses use the 50-epoch model, but the LSTM row of **Table III** and the Overall row of **Table IV** are the **20-epoch** numbers (0.007694 / 0.087716 / 0.056462). The Abstract and Section 5.1 then pair the 50-epoch MSE (0.007447) with "61.2% better than SMA", which was computed from the 20-epoch MSE. With 0.007447 it is **62.5%**.

The code for these runs was not saved, so the original numbers can't be regenerated exactly. `CloudGaming_Full_Analysis.ipynb` now regenerates everything with a fixed seed. Pick **one** of the two options below.

### Option A — keep the report's numbers, fix the labels (smallest change)

- **p. 13, Section 4.1 — current:** "Unless otherwise stated, the subsequent analyses use the 50-epoch model."
  **Replace with:** "Tables III and IV report the 20-epoch model; the 50-epoch model leads to the same conclusions."
- **p. 14, Table III caption:** "COMPARISON OF LSTM AGAINST BASELINE PREDICTORS (20-EPOCH LSTM)"
- **p. 15, Table IV caption:** "LSTM PREDICTION PERFORMANCE BY MOBILITY MODE (20-EPOCH LSTM)"
- **p. 7 Abstract and p. 18 Section 5.1:** "outperforming a Simple Moving Average baseline by 61.2% on MSE" → "**by 62.5% on MSE**" (since these sentences quote the 50-epoch MSE 0.007447).

### Option B — replace every LSTM number with the reproducible notebook run (seed 42, 50 epochs)

Every number then comes from code you can show. Old → new:

| Where | Item | Report now | Notebook |
|---|---|---|---|
| Table II | 20 epochs MSE / RMSE / MAE | 0.007694 / 0.087716 / 0.056462 | 0.007931 / 0.089056 / 0.059972 |
| Table II, Abstract, 5.1 | 50 epochs MSE / RMSE / MAE | 0.007447 / 0.086299 / 0.053223 | 0.007575 / 0.087036 / 0.055966 |
| 4.1 text | MSE reduction 20 → 50 epochs | ≈3.2% | ≈4.5% |
| Table III | LSTM row | 0.007694 / 0.087716 / 0.056462 | 0.007575 / 0.087036 / 0.055966 |
| Table III | SMA MAE | 0.090253 | 0.090254 |
| Abstract, 4.2, 5.1 | MSE reduction vs SMA | 61.2% | 61.8% |
| Table IV | Walking MSE / RMSE / MAE | 0.008121 / 0.090116 / 0.058234 | 0.007993 / 0.089402 / 0.057765 |
| Table IV | Driving MSE / RMSE / MAE | 0.004668 / 0.068320 / 0.043902 | 0.004616 / 0.067942 / 0.043214 |
| Table IV | Overall | 0.007694 / 0.087716 / 0.056462 | 0.007575 / 0.087036 / 0.055966 |
| 4.3 text | Driving MSE lower than walking | 42.5% | 42.2% |
| Table V | 1440p / 1080p / 720p / 480p / 360p samples | 5,491 / 2,221 / 4,403 / 1,504 / 3 | 5,452 / 2,147 / 4,315 / 1,682 / 26 |
| Table V | percentages | 40.3 / 16.3 / 32.3 / 11.0 / 0.0 | 40.0 / 15.8 / 31.7 / 12.3 / 0.2 |
| 4.4 text | "360p tier appeared only three times" | 3 | 26 times (0.2%) |
| Abstract, 4.4, 5.1 | 720p or higher | 88.9% | 87.5% |
| Table VI, Abstract, 4.4, 5.1 | LSTM switching | 19.34% | 18.62% |
| Table VI | SMA switching | 8.60% | 8.61% |
| Table VI | Last Value / Actual | 27.86% / 27.87% | 27.86% / 27.87% (unchanged) |
| Figs. 6–10 | images | — | `figures/fig6_…` to `figures/fig9_…`, `figures/fig5_fig10_handover_analysis.png` |

If you choose Option B, also apply item 4 using the new Table V values instead of the ones written there.

### Either option — add a robustness sentence (end of Section 4.1, p. 13)

> "To check run-to-run variation, the 50-epoch model was retrained with four random seeds. Test MSE ranged from 0.00743 to 0.00758 (mean 0.00751 ± 0.00006), bitrate switching from 18.60% to 18.72%, and the share of 720p-or-higher selections from 86.5% to 87.5%. In every run the LSTM had lower MSE than both the Last Value (0.00821) and SMA (0.01984) baselines."

---

## Notes for questions (no report change needed)

- **Switching frequency of "Actual" is 27.87%** because Mbps values pass through 32-bit floats (as in the original notebook): throughput exactly at 500/200/100/50 Mbps rounds a hair below the threshold. In exact arithmetic it is 27.52%. Conclusion unchanged.
- **72 of the 1,370 tower changes are between two separate runs**, not inside one walk or drive. Counting only within-run changes gives 1,298. The notebook prints both.
