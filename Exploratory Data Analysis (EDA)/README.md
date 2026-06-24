# Project 2 – Exploratory Data Analysis (EDA)
**DecodeLabs Industrial Training Kit | Batch 2026**

## Objective
Perform Exploratory Data Analysis (EDA) on the cleaned e-commerce order dataset to identify patterns, trends, distributions, and outliers, and extract meaningful business insights.

## Dataset
`cleaned_dataset.csv` — an e-commerce order-level dataset including columns such as `Product`, `PaymentMethod`, `OrderStatus`, `ReferralSource`, `Date`, and `TotalPrice`, among others.

## Requirements
This notebook uses `pandas` and `matplotlib`. Make sure both are imported before running (add the following at the top if not already present):
```python
import pandas as pd
import matplotlib.pyplot as plt
```

## Notebook Structure
The notebook is organized into the following sections, each with a markdown explanation followed by the corresponding code:

### 1. Dataset Preview
- Loads `cleaned_dataset.csv` into a DataFrame.
- Displays the first five (`df.head()`) and last five (`df.tail()`) records to understand structure and contents.

### 2. Dataset Dimensions
- `df.shape` — total number of rows and columns.

### 3. Dataset Information
- `df.info()` — data types, number of entries, and memory usage.

### 4. Column Names
- `df.columns` — lists all column names in the dataset.

### 5. Checking Missing Values
- `df.isnull().sum()` — counts null values per column to confirm data completeness.

### 6. Statistical Summary
- `df.describe()` — descriptive statistics (mean, std, min, max, quartiles) for numerical columns.
- `df.mean(numeric_only=True)` — mean of each numerical column.
- `df.median(numeric_only=True)` — median of each numerical column.
- `df.count()` — count of non-null observations per column.

### 7. Product Distribution
- `df["Product"].value_counts()` — frequency of each product.
- Bar chart visualizing product frequency.

### 8. Payment Method Analysis
- `df["PaymentMethod"].value_counts()` — frequency of each payment method.
- Pie chart showing the percentage share of each payment method.

### 9. Order Status Analysis
- `df["OrderStatus"].value_counts()` — frequency of each order status.
- Bar chart visualizing order status distribution.

### 10. Referral Source Analysis
- `df["ReferralSource"].value_counts()` — frequency of each referral channel.
- Bar chart visualizing referral source distribution.

### 11. Sales Trend Analysis
- Converts `Date` to datetime and extracts the `Month` number.
- Groups `TotalPrice` by month to get total monthly sales.
- Line plot (with markers) showing the monthly sales trend.

### 12. Outlier Detection
- Boxplot of `TotalPrice` to visually identify outliers.

### 13. Correlation Analysis
- Computes the correlation matrix for all numeric columns (`int64`/`float64`).
- Visualizes the matrix as a heatmap using `imshow()`, with a colorbar and labeled axes.

### 14. Key Findings and Observations
A markdown summary covering:
1. Successful loading and exploration of the dataset.
2. Examination of dataset dimensions and structure.
3. Missing-value check confirming data completeness.
4. Descriptive statistics on numerical variables.
5. Most frequently ordered products.
6. Customer payment method preferences.
7. Distribution of order statuses (completed, pending, cancelled, etc.).

> **Note:** Point 8 of the Key Findings section is currently incomplete in the notebook — add a closing observation (e.g. on referral source performance, sales trend, or correlation results) before submission.

## How to Run
1. Ensure `cleaned_dataset.csv` is in the same folder as the notebook.
2. Open in Jupyter Notebook or Google Colab.
3. Add the missing `import pandas as pd` / `import matplotlib.pyplot as plt` line if not already present in an earlier cell.
4. Run all cells in order (Jupyter: **Cell → Run All**; Colab: **Runtime → Run all**).

## Tools Used
Python, pandas, Matplotlib (Jupyter Notebook environment).
