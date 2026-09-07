### Hi, I'm Muhammad Yassin👋
### 📊 Statistics Graduate | Aspiring Data Scientist

I'm a Statistics graduate, currently building out a portfolio of data analysis projects while preparing applications for a Master of Science in Data Science or Financial Mathematics in Australia.

### 📂 Portfolio Structure & Architecture
To keep my work reproducible and easy to follow, every project in this portfolio rejects messy, all-in-one notebooks in favor of a Standardized Modular Pipeline: each stage lives in its own notebook, with data passed cleanly between them.

```
├── data/
│   ├── raw/                 # Original, untouched data dumps (.csv, etc.)
│   └── processed/           # Transformed datasets passed between notebooks
├── notebooks/
|   ├── 00_data_acquisition.ipynb     -> Documents data source and extraction method
│   ├── 01_data_preprocessing.ipynb   -> Ingestion, quality checks, cleaning
│   ├── 02_data_analysis.ipynb        -> Distribution/EDA and feature engineering decisions
│   ├── 03_modelling.ipynb            -> Training, tuning, and evaluation
│   └── 04_finalizing.ipynb           -> Validation, final export, and summary
├── .gitignore                # Excludes large data, virtual envs, and secrets
├── README.md                 # Comprehensive executive summary of the project
└── requirements.txt          # Package dependencies (pandas, scikit-learn, etc.)
```

Each notebook is single-responsibility and self-documenting — every one opens with a pipeline status header, its objective, and data ingestion details, and closes with an explicit handoff to the next stage.

### 🚀 Featured Projects

#### 🛠️ 1. Indonesian Municipal Fiscal Analytics Framework
* **Core Goal:** Segment Indonesia's 508 regencies/cities into interpretable fiscal-autonomy profiles from 2023 BPS data, surfacing patterns like the eastern-Indonesia dependency gap and resource-windfall regions.
* **Tech Stack:** Python, Pandas, NumPy, Scikit-Learn (StandardScaler, KMeans, PCA), Matplotlib, Seaborn
* **Architecture:** 4-Part Modular Pipeline (Preprocessing ➔ Analysis ➔ Modelling ➔ Finalizing)
* 🔗 [View Repository](https://github.com/muhyassin09/yasinnn/tree/main/menuju-indonesia-emas-2045)

#### 🛠️ 2. [Project 2 Title]
* **Core Goal:** [1-sentence summary of the business/technical problem solved]
* **Tech Stack:** Python, [Library X], [Library Y], [Library Z]
* **Architecture:** 4-Part Modular Pipeline (Preprocessing ➔ Analysis ➔ Modelling ➔ Finalizing)
* 🔗 [View Repository](https://github.com/your-username/project-2-repo)

### 🛠️ Technical Toolbox
* **Language:** Python, R, SQL, etc.
* **Visualization:** Matplotlib, Seaborn, etc.
* **Tools & Workflow:** Git, GitHub, Jupyter Ecosystem, Microsoft Excel, etc.

### 📬 Connect With Me
* **Email:** [muh.yassin09@gmail.com](mailto:muh.yassin09@gmail.com)
