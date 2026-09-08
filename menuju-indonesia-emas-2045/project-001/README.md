### Project Title: Indonesian Municipal Fiscal Analytics Framework
### 📌 Project Overview
This project analyzes fiscal autonomy across Indonesia's 508 kabupaten/kota (regencies and cities) using 2023 BPS fiscal indicator data. The goal is to segment regions into interpretable fiscal profiles — from centrally-dependent to fiscally autonomous — so that patterns in revenue independence, tax capacity, and spending behavior can inform policy discussion around eastern-Indonesia fiscal gaps and resource-windfall regions.

# Why?
**Situation:** Since 2001, Indonesia has let each of its 508 regions (kabupaten/kota — regencies and cities) manage its own budget instead of depending entirely on the central government.

**Task:** Some regions handle that freedom well; others still lean heavily on Jakarta for money. But with 508 regions and 8 different financial ratios per region, that pattern is invisible in a spreadsheet — you'd need to stare at thousands of numbers to see it.

**Action:** This project groups all 508 regions by how similar their financial behavior is (using a clustering technique called K-Means), so regions that "act alike" financially end up in the same group — without assuming in advance what those groups should look like.

**Result:** Four distinct financial "personalities" emerged from the data. No region was told which group it belonged to — the numbers sorted themselves.

### 🛠️ Tech Stack & Tools
* **Languages:** Python (v3.12+)
* **Libraries:** Pandas, NumPy, pdfplumber, re (regex), Scikit-Learn (StandardScaler, KMeans, PCA, silhouette_score), Matplotlib, Seaborn
* **Environment:** Jupyter Notebooks (Google Colab compatible), Git/GitHub

### 📂 Modular Project Pipeline
This repository is split into five functional modules to mimic production-level data engineering pipelines. Below is the breakdown of each notebook's role, methodology, and technical handoff.

### 📓 00. Data Acquisition
* **Notebook Link:** [🔗 View Notebook](./notebooks/00_data_acquisition.ipynb)
* **Objective:** Document where the raw fiscal data comes from and how all 8 fiscal ratio tables were extracted from the BPS PDF publication into a flat, analysis-ready CSV.
* **Source:** *Statistik Keuangan Pemerintah Kabupaten/Kota 2023 dan 2024* (BPS-Statistics Indonesia), 520 pages, text-based tables. [View publication](https://www.bps.go.id/id/publication/2024/12/31/6a4becee62edbb7320b6a81e/statistik-keuangan-pemerintah-kabupaten-kota-2023-dan-2024.html)
* **Key Findings:**
  * Figures are the DJPK/Ministry of Finance's **realized 2023** APBD data — the 2024 column in the same publication is budget/anggaran (not yet realized) and was deliberately excluded to keep every source in the project on the same fiscal year.
  * Every page carries a diagonal BPS watermark rendered in an oversized font (>10.5pt) that interleaves with the real table text (7–10pt) during naive extraction (e.g. `Kab. Aceh Besar o8,59`). Filtering `page.chars` by font size before calling `extract_text()` removes it cleanly.
  * `~0` values (e.g. Kab. Pegunungan Arfak's `rasio_pajak`) are BPS's own notation for a real near-zero figure and are recorded as `0.0`, not treated as missing.
  * DKI Jakarta is absent from the output **by design**: it has no separate kabupaten/kota APBDs, since its kotamadya are administrative subdivisions of one integrated provincial budget.
  * Extraction was validated two ways: shape/merge asserts (508 rows, 37 provinces, 0 NaNs after an outer join) and independent spot-checks against known facts — Kab. Badung (Bali) tops `rasio_kemandirian_2023` as expected from its tourism revenue base, and remote Papuan regencies sit at the bottom of `rasio_pajak`, consistent with near-zero local tax bases.
* **Handoff:** Saves the consolidated raw dataset to `data/raw/fiscal_indicators_indonesia_2023.csv` (508 rows × 10 columns, kabupaten/kota-level, 2023 realization).

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

# So..

In plain terms, here's what the four groups actually look like:
* **🟢 Fiscally Autonomous (156 regions):** Mostly Java and Bali — forming the nation's core industrial and tourism belt driven by mature internal consumer markets. These regions raise enough of their own tax and local revenue that they do not need to lean on the central government much. Example: Kab. Badung, Bali — its tourism income alone comfortably covers its budget.
* **⚪ Stable / Average (232 regions):** Anchoring across Sumatera and Sulawesi provinces — representing the largest, most "typical" macroeconomic baseline group in the distribution. Not struggling, not standout — just steady, typical regions with no extreme story either way.
* **🟡 Resource-Windfall Overperformers (46 regions):** Concentrated in Kalimantan and Papua Tengah — mining and resource royalty regions whose numbers look strong, but mostly because of massive commodity injections like sudden windfalls from coal, nickel, palm oil, or gold rather than independent revenue systems. That kind of income can vanish fast if commodity prices fall.
* **🔴 Dependent / Underperforming (74 regions):** Concentrated heavily in eastern Indonesia (NTT, Maluku, and Papua) — mapping perfectly to the well-known eastern-Indonesia fiscal gap where local economies still rely heavily on central government transfers and have not built up much of their own tax base.
  
**The takeaway:** Indonesia's decentralization did not simply "succeed" or "fail" — it produced four different outcomes depending on the region. A single national funding formula would help some of these groups and completely miss the other three.

**A challenge worth raising:** a region landing in the "Dependent" cluster does not automatically mean poor management — it could just be small, rural, or remote, with little to tax in the first place. Grouping by financial ratios alone cannot tell the difference between "mismanaged" and "genuinely has less to work with." A fair next step would be to check each cluster against basic facts like population size and geography before drawing conclusions about *why* a region ended up where it did.

**Policy directions that could work:** Instead of using one rigid policy for all 508 regions, the central government could align support with modern laws like the *UU HKPD no.1 Tahun 2022* to match how each group naturally behaves:
* **For the Dependent Group:** Shift the focus away from cutting their funding. Instead of complex tax overhauls, use the national **ETPD (Elektronifikasi Transaksi Pemerintah Daerah)** initiative to help them digitize simple, local fees like marketplace and parking retributions, making revenue collection leak-proof and straightforward.
* **For the Resource-Windfall Group:** Enforce the utilization of the newly regulated **Dana Abadi Daerah (Regional Endowment Funds)** under *PP 1/2024*. When commodity prices surge, these regions should lock away a slice of their mining or gas royalties into these long-term generation funds so they have a financial safety net when global prices drop.
* **For the Fiscally Autonomous Group:** Treat them as regional hubs for innovation. Grant them higher baseline spending flexibility so they can pioneer large regional infrastructure setups, and use their success stories to build local tax playbooks that emerging cities can safely copy.

A caveat worth stating plainly in any writeup is that the silhouette scores were modest (~0.21), meaning these are soft, overlapping groupings rather than hard-edged, rigid categories. Which is an accurate reflection of real-world public finance data where administrative boundaries blur, not a modeling weakness to hide.

---
### 🚀 How To Run This Project
1. Clone this repository to your local machine:
```bash
git clone https://github.com/muhyassin09/yasinnn/tree/main/menuju-indonesia-emas-2045
```
2. Install the exact environment dependencies:
```bash
pip install -r requirements.txt
```
3. Place `statistik-keuangan-pemerintah-kabupaten-kota-2023-dan-2024.pdf` in `data/raw/pdf/` — or run `00_data_acquisition.ipynb` to reproduce the extraction from scratch.
4. Navigate to the `notebooks/` directory and execute them sequentially from 00 to 04.
