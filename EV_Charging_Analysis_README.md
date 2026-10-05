# EV Charging Infrastructure Analysis

## Project Overview

This project performs **Exploratory Data Analysis (EDA)** on an EV charging station dataset to understand the distribution, availability, charging capacity, power levels, and geographic patterns of electric vehicle charging infrastructure.

The analysis focuses on data cleaning, statistical analysis, aggregation, outlier detection, and visualization to identify useful patterns and insights from EV charging infrastructure data.

---

## Objectives

- Clean and validate the EV charging dataset
- Analyze the number of charging stations by country
- Analyze the total number of charging ports
- Compare charging stations and ports across countries
- Calculate average charging ports per station
- Analyze charging-power distribution
- Understand different EV charging power classes
- Analyze DC fast-charging infrastructure
- Calculate DC charging penetration by country
- Identify outliers and unusual charging-power values
- Use statistical measures to understand the charging-power distribution
- Create meaningful visualizations
- Extract business and infrastructure insights

---

## Dataset

The dataset contains information related to EV charging stations, including fields such as:

- Station ID
- Station name
- City
- State/Province
- Country code
- Number of charging ports
- Charging power in kW
- Charging power class

### Dataset Size

After the cleaning and outlier-removal steps shown in the analysis:

- **Cleaned records:** 237,718
- **Charging-power records:** 237,718
- **Columns used in the analysis:** station information, location, ports, charging power, and power class

---

# Data Cleaning

The dataset was examined for data-quality issues before performing the analysis.

The cleaning process included:

- Checking the structure of the dataset
- Inspecting relevant columns
- Checking charging-power values
- Identifying extreme values
- Investigating suspicious observations
- Removing an erroneous `1,000,000 kW` record
- Resetting the DataFrame index
- Verifying the cleaned data
- Rechecking descriptive statistics after cleaning

### Extreme Value Identified

One record contained:

```text
power_kw = 1,000,000 kW
```

This value was identified as erroneous and removed.

After removal:

```text
Rows after removal: 237,718
Maximum power: 1,000 kW
```

The remaining maximum of `1,000 kW` belongs to the `DC_ULTRA_(>=150kW)` power class and was retained for analysis.

---

# Exploratory Data Analysis

## 1. Charging Stations by Country

The number of charging stations was aggregated by `country_code`.

Example approach:

```python
stations_by_country = (
    df_clean.groupby("country_code")["id"]
    .count()
    .sort_values(ascending=False)
    .reset_index(name="station_count")
)
```

### Top Countries by Charging Stations

| Rank | Country | Charging Stations |
|---:|:---:|---:|
| 1 | US | 81,357 |
| 2 | ES | 17,795 |
| 3 | GB | 26,628 |
| 4 | CA | 16,346 |
| 5 | DE | 23,240 |
| 6 | IT | 9,935 |
| 7 | FR | 12,945 |
| 8 | PT | 3,696 |
| 9 | RU | 2,174 |
| 10 | NO | 4,662 |

The country-level analysis shows that charging infrastructure is highly concentrated among a relatively small number of countries.

---

# 2. DC Fast-Charging Stations by Country

DC fast-charging stations were identified using the following power classes:

```python
dc_classes = [
    "DC_FAST_(50-149kW)",
    "DC_ULTRA_(>=150kW)"
]
```

The analysis grouped stations by country and counted stations belonging to these DC categories.

Example:

```python
dc_by_country = (
    df_clean.groupby("country_code")
    .agg(
        dc_station_count=(
            "power_class",
            lambda x: x.isin(dc_classes).sum()
        )
    )
    .sort_values("dc_station_count", ascending=False)
    .reset_index()
)
```

### Top Countries by DC Fast-Charging Stations

| Country | DC Stations |
|:---:|---:|
| US | 14,298 |
| ES | 6,324 |
| GB | 4,438 |
| CA | 3,285 |
| DE | 3,259 |
| IT | 2,288 |
| FR | 1,793 |
| PT | 1,750 |
| RU | 1,542 |
| NO | 1,408 |
| SE | 1,193 |
| AU | 662 |
| TR | 594 |
| UA | 502 |
| IE | 482 |

The United States has the largest number of DC fast-charging stations in the dataset.

---

# 3. Total Charging Ports by Country

Charging ports were aggregated by country.

Example:

```python
country_avg_ports = (
    df_clean.groupby("country_code")
    .agg(
        total_stations=("id", "count"),
        total_ports=("ports", "sum")
    )
    .reset_index()
)
```

This allows comparison between the number of stations and the total number of available charging ports.

---

# 4. Average Charging Ports per Station

The average number of ports per station was calculated as:

```text
Average Ports per Station =
Total Charging Ports / Total Charging Stations
```

Python implementation:

```python
country_avg_ports["avg_ports_per_station"] = (
    country_avg_ports["total_ports"]
    / country_avg_ports["total_stations"]
)
```

Countries were sorted by average ports per station.

Because countries with only a few stations can produce misleading averages, a second analysis was performed using only countries with at least **100 charging stations**.

Example:

```python
country_avg_ports_100 = (
    country_avg_ports[
        country_avg_ports["total_stations"] >= 100
    ]
    .sort_values(
        "avg_ports_per_station",
        ascending=False
    )
)
```

### Highest Average Ports per Station Among Countries With ≥100 Stations

| Country | Stations | Total Ports | Average Ports/Station |
|:---:|---:|---:|---:|
| KR | 161 | 1,098 | 6.819876 |
| SE | 4,892 | 30,311 | 6.196034 |
| NO | 4,662 | 28,852 | 6.188760 |
| TW | 162 | 739 | 4.561728 |
| FI | 1,770 | 6,843 | 3.866102 |
| IE | 1,926 | 6,799 | 3.530114 |
| DK | 2,151 | 6,880 | 3.198512 |
| ES | 17,795 | 53,715 | 3.018545 |
| IS | 415 | 1,231 | 2.966266 |
| AT | 1,268 | 3,454 | 2.723975 |
| AE | 127 | 334 | 2.629921 |
| EG | 457 | 1,190 | 2.603939 |
| NZ | 789 | 1,915 | 2.427123 |
| RO | 401 | 973 | 2.426434 |
| BE | 1,065 | 2,570 | 2.413146 |

This analysis provides a more meaningful comparison of station capacity because it reduces the effect of countries with very small numbers of stations.

---

# 5. EV Charging Power Distribution

The `power_kw` column was analyzed using descriptive statistics.

```python
df_clean["power_kw"].describe()
```

After removing the erroneous `1,000,000 kW` record:

| Statistic | Value |
|:---|---:|
| Count | 237,718 |
| Mean | 31.051659 kW |
| Standard Deviation | 56.341690 kW |
| Minimum | 0.2 kW |
| 25% | 3.7 kW |
| Median | 11.0 kW |
| 75% | 22.0 kW |
| Maximum | 1,000.0 kW |

### Interpretation

The median charging power is only **11 kW**, while the mean is approximately **31.05 kW**.

This difference occurs because a relatively small number of high-power charging stations increase the mean.

Therefore, the distribution is strongly right-skewed.

---

# 6. Most Common Charging-Power Values

The most frequent charging-power values were identified using:

```python
df_clean["power_kw"].value_counts().head(20)
```

### Top Charging-Power Values

| Charging Power | Number of Stations |
|---:|---:|
| 3.7 kW | 77,460 |
| 22 kW | 49,817 |
| 50 kW | 28,332 |
| 11 kW | 14,072 |
| 7 kW | 10,151 |
| 3 kW | 5,425 |
| 150 kW | 4,713 |
| 7.4 kW | 3,474 |
| 18 kW | 2,992 |
| 250 kW | 2,905 |
| 60 kW | 2,619 |
| 5 kW | 2,567 |
| 120 kW | 2,386 |
| 16 kW | 2,277 |
| 350 kW | 2,146 |
| 5.1 kW | 1,788 |
| 4.8 kW | 1,657 |
| 40 kW | 1,599 |
| 100 kW | 1,467 |
| 48 kW | 1,280 |

The most common values demonstrate that EV charging infrastructure is concentrated around several standard charging-power levels.

---

# 7. Charging Power Classes

The dataset was divided into five charging-power classes:

```text
AC_L1_(<7.5kW)
AC_L2_(7.5-21kW)
AC_HIGH_(22-49kW)
DC_FAST_(50-149kW)
DC_ULTRA_(>=150kW)
```

The number of stations in each class was calculated using:

```python
power_class_count = (
    df_clean["power_class"]
    .value_counts()
    .reset_index()
)

power_class_count.columns = [
    "power_class",
    "station_count"
]
```

---

# 8. EV Charging Stations by Power Class

### Distribution

| Power Class | Station Count | Percentage |
|:---|---:|---:|
| AC_L1 (<7.5 kW) | 107,111 | 45.06% |
| AC_HIGH (22–49 kW) | 55,539 | 23.36% |
| DC_FAST (50–149 kW) | 37,328 | 15.70% |
| AC_L2 (7.5–21 kW) | 24,238 | 10.20% |
| DC_ULTRA (≥150 kW) | 13,503 | 5.68% |

### Interpretation

The largest category is:

```text
AC_L1 (<7.5 kW)
```

with approximately:

```text
107,111 stations
45.06% of the analyzed records
```

DC ultra-fast charging represents the smallest category:

```text
DC_ULTRA (>=150 kW)
13,503 stations
5.68%
```

---

# 9. Power Class Statistics

The minimum, median, and maximum charging power were calculated for every power class.

```python
df_clean.groupby("power_class")["power_kw"].agg(
    ["count", "min", "median", "max"]
)
```

### Results

| Power Class | Count | Minimum | Median | Maximum |
|:---|---:|---:|---:|---:|
| AC_HIGH (22–49 kW) | 55,539 | 22.0 | 22.0 | 49.0 |
| AC_L1 (<7.5 kW) | 107,111 | 0.2 | 3.7 | 7.4 |
| AC_L2 (7.5–21 kW) | 24,238 | 7.5 | 11.0 | 21.6 |
| DC_FAST (50–149 kW) | 37,328 | 50.0 | 50.0 | 149.0 |
| DC_ULTRA (≥150 kW) | 13,502 | 150.0 | 200.0 | 1,000.0 |

The results demonstrate distinct charging-power tiers in the dataset.

---

# 10. DC Charging Penetration

DC charging penetration was calculated as:

```text
DC Charging Penetration =
DC Stations / Total Stations × 100
```

Python:

```python
country_dc_analysis["dc_percentage"] = (
    country_dc_analysis["dc_stations"]
    / country_dc_analysis["total_stations"]
    * 100
)
```

To avoid very small country samples affecting the result, countries with at least **100 stations** were analyzed.

### Highest DC Charging Penetration for Countries With ≥100 Stations

| Country | Total Stations | DC Stations | DC Percentage |
|:---:|---:|---:|---:|
| KR | 161 | 161 | 100.00% |
| IL | 291 | 276 | 94.85% |
| TW | 162 | 151 | 93.21% |
| EE | 169 | 156 | 92.31% |
| UA | 555 | 502 | 90.45% |
| RU | 2,174 | 1,542 | 70.93% |
| RO | 401 | 279 | 69.58% |
| ZA | 154 | 96 | 62.34% |
| UY | 138 | 75 | 54.35% |
| AU | 1,219 | 662 | 54.31% |
| HR | 265 | 143 | 53.96% |
| LT | 765 | 399 | 52.16% |
| TR | 1,190 | 594 | 49.92% |
| PT | 3,696 | 1,750 | 47.35% |
| ID | 412 | 188 | 45.63% |

### Important Interpretation

A high DC percentage does **not necessarily mean a country has the largest number of DC charging stations**.

For example:

- South Korea has a very high DC percentage but only 161 stations in this filtered analysis.
- The United States has many more DC stations overall but a lower DC percentage relative to its total station count.

Therefore, both **absolute DC station count** and **DC penetration percentage** should be considered.

---

# 11. DC Penetration for Countries With ≥500 Stations

A more conservative analysis was also performed by considering countries with at least **500 charging stations**.

### Results

| Country | Total Stations | DC Stations | DC Percentage |
|:---:|---:|---:|---:|
| UA | 555 | 502 | 90.45% |
| RU | 2,174 | 1,542 | 70.93% |
| AU | 1,219 | 662 | 54.31% |
| LT | 765 | 399 | 52.16% |
| TR | 1,190 | 594 | 49.92% |
| PT | 3,696 | 1,750 | 47.35% |
| NZ | 789 | 357 | 45.25% |
| BR | 624 | 259 | 41.51% |
| IN | 1,170 | 449 | 38.38% |
| ES | 17,795 | 6,324 | 35.54% |
| NO | 4,662 | 1,408 | 30.20% |
| MY | 608 | 182 | 29.93% |
| IT | 9,935 | 2,288 | 23.03% |

This filtering gives a more stable comparison because countries with only a handful of stations are excluded.

---

# 12. Charging Power Outlier Analysis

The distribution of charging power was visualized using a histogram.

The original distribution was highly compressed near the lower values because of extreme observations.

The analysis showed:

```text
Mean ≈ 35.26 kW before removing the extreme value
Median = 11 kW
Maximum = 1,000,000 kW
```

After removing the erroneous observation:

```text
Mean ≈ 31.05 kW
Median = 11 kW
Maximum = 1,000 kW
```

This demonstrates the significant influence of the erroneous extreme value on the mean and standard deviation.

---

# 13. Charging Power Distribution Below 1,000 kW

To better visualize the normal range of charging power, stations below 1,000 kW were plotted.

Example:

```python
power_normal = df_clean[
    df_clean["power_kw"] < 1000
]["power_kw"]

sns.histplot(
    power_normal,
    bins=50,
    kde=True
)
```

The visualization shows a strong concentration of charging stations at lower power levels and a long tail toward higher charging power.

---

# 14. Log Transformation

Because charging power is highly right-skewed, a logarithmic transformation was applied.

```python
power_log = np.log1p(df_clean["power_kw"])
```

`np.log1p(x)` calculates:

```text
log(1 + x)
```

This is useful because it:

- Reduces the influence of extreme values
- Compresses the long right tail
- Makes lower and higher values easier to compare
- Helps visualize highly skewed data

### Example

```python
np.exp([1.6, 2.1, 2.5, 3.1, 3.8]) - 1
```

produces approximately:

```text
[ 3.95,  7.17, 11.18, 21.20, 43.70 ]
```

This demonstrates how log-transformed values can be interpreted back on the original charging-power scale.

---

# 15. Skewness and Kurtosis

The distribution was further evaluated using skewness and kurtosis.

```python
print("Skewness:", df_clean["power_kw"].skew())
print("Kurtosis:", df_clean["power_kw"].kurt())
```

### Results

```text
Skewness ≈ 3.8316
Kurtosis ≈ 17.2054
```

### Interpretation

#### Skewness

A skewness value of approximately **3.83** indicates strong positive/right skewness.

This means:

- Most observations are concentrated toward lower charging-power values.
- A smaller number of high-power stations create a long right tail.

#### Kurtosis

A kurtosis value of approximately **17.21** indicates a heavy-tailed distribution with substantial extreme observations.

This supports the earlier outlier analysis.

---

# Visualizations

The project includes the following visualizations:

## Country Analysis

- Top 15 countries by number of EV charging stations
- Top countries by total charging ports
- Top countries by average ports per station

## Charging Power Analysis

- Distribution of EV charging power
- Distribution of charging power below 1,000 kW
- Log-transformed charging-power distribution
- Charging stations by power class

## DC Charging Analysis

- Top 15 countries by DC fast-charging stations
- DC charging penetration versus total charging stations

---

# Key Business Insights

## 1. Charging Infrastructure Is Geographically Concentrated

A relatively small number of countries account for a large proportion of the charging infrastructure represented in the dataset.

This indicates that EV infrastructure development is not evenly distributed globally.

---

## 2. The United States Has the Largest Absolute Infrastructure

The US has:

```text
81,357 total charging stations
14,298 DC charging stations
```

Therefore, it has the largest absolute charging infrastructure among the countries shown in the analysis.

---

## 3. Station Count Alone Does Not Tell the Full Story

Two countries may have a similar number of stations but very different numbers of ports.

Therefore, both:

```text
Number of Stations
```

and:

```text
Number of Charging Ports
```

should be considered when evaluating charging infrastructure.

---

## 4. Average Ports per Station Provides Additional Context

The average ports-per-station metric helps identify countries where charging stations tend to contain more charging points.

This is useful for evaluating the potential capacity of individual charging locations.

---

## 5. AC Charging Dominates the Dataset

The largest power class is:

```text
AC_L1 (<7.5 kW)
```

with:

```text
107,111 stations
45.06%
```

This shows that lower-power AC charging represents a substantial part of the infrastructure.

---

## 6. DC Ultra-Fast Charging Is Less Common

The `DC_ULTRA_(>=150kW)` category contains approximately:

```text
13,503 stations
5.68%
```

This is considerably smaller than the AC_L1 category.

This suggests that ultra-fast charging infrastructure is still a smaller portion of the overall station distribution represented in the dataset.

---

## 7. Charging Power Has Distinct Tiers

The most common charging-power values reveal clear clusters around:

```text
3.7 kW
7–7.4 kW
11 kW
22 kW
50 kW
150 kW
250 kW
350 kW
```

This suggests that charging infrastructure is built around commonly used power configurations.

---

## 8. High-Power Chargers Create a Long Tail

Although most charging stations have relatively low charging power, some stations provide substantially higher power.

This produces:

```text
Mean > Median
```

and strong positive skewness.

---

## 9. Data Quality Has a Significant Effect on Analysis

The erroneous:

```text
1,000,000 kW
```

record substantially affected the mean and standard deviation.

Removing the record changed the distribution and made the statistical analysis more representative.

This demonstrates why **data validation and outlier investigation are important before drawing conclusions from a dataset**.

---

# Technical Implementation

## Libraries

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```

---

## Country-Level Station Analysis

```python
stations_by_country = (
    df_clean
    .groupby("country_code")["id"]
    .count()
    .sort_values(ascending=False)
    .reset_index(name="station_count")
)
```

---

## Country-Level Port Analysis

```python
country_avg_ports = (
    df_clean
    .groupby("country_code")
    .agg(
        total_stations=("id", "count"),
        total_ports=("ports", "sum")
    )
    .reset_index()
)

country_avg_ports["avg_ports_per_station"] = (
    country_avg_ports["total_ports"]
    / country_avg_ports["total_stations"]
)
```

---

## Power-Class Analysis

```python
power_class_count = (
    df_clean["power_class"]
    .value_counts()
    .reset_index()
)

power_class_count.columns = [
    "power_class",
    "station_count"
]

power_class_count["percentage"] = (
    power_class_count["station_count"]
    / len(df_clean)
    * 100
)
```

---

## DC Charging Analysis

```python
dc_classes = [
    "DC_FAST_(50-149kW)",
    "DC_ULTRA_(>=150kW)"
]

country_dc_analysis = (
    df_clean
    .groupby("country_code")
    .agg(
        total_stations=("id", "count"),
        dc_stations=(
            "power_class",
            lambda x: x.isin(dc_classes).sum()
        )
    )
    .reset_index()
)

country_dc_analysis["dc_percentage"] = (
    country_dc_analysis["dc_stations"]
    / country_dc_analysis["total_stations"]
    * 100
)
```

---

## Outlier Removal

```python
df_clean = df_clean[
    df_clean["power_kw"] != 1_000_000
].copy()

df_clean.reset_index(
    drop=True,
    inplace=True
)
```

---

## Charging-Power Distribution

```python
plt.figure(figsize=(12, 6))

sns.histplot(
    df_clean["power_kw"],
    bins=50,
    kde=True
)

plt.title("Distribution of EV Charging Power")
plt.xlabel("Charging Power (kW)")
plt.ylabel("Number of Stations")

plt.tight_layout()
plt.show()
```

---

## Log-Transformed Distribution

```python
power_log = np.log1p(
    df_clean["power_kw"]
)

plt.figure(figsize=(12, 6))

sns.histplot(
    power_log,
    bins=50,
    kde=True
)

plt.title(
    "Log-Transformed Distribution of EV Charging Power"
)

plt.xlabel("log(Charging Power + 1)")
plt.ylabel("Number of Stations")

plt.tight_layout()
plt.show()
```

---

# Project Workflow

```text
Raw Dataset
     ↓
Data Inspection
     ↓
Data Cleaning
     ↓
Outlier Detection
     ↓
Outlier Removal
     ↓
Descriptive Statistics
     ↓
Country-Level Analysis
     ↓
Charging-Port Analysis
     ↓
Charging-Power Analysis
     ↓
Power-Class Analysis
     ↓
DC Charging Analysis
     ↓
Visualization
     ↓
Business Insights
```

---

# Tools and Technologies

| Technology | Purpose |
|:---|:---|
| Python | Main programming language |
| Pandas | Data manipulation and aggregation |
| NumPy | Numerical operations and transformations |
| Matplotlib | Data visualization |
| Seaborn | Statistical visualization |
| Jupyter Notebook | Interactive analysis environment |

---

# Project Structure

```text
EV-Charging-Analysis/
│
├── EV Charging Analysis.ipynb
├── dataset/
│   └── EV charging dataset
│
└── README.md
```

---

# How to Run

## 1. Clone the Repository

```bash
git clone <your-repository-url>
```

## 2. Navigate to the Project

```bash
cd EV-Charging-Analysis
```

## 3. Install Required Libraries

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

## 4. Start Jupyter Notebook

```bash
jupyter notebook
```

Open:

```text
EV Charging Analysis.ipynb
```

Run the notebook cells sequentially.

---

# Questions Answered by the Project

This analysis answers practical questions such as:

1. Which countries have the most EV charging stations?
2. Which countries have the most charging ports?
3. Which countries have the highest average ports per station?
4. What are the most common charging-power levels?
5. Which charging-power class is most common?
6. How many stations belong to DC fast-charging categories?
7. Which countries have the highest DC charging penetration?
8. How does charging power vary across stations?
9. Are there extreme or erroneous charging-power values?
10. How does an outlier affect mean and standard deviation?
11. What does the charging-power distribution look like?
12. Does the charging-power distribution contain skewness?
13. Does log transformation make the distribution easier to analyze?
14. How does charging infrastructure differ between countries?
15. What charging-power tiers are most common?

---

# Final Conclusion

This project demonstrates a complete **Python-based Exploratory Data Analysis workflow** for EV charging infrastructure.

The analysis covered:

- Data cleaning
- Data validation
- Outlier detection
- Descriptive statistics
- Country-level aggregation
- Charging-port analysis
- Charging-power analysis
- Power-class analysis
- DC fast-charging analysis
- DC penetration analysis
- Skewness and kurtosis
- Log transformation
- Data visualization
- Business interpretation

### Main Findings

- EV charging infrastructure is concentrated in a limited number of countries.
- The United States has the largest absolute number of charging stations and DC fast-charging stations in the analyzed data.
- Lower-power AC charging represents the largest power class.
- Charging power is strongly right-skewed.
- The median charging power is much lower than the mean.
- Several standardized charging-power levels occur frequently.
- DC charging penetration varies significantly between countries.
- High-power charging stations form a long tail in the distribution.
- Data-quality validation is critical because the erroneous `1,000,000 kW` record significantly distorted the original statistics.
- Log transformation provides a clearer view of the highly skewed charging-power distribution.

Overall, the project demonstrates how **Python, Pandas, NumPy, Matplotlib, and Seaborn** can be used to convert raw EV infrastructure data into meaningful analytical and business insights.

---

# Author

**Sridhar Methuku**

**Data Analytics | Python | SQL | Power BI | Data Visualization**
