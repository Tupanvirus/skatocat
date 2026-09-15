# skatocat 
## Processing of results for triboelectric series featuring cat fur as the reference element


## Abstract

This repository presents a computational pipeline for the automated analysis of experimental potentiometric data obtained from repeated trials on various materials (cotton, polyester, cat fur, leather, nitrile). The script ingests Excel files, performs robust preprocessing—including Isolation Forest for global outlier detection, RANSAC for local outlier removal, and Savitzky–Golay filtering for noise reduction—and evaluates a set of candidate mathematical models (linear, polynomial, logarithmic, exponential, power, and root functions). The best-fitting model is selected based on the coefficient of determination (\(R^2\)). The pipeline produces publication-quality visualisations and explicit model equations, facilitating reproducible analysis and interpretation of experimental trends.

This project is intended as a portfolio piece demonstrating practical skills in scientific computing, data cleaning, regression analysis, and reproducible research workflows.

---

## 1. Introduction

Experimental studies often yield datasets containing noise, missing values, and outliers. Manual analysis is time-consuming and subjective. This project provides an automated, extensible framework for processing such data, with a focus on:

- **Robust preprocessing** to mitigate the influence of outliers.
- **Model selection** among a family of common functional forms.
- **Quantitative evaluation** using \(R^2\).
- **Clear visualisation** and equation reporting.

The pipeline is particularly suited for potentiometric measurements across repeated attempts, where the relationship between attempt number and measured potential may follow non-linear trends.

---

## 2. Features

- **Excel ingestion** (`.xlsx`, `.xls`) with automatic detection of numeric columns.
- **Grouping and aggregation** of series by material (e.g., cotton, polyester, nitrile).
- **Scaling and concatenation** of multiple series for combined analysis.
- **Outlier detection**:
  - Isolation Forest (global outliers)
  - RANSAC (local outliers)
- **Smoothing**: Savitzky–Golay filter.
- **Model fitting**: linear, quadratic, cubic, logarithmic, exponential, power, and root models.
- **Model selection** via \(R^2\).
- **Equation formatting** for polynomial models (e.g., \(y = 3.45x^2 + 1.23x - 0.5\)).
- **Visualisation**: raw data, filtered inliers, smoothed curves, RANSAC fits, and best-model curves.
- **Console reporting** of extrema, \(R^2\), and equations.

---

## 3. Methodology

The pipeline follows a sequential workflow:

1. **Data Loading**  
   The Excel file is read into a pandas DataFrame. The first column is assumed to represent the trial number (`attempts`); all subsequent numeric columns are treated as experimental measurements.

2. **Grouping and Aggregation**  
   Columns are grouped by material keywords. For each group, row-wise means are computed. Optionally, multiple series for the same material are scaled and concatenated to form a single dataset.

3. **Outlier Detection**  
   - **Isolation Forest**: Applied to the \((x, y)\) pairs to remove global outliers. The contamination parameter controls the expected proportion of outliers.
   - **RANSAC**: A robust regression technique that fits a polynomial (default degree 2) while iteratively identifying inliers.

4. **Smoothing**  
   The Savitzky–Golay filter is applied to the RANSAC-cleaned data to reduce noise while preserving signal shape.

5. **Model Fitting**  
   The following models are fitted to the cleaned data:
   - Linear: \(y = ax + b\)
   - Quadratic: \(y = ax^2 + bx + c\)
   - Cubic: \(y = ax^3 + bx^2 + cx + d\)
   - Logarithmic: \(y = a \ln(x) + b\)
   - Exponential: \(y = a e^{bx}\)
   - Power: \(y = a x^b\)
   - Root: \(y = a \sqrt{x} + b\)

   For non-linear models, linear regression is performed on transformed variables (e.g., \(\ln y\) vs. \(x\)).

6. **Model Selection**  
   The model with the highest \(R^2\) on the cleaned data is selected as the best fit.

7. **Visualisation and Reporting**  
   For each series or aggregated group, the pipeline generates a plot showing:
   - Raw data
   - Isolation Forest inliers
   - RANSAC-cleaned data
   - Savitzky–Golay smoothed curve
   - RANSAC model curve
   - Best-fitting model curve

   Equations and \(R^2\) values are displayed in the legend and printed to the console.

---

## 4. Data Requirements

### Input Format

The input must be an Excel file with the following structure:

- **First column**: trial number (integer or float).
- **Remaining columns**: experimental measurements (numeric).

Example:

| Attempt | Cotton (Series 1) | Cotton (Series 2) | Nitrile (Series 1) | ... |
|--------:|------------------:|------------------:|-------------------:|---:|
| 1       | 12.4              | 11.8              | 9.7                | ... |
| 2       | 13.1              | 12.5              | 9.9                | ... |
| 3       | 14.0              | 13.2              | 10.4               | ... |

### Column Naming Conventions

Grouping relies on keywords in column names. The script recognises:

- `хлопок` (cotton)
- `полиэстер` (polyester)
- `шерсть кота` (cat fur)
- `кожа` (leather)
- `нитрил (серия 1)`, `нитрил (серия 2)`, ... (nitrile series)

If your column names differ, adjust the `material_groups_to_process` and `nitril_group_*` lists in the script.

### Data Quality

- Non-numeric columns are ignored.
- Rows with `NaN` are dropped before analysis.
- Columns with fewer than 2 valid data points are skipped.

---

## 5. Installation

### Dependencies

- Python 3.9+
- `numpy`
- `pandas`
- `matplotlib`
- `scikit-learn`
- `scipy`
- `openpyxl`

Create a `requirements.txt` file:

```txt
numpy
pandas
matplotlib
scikit-learn
scipy
openpyxl
```

Install with:

```bash
pip install -r requirements.txt
```

---

## 6. Usage

### Google Colab

1. Open the notebook or script in Google Colab.
2. Run the cells sequentially.
3. When prompted, upload your Excel file via `files.upload()`.
4. Review the generated plots and printed equations.

### Local Execution

Replace the Colab-specific upload block:

```python
from google.colab import files
df = files.upload()
```

with:

```python
import pandas as pd
df = pd.read_excel("results.xlsx")
```

Then run:

```bash
python обсчет_скатокошки.py
```

---

## 7. Output

The script produces:

- **Console output**:
  - Number of outliers detected by Isolation Forest and RANSAC.
  - Best model type and \(R^2\).
  - Polynomial equation (if applicable).
  - Coordinates of extrema.

- **Plots**:
  - Scatter plot of raw data.
  - Overlay of Isolation Forest inliers.
  - RANSAC-cleaned points.
  - Savitzky–Golay smoothed curve.
  - RANSAC model curve.
  - Best-fitting model curve.
  - Legend containing model type, \(R^2\), and equation.

Example console output:

```text
Processing series: Cotton (Series 1)
  Isolation Forest identified 3 outliers, 17 inliers.
  RANSAC identified 1 local outlier, 16 inliers.
  Savitzky-Golay smoothing applied with window_length=5, polyorder=2.
  Best-fitting model: poly2 (R²=0.987).
  Best Model Extrema: Min Y=10.234 (at X=2.345), Max Y=15.678 (at X=8.901)
  Best Model Equation: y = 0.012x^2 + 1.45x + 3.2
```

---

## 8. Customisation

Key parameters can be adjusted in the script:

- `contamination` in `isolation_forest_filter` – expected proportion of global outliers.
- `degree` in `PolynomialFeatures` within `ransac_fit` – polynomial degree for RANSAC.
- `window_length` and `polyorder` in `savitzky_golay_smooth` – smoothing parameters.
- Material grouping lists – adapt to your column naming.

---

## 9. Limitations and Future Work

- The pipeline assumes the first column is the independent variable (trial number).
- Grouping is keyword-based; non-standard column names require manual configuration.
- RANSAC requires a minimum number of samples (typically ≥4) for quadratic fits.
- Model selection is limited to the provided set; other functional forms could be added.
- Outlier removal is heuristic; visual inspection is recommended.
- The script currently does not export results to files; adding CSV/JSON export is straightforward.

Future enhancements could include:

- Automated parameter tuning via cross-validation.
- Support for multiple independent variables.
- Integration with experiment tracking tools (e.g., MLflow).
- Interactive dashboards (e.g., Streamlit).

---

## 10. Project Structure

```text
.
├── обсчет_скатокошки.py     # Main analysis script
├── results.xlsx             # Example input data
├── requirements.txt         # Python dependencies
└── README.md                # This file
```

---

## 11. License

This project is released under the MIT License. See `LICENSE` for details.

---

## 12. Author

**<Your Name>**  
<Your Email>  
<Your GitHub / LinkedIn>

---

*This README is part of a portfolio demonstrating expertise in scientific computing, data analysis, and reproducible research. The pipeline is intentionally modular and extensible, reflecting best practices in computational science.*
```
