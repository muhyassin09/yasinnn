### Project Title: Indonesian Municipal Fiscal Analytics Framework
### 📌 Project Overview
This project analyzes fiscal autonomy across Indonesia's 508 regencies and cities using 2023 BPS fiscal indicator data [🔗 View Data](https://www.bps.go.id/id/publication/2024/12/20/6453851e55f22f31c4d30141/statistik-keuangan-pemerintah-provinsi-2023-dan-2024.html). The goal is to segment regions into interpretable fiscal profiles — from centrally-dependent to fiscally autonomous — so that patterns in revenue independence, tax capacity, and spending behavior can inform policy discussion around eastern-Indonesia fiscal gaps and resource-windfall regions.

### 🛠️ Tech Stack & Tools
* **Languages:** Python (v3.12+)
* **Libraries:** Pandas, NumPy, Scikit-Learn (StandardScaler, KMeans, PCA, silhouette_score), Matplotlib, Seaborn
* **Environment:** Jupyter Notebooks (Google Colab compatible), Git/GitHub

### 📂 Modular Project Pipeline
This repository is split into four functional modules to mimic production-level data engineering pipelines. Below is the breakdown of each notebook's role, methodology, and technical handoff.

### 📓 01. Data Preprocessing
* **Notebook Link:** [🔗 View Notebook](./notebooks/01_data_preprocessing.ipynb)
* **Objective:** Ingest the raw fiscal indicator CSV and run a data-quality pass before any transformation.
* **Key Findings** (verified against `processed_fiscal_indicators_indonesia_2023.csv`):
  * 508 rows confirmed, matching BPS's official regency/city count, with exactly 37 provinces represented.
  * Zero missing values and zero duplicate region names across all 508 rows — the raw data is clean going in.
* **Handoff:** Saves the quality-checked dataset to `data/processed/checked_fiscal_indicators_indonesia_2023.csv`.

### 📓 02. Data Analysis
* **Notebook Link:** [🔗 View Notebook](./notebooks/02_data_analysis.ipynb)
* **Objective:** Measure and visualize the distribution shape of each of the 8 fiscal ratios, then correct skew where a transform genuinely helps.
* **Key Findings** (verified skewness, before → after `log1p`):

| Indicator | Skew Before | Skew After | Verdict |
|---|---|---|---|
| `rasio_kemandirian_2023` | 14.09 | 0.29 | Fixed |
| `rasio_efektivitas_pad` | 7.68 | 0.06 | Fixed |
| `rasio_pajak` | 3.43 | 0.74 | Fixed |
| `rasio_efektivitas_pajak` | 2.86 | -2.28 | **Worse** — overcorrected in the opposite direction |
| `derajat_desentralisasi_2023` | 2.11 | -0.17 | Fixed |
| `tingkat_penyerapan_pendapatan` | 2.10 | 1.41 | Partially improved |
| `tingkat_penyerapan_belanja` | 1.68 | 0.99 | Fixed |
| `rasio_belanja_thd_pendapatan` | 0.40 | -0.14 | Already near-symmetric |

  * The two heaviest raw skews — `rasio_kemandirian_2023` (14.09) and `rasio_efektivitas_pad` (7.68) — are driven by real high-capacity outliers (e.g. Kab. Badung's tourism tax base), not data errors.
  * `log1p` fixed 6 of 8 indicators well, but `rasio_efektivitas_pajak` actually got *more* skewed in the opposite direction (2.86 → -2.28) since it's centered near 100 rather than near 0 — flagged honestly rather than hidden, and kept as a known limitation.
* **Figures:** [Raw distributions](./reports/figures/fig1_raw_distributions.png) · [Log-transformed distributions](./reports/figures/fig2_log_distributions.png)
* **Handoff:** Exports the log-transformed feature set to `data/processed/processed_fiscal_indicators_indonesia_2023.csv`.

### 📓 03. Modelling
* **Notebook Link:** [🔗 View Notebook](./notebooks/03_modelling.ipynb)
* **Objective:** Standardize the log-transformed indicators and segment the 508 regions into fiscal profiles via K-Means.
* **Methodology:**
  * Applied `StandardScaler` to the 8 log-transformed indicators (z-score, mean 0 / std 1).
  * Swept `k = 2…8`, evaluating inertia (elbow) and silhouette score. The elbow curve has no sharp bend, and silhouette score declines steadily from k=2 (0.225) through k=4 (0.212), dips further to a low of ~0.17 at k=6, then ticks back up. Selected **k = 4** for a policy-readable segmentation rather than chasing the marginally higher but less interpretable k=2/3 scores.
  * Used PCA (2 components, explaining 36.8% + 29.3% = 66.1% of variance) purely for 2-D visualization — clustering itself ran in the full 8-dimensional space.
* **Results Matrix** (verified against `fiscal_clustered_indonesia_2023.csv`):

| Cluster Label | n Regions | Decentralization | Fiscal Independence | Tax Ratio | PAD Effectiveness | Tax Effectiveness | Spend-to-Revenue |
|---|---|---|---|---|---|---|---|
| Dependent / Underperforming | 74 | 4.62 | 5.02 | 1.35 | 67.05 | 63.45 | 99.51 |
| Resource-Windfall Overperformers | 46 | 8.12 | 9.33 | 2.94 | 187.86 | 140.77 | 95.44 |
| Fiscally Autonomous | 156 | 24.95 | 40.23 | 14.07 | 103.57 | 107.17 | 101.70 |
| Stable / Average | 232 | 8.29 | 9.32 | 2.42 | 109.91 | 106.38 | 102.07 |

* **Conclusion:** Silhouette scores were modest throughout (peaking at 0.225 for k=2, 0.212 for k=4), indicating fiscal capacity is more of a continuum than hard-bounded groups — an honest, realistic read of socioeconomic data rather than a modeling shortfall. The **Fiscally Autonomous** group stands out clearly (decentralization ~25 vs. ~4–8 elsewhere, fiscal independence ~40 vs. ~5–9 elsewhere), while **Resource-Windfall Overperformers** are distinguished by outsized PAD (187.9) and tax effectiveness (140.8) relative to modest targets, not by decentralization itself.
* **Figures:** [Elbow curve & silhouette score by k](./reports/figures/fig3_elbow_silhouette.png) · [Cluster PCA projection](./reports/figures/fig4_cluster_pca.png)
* **Handoff:** Exports cluster assignments and PCA coordinates to `data/processed/fiscal_clustered_with_pca.csv`.

### 📓 04. Finalizing
* **Notebook Link:** [🔗 View Notebook](./notebooks/04_finalizing.ipynb)
* **Objective:** Validate clusters against real geography, save the final labeled dataset, and summarize the pipeline.
* **Key Findings** (verified top-5 provinces per cluster):
  * **Dependent / Underperforming** (n=74) — Nusa Tenggara Timur (9), Maluku (8), Maluku Utara (6), Lampung (4), Papua (4): the known eastern-Indonesia fiscal gap.
  * **Resource-Windfall Overperformers** (n=46) — Kalimantan Selatan (8), Kalimantan Timur (7), Papua Tengah (5), Kalimantan Tengah (4), Kalimantan Utara (4): mining/resource royalty regions.
  * **Fiscally Autonomous** (n=156) — Jawa Tengah (30), Jawa Timur (26), Jawa Barat (20), Bali (9), Banten (7): the industrial/tourism belt.
  * **Stable / Average** (n=232) — Sumatera Utara (23), Aceh (20), Sumatera Barat (15), Sulawesi Selatan (14), Sulawesi Tenggara (13): the largest and most "typical" group.
* **Not covered** (deliberately out of scope): spatial autocorrelation (needs region boundary geometry), VaR/CVaR-style risk modeling (not supportable on single-year cross-sectional ratios), transfer allocation optimization (inputs not in current data).
* **Handoff:** Saves the final dataset to `data/processed/fiscal_clustered_indonesia_2023.csv`.

### 🚀 How To Run This Project
1. Clone this repository to your local machine:
```bash
git clone https://github.com/muhyassin09/yasinnn/tree/main/menuju-indonesia-emas-2045
```
2. Install the exact environment dependencies:
```bash
pip install -r requirements.txt
```
3. Place `fiscal_indicators_indonesia_2023.csv` in `data/raw/`.
4. Navigate to the `notebooks/` directory and execute them sequentially from 01 to 04.
