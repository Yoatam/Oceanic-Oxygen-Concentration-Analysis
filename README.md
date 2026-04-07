# 🌊 Ocean Oxygen Analysis — Gulf of St. Lawrence

This project investigates how **temperature**, **salinity**, and **depth/pressure** 
influence **dissolved oxygen concentration** in the Gulf of St. Lawrence, eastern 
Canada, a region with a well documented and expanding hypoxic zone that threatens 
marine ecosystems.

We worked with two real oceanographic datasets (bottle samples + glider sensor data) 
to clean, visualize, model, and validate the key drivers of oxygen concentration 
using statistical analysis and machine learning.

---

## 📌 Project Overview

Dissolved oxygen in seawater is essential for the survival of marine life. When 
oxygen drops too low, areas become hypoxic, unable to support most species. 
The Gulf of St. Lawrence is one such region where scientists have tracked an 
expanding oxygen minimum zone for decades.

This project builds predictive models, identifies the dominant drivers of oxygen 
concentration, and validates findings using ablation testing and cross-validation.

---

## ❓ Key Questions

- What are the most influential factors affecting oxygen levels in the ocean?
- How are oxygen levels distributed across depths, locations, and time?
- Can we build reliable models to predict oxygen concentration from environmental data?
- Does feature importance accurately reflect each variable's true contribution?

---

## 📂 Dataset

Two real datasets from the Gulf of St. Lawrence:
- **Bottle sample data** — 481 observations after cleaning, discrete cast measurements
- **Glider sensor data** — continuous high-resolution depth profiles

### Key Variables
- `CTDTEMP` — Temperature of seawater (°C)
- `CTDSAL` — Salinity of seawater (PSU)
- `CTDPRES` — Pressure / depth (dbar)
- `best_Oxygen` — Bottle oxygen concentration (µmol/kg) — model target
- `Latitude`, `Longitude` — Geographic coordinates of measurement points

> **Note on the merge:** The two datasets were joined using a nearest-time approach. 
> A merge quality check revealed only 27 unique glider readings matched across 481 
> bottle rows, so the glider oxygen variable was excluded as a model target. 
> Bottle oxygen was used instead.

---

## ⚙️ Methods

### 🔧 Data Cleaning
- Replaced -999 sentinel values (failed sensor readings) with NaN
- Dropped rows missing key variables (oxygen, location, temperature, salinity)
- Constructed datetime column from separate year/month/day/hour/minute fields
- Validated merge quality between bottle and glider datasets
- Removed outliers using Z-score filtering (threshold: 3 standard deviations)

### 📊 Exploratory Data Analysis
- Pearson correlation and hypothesis testing
- Pairplots, scatter plots, correlation heatmap
- Depth profile plots (inverted y-axis — standard oceanographic format)
- Time series analysis of oxygen over the study period
- Interactive geographic map (folium) with clickable station-level data

### 🤖 Modeling
- **Random Forest Regression** — primary model
- **Linear Regression** — baseline comparison
- **GridSearchCV** — hyperparameter tuning (108 combinations)
- **5-fold cross-validation** — generalisation check
- **Ablation testing** — feature importance validation

---

## 🔍 Key Findings

- **Salinity is the strongest individual predictor** of oxygen concentration 
  (r = -0.899), but salinity and pressure are correlated at r = 0.927 — both 
  encode the same underlying water mass structure of the Gulf.

- **Ablation testing** revealed that removing salinity only dropped R² from 
  0.916 to 0.897 (2%), despite salinity holding 91.4% feature importance. 
  This exposes a known limitation of Random Forest feature importance under 
  multicollinearity — pressure compensated when salinity was removed.

- **Oxygen Minimum Zone (OMZ)** identified at 200–400 dbar in the depth 
  profile — a well-documented hypoxic feature of the Gulf of St. Lawrence 
  consistent with published oceanographic research.

- **Seasonal pattern** detected in the time series: oxygen increased from 
  November 2021 to June 2022, consistent with winter convective mixing and 
  the spring phytoplankton bloom.

- **Random Forest outperformed Linear Regression** (R² 0.916 vs 0.857), 
  though the gap is smaller than expected because salinity's near-linear 
  relationship with oxygen allows a linear model to capture most of the pattern.

---

## 📈 Model Results

| Model | R² | MAE |
|---|---|---|
| Random Forest (baseline) | 0.916 | 12.51 µmol/kg |
| Random Forest (tuned) | 0.914 | 13.03 µmol/kg |
| Linear Regression | 0.857 | 27.10 µmol/kg |
| Cross-validation mean (RF) | 0.959 | std = 0.016 |
| GridSearchCV best CV score | 0.974 | — |

---

## ⚠️ Limitations

- The nearest-time merge produced only 27 unique glider readings matched 
  across 481 bottle rows — future work should validate time alignment or 
  use spatial matching instead.
- Missing variables such as ocean currents and biological activity could 
  improve model accuracy.
- 481 observations is a small sample — results should be validated on a 
  larger multi-region dataset.
- Multicollinearity between salinity and pressure (r = 0.927) makes it 
  difficult to isolate each variable's independent contribution.

---

## ▶️ How to Run

1. **Clone this repository:**
```bash
   git clone https://github.com/your-username/your-repo-name.git
   cd your-repo-name
```

2. **Install dependencies:**
```bash
   pip install pandas numpy matplotlib seaborn scikit-learn scipy folium openpyxl
```

3. **Run the notebook:**
   Open `oxygen_CC_final.ipynb` in Jupyter and run all cells top to bottom.

4. **View the interactive map:**
   After running the folium cell, open `oxygen_map.html` in any browser.

---

## 🙌 Acknowledgments

Thanks to oceanographic institutions and open-data initiatives for providing 
the real-world datasets that made this analysis possible.
