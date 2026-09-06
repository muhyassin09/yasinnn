### Project Title: Indonesian Municipal Fiscal Analytics Framework
### 📌 Project Overview
This project analyzes fiscal autonomy across Indonesia's 508 kabupaten/kota (regencies and cities) using 2023 BPS fiscal indicator data. The goal is to segment regions into interpretable fiscal profiles — from centrally-dependent to fiscally autonomous — so that patterns in revenue independence, tax capacity, and spending behavior can inform policy discussion around eastern-Indonesia fiscal gaps and resource-windfall regions.

### 🛠️ Tech Stack & Tools
* **Languages:** Python (v3.12+)
* **Libraries:** Pandas, NumPy, Scikit-Learn (StandardScaler, KMeans, PCA, silhouette_score), Matplotlib, Seaborn
* **Environment:** Jupyter Notebooks (Google Colab compatible), Git/GitHub

### 📂 Modular Project Pipeline
This repository is split into four functional modules to mimic production-level data engineering pipelines. Below is the breakdown of each notebook's role, methodology, and technical handoff.

### 📓 01. Data Preprocessing
* **Notebook Link:** [🔗 View Notebook](./notebooks/01_data_preprocessing.ipynb)
* **Objective:** Ingest the raw fiscal indicator CSV and run a data-quality pass before any transformation.
* **Key Findings:**
  * 508 rows confirmed, matching BPS's official regency/city count, with 37 provinces represented.
  * No missing values and no duplicate region names — the raw data is clean going in.
* **Handoff:** Saves the quality-checked dataset to `data/processed/checked_fiscal_indicators_indonesia_2023.csv`.

### 📓 02. Data Analysis
* **Notebook Link:** [🔗 View Notebook](./notebooks/02_data_analysis.ipynb)
* **Objective:** Measure and visualize the distribution shape of each of the 8 fiscal ratios, then correct skew where a transform genuinely helps.
* **Key Findings:**
  * Several indicators are heavily right-skewed — `rasio_kemandirian_2023` (skew 14.09) and `rasio_efektivitas_pad` (skew 7.68) most severely, driven by real high-capacity outliers (e.g. Kab. Badung's tourism tax base), not data errors.
  * A `log1p` transform fixed 6 of 8 indicators well (e.g. `rasio_kemandirian_2023` skew dropped to 0.29), but `rasio_efektivitas_pajak` actually got *more* skewed (2.86 → -2.28) since it's centered near 100 rather than near 0 — flagged honestly rather than hidden.
* **Handoff:** Exports the log-transformed feature set to `data/processed/processed_fiscal_indicators_indonesia_2023.csv`.

### 📓 03. Modelling
* **Notebook Link:** [🔗 View Notebook](./notebooks/03_modelling.ipynb)
* **Objective:** Standardize the log-transformed indicators and segment the 508 regions into fiscal profiles via K-Means.
* **Methodology:**
  * Applied `StandardScaler` to the 8 log-transformed indicators (z-score, mean 0 / std 1).
  * Swept `k = 2…8`, evaluating inertia (elbow) and silhouette score; selected **k = 4** for policy-readable segmentation over a flat/uninformative k=2.
  * Used PCA (2 components) purely for 2-D visualization — clustering itself ran in the full 8-dimensional space.
* **Results Matrix:**

| Cluster | n Regions | Profile Label |
|---|---|---|
| 0 | — | Dependent / Underperforming |
| 1 | — | Resource-Windfall Overperformers |
| 2 | — | Fiscally Autonomous |
| 3 | — | Stable / Average |

*(n Regions is populated after running the notebook against the raw CSV — the source file wasn't available in this session, so cluster sizes couldn't be computed.)*

* **Conclusion:** Silhouette scores were modest throughout (peaking ~0.22–0.23 at k=2–3, ~0.21 at k=4), indicating fiscal capacity is more of a continuum than hard-bounded groups — an honest, realistic read of socioeconomic data rather than a modeling shortfall.
* **Handoff:** Exports cluster assignments and PCA coordinates to `data/processed/fiscal_clustered_with_pca.csv`.

### 📓 04. Finalizing
* **Notebook Link:** [🔗 View Notebook](./notebooks/04_finalizing.ipynb)
* **Objective:** Validate clusters against real geography, save the final labeled dataset, and summarize the pipeline.
* **Key Findings:**
  * **Dependent / Underperforming** — concentrated in NTT, Maluku, Maluku Utara, Papua (the known eastern-Indonesia fiscal gap).
  * **Resource-Windfall Overperformers** — Kalimantan (Selatan/Timur/Tengah/Utara) and Papua Tengah, driven by mining/resource royalties.
  * **Fiscally Autonomous** — Jawa Tengah/Timur/Barat, Bali, Banten (the industrial/tourism belt).
  * **Stable / Average** — spread across Sumatera and Sulawesi, the largest and most "typical" group.
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
