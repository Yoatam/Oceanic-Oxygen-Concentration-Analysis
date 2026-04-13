# 🌊 Ocean Oxygen Analysis — Gulf of St. Lawrence

An analysis of dissolved oxygen concentration in the Gulf of St. Lawrence using 15 underwater glider missions spanning 5.5 years (2018–2023) and high accuracy ship based bottle samples. This project was completed as part of the Statistical Society of Canada (SSC) 2025 Case Study Competition.

---

## 📌 Project Overview

Dissolved oxygen in seawater is essential for the survival of marine life. When oxygen drops too low, areas become hypoxic — unable to support most species. The Gulf of St. Lawrence is one such region where scientists have tracked an expanding oxygen minimum zone for decades.

This project addresses all five challenges from the SSC competition:

1. Sensor validation: comparing glider oxygen readings against high accuracy bottle measurements
2. Depth and regional categorization with trend analysis across 5.5 years
3. Relationships between oxygen, temperature, salinity, and density
4. Geospatial variation in oxygen across the Gulf
5. Temporal variation across seasons and years

Beyond the competition requirements, a machine learning model was built to predict oxygen concentration from temperature, salinity, and depth.

---

## ❓ Key Questions

- How does oxygen vary by depth, location, and season across the Gulf of St. Lawrence?
- Is oxygen declining over time in the deep Gulf, and can that trend be measured from glider data?
- Which variables most strongly predict oxygen concentration, and does feature importance accurately reflect their true contribution?
- Can a reliable model predict oxygen from temperature, salinity, and depth alone?

---

## 📂 Dataset

**Glider data: 15 missions from 6 gliders (2018 to 2023)**

| Glider | Missions | Coverage |
|---|---|---|
| Cabot | 4 | Western Gulf, various seasons |
| Fundy | 2 | Central Gulf |
| Peggy | 2 | Eastern Gulf |
| Sambro | 1 | Western Gulf |
| Scotia | 5 | Eastern Gulf, 2018 to 2022 |
| Shad | 1 | Western Gulf |

Total: 12,018,966 clean observations after removing physically impossible values.

**Bottle sample data: 481 high accuracy observations**

Collected via Winkler titration on discrete water samples. Used as the target variable for machine learning modelling.

### Key Variables
- `time`: UTC timestamp
- `latitude`, `longitude`: geographic coordinates
- `depth` — depth in meters (glider) / `CTDPRES`: pressure in dbar (bottle)
- `sea_water_temperature` / `CTDTEMP`: temperature (°C)
- `sea_water_practical_salinity` / `CTDSAL`: salinity (PSU)
- `sea_water_density` — density (kg/m³): glider only
- `micromoles_of_oxygen_per_unit_mass_in_sea_water`: glider oxygen (µmol/kg)
- `best_Oxygen` — bottle oxygen (µmol/kg): model target

---

## ⚙️ Methods

### Part 1:  Glider Data Analysis
- Loaded and merged all 15 glider CSV files into a single dataset
- Removed physically impossible values (oxygen outside 0–500 µmol/kg range)
- Categorized data into three depth layers: surface (0–50m), mid water (50–150m), deep (150m+)
- Divided the Gulf into three geographic regions by longitude
- Computed annual oxygen trends from 2018 to 2023 by depth layer
- Analyzed seasonal oxygen patterns across all missions
- Computed correlations between oxygen, temperature, salinity, density, and depth
- Built an interactive folium map of all 15 glider tracks
- Attempted sensor validation by co-locating bottle and glider measurements within 10km and 6 hours

### Part 2: Predictive Modelling (Bottle Data)
- Replaced -999 sentinel values with NaN
- Dropped rows missing key variables
- Constructed datetime from separate year/month/day/hour/minute columns
- Merged bottle and glider datasets using nearest time approach
- Removed outliers using Z score filtering (threshold: 3 standard deviations)
- Pearson correlation and hypothesis testing
- Random Forest Regression (primary model)
- Linear Regression (baseline comparison)
- GridSearchCV hyperparameter tuning (108 combinations, 5 fold CV)
- Standalone cross validation on scaled features
- Feature importance analysis and ablation testing

---

## 🔍 Key Findings

**Depth is the dominant driver across the full glider dataset (r = -0.902).** Across 12 million readings spanning 5.5 years, oxygen drops consistently as depth increases. Salinity (r = -0.749) and density (r = -0.657) follow because deeper water is consistently saltier and denser in the Gulf of St. Lawrence.

**Deep water oxygen is declining.** Mean deep water oxygen dropped from 105.4 µmol/kg in 2021 to 92.8 µmol/kg in 2023: a decrease of 12.6 µmol/kg in three years. The hypoxic threshold for most marine species is around 62.5 µmol/kg.

**An Oxygen Minimum Zone** was identified at 200–400 dbar in the bottle data depth profile: a well documented hypoxic feature of the Gulf that has been expanding due to climate change.

**A seasonal oxygen cycle** was detected: surface oxygen peaks in spring and early summer due to the phytoplankton bloom and winter mixing, then drops in late summer as water warms. This pattern was confirmed across both the glider seasonal analysis and the bottle data time series.

**Feature importance can be misleading under multicollinearity.** In the bottle data model, salinity scored 91.4% feature importance but removing it only dropped R² from 0.916 to 0.897, a 2% decrease. This is because salinity and pressure are correlated at r = 0.927 in this dataset. When salinity is removed, pressure compensates. Ablation testing is the correct way to verify feature importance claims.

**Sensor validation was not achievable.** Despite 273 bottle samples falling within the glider geographic area and time period, the ship and gliders were never within 10km of each other simultaneously, the closest approach was approximately 548km apart. Coordinated sampling is recommended for future monitoring programs.

---

## 📈 Model Results (Bottle Data — Part 2)

| Model | R² | MAE |
|---|---|---|
| Random Forest (baseline) | 0.916 | 12.51 µmol/kg |
| Random Forest (tuned) | 0.914 | 13.03 µmol/kg |
| Linear Regression | 0.857 | 27.10 µmol/kg |
| Cross validation mean (RF) | 0.959 | std = 0.016 |
| GridSearchCV best CV score | 0.974 | — |

**Glider Data Correlations with Oxygen (Part 1 — 12M rows):**

| Variable | Correlation with Oxygen |
|---|---|
| Depth | r = -0.902 |
| Salinity | r = -0.749 |
| Density | r = -0.657 |
| Temperature | r = -0.308 |

---

## ⚠️ Limitations

- Deep water glider data only available from 2021 onward: earlier missions did not dive below 150m
- Most glider deployments occurred in summer: winter coverage is limited
- Sensor validation was not achievable due to lack of coordinated sampling
- Variables such as ocean currents, biological activity, and nutrients are not included
- The bottle glider merge produced only 27 unique matched glider readings across 481 bottle rows: the glider oxygen variable was excluded from modelling as a result
- 481 bottle observations is a small sample for machine learning

---

## ▶️ How to Run

1. **Clone this repository:**
   ```bash
   git clone https://github.com/Yoatam/your-repo-name.git
   cd your-repo-name
   ```

2. **Install dependencies:**
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn scipy folium openpyxl
   ```

3. **Place data files in the correct folders:**
   - `Dataset/Bottle data/Bottle Oxygen Data.xlsx`
   - `Dataset/Glider data/*.csv` (all 15 glider mission CSV files)

4. **Run the notebook:**
   Open `oxygen_CC_final_clean.ipynb` in Jupyter and run all cells top to bottom.

5. **View the interactive maps:**
   After running the folium cells, open `oxygen_map.html` and `glider_tracks_map.html` in any browser.

---

## 🙌 Acknowledgments

Data provided by the Ocean Tracking Network and the TReX project (Dalhousie University) as part of the SSC 2025 Case Study Competition. Case study organized by Adam Comeau and Will Nesbitt.
