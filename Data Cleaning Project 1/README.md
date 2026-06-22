# 🧹 DecodeLabs Data Analytics Internship — Project 1
## Data Cleaning and Preparation

---

**Intern:** Data Analytics Intern  
**Organization:** DecodeLabs  
**Project:** Project 1 — Data Cleaning and Preparation  
**Dataset:** E-Commerce Orders Dataset (1,200 records × 14 features)  
**Tools:** Python 3, Pandas, NumPy  
**Date:** June 2026  

---

> **Objective:** Load a real-world e-commerce orders dataset, perform systematic data quality assessment, apply professional data cleaning techniques, and deliver a production-ready cleaned dataset with full documentation.
## 📦 Step 1: Import Libraries
# ─────────────────────────────────────────────────────────────────
# Step 1 | Import all required libraries
# ─────────────────────────────────────────────────────────────────
import pandas as pd          # Core data manipulation library
import numpy as np           # Numerical operations and statistical functions
import warnings              # Suppress non-critical warnings for clean output

warnings.filterwarnings('ignore')  # Suppress pandas future/deprecation warnings

# Display settings — show all columns and limit float precision for readability
pd.set_option('display.max_columns', None)
pd.set_option('display.float_format', lambda x: f'{x:.2f}')
pd.set_option('display.width', 120)

print('✅ Libraries imported successfully.')
print(f'   Pandas  version : {pd.__version__}')
print(f'   NumPy   version : {np.__version__}')
## 📂 Step 2: Load the Dataset
# ─────────────────────────────────────────────────────────────────
# Step 2 | Load the raw dataset from Excel into a Pandas DataFrame
# ─────────────────────────────────────────────────────────────────

# Load the Excel file — Sheet1 contains all 1,200 order records
df_raw = pd.read_excel('Dataset for Data Analytics.xlsx', sheet_name='Sheet1')

# Create a working copy so the original stays intact for comparison
df = df_raw.copy()

print('✅ Dataset loaded successfully!')
print(f'   Shape  : {df.shape[0]:,} rows  ×  {df.shape[1]} columns')
print(f'   Memory : {df.memory_usage(deep=True).sum() / 1024:.1f} KB')
## 👀 Step 3: Initial Data Exploration
# ─────────────────────────────────────────────────────────────────
# Step 3a | Display the first 5 rows to understand data structure
# ─────────────────────────────────────────────────────────────────
print('=' * 80)
print('FIRST 5 ROWS OF THE DATASET')
print('=' * 80)
df.head()
# ─────────────────────────────────────────────────────────────────
# Step 3b | Display dataset information (dtypes, non-null counts)
# ─────────────────────────────────────────────────────────────────
print('=' * 80)
print('DATASET INFORMATION')
print('=' * 80)
df.info()
# ─────────────────────────────────────────────────────────────────
# Step 3c | Statistical summary of numerical columns
# ─────────────────────────────────────────────────────────────────
print('=' * 80)
print('STATISTICAL SUMMARY — NUMERICAL COLUMNS')
print('=' * 80)
df.describe()
# ─────────────────────────────────────────────────────────────────
# Step 3d | Statistical summary of categorical / object columns
# ─────────────────────────────────────────────────────────────────
print('=' * 80)
print('STATISTICAL SUMMARY — CATEGORICAL COLUMNS')
print('=' * 80)
df.describe(include=['object', 'string'])
## 🔍 Step 4: Identify Missing / Null Values
# ─────────────────────────────────────────────────────────────────
# Step 4 | Identify all missing values using isnull().sum()
# ─────────────────────────────────────────────────────────────────

missing_counts = df.isnull().sum()
missing_pct    = (missing_counts / len(df) * 100).round(2)

missing_report = pd.DataFrame({
    'Column'        : missing_counts.index,
    'Missing Count' : missing_counts.values,
    'Missing %'     : missing_pct.values,
    'Data Type'     : df.dtypes.values
}).set_index('Column')

# Display only columns that have at least 1 missing value
affected = missing_report[missing_report['Missing Count'] > 0]

print('=' * 60)
print('MISSING VALUE ANALYSIS')
print('=' * 60)
if affected.empty:
    print('  ✅  No missing values detected in any column.')
else:
    print(affected.to_string())

print(f'\n  Total cells       : {df.size:,}')
print(f'  Total missing     : {df.isnull().sum().sum():,}')
print(f'  Columns affected  : {(missing_counts > 0).sum()}')
print(f'  Overall missing % : {df.isnull().sum().sum() / df.size * 100:.2f}%')
## 🛠️ Step 5: Handle Missing Values

**Finding:** Column `CouponCode` has **309 missing values** (25.75% of rows).  
**Strategy:** `CouponCode` is a categorical column → fill with **Mode** (most frequent value).  
This represents customers who did not apply any coupon; filling with Mode preserves the existing distribution of coupon usage patterns without distorting the data.
# ─────────────────────────────────────────────────────────────────
# Step 5 | Handle missing values using appropriate strategy
# ─────────────────────────────────────────────────────────────────

# ── 5a: Numerical columns — check skewness to choose Mean vs Median ─
numerical_cols = df.select_dtypes(include=['number']).columns.tolist()

print('Skewness analysis of numerical columns:')
print('-' * 55)
for col in numerical_cols:
    skew   = df[col].skew()
    method = 'Mean' if abs(skew) <= 1 else 'Median'
    print(f'  {col:15s}  skew = {skew:+.2f}  → strategy: {method}')

# No numerical columns have missing values — confirming no action needed
print('\n  ✅ No missing values in numerical columns. No imputation required.')

# ── 5b: Categorical columns — fill with Mode ─────────────────────
print('\nCategorical column treatment:')
print('-' * 55)
categorical_missing = ['CouponCode']   # Only affected categorical column

for col in categorical_missing:
    mode_value     = df[col].mode()[0]           # Compute most frequent value
    missing_before = df[col].isnull().sum()      # Count before filling
    df[col]        = df[col].fillna(mode_value)  # Fill NaNs with mode
    missing_after  = df[col].isnull().sum()      # Count after filling

    print(f'  Column        : {col}')
    print(f'  Mode value    : "{mode_value}"')
    print(f'  Nulls before  : {missing_before}')
    print(f'  Nulls after   : {missing_after}')

print('\n✅ Missing value treatment complete.')
## 📋 Step 6: Identify and Remove Duplicate Rows
# ─────────────────────────────────────────────────────────────────
# Step 6 | Detect and remove duplicate rows
# ─────────────────────────────────────────────────────────────────

dup_count = df.duplicated().sum()
print('=' * 50)
print('DUPLICATE ROW ANALYSIS')
print('=' * 50)
print(f'  Total rows before : {len(df):,}')
print(f'  Duplicate rows    : {dup_count:,}')

if dup_count > 0:
    print('\n  Sample duplicate rows:')
    print(df[df.duplicated(keep=False)].head(4))
    # Remove all duplicate rows, keeping the first occurrence
    df = df.drop_duplicates(keep='first').reset_index(drop=True)
    print(f'\n  ✅ Duplicates removed. Rows remaining: {len(df):,}')
else:
    print('\n  ✅ No duplicate rows found. No action required.')

print(f'  Total rows after  : {len(df):,}')
## 🗓️ Step 7: Correct Data Formats
# ─────────────────────────────────────────────────────────────────
# Step 7a | Standardize Date column to YYYY-MM-DD string format
# ─────────────────────────────────────────────────────────────────

print('Date column BEFORE formatting:')
print(f'  dtype  : {df["Date"].dtype}')
print(f'  sample : {df["Date"].head(3).tolist()}')

# Convert to datetime, then format as YYYY-MM-DD string
df['Date'] = pd.to_datetime(df['Date'], errors='coerce').dt.strftime('%Y-%m-%d')

print('\nDate column AFTER formatting:')
print(f'  dtype  : {df["Date"].dtype}')
print(f'  sample : {df["Date"].head(3).tolist()}')
print('  ✅ Date column standardized to YYYY-MM-DD format.')
# ─────────────────────────────────────────────────────────────────
# Step 7b | Strip leading/trailing whitespace from all text columns
# ─────────────────────────────────────────────────────────────────

text_columns = df.select_dtypes(include=['object', 'string']).columns.tolist()
print(f'Text columns to clean ({len(text_columns)} columns):')
print('-' * 50)

for col in text_columns:
    before = df[col].astype(str).copy()
    df[col] = df[col].astype(str).str.strip()
    spaces_removed = (before != df[col]).sum()
    status = f'{spaces_removed} cells cleaned' if spaces_removed > 0 else 'no extra spaces ✓'
    print(f'  {col:22s} : {status}')

print('\n✅ Whitespace stripping complete.')
# ─────────────────────────────────────────────────────────────────
# Step 7c | Standardize text capitalization to Title Case
# ─────────────────────────────────────────────────────────────────
# Applied to all descriptive categorical columns

capitalize_cols = ['Product', 'PaymentMethod', 'OrderStatus',
                   'CouponCode', 'ReferralSource']

print('Capitalization standardization (→ Title Case):')
print('-' * 60)
for col in capitalize_cols:
    df[col] = df[col].str.title()
    print(f'  {col:20s} → {sorted(df[col].unique().tolist())}')

print('\n✅ Capitalization standardization complete.')
# ─────────────────────────────────────────────────────────────────
# Step 7d | Ensure numerical columns have correct data types
# ─────────────────────────────────────────────────────────────────

print('Data type corrections:')
print('-' * 50)

# Quantity and ItemsInCart → integers (whole numbers)
for col in ['Quantity', 'ItemsInCart']:
    before_dtype = df[col].dtype
    df[col] = pd.to_numeric(df[col], errors='coerce').astype('Int64')
    print(f'  {col:15s} : {before_dtype} → {df[col].dtype}')

# UnitPrice and TotalPrice → float64 (decimal numbers)
for col in ['UnitPrice', 'TotalPrice']:
    before_dtype = df[col].dtype
    df[col] = pd.to_numeric(df[col], errors='coerce').astype('float64')
    print(f'  {col:15s} : {before_dtype} → {df[col].dtype}')

print('\n✅ Data type corrections applied.')
print('\nFinal column dtypes:')
print(df.dtypes.to_string())
## ✅ Step 8: Post-Cleaning Data Quality Verification
# ─────────────────────────────────────────────────────────────────
# Step 8 | Comprehensive post-cleaning quality verification
# ─────────────────────────────────────────────────────────────────
import re

print('=' * 65)
print('          POST-CLEANING DATA QUALITY REPORT')
print('=' * 65)

# ── Verification 1: Missing Values ───────────────────────────────
total_missing = df.isnull().sum().sum()
mv_pass  = total_missing == 0
mv_icon  = '✅ PASS' if mv_pass else f'❌ FAIL ({total_missing} remaining)'
print(f'\n  [1] Missing Values          : {mv_icon}')
print('       Per column breakdown   :')
for col, cnt in df.isnull().sum().items():
    flag = '  ✅' if cnt == 0 else '  ❌'
    print(f'{flag}  {col:20s} : {cnt}')

# ── Verification 2: Duplicate Rows ───────────────────────────────
dup_count = df.duplicated().sum()
dup_pass  = dup_count == 0
dup_icon  = '✅ PASS' if dup_pass else f'❌ FAIL ({dup_count} duplicates)'
print(f'\n  [2] Duplicate Rows          : {dup_icon}')

# ── Verification 3: Date Format ──────────────────────────────────
date_pattern   = r'^\d{4}-\d{2}-\d{2}$'
invalid_dates  = df['Date'].apply(
    lambda x: not bool(re.match(date_pattern, str(x)))
).sum()
date_pass = invalid_dates == 0
date_icon = '✅ PASS' if date_pass else f'❌ FAIL ({invalid_dates} invalid)'
print(f'\n  [3] Date Format (YYYY-MM-DD): {date_icon}')
print(f'       Sample: {df["Date"].head(3).tolist()}')

# ── Verification 4: Numerical Types ──────────────────────────────
print('\n  [4] Numerical Data Types    :')
type_checks = {'Quantity': 'Int64', 'ItemsInCart': 'Int64',
               'UnitPrice': 'float64', 'TotalPrice': 'float64'}
all_types_ok = True
for col, expected in type_checks.items():
    actual = str(df[col].dtype)
    ok = actual == expected
    all_types_ok = all_types_ok and ok
    icon = '  ✅' if ok else '  ⚠️'
    print(f'{icon}  {col:15s} : {actual}')

# ── Overall Summary ──────────────────────────────────────────────
all_pass = mv_pass and dup_pass and date_pass and all_types_ok
print(f"\n{'=' * 65}")
print(f'  FINAL DATASET SHAPE  : {df.shape[0]:,} rows × {df.shape[1]} columns')
grade = '100% — All checks passed ✅' if all_pass else 'Review required ⚠️'
print(f'  DATA QUALITY SCORE   : {grade}')
print(f"{'=' * 65}")
## 💾 Step 9: Export Cleaned Dataset
# ─────────────────────────────────────────────────────────────────
# Step 9 | Export the cleaned DataFrame to cleaned_dataset.csv
# ─────────────────────────────────────────────────────────────────

output_path = 'cleaned_dataset.csv'

# utf-8-sig encoding adds BOM so Excel opens the file correctly
df.to_csv(output_path, index=False, encoding='utf-8-sig')

print('=' * 55)
print('  DATASET SAVED SUCCESSFULLY')
print('=' * 55)
print(f'  File     : {output_path}')
print(f'  Rows     : {len(df):,}')
print(f'  Columns  : {len(df.columns)}')
print(f'  Encoding : UTF-8 with BOM (Excel-compatible)')
print('=' * 55)

# Read-back verification
df_check = pd.read_csv(output_path)
print(f'\n  Read-back check: {df_check.shape[0]:,} rows × {df_check.shape[1]} columns ✅')
## 📊 Step 10: Final Cleaned Dataset Preview
# ─────────────────────────────────────────────────────────────────
# Step 10 | Final preview of the cleaned and formatted dataset
# ─────────────────────────────────────────────────────────────────
print('=' * 80)
print('CLEANED DATASET — FIRST 10 ROWS')
print('=' * 80)
df.head(10)
# Final dataset info after all cleaning operations
print('=' * 80)
print('CLEANED DATASET — FINAL INFO')
print('=' * 80)
df.info()
print(f'\n  Unique values per categorical column:')
for col in ['Product', 'PaymentMethod', 'OrderStatus', 'CouponCode', 'ReferralSource']:
    print(f'  {col:20s}: {sorted(df[col].unique().tolist())}')
## 📋 Change Log

| Change ID | Description of Change | Impact | Status |
|-----------|----------------------|--------|--------|
| CLN-001 | Loaded raw dataset from Excel (`Dataset for Data Analytics.xlsx`, 1,200 rows × 14 cols) | Dataset available for processing | ✅ Done |
| CLN-002 | Identified 309 missing values in `CouponCode` column (25.75% of rows) | Data quality issue documented | ✅ Done |
| CLN-003 | Filled `CouponCode` nulls using **Mode imputation** (`Save10`) — categorical best practice | 309 missing values resolved; zero nulls remain | ✅ Done |
| CLN-004 | Confirmed **zero duplicate rows** in the dataset — no action needed | Data integrity verified | ✅ Done |
| CLN-005 | Standardized `Date` column to **YYYY-MM-DD** string format via `pd.to_datetime().dt.strftime()` | Consistent date representation across all 1,200 records | ✅ Done |
| CLN-006 | Stripped leading/trailing whitespace from all 9 text columns using `.str.strip()` | Prevents mismatches in groupby/merge operations | ✅ Done |
| CLN-007 | Applied **Title Case** to `Product`, `PaymentMethod`, `OrderStatus`, `CouponCode`, `ReferralSource` | Uniform categorical values; prevents duplicated categories | ✅ Done |
| CLN-008 | Converted `Quantity` and `ItemsInCart` to `Int64`; `UnitPrice` and `TotalPrice` to `float64` | Correct types for arithmetic, aggregation, and ML pipelines | ✅ Done |
| CLN-009 | Post-cleaning validation: 0 missing values, 0 duplicates, all dates in YYYY-MM-DD | Full data quality confirmation — 100% pass | ✅ Done |
| CLN-010 | Exported cleaned dataset to `cleaned_dataset.csv` (UTF-8 BOM, Excel-compatible) | Production-ready file delivered | ✅ Done |

## 🏁 Project Summary

### Results Achieved

| Metric | Before Cleaning | After Cleaning |
|--------|----------------|----------------|
| Total Rows | 1,200 | 1,200 |
| Total Columns | 14 | 14 |
| Missing Values | **309** (CouponCode only) | **0** ✅ |
| Duplicate Rows | 0 | 0 ✅ |
| Date Format Compliance | datetime64 (no standard format) | 100% YYYY-MM-DD ✅ |
| Correct Numerical Types | Partial | 100% ✅ |
| Text Standardization | Inconsistent | Title Case ✅ |
| Whitespace Issues | Present | Fully resolved ✅ |

### Conclusion

The e-commerce orders dataset was systematically cleaned using industry-standard data preparation techniques. Missing categorical values were imputed using Mode strategy. Dates were standardized to ISO format. Text columns were normalized to Title Case and stripped of whitespace. Numerical columns were cast to their appropriate types. The final `cleaned_dataset.csv` is production-ready for downstream analytics, dashboard visualization, and machine learning pipelines.

---
*DecodeLabs Data Analytics Internship — Project 1 | Completed June 2026*
