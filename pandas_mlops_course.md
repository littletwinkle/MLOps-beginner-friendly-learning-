# Pandas for Testers & MLOps Engineers — Beginner Course

> **Who this is for:** Software testers moving into ML, and beginners learning MLOps fundamentals.
> **Pre-requisite:** numpy_mlops_course.md
> **Environment:** Kaggle Notebooks / Jupyter (pandas pre-installed)
> **Goal:** Use pandas to load, clean, validate, and export structured data for ML pipelines.

---

## Table of Contents
1. [Why Pandas Matters in MLOps](#1-why-pandas-matters-in-mlops)
2. [Creating and Loading DataFrames](#2-creating-and-loading-dataframes)
3. [Exploring Your Data](#3-exploring-your-data)
4. [Selecting Columns and Rows](#4-selecting-columns-and-rows)
5. [Filtering Data](#5-filtering-data)
6. [Handling Missing Values](#6-handling-missing-values)
7. [Adding and Transforming Columns](#7-adding-and-transforming-columns)
8. [Groupby and Aggregation](#8-groupby-and-aggregation)
9. [Merging DataFrames](#9-merging-dataframes)
10. [Saving and Exporting Data](#10-saving-and-exporting-data)
11. [Common Errors and How to Fix Them](#11-common-errors-and-how-to-fix-them)
12. [Mini Project: ML-Ready Data Pipeline](#12-mini-project-ml-ready-data-pipeline)

---

## 1. Why Pandas Matters in MLOps

NumPy works with raw numbers. Pandas works with **labelled, structured data** — the kind that arrives from databases, CSVs, and APIs in the real world.

| Role | How Pandas is Used |
|------|-------------------|
| **Tester** | Validate column names, check data types, detect nulls, verify row counts after transformations |
| **MLOps Engineer** | Load datasets, clean and transform features, encode categories, export ML-ready data |

```python
import pandas as pd
import numpy as np

# Pandas sits between raw data and your ML model
raw_csv      → pd.read_csv()   → clean DataFrame   → np.array / sklearn
```

---

## 2. Creating and Loading DataFrames

### Create from a dictionary
```python
import pandas as pd

data = {
    'name':   ['Alice', 'Bob', 'Carol', 'Dave'],
    'age':    [25, 32, 28, 45],
    'salary': [50000, 75000, 62000, 90000],
    'dept':   ['QA', 'MLOps', 'QA', 'MLOps']
}

df = pd.DataFrame(data)
print(df)
```

Output:
```
    name  age  salary   dept
0  Alice   25   50000     QA
1    Bob   32   75000  MLOps
2  Carol   28   62000     QA
3   Dave   45   90000  MLOps
```

### Load from CSV (most common in real projects)
```python
# From a local file
df = pd.read_csv('data.csv')

# From Kaggle input
df = pd.read_csv('/kaggle/input/your-dataset/data.csv')

# With options
df = pd.read_csv('data.csv',
                 sep=',',
                 header=0,
                 index_col=None,
                 na_values=['NA', 'N/A', '-', ''])
```

### Tester use case — verify load
```python
df = pd.read_csv('/kaggle/input/titanic/train.csv')

# Sanity checks immediately after loading
assert df is not None, "DataFrame is None — load failed"
assert len(df) > 0, "DataFrame is empty"
print(f"Loaded {len(df)} rows and {len(df.columns)} columns")
print(f"Columns: {list(df.columns)}")
```

---

## 3. Exploring Your Data

These are your first commands on any new dataset.

```python
df.head()          # first 5 rows
df.tail(3)         # last 3 rows
df.shape           # (rows, columns)
df.info()          # column names, dtypes, non-null counts
df.describe()      # statistics for numeric columns
df.dtypes          # data type of each column
df.columns         # column names as Index object
df.isnull().sum()  # count of missing values per column
df.duplicated().sum()  # count of duplicate rows
```

### Tester checklist after loading
```python
def validate_dataframe(df, expected_cols, min_rows=1):
    """Basic data quality checks — run after every load."""
    
    # Check columns
    missing_cols = [c for c in expected_cols if c not in df.columns]
    assert not missing_cols, f"Missing columns: {missing_cols}"
    
    # Check row count
    assert len(df) >= min_rows, f"Too few rows: {len(df)}"
    
    # Check for all-null columns
    all_null = df.columns[df.isnull().all()].tolist()
    assert not all_null, f"Completely empty columns: {all_null}"
    
    print(f"Validation passed: {df.shape[0]} rows, {df.shape[1]} cols")

# Usage
validate_dataframe(df, expected_cols=['PassengerId', 'Survived', 'Age', 'Fare'])
```

---

## 4. Selecting Columns and Rows

### Select columns
```python
# Single column → returns a Series
ages = df['age']
print(type(ages))   # <class 'pandas.core.series.Series'>

# Multiple columns → returns a DataFrame
subset = df[['name', 'salary']]
print(type(subset))   # <class 'pandas.core.frame.DataFrame'>
```

### Select rows by position — `.iloc`
```python
df.iloc[0]        # first row (as Series)
df.iloc[0:3]      # first 3 rows
df.iloc[1, 2]     # row 1, column 2 (by number)
df.iloc[-1]       # last row
```

### Select rows by label — `.loc`
```python
df.loc[0]              # row with index label 0
df.loc[0:2]            # rows 0 to 2 INCLUSIVE (unlike Python slicing)
df.loc[0, 'salary']    # row 0, column 'salary' (by name)
df.loc[:, 'age']       # all rows, 'age' column
```

### Tester use case — spot check specific cells
```python
# Verify a known value in a specific position
assert df.loc[0, 'name'] == 'Alice', "Expected 'Alice' in row 0"
assert df.iloc[2, 1] == 28, "Expected age 28 in row 2"
print("Cell-level checks passed.")
```

### MLOps use case — select feature columns
```python
feature_cols = ['age', 'salary']
target_col   = 'dept'

X = df[feature_cols]
y = df[target_col]

print(f"Features: {X.shape}, Target: {y.shape}")
```

---

## 5. Filtering Data

```python
# Single condition
senior = df[df['age'] > 30]

# Multiple conditions — use & and |, wrap each in ()
qateam = df[(df['dept'] == 'QA') & (df['age'] < 30)]

# Exclude values
not_qa = df[df['dept'] != 'QA']

# isin — match a list of values
two_depts = df[df['dept'].isin(['QA', 'MLOps'])]

# String contains
df_str = df[df['name'].str.contains('a', case=False)]
```

### MLOps use case — remove training data outliers
```python
# Remove salary outliers before training
Q1 = df['salary'].quantile(0.25)
Q3 = df['salary'].quantile(0.75)
IQR = Q3 - Q1

clean_df = df[(df['salary'] >= Q1 - 1.5*IQR) &
              (df['salary'] <= Q3 + 1.5*IQR)]

print(f"Removed {len(df) - len(clean_df)} outlier rows")
```

### Tester use case — check no invalid values exist
```python
# Verify no negative salaries exist in the dataset
invalid = df[df['salary'] < 0]
assert len(invalid) == 0, f"Found {len(invalid)} negative salaries"
print("Salary range check passed.")
```

---

## 6. Handling Missing Values

```python
# Detect missing values
df.isnull()          # True/False for each cell
df.isnull().sum()    # count per column
df.isnull().any()    # True if column has any null

# Drop rows with any missing value
df_clean = df.dropna()

# Drop only if ALL values in a row are null
df_clean = df.dropna(how='all')

# Drop if missing in specific columns
df_clean = df.dropna(subset=['age', 'salary'])

# Fill with a constant
df['age'] = df['age'].fillna(0)

# Fill with column mean (most common for ML)
df['salary'] = df['salary'].fillna(df['salary'].mean())

# Forward fill (useful for time-series data)
df['salary'] = df['salary'].ffill()
```

### Tester use case — null audit
```python
def null_audit(df, max_null_pct=0.05):
    """Fail if any column exceeds allowed null percentage."""
    null_pct = df.isnull().mean()
    
    for col, pct in null_pct.items():
        assert pct <= max_null_pct, \
            f"Column '{col}' has {pct:.1%} nulls — exceeds {max_null_pct:.1%} threshold"
    
    print("Null audit passed.")

null_audit(df)
```

### MLOps use case — impute before training
```python
# Standard pre-training null handling
numeric_cols = df.select_dtypes(include='number').columns
for col in numeric_cols:
    df[col] = df[col].fillna(df[col].median())

categorical_cols = df.select_dtypes(include='object').columns
for col in categorical_cols:
    df[col] = df[col].fillna(df[col].mode()[0])

print(f"Remaining nulls: {df.isnull().sum().sum()}")
```

---

## 7. Adding and Transforming Columns

```python
# Add a new column
df['annual_bonus'] = df['salary'] * 0.10

# Derived column
df['experience_level'] = df['age'].apply(
    lambda x: 'junior' if x < 30 else 'senior'
)

# String operations
df['name_upper'] = df['name'].str.upper()
df['name_length'] = df['name'].str.len()

# Rename columns
df = df.rename(columns={'dept': 'department', 'age': 'age_years'})

# Drop columns
df = df.drop(columns=['annual_bonus'])

# Change data type
df['salary'] = df['salary'].astype(float)
```

### MLOps use case — encode categorical column
```python
# One-hot encoding for ML (no sklearn needed for simple cases)
df_encoded = pd.get_dummies(df, columns=['dept'], drop_first=True)
print(df_encoded.columns.tolist())
# ['name', 'age', 'salary', 'dept_MLOps']
```

### Tester use case — verify transformation
```python
# Confirm derived column is correct
df['salary_k'] = df['salary'] / 1000
assert df.loc[0, 'salary_k'] == df.loc[0, 'salary'] / 1000, "Transformation error"
print("Column transformation check passed.")
```

---

## 8. Groupby and Aggregation

```python
# Average salary per department
avg_by_dept = df.groupby('dept')['salary'].mean()
print(avg_by_dept)

# Multiple aggregations at once
summary = df.groupby('dept').agg(
    avg_salary=('salary', 'mean'),
    max_age=('age', 'max'),
    headcount=('name', 'count')
)
print(summary)
```

### MLOps use case — feature engineering
```python
# Add department-level statistics as features
dept_avg = df.groupby('dept')['salary'].transform('mean')
df['salary_vs_dept_avg'] = df['salary'] - dept_avg
print(df[['name', 'dept', 'salary', 'salary_vs_dept_avg']])
```

### Tester use case — validate group integrity
```python
# Each department should have at least 1 person
dept_counts = df.groupby('dept')['name'].count()
assert (dept_counts >= 1).all(), f"Empty department found:\n{dept_counts}"
print("Group integrity check passed.")
```

---

## 9. Merging DataFrames

```python
employees = pd.DataFrame({
    'emp_id': [1, 2, 3, 4],
    'name':   ['Alice', 'Bob', 'Carol', 'Dave'],
    'dept_id': [101, 102, 101, 102]
})

departments = pd.DataFrame({
    'dept_id':   [101, 102],
    'dept_name': ['QA', 'MLOps']
})

# Inner join — only matching rows
merged = pd.merge(employees, departments, on='dept_id', how='inner')
print(merged)

# Left join — keep all employees, fill unmatched dept with NaN
merged_left = pd.merge(employees, departments, on='dept_id', how='left')
```

### Tester use case — verify no rows lost
```python
result = pd.merge(employees, departments, on='dept_id', how='inner')
assert len(result) == len(employees), \
    f"Row count changed after merge: {len(employees)} → {len(result)}"
print("Merge row count check passed.")
```

### MLOps use case — enrich feature table
```python
# Add external feature (e.g. regional cost of living index)
cost_index = pd.DataFrame({
    'dept': ['QA', 'MLOps'],
    'cost_index': [1.0, 1.3]
})

enriched = pd.merge(df, cost_index, left_on='dept', right_on='dept', how='left')
enriched['adjusted_salary'] = enriched['salary'] * enriched['cost_index']
print(enriched[['name', 'dept', 'salary', 'adjusted_salary']])
```

---

## 10. Saving and Exporting Data

```python
# Save to CSV
df.to_csv('/kaggle/working/clean_data.csv', index=False)

# Load back
df_loaded = pd.read_csv('/kaggle/working/clean_data.csv')

# Save to multiple formats
df.to_json('/kaggle/working/data.json', orient='records')
df.to_parquet('/kaggle/working/data.parquet')   # faster, smaller, production preferred

# Convert to NumPy for model training
X = df[['age', 'salary']].values   # .values returns numpy array
print(type(X), X.shape)
```

### Tester use case — round-trip integrity test
```python
original = df.copy()
original.to_csv('/kaggle/working/test_export.csv', index=False)
reloaded = pd.read_csv('/kaggle/working/test_export.csv')

assert list(original.columns) == list(reloaded.columns), "Column mismatch after save/load"
assert len(original) == len(reloaded), "Row count mismatch after save/load"
print("Round-trip integrity check passed.")
```

---

## 11. Common Errors and How to Fix Them

| Error | Cause | Fix |
|-------|-------|-----|
| `KeyError: 'column_name'` | Column doesn't exist | Check `df.columns` first |
| `ValueError: could not convert string to float` | Strings in numeric column | Use `.astype(float)` after `.str.strip()` |
| `SettingWithCopyWarning` | Modifying a slice | Use `.copy()` or `.loc[]` assignment |
| `MergeError: key not found` | Column name mismatch | Check spelling + `df.columns` |
| `AttributeError: 'DataFrame' has no attribute 'X'` | Typo in method/property | Check pandas docs for correct name |

```python
# Fix: SettingWithCopyWarning
# Wrong — modifying a slice
subset = df[df['dept'] == 'QA']
subset['salary'] = 99999   # Warning!

# Correct
subset = df[df['dept'] == 'QA'].copy()
subset['salary'] = 99999   # No warning

# Fix: convert column with mixed types
df['salary'] = pd.to_numeric(df['salary'], errors='coerce')   # non-numeric → NaN
df['salary'] = df['salary'].fillna(df['salary'].median())
```

---

## 12. Mini Project: ML-Ready Data Pipeline

> **Scenario:** Load raw employee data, clean it, add features, and export an ML-ready dataset.

```python
import pandas as pd
import numpy as np

# --- Step 1: Create raw data (simulating a CSV load) ---
raw = pd.DataFrame({
    'emp_id':  [1, 2, 3, 4, 5, 6, 7, 8],
    'name':    ['Alice', 'Bob', 'Carol', 'Dave', 'Eve', 'Frank', 'Grace', 'Hank'],
    'age':     [25, 32, None, 45, 28, 38, None, 41],
    'salary':  [50000, 75000, 62000, None, 58000, 88000, 70000, 95000],
    'dept':    ['QA', 'MLOps', 'QA', 'MLOps', 'QA', 'MLOps', 'QA', 'MLOps'],
    'perf':    ['good', 'excellent', 'good', 'excellent', None, 'good', 'excellent', 'good']
})

print(f"Raw data: {raw.shape}")
print(raw.isnull().sum())

# --- Step 2: Validate (Tester role) ---
required_cols = ['emp_id', 'name', 'age', 'salary', 'dept', 'perf']
missing = [c for c in required_cols if c not in raw.columns]
assert not missing, f"Missing columns: {missing}"
print("Column check passed.")

# No duplicate employee IDs
assert raw['emp_id'].duplicated().sum() == 0, "Duplicate emp_id found!"
print("Duplicate check passed.")

# --- Step 3: Handle missing values (MLOps role) ---
df = raw.copy()
df['age']    = df['age'].fillna(df['age'].median())
df['salary'] = df['salary'].fillna(df['salary'].median())
df['perf']   = df['perf'].fillna(df['perf'].mode()[0])

print(f"Nulls remaining: {df.isnull().sum().sum()}")

# --- Step 4: Feature engineering ---
dept_avg_salary = df.groupby('dept')['salary'].transform('mean')
df['salary_vs_avg']  = (df['salary'] - dept_avg_salary).round(2)
df['senior_flag']    = (df['age'] >= 35).astype(int)
df['age_group']      = pd.cut(df['age'],
                               bins=[0, 29, 39, 100],
                               labels=['junior', 'mid', 'senior'])

# --- Step 5: Encode categoricals ---
df_encoded = pd.get_dummies(df, columns=['dept', 'perf', 'age_group'], drop_first=True)
df_encoded = df_encoded.drop(columns=['name'])   # drop non-feature columns

print(f"Encoded features: {list(df_encoded.columns)}")

# --- Step 6: Extract feature matrix and target ---
target_col = 'senior_flag'
feature_cols = [c for c in df_encoded.columns if c not in ['emp_id', target_col]]

X = df_encoded[feature_cols]
y = df_encoded[target_col]

print(f"X: {X.shape}, y: {y.shape}")

# --- Step 7: Final validation (Tester role) ---
assert X.isnull().sum().sum() == 0, "Nulls remain in feature matrix!"
assert y.isnull().sum() == 0, "Nulls in target column!"
assert len(X) == len(y), "Feature/target row mismatch!"
print("Final quality checks passed.")

# --- Step 8: Save ML-ready data ---
df_encoded.to_csv('/kaggle/working/ml_ready_data.csv', index=False)
X.to_csv('/kaggle/working/features.csv', index=False)
y.to_csv('/kaggle/working/target.csv', index=False)
print("Saved to /kaggle/working/")

# --- Step 9: Verify round-trip ---
X_check = pd.read_csv('/kaggle/working/features.csv')
assert X_check.shape == X.shape, "Feature file shape mismatch!"
print(f"Pipeline complete. Output shape: {X.shape}")
```

---

## Quick Reference Card

```python
import pandas as pd

# Load
pd.read_csv('file.csv')              # from file
pd.DataFrame({'col': [1,2,3]})       # from dict

# Explore
df.head() / df.tail()               # first/last rows
df.shape / df.info() / df.describe()# structure & stats
df.dtypes / df.columns              # types & names
df.isnull().sum()                   # null counts
df.duplicated().sum()               # duplicate rows

# Select
df['col']                           # single column (Series)
df[['a','b']]                       # multiple columns (DataFrame)
df.loc[0, 'col']                    # by label
df.iloc[0, 1]                       # by position

# Filter
df[df['age'] > 30]                  # single condition
df[(df['age'] > 30) & (df['dept'] == 'QA')]  # multiple
df[df['dept'].isin(['QA','MLOps'])] # match list

# Clean
df.dropna()                         # drop nulls
df['col'].fillna(df['col'].mean())  # fill nulls
df.drop_duplicates()                # remove duplicates
df['col'].astype(float)             # change dtype

# Transform
df['new'] = df['col'] * 2          # new column
df.rename(columns={'old':'new'})    # rename
df.drop(columns=['col'])            # remove column
pd.get_dummies(df, columns=['cat']) # one-hot encode

# Groupby
df.groupby('dept')['salary'].mean() # group aggregate
df.groupby('dept').agg({'salary':'mean', 'age':'max'})

# Merge
pd.merge(df1, df2, on='id', how='inner')  # inner join
pd.merge(df1, df2, on='id', how='left')   # left join

# Save
df.to_csv('out.csv', index=False)  # CSV
df.to_parquet('out.parquet')       # parquet (production)
df[cols].values                    # to numpy array
```

---

*Built as part of a 6-week MLOps portfolio roadmap.*
*Previous: numpy_mlops_course.md | Next: scikit-learn basics*
