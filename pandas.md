# 🐼 The Definitive Beginner’s Guide to Python Pandas

> **A comprehensive, beginner-friendly reference for the most important data manipulation library in Python.**
> Every topic includes plain-English definitions, fully commented code, and hard-won tips from the trenches.

-----

## 📋 Table of Contents

1. [What is Pandas?](#-what-is-pandas)
1. [Core Structures: Series vs. DataFrames](#-core-structures-series-vs-dataframes)
1. [Data I/O: Reading & Writing Files](#-data-io-reading--writing-files)
1. [Inspection: Understanding Your Data](#-inspection-understanding-your-data)
1. [Selection & Indexing: `[]` vs `.loc` vs `.iloc`](#-selection--indexing--vs-loc-vs-iloc)
1. [Filtering: Boolean Logic & Smart Queries](#-filtering-boolean-logic--smart-queries)
1. [Missing Data: Detecting & Handling NaN](#-missing-data-detecting--handling-nan)
1. [Transformation: Reshaping Your DataFrame](#-transformation-reshaping-your-dataframe)
1. [Sorting: sort_values, nlargest & nsmallest](#-sorting-sort_values-nlargest--nsmallest)
1. [Aggregation: groupby, agg & pivot_table](#-aggregation-groupby-agg--pivot_table)
1. [String Operations: Working with Text Data](#-string-operations-working-with-text-data)
1. [Functions: apply, map & applymap / map](#-functions-apply-map--applymap--map)
1. [Combining Data: merge, concat & join](#-combining-data-merge-concat--join)
1. [Time Series: Dates & Datetime Components](#-time-series-dates--datetime-components)
1. [Duplicates: Finding & Removing](#-duplicates-finding--removing)
1. [Advanced: category dtype & query()](#-advanced-category-dtype--query)
1. [Best Practices & Final Wisdom](#-best-practices--final-wisdom)

-----

## 🐼 What is Pandas?

Pandas is the go-to Python library for **data manipulation and analysis**. Think of it as a supercharged spreadsheet that lives inside your Python code. It lets you load data from CSVs, Excel files, databases, and dozens of other sources, then clean, reshape, filter, and summarize it — all with concise, readable code.

The name comes from **“panel data”**, an econometrics term for multi-dimensional structured datasets. Today it powers everything from quick data explorations to production ETL pipelines.

```bash
# Install pandas (run this in your terminal, not Python)
pip install pandas

# Import it — the alias 'pd' is universal convention
import pandas as pd
import numpy as np  # NumPy often works alongside Pandas
```

-----

## 🧱 Core Structures: Series vs. DataFrames

### What Is a Series?

A **Series** is a one-dimensional labeled array — imagine a single column from a spreadsheet. Every value has an **index label** (which defaults to 0, 1, 2… but can be anything). You can think of it as a Python dictionary that also supports fast numerical operations.

```python
import pandas as pd

# --- Creating a Series from a Python list ---
temperatures = pd.Series([72, 68, 75, 80, 77])
# Index is automatically 0, 1, 2, 3, 4 (the default RangeIndex)
print(temperatures)
# 0    72
# 1    68
# 2    75
# dtype: int64

# --- Creating a Series with custom index labels ---
temps_named = pd.Series(
    [72, 68, 75],           # The actual data values
    index=["Mon", "Tue", "Wed"],  # Custom labels for each value
    name="Temp_F"           # Give the Series a name (useful when building a DataFrame later)
)
print(temps_named["Tue"])  # Access by label: 68

# --- Creating a Series from a dictionary ---
# Dictionary keys become the index labels automatically
city_pop = pd.Series({"New York": 8_336_817, "Los Angeles": 3_979_576, "Chicago": 2_693_976})
print(city_pop["Chicago"])  # 2693976
```

### What Is a DataFrame?

A **DataFrame** is a two-dimensional table with labeled rows (index) and labeled columns — exactly like a spreadsheet or a SQL table. Under the hood, it’s a collection of Series that all share the same index.

```python
# --- Creating a DataFrame from a list of dictionaries ---
# Each dictionary represents one row; keys become column names
employees = pd.DataFrame([
    {"name": "Alice", "dept": "Engineering", "salary": 95000},
    {"name": "Bob",   "dept": "Marketing",   "salary": 72000},
    {"name": "Carol", "dept": "Engineering", "salary": 105000},
    {"name": "Dave",  "dept": "HR",           "salary": 68000},
])
# pandas aligns columns automatically across all rows

# --- Creating a DataFrame from a dictionary of lists ---
# Each key is a column name; each list is that column's data
scores = pd.DataFrame({
    "student": ["Alice", "Bob", "Carol"],   # Column 1
    "math":    [92, 85, 78],                 # Column 2
    "science": [88, 91, 95],                 # Column 3
})

# --- Checking the shape: (rows, columns) ---
print(employees.shape)    # (4, 3)
print(employees.columns)  # Index(['name', 'dept', 'salary'], dtype='object')
print(employees.dtypes)   # Each column's data type
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — Mixed types corrupt columns:** If you accidentally mix numbers and strings in one column (e.g., `[100, "N/A", 200]`), pandas will store the column as `object` dtype, which is slow and disables numeric operations. Always clean your data before or right after loading.

**Pro-Tip — Set a meaningful index:** By default, pandas uses `0, 1, 2…` as the row index. If your data has a natural unique identifier (like employee ID or date), set it as the index with `df.set_index("employee_id")`. This makes row lookups much faster and your code more readable.

**Pro-Tip — A single-column DataFrame ≠ a Series:** `df["salary"]` returns a Series. `df[["salary"]]` (double brackets) returns a single-column DataFrame. They behave differently — knowing which one you have prevents confusing errors downstream.

-----

## 📂 Data I/O: Reading & Writing Files

### What Is Data I/O?

“I/O” stands for Input/Output. Before you can do anything useful, you need to get data *into* pandas (usually from a file or database) and eventually get results *out*. Pandas supports dozens of formats; the most common are CSV and Excel.

### Reading a CSV

```python
# --- Basic CSV read ---
df = pd.read_csv("sales_data.csv")
# pandas infers column names from the first row and data types automatically

# --- Reading with common options ---
df = pd.read_csv(
    "sales_data.csv",         # File path (relative or absolute)
    sep=",",                   # Delimiter — use sep="\t" for tab-separated files
    header=0,                  # Row number to use as column names (0 = first row)
    index_col="order_id",      # Use this column as the row index instead of 0,1,2...
    usecols=["date", "amount", "region"],  # Only load these columns (saves memory!)
    dtype={"amount": "float32"},           # Force a specific type for a column
    parse_dates=["date"],      # Automatically convert this column to datetime
    na_values=["N/A", "–", "none"],  # Treat these strings as NaN (missing values)
    encoding="utf-8",          # File encoding — try "latin-1" if utf-8 fails
    nrows=1000,                # Only read the first 1000 rows (great for testing)
    skiprows=[1, 2],           # Skip specific row numbers after the header
)
```

### Reading an Excel File

```python
# pip install openpyxl  ← required for .xlsx files
df = pd.read_excel(
    "report.xlsx",        # File path
    sheet_name="Q1",      # Name or index of the sheet to read (0 = first sheet)
    header=1,             # Sometimes Excel files have a title row before the real headers
    usecols="A:F",        # Excel-style column range, OR use a list: usecols=[0, 1, 2]
    skiprows=2,           # Skip the first 2 rows (e.g., a logo or title block)
)

# --- Reading all sheets at once (returns a dict of DataFrames) ---
all_sheets = pd.read_excel("report.xlsx", sheet_name=None)
# Access a specific sheet: all_sheets["Q2"]
```

### Exporting with `to_csv`

```python
# --- Save DataFrame to CSV ---
df.to_csv(
    "output.csv",         # Output file path
    index=False,          # IMPORTANT: Don't write the row index as a column
                          # (Without this, you get an ugly unnamed "Unnamed: 0" column)
    sep=",",              # Delimiter
    encoding="utf-8-sig", # "utf-8-sig" adds a BOM marker — fixes Excel opening issues on Windows
    float_format="%.2f",  # Round all floats to 2 decimal places in the file
    na_rep="",            # How to represent NaN values (empty string is safe default)
    columns=["date", "amount"],  # Only export specific columns
)
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — Forgetting `index=False`:** This is the single most common mistake when exporting. If you don’t set `index=False`, pandas writes the integer index (0, 1, 2…) as the first column. When you re-import that file, you’ll have an ugly `Unnamed: 0` column. Always use `index=False` unless your index is meaningful data.

**Pitfall — Encoding errors:** If you see `UnicodeDecodeError`, try `encoding="latin-1"` or `encoding="cp1252"` in `read_csv`. These cover most Windows-generated files with special characters.

**Pro-Tip — Use `chunksize` for large files:** If a CSV is too large to fit in memory, read it in pieces with `pd.read_csv("big_file.csv", chunksize=10000)`. This returns an iterator of DataFrames, which you can process one at a time.

**Pro-Tip — `usecols` is your memory friend:** Only loading the columns you actually need can reduce memory usage by 50-80% on wide datasets.

-----

## 🔍 Inspection: Understanding Your Data

### What Is Data Inspection?

The very first thing you should do after loading data is *look at it*. Inspection functions tell you what you have: how many rows and columns, what types the data is, whether there are missing values, and what the distribution of values looks like. Never skip this step — surprises in data are best caught early.

```python
# Assume we have a DataFrame called df loaded from a sales CSV

# --- head() & tail(): Preview the data ---
df.head()     # Shows the first 5 rows (default)
df.head(10)   # Shows the first 10 rows
df.tail(3)    # Shows the last 3 rows — useful for checking if loading cut off early

# --- info(): Your data's "vital signs" ---
df.info()
# Prints:
#   - Total number of rows and columns
#   - Each column name and its dtype (int64, float64, object, datetime64, etc.)
#   - Non-null count per column (instantly reveals missing data)
#   - Total memory usage of the DataFrame
# This is the FIRST thing to run on any new dataset.

df.info(memory_usage="deep")
# memory_usage="deep" gives a more accurate (but slower) memory estimate
# especially important for object/string columns which are often larger than they appear

# --- describe(): Statistical summary ---
df.describe()
# For numeric columns: count, mean, std, min, 25%, 50% (median), 75%, max
# Quickly shows outliers (e.g., a max 100x the mean) and data range

df.describe(include="object")
# For string columns: count, unique count, top (most frequent value), freq (its count)

df.describe(include="all")
# Shows stats for ALL column types at once (mixes numeric and categorical stats)

# --- shape: Rows and columns as a tuple ---
print(df.shape)         # e.g., (50000, 12) → 50,000 rows, 12 columns
print(df.shape[0])      # Just the row count
print(df.shape[1])      # Just the column count

# --- value_counts(): Frequency table for a column ---
print(df["region"].value_counts())       # Counts of each region (sorted descending)
print(df["region"].value_counts(normalize=True))  # Proportions (0.0–1.0) instead of counts
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — `describe()` skips object columns by default:** If you just call `df.describe()`, you’ll only see stats for numeric columns. Call `df.describe(include="all")` to see everything, or you’ll miss problems in string columns entirely.

**Pro-Tip — `info()` is faster than `isnull().sum()` for a first look:** `df.info()` shows you the non-null count per column at a glance. You can instantly see which columns have missing data without an extra method call.

**Pro-Tip — `nunique()` is underrated:** `df.nunique()` tells you how many unique values each column has. A column with 2 unique values in 50,000 rows is a great candidate for `category` dtype. A column with 49,999 unique values in 50,000 rows is probably an ID column that shouldn’t be treated as categorical.

-----

## 🎯 Selection & Indexing: `[]` vs `.loc` vs `.iloc`

### The Core Concept

This is the topic that trips up almost every beginner, so let’s be very precise. Pandas gives you three ways to select data, and they answer three different questions:

- **`df["col"]` or `df[["col1", "col2"]]`** — “Give me this column (or these columns) by name.”
- **`df.loc[row_label, col_label]`** — “Give me data by **label** — the actual name of the row and column.”
- **`df.iloc[row_number, col_number]`** — “Give me data by **position** — the integer position (0, 1, 2…).”

The `loc` vs `iloc` distinction is critical: **`loc` uses labels, `iloc` uses integer positions.**

```python
import pandas as pd

# Create a DataFrame with a custom string index (not the default 0,1,2...)
df = pd.DataFrame({
    "name":   ["Alice", "Bob", "Carol", "Dave"],
    "dept":   ["Eng",   "Mkt", "Eng",   "HR"],
    "salary": [95000,   72000, 105000,  68000],
    "years":  [5,       3,     8,       2],
}, index=["emp01", "emp02", "emp03", "emp04"])  # Custom row labels

# ─────────────────────────────────────────────────────────────
# BRACKET SELECTION [ ]
# ─────────────────────────────────────────────────────────────

# Single column → returns a Series
salaries = df["salary"]

# Multiple columns → returns a DataFrame (note the double brackets)
subset = df[["name", "salary"]]

# Row slicing by position (confusingly, brackets DO support this, but only for slices)
first_two = df[0:2]   # Rows at positions 0 and 1 (position-based slicing)
# NOTE: This is the exception — for everything else, use .loc or .iloc

# ─────────────────────────────────────────────────────────────
# .loc — LABEL-BASED (uses row/column NAMES)
# ─────────────────────────────────────────────────────────────

# Single row by label
alice_row = df.loc["emp01"]             # Returns a Series (one row as a Series)

# Multiple rows by label
top_two = df.loc[["emp01", "emp03"]]   # Returns a DataFrame

# Row + column selection: df.loc[row_label, column_label]
bobs_dept = df.loc["emp02", "dept"]    # Single value: "Mkt"

# Row range + specific columns (label-based slicing is INCLUSIVE on both ends)
subset = df.loc["emp01":"emp03", ["name", "salary"]]
# Includes emp01, emp02, AND emp03 — unlike Python slicing, the end is included!

# All rows, specific columns
salaries_only = df.loc[:, "salary"]   # The colon means "all rows"

# Boolean array with .loc (the most common real-world usage)
engineers = df.loc[df["dept"] == "Eng"]  # All rows where dept is Engineering

# ─────────────────────────────────────────────────────────────
# .iloc — POSITION-BASED (uses integer positions 0, 1, 2...)
# ─────────────────────────────────────────────────────────────

# Single row by position (0 = first row)
first_row = df.iloc[0]                  # Returns a Series

# Row + column by position: df.iloc[row_pos, col_pos]
# salary is column index 2 (0=name, 1=dept, 2=salary, 3=years)
first_salary = df.iloc[0, 2]           # 95000

# Slices with iloc — end is EXCLUSIVE (like standard Python slicing)
first_three = df.iloc[0:3]             # Rows 0, 1, 2 (NOT row 3)

# Last row (negative indexing works!)
last_row = df.iloc[-1]                  # Dave's row

# Every other row, first two columns
sampled = df.iloc[::2, 0:2]            # Rows 0 and 2, columns 0 and 1
```

### The Label vs. Position Confusion — A Concrete Example

```python
# Here's WHY this matters so much:
# If your index is already integers (the default), loc and iloc look the same...
df_default = pd.DataFrame({"val": [10, 20, 30]})  # Index: 0, 1, 2
df_default.loc[1]   # Label "1" → row with val=20
df_default.iloc[1]  # Position 1 → row with val=20  (same result here!)

# BUT after filtering, the integer index labels may no longer match positions:
df_filtered = df_default[df_default["val"] > 10]  # Keeps rows with index 1 and 2
print(df_filtered)
#    val
# 1   20
# 2   30

df_filtered.iloc[0]   # Position 0 → row with val=20 (index label=1)
df_filtered.loc[0]    # Label 0 → KeyError! There's no row labeled "0" anymore.
# This is the most common indexing bug in pandas.
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — The classic `loc` vs `iloc` index drift bug:** After filtering or sorting a DataFrame, the integer row labels and integer positions diverge. Always be explicit: use `.loc` if you know the label, `.iloc` if you know the position. Never assume they’re the same.

**Pitfall — `loc` slicing is inclusive, Python slicing is exclusive:** `df.loc["a":"c"]` includes row “c”. `df.iloc[0:3]` does NOT include position 3. This inconsistency is a known quirk of pandas.

**Pro-Tip — Use `.at` and `.iat` for single-cell access:** `df.at["emp01", "salary"]` (label-based) and `df.iat[0, 2]` (position-based) are optimized for reading or writing a single value and are significantly faster than `.loc` or `.iloc` for that use case.

**Pro-Tip — `.loc` works perfectly for updating values:** `df.loc[df["dept"] == "Eng", "salary"] *= 1.05` gives a 5% raise to all engineers. This is the clean, warning-free way to update a subset of values.

-----

## 🔎 Filtering: Boolean Logic & Smart Queries

### What Is Boolean Filtering?

Filtering means keeping only the rows that meet certain conditions. Pandas achieves this by creating a **boolean Series** — a column of `True`/`False` values — where `True` marks the rows you want to keep. You then pass that Series into `df[...]` to select only the matching rows.

```python
import pandas as pd

df = pd.DataFrame({
    "product": ["Widget A", "Widget B", "Gadget X", "Gadget Y", "Widget C"],
    "category": ["Widget", "Widget", "Gadget", "Gadget", "Widget"],
    "price":    [29.99, 49.99, 89.99, 120.00, 19.99],
    "in_stock": [True, False, True, True, True],
    "region":   ["North", "South", "North", "West", "South"],
})

# ─────────────────────────────────────────────────────────────
# SIMPLE CONDITIONS
# ─────────────────────────────────────────────────────────────

# Single condition: price above 50
expensive = df[df["price"] > 50]

# Equality check
widgets = df[df["category"] == "Widget"]

# Not equal
not_south = df[df["region"] != "South"]

# ─────────────────────────────────────────────────────────────
# COMPOUND CONDITIONS with & (AND) and | (OR)
# ─────────────────────────────────────────────────────────────

# CRITICAL: Each condition MUST be wrapped in parentheses!
# Without parentheses, Python's operator precedence causes incorrect results.
cheap_and_available = df[
    (df["price"] < 50) &    # AND: both must be True
    (df["in_stock"] == True)
]

expensive_or_gadget = df[
    (df["price"] > 100) |   # OR: at least one must be True
    (df["category"] == "Gadget")
]

# NOT operator: ~ (tilde) inverts a boolean Series
out_of_stock = df[~df["in_stock"]]  # Rows where in_stock is False

# Combining all three
results = df[
    (df["price"] < 100) &
    (~df["in_stock"]) |     # Note: be careful with precedence even with parens
    (df["region"] == "West")
]

# ─────────────────────────────────────────────────────────────
# isin(): Check membership in a list
# ─────────────────────────────────────────────────────────────

# More concise than writing: (df["region"] == "North") | (df["region"] == "West")
north_or_west = df[df["region"].isin(["North", "West"])]

# Negate isin() to EXCLUDE values
not_south_or_west = df[~df["region"].isin(["South", "West"])]

# ─────────────────────────────────────────────────────────────
# str.contains(): Partial string matching
# ─────────────────────────────────────────────────────────────

# Find rows where product name contains "Widget"
widget_products = df[df["product"].str.contains("Widget")]

# Case-insensitive match (default is case-sensitive)
widget_ci = df[df["product"].str.contains("widget", case=False)]

# Using regex=False for literal strings (faster, safer when no regex needed)
widget_literal = df[df["product"].str.contains("Widget A", regex=False)]

# str.contains() returns NaN for missing values — handle with na=False
safe_filter = df[df["product"].str.contains("Widget", na=False)]
# na=False treats NaN values as "not matching" (returns False instead of NaN)

# ─────────────────────────────────────────────────────────────
# between(): Range filtering (inclusive on both ends by default)
# ─────────────────────────────────────────────────────────────
mid_range = df[df["price"].between(25, 75)]  # 25 <= price <= 75
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — Forgetting parentheses in compound conditions:** `df[df["price"] > 50 & df["in_stock"]]` will raise a `TypeError` or give wrong results. Python evaluates `50 & df["in_stock"]` first due to operator precedence. Always wrap each condition: `df[(df["price"] > 50) & (df["in_stock"])]`.

**Pitfall — Using `and`/`or` instead of `&`/`|`:** Python’s `and` and `or` keywords don’t work element-wise on Series. Using them will raise `ValueError: The truth value of a Series is ambiguous`. Always use `&` for AND and `|` for OR.

**Pro-Tip — Store your boolean mask separately for readability:** Instead of a one-liner, write `mask = (df["price"] > 50) & (df["in_stock"])` then `df[mask]`. It’s much easier to read, debug, and reuse.

**Pro-Tip — Prefer `.query()` for complex filters:** `df.query("price > 50 and in_stock == True and region in ['North', 'West']")` is often far more readable than a long boolean expression. See the [Advanced section](#-advanced-category-dtype--query) for full details.

-----

## 🩺 Missing Data: Detecting & Handling NaN

### What Is NaN?

`NaN` stands for “Not a Number” — it’s the sentinel value pandas uses for missing data in numeric columns. For object (string) columns, missing values can be `NaN` or Python’s `None`. Both behave similarly in most contexts. Missing data is real-world normal; knowing how to handle it cleanly is essential.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "name":    ["Alice", "Bob", None,    "Dave"],
    "score":   [92.0,    np.nan, 78.0,   85.0],
    "dept":    ["Eng",   "Mkt",  "Eng",  None],
    "tenure":  [5,       3,      np.nan, 2],
})

# ─────────────────────────────────────────────────────────────
# DETECTING MISSING VALUES
# ─────────────────────────────────────────────────────────────

# isnull() (alias: isna()) returns a boolean DataFrame
df.isnull()        # True where data is missing, False where it's present

# Count missing values per column — the most useful one-liner
df.isnull().sum()
# name      1
# score     1
# dept      1
# tenure    1

# Percentage of missing values per column
(df.isnull().sum() / len(df) * 100).round(2)

# notnull() is the inverse — True where data IS present
df.notnull()

# Filter for rows that have at least one missing value
rows_with_missing = df[df.isnull().any(axis=1)]

# Filter for rows with NO missing values
complete_rows = df[df.notnull().all(axis=1)]

# ─────────────────────────────────────────────────────────────
# DROPPING MISSING VALUES: dropna()
# ─────────────────────────────────────────────────────────────

# Drop any row with at least one NaN (most aggressive option)
df_clean = df.dropna()

# Drop rows only if ALL values in that row are NaN
df_mostly_clean = df.dropna(how="all")

# Drop rows where specific columns have NaN (often the right approach)
df_needs_score = df.dropna(subset=["score"])  # Must have a score to keep the row

# Drop columns with any missing values (useful for feature selection)
df_no_missing_cols = df.dropna(axis=1)

# thresh: Keep rows with at least N non-NaN values
df_thresh = df.dropna(thresh=3)  # Keep rows that have at least 3 non-null values

# ─────────────────────────────────────────────────────────────
# FILLING MISSING VALUES: fillna()
# ─────────────────────────────────────────────────────────────

# Fill all NaN with a single constant
df_filled = df.fillna(0)           # Risky — zeros in "name" doesn't make sense!

# Fill specific columns with specific values using a dictionary
df_filled = df.fillna({
    "score":  df["score"].mean(),   # Fill missing scores with the column average
    "dept":   "Unknown",            # Fill missing department with a placeholder string
    "tenure": df["tenure"].median() # Fill missing tenure with the median
})

# Forward fill: propagate the last valid value forward (great for time series)
df_ffill = df.fillna(method="ffill")  # Each NaN gets the value from the row above

# Backward fill: propagate the next valid value backward
df_bfill = df.fillna(method="bfill")  # Each NaN gets the value from the row below

# Limit how many consecutive NaNs to fill
df_limited = df.fillna(method="ffill", limit=1)  # Fill at most 1 consecutive NaN
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — `None` vs `NaN` can behave differently:** In object-dtype columns, missing values are `None`. In numeric columns, they are `NaN` (a float). `isnull()` catches both, but direct comparisons like `df["name"] == None` won’t work reliably. Always use `isnull()` to detect missing values.

**Pitfall — Dropping rows too aggressively:** `df.dropna()` with no arguments drops any row with even one missing value. On a dataset with 20 columns and normal rates of missingness, this can silently delete most of your data. Use `subset=` to specify which columns actually matter for completeness.

**Pro-Tip — Always investigate *why* data is missing:** Missing data can be MCAR (Missing Completely At Random), MAR (Missing At Random), or MNAR (Missing Not At Random). Blindly filling with the mean hides patterns. Consider visualizing missingness with the `missingno` library before deciding on a strategy.

**Pro-Tip — Avoid `inplace=True` with `fillna` and `dropna`:** See the [Best Practices](#-best-practices--final-wisdom) section — `inplace=True` is being deprecated and can cause subtle bugs. Always reassign: `df = df.fillna(0)`.

-----

## 🔧 Transformation: Reshaping Your DataFrame

### What Is Transformation?

Transformation is the act of changing the *shape or content* of your DataFrame — adding new columns, removing unwanted ones, renaming for clarity, or changing a column’s data type to save memory or enable operations.

```python
import pandas as pd

df = pd.DataFrame({
    "first_name": ["Alice", "Bob", "Carol"],
    "last_name":  ["Smith", "Jones", "Williams"],
    "salary":     [95000,   72000,  105000],
    "dept_code":  ["E", "M", "E"],
})

# ─────────────────────────────────────────────────────────────
# ADDING NEW COLUMNS
# ─────────────────────────────────────────────────────────────

# Simple assignment (vectorized — no loop needed)
df["annual_bonus"] = df["salary"] * 0.10       # 10% bonus column
df["full_name"]    = df["first_name"] + " " + df["last_name"]  # String concatenation

# Conditional column using np.where (like a vectorized if/else)
import numpy as np
df["level"] = np.where(df["salary"] > 90000, "Senior", "Junior")
# np.where(condition, value_if_true, value_if_false)

# Multiple conditions with np.select
conditions = [
    df["salary"] < 75000,
    (df["salary"] >= 75000) & (df["salary"] < 100000),
    df["salary"] >= 100000
]
choices = ["Junior", "Mid-Level", "Senior"]
df["grade"] = np.select(conditions, choices, default="Unknown")

# ─────────────────────────────────────────────────────────────
# DROPPING COLUMNS & ROWS: drop()
# ─────────────────────────────────────────────────────────────

# Drop columns by name
df_slim = df.drop(columns=["first_name", "last_name"])
# Always use the keyword argument "columns=" for clarity

# Drop rows by index label
df_no_first = df.drop(index=[0])    # Drop row with label 0

# Drop rows by a condition (usually better done with filtering, but this works too)
df_no_senior = df.drop(index=df[df["grade"] == "Senior"].index)

# ─────────────────────────────────────────────────────────────
# RENAMING COLUMNS: rename()
# ─────────────────────────────────────────────────────────────

# Use a dictionary: {old_name: new_name}
df_renamed = df.rename(columns={
    "dept_code": "department",
    "annual_bonus": "bonus"
})

# You can also rename via list assignment if renaming ALL columns at once
# df.columns = ["col1", "col2", "col3", ...]  ← only when you know the order

# ─────────────────────────────────────────────────────────────
# CHANGING DATA TYPES: astype() — critical for memory management
# ─────────────────────────────────────────────────────────────

# Check current types
print(df.dtypes)
# salary:    int64  ← uses 8 bytes per value
# dept_code: object ← stores strings as Python objects (expensive)

# Downcast integer (int64 → int32 saves 4 bytes per row)
df["salary"] = df["salary"].astype("int32")

# Float downcast (float64 → float32 — halves memory, some precision loss)
df["annual_bonus"] = df["annual_bonus"].astype("float32")

# Convert to category (huge savings for low-cardinality string columns)
df["dept_code"] = df["dept_code"].astype("category")
# "category" stores unique values once, then uses integers as pointers
# Perfect for columns with few unique values (like dept, status, region)

# Convert to boolean
df["is_senior"] = (df["salary"] > 90000).astype("bool")

# Convert string to numeric (errors="coerce" turns invalid values into NaN)
df["salary_from_string"] = pd.to_numeric(df["salary"].astype(str), errors="coerce")
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — `astype("int")` fails if there are NaN values:** Integer dtypes in pandas traditionally can’t hold NaN. If you try to cast a float column with NaN to `int`, you’ll get an error. Use pandas’ nullable integer type instead: `df["col"].astype("Int64")` (capital I) — it supports NaN.

**Pitfall — Chained assignment creates the “SettingWithCopyWarning”:** If you write `df[df["dept"] == "Eng"]["salary"] = 100000`, pandas may raise a `SettingWithCopyWarning` because it’s unclear whether you’re modifying a copy or the original. Use `.loc` instead: `df.loc[df["dept"] == "Eng", "salary"] = 100000`.

**Pro-Tip — `pd.to_numeric(series, errors="coerce")` is your cleaner for dirty numeric data:** When a supposedly numeric column contains strings like “N/A” or “$100”, `astype(float)` crashes. `pd.to_numeric` with `errors="coerce"` silently converts bad values to NaN so you can handle them deliberately.

-----

## 📊 Sorting: sort_values, nlargest & nsmallest

### What Is Sorting?

Sorting reorders your rows based on the values in one or more columns. It’s fundamental for ranking, reporting, and finding extreme values. Pandas gives you both full sorting and efficient shortcuts for finding top/bottom N records.

```python
import pandas as pd

df = pd.DataFrame({
    "product":  ["Widget A", "Gadget X", "Widget B", "Gadget Y", "Widget C"],
    "category": ["Widget",   "Gadget",   "Widget",   "Gadget",   "Widget"],
    "sales":    [1500,       3200,       800,        4100,       2200],
    "price":    [29.99,      89.99,      49.99,      120.00,     19.99],
})

# ─────────────────────────────────────────────────────────────
# sort_values(): Sort by one or more columns
# ─────────────────────────────────────────────────────────────

# Simple sort by one column, ascending (default)
by_sales = df.sort_values("sales")

# Sort descending (highest sales first)
by_sales_desc = df.sort_values("sales", ascending=False)

# Multi-column sort: sort by category first, then by sales within each category
by_cat_then_sales = df.sort_values(
    by=["category", "sales"],        # List of column names to sort by
    ascending=[True, False]          # ascending per column: category asc, sales desc
)

# Sort by index (useful after operations that scramble row order)
df_sorted_index = df.sort_index()

# na_position: where to put NaN values
df_with_nan = df.copy()
df_with_nan.loc[0, "sales"] = None
sorted_nans_first = df_with_nan.sort_values("sales", na_position="first")
# na_position="last" is the default (NaNs go to the bottom)

# ─────────────────────────────────────────────────────────────
# nlargest(): Get top N rows by a column (faster than sort + head)
# ─────────────────────────────────────────────────────────────

# Top 3 products by sales
top3 = df.nlargest(3, "sales")
# More efficient than: df.sort_values("sales", ascending=False).head(3)

# Tie-breaking: "keep" parameter handles duplicate values
top3_all = df.nlargest(3, "sales", keep="all")
# keep="first" (default): keep first occurrence of ties
# keep="last": keep last occurrence of ties
# keep="all": keep all tied rows (result may be larger than N)

# ─────────────────────────────────────────────────────────────
# nsmallest(): Get bottom N rows by a column
# ─────────────────────────────────────────────────────────────

# 2 cheapest products
cheapest2 = df.nsmallest(2, "price")
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — Forgetting to capture the result:** `df.sort_values("sales")` does NOT modify `df` in place (by default). You must reassign: `df = df.sort_values("sales")`. This is intentional and actually good design — see [Best Practices](#-best-practices--final-wisdom).

**Pro-Tip — `nlargest`/`nsmallest` are faster than sort + head:** When you only need the top/bottom N rows, these methods use a partial sort algorithm that’s significantly faster on large DataFrames than sorting the entire dataset first.

**Pro-Tip — Reset the index after sorting:** After a sort, the original row labels are preserved but scrambled. If you want clean 0, 1, 2… labels, call `.reset_index(drop=True)`: `df = df.sort_values("sales").reset_index(drop=True)`. The `drop=True` prevents the old index from becoming a column.

-----

## 📈 Aggregation: groupby, agg & pivot_table

### What Is Aggregation?

Aggregation means **collapsing multiple rows into a summary** by applying a function (sum, mean, count, etc.). This is the pandas equivalent of SQL’s `GROUP BY`. It’s one of the most powerful and frequently used operations in data analysis.

```python
import pandas as pd

df = pd.DataFrame({
    "dept":    ["Eng", "Eng", "Mkt", "Mkt", "HR", "Eng", "HR"],
    "gender":  ["F",   "M",   "F",   "M",   "F",  "F",   "M"],
    "salary":  [95000, 88000, 72000, 68000, 65000, 105000, 70000],
    "years":   [5,     3,     4,     2,     6,     8,      3],
})

# ─────────────────────────────────────────────────────────────
# groupby(): The foundation of aggregation
# ─────────────────────────────────────────────────────────────

# Group by one column and apply a single aggregation
avg_salary_by_dept = df.groupby("dept")["salary"].mean()
# Result is a Series: dept → average salary

# Group by one column and aggregate multiple columns with different functions
dept_stats = df.groupby("dept").agg(
    avg_salary=("salary", "mean"),  # New column name = ("source column", "function")
    max_salary=("salary", "max"),
    min_salary=("salary", "min"),
    headcount=("salary", "count"),
    avg_tenure=("years", "mean"),
)
# This named aggregation syntax (pandas ≥ 0.25) is the modern approach

# Group by multiple columns
by_dept_gender = df.groupby(["dept", "gender"])["salary"].mean()
# Result has a MultiIndex: (dept, gender) → average salary

# ─────────────────────────────────────────────────────────────
# .agg() with multiple functions per column
# ─────────────────────────────────────────────────────────────

# Apply a list of functions to each column
summary = df.groupby("dept")[["salary", "years"]].agg(["mean", "min", "max"])
# Creates a MultiIndex column: (salary_mean, salary_min, ..., years_mean, ...)

# Apply different functions to different columns using a dict
custom = df.groupby("dept").agg({
    "salary": ["mean", "sum"],    # Two functions for salary
    "years":  "max",              # One function for years
})

# Flatten the MultiIndex column names after .agg()
summary.columns = ["_".join(col).strip() for col in summary.columns]
# Turns ("salary", "mean") into "salary_mean" — much easier to work with

# ─────────────────────────────────────────────────────────────
# pivot_table(): Cross-tabulation and 2D aggregation
# ─────────────────────────────────────────────────────────────

# Create a cross-tab: rows=dept, columns=gender, values=avg salary
pivot = pd.pivot_table(
    df,                    # The source DataFrame
    values="salary",       # The column to aggregate
    index="dept",          # Becomes the row index (unique values = unique rows)
    columns="gender",      # Becomes column headers
    aggfunc="mean",        # Aggregation function ("mean", "sum", "count", etc.)
    fill_value=0,          # Replace NaN in the result with 0 (happens when a combo doesn't exist)
    margins=True,          # Add a "All" row and column with grand totals
    margins_name="Total",  # Custom name for the totals row/column
)
# pivot_table is perfect for building report-style summary tables

# Multiple aggregation functions in pivot_table
multi_pivot = pd.pivot_table(
    df,
    values="salary",
    index="dept",
    columns="gender",
    aggfunc=["mean", "count"],  # Get both mean and count
)
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — `groupby` drops NaN keys by default:** If your grouping column has NaN values, those rows are silently excluded from the result. Use `df.groupby("dept", dropna=False)` to include NaN as a group.

**Pitfall — The result of `groupby` is a GroupBy *object*, not a DataFrame:** `df.groupby("dept")` by itself does nothing visible. You must chain an aggregation or transformation method to it.

**Pro-Tip — `transform()` instead of `agg()` when you need the result aligned to the original DataFrame:** `df.groupby("dept")["salary"].transform("mean")` returns a Series with the same length as `df` where each row contains the group mean. This is perfect for adding a “dept_avg_salary” column back to the original DataFrame without merging.

**Pro-Tip — `size()` vs `count()`:** `groupby().size()` counts all rows including those with NaN. `groupby().count()` counts only non-NaN values. If you want total group size, use `size()`.

-----

## 🔤 String Operations: Working with Text Data

### What Is the `.str` Accessor?

The `.str` accessor lets you apply string operations to an entire pandas Series element-by-element — without writing a loop. It’s vectorized, clean, and dramatically faster than using `apply(lambda x: x.upper())` for common string tasks.

```python
import pandas as pd

df = pd.DataFrame({
    "full_name": ["  Alice Smith  ", "bob JONES", "Carol Williams", "dave  brown"],
    "email":     ["alice@company.com", "bob@company.com", "carol@other.org", None],
    "phone":     ["(555) 123-4567", "555-987-6543", "(555) 456-7890", "555.111.2222"],
})

# ─────────────────────────────────────────────────────────────
# CLEANING & CASE OPERATIONS
# ─────────────────────────────────────────────────────────────

df["name_clean"]  = df["full_name"].str.strip()        # Remove leading/trailing whitespace
df["name_upper"]  = df["full_name"].str.upper()        # ALL CAPS
df["name_lower"]  = df["full_name"].str.lower()        # all lowercase
df["name_title"]  = df["full_name"].str.strip().str.title()  # Title Case (chain operations!)

# ─────────────────────────────────────────────────────────────
# str.replace(): Substitute patterns
# ─────────────────────────────────────────────────────────────

# Replace all non-digit characters from phone numbers (using regex)
df["phone_digits"] = df["phone"].str.replace(r"[^\d]", "", regex=True)
# r"[^\d]" is a regex meaning "any character that is NOT a digit"
# Result: "5551234567", "5559876543", etc.

# Simple literal replacement (no regex)
df["email_domain"] = df["email"].str.replace("@company.com", "", regex=False)

# ─────────────────────────────────────────────────────────────
# str.split(): Split a string into parts
# ─────────────────────────────────────────────────────────────

# Split full name into a list of parts
df["name_parts"] = df["full_name"].str.strip().str.split(" ")
# Returns a Series of lists: [["Alice", "Smith"], ["bob", "JONES"], ...]

# expand=True splits into multiple columns (like str.extract)
name_cols = df["full_name"].str.strip().str.split(" ", expand=True)
# Returns a DataFrame with columns 0, 1, 2...
df["first"] = name_cols[0]
df["last"]  = name_cols[1]

# n= limits the number of splits
df["domain"] = df["email"].str.split("@", n=1, expand=True)[1]
# n=1: split only at the first "@" → gets everything after it

# ─────────────────────────────────────────────────────────────
# str.contains(), str.startswith(), str.endswith()
# ─────────────────────────────────────────────────────────────

df["is_company"] = df["email"].str.endswith("@company.com")
df["has_area_code"] = df["phone"].str.startswith("(")

# ─────────────────────────────────────────────────────────────
# str.extract(): Pull out groups using regex
# ─────────────────────────────────────────────────────────────

# Extract area code and main number from phone
df[["area", "number"]] = df["phone"].str.extract(
    r"[\(]?(\d{3})[\)\-\. ]*(\d{3}[\-\.]\d{4})"
)
# Regex groups in () become separate columns when expand=True (default)

# ─────────────────────────────────────────────────────────────
# str.len(): Length of each string
# ─────────────────────────────────────────────────────────────

df["name_length"] = df["full_name"].str.strip().str.len()
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — `.str` operations propagate NaN silently:** If a value in your column is NaN, most `.str` methods return NaN for that row without raising an error. This can mask data quality issues. Always check with `.isnull().sum()` before and after string operations.

**Pitfall — `str.split()` without `expand=True` gives you a column of lists:** A column of lists is difficult to work with. If you want separate columns, always use `str.split(..., expand=True)`.

**Pro-Tip — Chain `.str` operations:** `df["col"].str.strip().str.lower().str.replace("-", "_")` reads naturally and is efficient. Pandas evaluates these lazily as needed.

**Pro-Tip — Use `regex=False` when you don’t need regex:** `str.replace(".", "", regex=False)` replaces literal dots. Without `regex=False`, `.` means “any character” in regex — a very common source of subtle bugs.

-----

## ⚙️ Functions: apply, map & applymap / map

### The Big Picture

These three methods apply a function across your data, but they operate at different *levels* — and choosing the wrong one is a common source of confusion and slow code.

```
apply()    → operates on rows or columns of a DataFrame, or on a whole Series
map()      → operates element-by-element on a SERIES only
DataFrame.map() (formerly applymap) → operates element-by-element on a whole DATAFRAME
```

### Comparison Table

|Method                         |Operates On                |Input To Function     |Best For                                      |
|-------------------------------|---------------------------|----------------------|----------------------------------------------|
|`Series.apply(func)`           |Each element of a Series   |Single element        |Complex per-element logic on one column       |
|`DataFrame.apply(func, axis=0)`|Each column                |A full Series (column)|Column-level aggregations or transformations  |
|`DataFrame.apply(func, axis=1)`|Each row                   |A full Series (row)   |Row-level logic that uses multiple columns    |
|`Series.map(func/dict)`        |Each element of a Series   |Single element        |Mapping values using a dict or simple function|
|`DataFrame.map(func)`          |Each element in a DataFrame|Single element        |Applying a simple function to every cell      |

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "name":   ["Alice", "Bob", "Carol"],
    "salary": [95000, 72000, 105000],
    "dept":   ["Eng", "Mkt", "Eng"],
})

# ─────────────────────────────────────────────────────────────
# Series.apply(): Element-wise on one column (can use lambda or def)
# ─────────────────────────────────────────────────────────────

# Simple lambda
df["salary_k"] = df["salary"].apply(lambda x: f"${x/1000:.0f}k")
# Applies the formatting function to each element in the salary column

# Named function (better for complex logic)
def classify_salary(s):
    if s >= 100000:
        return "High"
    elif s >= 80000:
        return "Medium"
    else:
        return "Low"

df["salary_tier"] = df["salary"].apply(classify_salary)

# ─────────────────────────────────────────────────────────────
# DataFrame.apply() with axis=1: Row-wise (uses multiple columns)
# ─────────────────────────────────────────────────────────────

# The function receives a row as a Series; access values by column name
def annual_review(row):
    bonus_pct = 0.15 if row["dept"] == "Eng" else 0.10
    return row["salary"] * bonus_pct

df["bonus"] = df.apply(annual_review, axis=1)  # axis=1 means "apply across each row"

# DataFrame.apply() with axis=0: Column-wise
col_stats = df[["salary"]].apply(lambda col: col.max() - col.min(), axis=0)
# axis=0 means "apply down each column" — less common, usually groupby is better

# ─────────────────────────────────────────────────────────────
# Series.map(): Value substitution (dict mapping is its superpower)
# ─────────────────────────────────────────────────────────────

# Map using a dictionary: perfect for encoding or labeling
dept_map = {"Eng": "Engineering", "Mkt": "Marketing", "HR": "Human Resources"}
df["dept_full"] = df["dept"].map(dept_map)
# Values not found in the dict become NaN — use fillna() afterward if needed

# Map using a function (similar to apply, but slightly faster for simple cases)
df["name_upper"] = df["name"].map(str.upper)

# ─────────────────────────────────────────────────────────────
# DataFrame.map() — formerly applymap() in pandas < 2.1
# ─────────────────────────────────────────────────────────────

# Apply a function to every single cell in the DataFrame
numeric_df = df[["salary"]]
rounded = numeric_df.map(lambda x: round(x, -3))  # Round to nearest thousand

# NOTE: In pandas >= 2.1, applymap() was renamed to DataFrame.map()
# For compatibility, check which version you're on:
print(pd.__version__)
```

### When to Use What — Decision Guide

**Use `map()` (on a Series)** when you need to substitute or transform values using a dictionary lookup or a very simple function on a single column. It’s the cleanest tool for encoding categories.

**Use `Series.apply()`** when your per-element logic is too complex for a vectorized operation but only involves one column. Prefer it over `map()` when your function has conditional logic.

**Use `DataFrame.apply(axis=1)`** when your transformation logic *needs multiple columns* to compute a result for each row. It’s the right tool, but it’s slow on large DataFrames — consider `np.where` or `np.select` for simple conditionals instead.

**Avoid `DataFrame.apply(axis=1)` for large DataFrames:** It’s essentially a Python loop under the hood. On 1 million rows, it can be 100x slower than an equivalent vectorized NumPy operation.

### 💡 Pitfalls & Pro-Tips

**Pitfall — `apply` is not magic speed:** Many beginners use `apply(lambda x: ...)` thinking it’s vectorized. It’s not — it’s a Python loop. Always look for a vectorized alternative first (`np.where`, arithmetic operations, `.str` methods).

**Pitfall — `map()` with a dict turns unmatched values to NaN:** If your dictionary doesn’t cover all values in the Series, unmapped values become NaN silently. Use `df["col"].map(d).fillna(df["col"])` to keep original values for unmatched entries.

**Pro-Tip — `apply` with `result_type="expand"` returns a DataFrame from a row-wise function:** If your function returns a tuple or list for each row, `df.apply(func, axis=1, result_type="expand")` automatically expands the result into multiple columns.

-----

## 🔗 Combining Data: merge, concat & join

### The Big Picture

Combining DataFrames is something you’ll do constantly. Pandas offers three main tools, and they solve different problems.

### Comparison Table

|Method       |Primary Use                                |SQL Equivalent      |Joins On                   |
|-------------|-------------------------------------------|--------------------|---------------------------|
|`pd.merge()` |Join two DataFrames on shared column values|`JOIN`              |Column values (key columns)|
|`pd.concat()`|Stack DataFrames vertically or horizontally|`UNION ALL`         |Position (row/column)      |
|`df.join()`  |Join on the row *index*                    |`JOIN` (index-based)|Index labels               |

-----

### `pd.merge()` — SQL-Style Joins

```python
import pandas as pd

employees = pd.DataFrame({
    "emp_id": [1, 2, 3, 4],
    "name":   ["Alice", "Bob", "Carol", "Dave"],
    "dept_id":[10, 20, 10, 30],
})

departments = pd.DataFrame({
    "dept_id":   [10, 20, 40],
    "dept_name": ["Engineering", "Marketing", "Finance"],
})

# INNER JOIN: Only rows with matching keys in BOTH DataFrames
inner = pd.merge(employees, departments, on="dept_id", how="inner")
# Dave (dept_id=30) is excluded — no match in departments
# Finance (dept_id=40) is excluded — no match in employees

# LEFT JOIN: All rows from the left (employees), matching from right
left = pd.merge(employees, departments, on="dept_id", how="left")
# Dave is kept with dept_name=NaN (no match in departments)
# Finance is excluded (only the left side is kept fully)

# RIGHT JOIN: All rows from the right, matching from left
right = pd.merge(employees, departments, on="dept_id", how="right")

# OUTER JOIN: All rows from both DataFrames (unmatched = NaN)
outer = pd.merge(employees, departments, on="dept_id", how="outer")

# When key column names differ between DataFrames
orders = pd.DataFrame({"order_id": [1,2], "customer": [101, 102]})
customers = pd.DataFrame({"cust_id": [101,102], "name": ["Alice","Bob"]})
merged = pd.merge(
    orders, customers,
    left_on="customer",  # Key column in the left DataFrame
    right_on="cust_id",  # Key column in the right DataFrame
    how="inner"
)

# Validate the merge to catch data quality issues
validated = pd.merge(
    employees, departments, on="dept_id", how="inner",
    validate="many_to_one"  # Assert: each dept_id in employees maps to one dept
    # Other options: "one_to_one", "one_to_many", "many_to_many"
)

# indicator=True adds a "_merge" column showing where each row came from
with_source = pd.merge(employees, departments, on="dept_id", how="outer", indicator=True)
# _merge values: "left_only", "right_only", "both"
```

### `pd.concat()` — Stacking DataFrames

```python
# Vertical stacking (adding rows) — most common use case
q1_sales = pd.DataFrame({"month": ["Jan","Feb","Mar"], "sales": [1000, 1200, 900]})
q2_sales = pd.DataFrame({"month": ["Apr","May","Jun"], "sales": [1100, 1400, 950]})

full_year = pd.concat(
    [q1_sales, q2_sales],   # List of DataFrames to stack
    axis=0,                  # axis=0 means "stack vertically (add rows)"
    ignore_index=True,       # Reset index to 0,1,2,... (avoid duplicate index values)
)

# Add a key to identify which DataFrame each row came from
labeled = pd.concat(
    [q1_sales, q2_sales],
    keys=["Q1", "Q2"],       # Creates a MultiIndex to identify the source
    ignore_index=False
)

# Horizontal stacking (adding columns)
names = pd.DataFrame({"first": ["Alice","Bob"], "last": ["Smith","Jones"]})
scores = pd.DataFrame({"math": [92, 85], "science": [88, 91]})
combined = pd.concat([names, scores], axis=1)  # axis=1 means "add columns side by side"
# IMPORTANT: rows are aligned by index label, not position
```

### `df.join()` — Index-Based Joining

```python
# join() uses the index as the key — not a column
employees.set_index("emp_id", inplace=True)
departments.set_index("dept_id", inplace=True)

# join() defaults to a left join on the index
result = employees.join(departments, on="dept_id")
# On= tells join which column of the LEFT df to use to look up the RIGHT df's index
```

### When to Use What — Decision Guide

**Use `pd.merge()`** when you need to join two tables on shared *column values*, just like a SQL JOIN. This is the most flexible and commonly used option. It handles all four join types and works regardless of the index.

**Use `pd.concat()`** when you want to *stack* DataFrames that have the same columns (like quarterly reports into a yearly one), or side-by-side when they share the same index.

**Use `df.join()`** when your DataFrames are already indexed on their join key and you want a quick, concise syntax. It’s essentially shorthand for `merge(..., left_index=True, right_index=True)`.

### 💡 Pitfalls & Pro-Tips

**Pitfall — Duplicate column names after merge:** If both DataFrames have a column called “date”, the merged result will have “date_x” and “date_y”. Use `suffixes=("_left", "_right")` to control this, or drop/rename before merging.

**Pitfall — `concat` aligns on index, not position:** If you concat two DataFrames with different index values and `axis=1`, pandas aligns them by index label, which can produce lots of NaN. Use `ignore_index=True` or reset indexes first.

**Pro-Tip — Always use `validate=` in merges for important pipelines:** A `many_to_one` merge accidentally becoming `many_to_many` is a silent data explosion. The `validate` argument turns this into a loud, immediate error — a huge help for data quality.

-----

## 📅 Time Series: Dates & Datetime Components

### What Is a Datetime in Pandas?

Pandas has a dedicated `datetime64` type for dates and times that unlocks powerful operations: sorting chronologically, filtering by date ranges, resampling by week or month, and extracting components like day-of-week or quarter. The key step is always *converting* your date column from a string to a proper `datetime64`.

```python
import pandas as pd

df = pd.DataFrame({
    "order_date": ["2024-01-15", "2024-03-22", "2024-07-04", "2024-11-28"],
    "revenue":    [1500, 2300, 800, 4200],
    "status":     ["shipped", "pending", "shipped", "cancelled"],
})

# ─────────────────────────────────────────────────────────────
# CONVERTING TO DATETIME
# ─────────────────────────────────────────────────────────────

# pd.to_datetime() is the Swiss Army knife of date parsing
df["order_date"] = pd.to_datetime(df["order_date"])
# Now dtype is datetime64[ns] — pandas knows this is a date

# Handle custom date formats explicitly for speed and safety
df["order_date"] = pd.to_datetime(
    df["order_date"],
    format="%Y-%m-%d",    # Explicitly tell pandas the format — faster than inference
    errors="coerce"        # Turn unparseable dates into NaT (Not a Time) instead of crashing
)

# Common format strings:
# "%Y-%m-%d"     → 2024-01-15
# "%d/%m/%Y"     → 15/01/2024
# "%m/%d/%Y"     → 01/15/2024 (US format)
# "%B %d, %Y"    → January 15, 2024
# "%Y-%m-%d %H:%M:%S" → 2024-01-15 14:30:00

# Set as index for time series operations
df = df.set_index("order_date")

# ─────────────────────────────────────────────────────────────
# EXTRACTING DATETIME COMPONENTS with .dt accessor
# ─────────────────────────────────────────────────────────────

df = df.reset_index()  # Bring order_date back as a column to demonstrate

df["year"]       = df["order_date"].dt.year          # Integer year: 2024
df["month"]      = df["order_date"].dt.month         # Integer month: 1–12
df["month_name"] = df["order_date"].dt.month_name()  # Full name: "January"
df["day"]        = df["order_date"].dt.day            # Day of month: 1–31
df["day_name"]   = df["order_date"].dt.day_name()    # "Monday", "Tuesday", etc.
df["dayofweek"]  = df["order_date"].dt.dayofweek     # 0=Monday … 6=Sunday
df["quarter"]    = df["order_date"].dt.quarter       # 1–4
df["week"]       = df["order_date"].dt.isocalendar().week  # ISO week number

# ─────────────────────────────────────────────────────────────
# DATE ARITHMETIC & FILTERING
# ─────────────────────────────────────────────────────────────

# Add time with DateOffset
from pandas.tseries.offsets import BDay  # Business days
df["due_date"] = df["order_date"] + pd.DateOffset(days=30)
df["next_bizday"] = df["order_date"] + BDay(1)   # Next business day

# Time difference (creates a timedelta)
df["days_ago"] = pd.Timestamp("today") - df["order_date"]
df["days_ago_int"] = df["days_ago"].dt.days   # Convert timedelta to integer days

# Date range filtering
start = pd.Timestamp("2024-01-01")
end   = pd.Timestamp("2024-06-30")
h1_sales = df[df["order_date"].between(start, end)]

# ─────────────────────────────────────────────────────────────
# RESAMPLING: Aggregate by time period
# ─────────────────────────────────────────────────────────────

df_indexed = df.set_index("order_date")  # Datetime index is required for resample

monthly = df_indexed["revenue"].resample("ME").sum()
# "ME" = Month End frequency; aggregates all rows within each month
# Other aliases: "W"=weekly, "QE"=quarter end, "YE"=year end, "D"=daily
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — `pd.to_datetime()` inference can be wrong with ambiguous dates:** “01/02/03” could be Jan 2, 2003 or Feb 1, 2003 or 2001-02-03. Always specify `format=` explicitly for production code. Ambiguity is silent and dangerous.

**Pitfall — NaT (Not a Time) is the datetime equivalent of NaN:** `errors="coerce"` produces NaT for bad dates. NaT propagates through arithmetic (NaT + 30 days = NaT) and is excluded from `.dt` operations. Check with `df["date"].isna()`.

**Pro-Tip — Store dates as `datetime64` as early as possible:** If you load a CSV and the date column comes in as `object` (string), convert it immediately at load time using `parse_dates=["date_col"]` in `read_csv`. Operating on string dates is slow and error-prone.

-----

## 🔁 Duplicates: Finding & Removing

### What Are Duplicates?

Duplicate rows occur when the same data appears more than once — common in database exports, API responses, and merged datasets. Finding and handling them carefully (rather than blindly dropping them) is a mark of clean data engineering.

```python
import pandas as pd

df = pd.DataFrame({
    "order_id":  [101, 102, 103, 101, 104, 102],
    "customer":  ["Alice", "Bob", "Carol", "Alice", "Dave", "Bob"],
    "amount":    [150.00, 200.00, 75.00, 150.00, 300.00, 200.00],
    "status":    ["paid", "paid", "pending", "paid", "paid", "refunded"],  # Note: last 102 differs!
})

# ─────────────────────────────────────────────────────────────
# duplicated(): Identify duplicate rows
# ─────────────────────────────────────────────────────────────

# Returns a boolean Series: True for rows that ARE duplicates
dup_mask = df.duplicated()
# By default, marks all occurrences AFTER the first as True
# (The first occurrence is considered the "original" → False)

print(df[dup_mask])   # Show only the duplicate rows

# Count duplicates
print(f"Number of duplicate rows: {dup_mask.sum()}")

# keep="first" (default): First occurrence = not duplicate, rest = duplicate
# keep="last": Last occurrence = not duplicate, rest = duplicate
# keep=False: ALL occurrences of a duplicate are marked True
all_dups = df.duplicated(keep=False)
print(df[all_dups])  # Shows every row that has at least one duplicate

# Check duplicates on specific columns (ignoring other columns)
dup_orders = df.duplicated(subset=["order_id"])
# order_id 101 and 102 appear twice, so rows 3 and 5 are marked as duplicates

# ─────────────────────────────────────────────────────────────
# drop_duplicates(): Remove duplicate rows
# ─────────────────────────────────────────────────────────────

# Drop based on ALL columns (keeps first occurrence of each unique row)
df_unique = df.drop_duplicates()

# Drop based on specific columns (keep first occurrence of each order_id)
df_unique_orders = df.drop_duplicates(subset=["order_id"], keep="first")
# Keeps order 102 with status="paid", drops the later "refunded" row
# WARNING: Think carefully about which duplicate to keep — it matters!

# Keep the LAST occurrence (useful for "latest update" scenarios)
df_latest = df.drop_duplicates(subset=["order_id"], keep="last")
# Keeps order 102 with status="refunded" — the most recent record

# keep=False: Drop ALL rows that have any duplicate (keeps only truly unique rows)
df_no_dups_at_all = df.drop_duplicates(keep=False)
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — Dropping duplicates without thinking about `keep=`:** `drop_duplicates()` defaults to keeping the first occurrence. In an event log or update stream, the *last* occurrence is usually more accurate. Always be deliberate about which duplicate to retain.

**Pitfall — Duplicate detection is case-sensitive:** `"Alice"` and `"alice"` are not considered duplicates. Normalize strings with `.str.lower().str.strip()` before checking for duplicates in string columns.

**Pro-Tip — Always investigate before dropping:** Before calling `drop_duplicates`, look at the duplicates with `df[df.duplicated(keep=False)]`. Sometimes “duplicates” reveal real data issues — like two legitimate orders with the same ID, which is a data integrity problem, not just noise to be cleaned.

**Pro-Tip — Use `duplicated().sum()` as a data quality metric:** Adding `df.duplicated().sum()` to your data pipeline logging gives you a quick sanity check on source data quality. A sudden spike in duplicates often signals an upstream data issue.

-----

## ⚡ Advanced: category dtype & query()

### The `category` Dtype: Memory’s Best Friend

When a string column has a small number of unique values (like “North”, “South”, “East”, “West”), storing it as `object` is wasteful — pandas stores the full string for every row. The `category` dtype instead stores unique strings once and uses compact integers as row-level pointers.

```python
import pandas as pd
import numpy as np

# Create a large DataFrame to demonstrate memory savings
np.random.seed(42)
n = 1_000_000  # 1 million rows

df = pd.DataFrame({
    "region": np.random.choice(["North", "South", "East", "West"], size=n),
    "status": np.random.choice(["active", "inactive", "pending"], size=n),
    "value":  np.random.randn(n),
})

# Check memory before conversion
print("Before:", df.memory_usage(deep=True).sum() / 1024**2, "MB")

# Convert low-cardinality string columns to category
df["region"] = df["region"].astype("category")
df["status"] = df["status"].astype("category")

# Check memory after
print("After:", df.memory_usage(deep=True).sum() / 1024**2, "MB")
# Typically 60-80% memory reduction for low-cardinality string columns!

# Inspect the categories (the unique values stored once)
print(df["region"].cat.categories)     # Index(['East', 'North', 'South', 'West'])
print(df["region"].cat.codes)          # Integer codes (0=East, 1=North, etc.)

# Add a new category that doesn't yet appear in the data
df["region"] = df["region"].cat.add_categories(["Central"])

# Rename categories
df["status"] = df["status"].cat.rename_categories({"active": "Active", "inactive": "Inactive", "pending": "Pending"})

# Set a meaningful order (makes sorting and comparison logical)
df["status"] = df["status"].cat.set_categories(
    ["Inactive", "Pending", "Active"],
    ordered=True   # Now you can use < > comparisons: "Inactive" < "Active"
)
high_priority = df[df["status"] > "Pending"]   # Works because ordered=True
```

### The `.query()` Method: Readable Filtering

```python
# Standard boolean filtering — functional but visually cluttered
result = df[
    (df["region"].isin(["North", "East"])) &
    (df["value"] > 0.5) &
    (df["status"] == "Active")
]

# .query() method — same result, much more readable
result = df.query("region in ['North', 'East'] and value > 0.5 and status == 'Active'")

# Reference external Python variables with @
threshold = 0.5
target_regions = ["North", "East"]
result = df.query("region in @target_regions and value > @threshold")
# @ prefix tells query() to look up the variable in the local Python scope

# query() works with datetime comparisons too
df["date"] = pd.to_datetime(pd.date_range("2024-01-01", periods=n, freq="1min"))
result = df.query("date > '2024-06-01' and value > 0")

# query() with column names that have spaces (use backticks)
df["my column"] = df["value"] * 2
result = df.query("`my column` > 1.0")
```

### 💡 Pitfalls & Pro-Tips

**Pitfall — `category` breaks some operations that work on strings:** Concatenating two DataFrames with `category` columns that have different categories produces an `object` column, not a combined `category`. Always merge/concat first, then cast to `category`.

**Pitfall — Adding a value not in the categories raises an error:** `df.loc[0, "region"] = "Pacific"` will raise a `ValueError` if “Pacific” is not in the categories. Use `cat.add_categories(["Pacific"])` first, then assign.

**Pro-Tip — Cast to `category` right after loading:** The best place to cast is right in your `read_csv` call with `dtype={"region": "category"}`. This avoids ever storing the large string representation in memory.

**Pro-Tip — `.query()` can be slightly faster on large DataFrames:** For complex multi-condition filters on very large DataFrames, `query()` can outperform standard boolean indexing because it uses `numexpr` under the hood (if installed). Install with `pip install numexpr`.

-----

## 🏆 Best Practices & Final Wisdom

### Memory Efficiency

Working with large datasets efficiently starts with good habits from the moment you load your data.

**Load only what you need.** Use `usecols=` in `read_csv` to skip unnecessary columns and `nrows=` or `chunksize=` for large files. A column you never use still takes up RAM.

**Cast types at load time.** Use `dtype={"status": "category", "id": "int32"}` in `read_csv`. Letting pandas infer types is convenient but rarely optimal — it defaults to `int64` and `float64` even when `int16` or `float32` would suffice.

**Use `category` for low-cardinality strings.** Any column with fewer than ~50 unique values in thousands of rows should be `category` dtype. The memory savings are enormous and operations on it are often faster.

**Downcast numerics when precision allows.** `int8` holds values 0–127, `int16` holds up to 32,767, `int32` holds up to ~2.1 billion. If your column holds ages, `int8` is fine. `float32` is sufficient for most financial and scientific data.

```python
# Efficient loading pattern
df = pd.read_csv(
    "data.csv",
    usecols=["id", "region", "status", "amount"],
    dtype={
        "region": "category",
        "status": "category",
        "amount": "float32",
        "id":     "int32",
    },
    parse_dates=["created_at"],
)
```

### Why You Should Avoid `inplace=True`

This deserves special attention because `inplace=True` feels natural to beginners but causes real problems.

```python
# The "inplace" way — looks reasonable but has serious issues
df.drop(columns=["temp_col"], inplace=True)
df.rename(columns={"old": "new"}, inplace=True)
df.sort_values("date", inplace=True)

# The correct way — always reassign
df = df.drop(columns=["temp_col"])
df = df.rename(columns={"old": "new"})
df = df.sort_values("date")
```

**Why `inplace=True` is problematic:**

First, it **doesn’t actually save memory**. Despite what the name implies, pandas almost always creates a copy internally even with `inplace=True`. The “in-place” mutation is an illusion — you pay the memory cost without the benefit of having the original available.

Second, it **prevents method chaining**, one of pandas’ most elegant patterns:

```python
# Without inplace (beautiful and readable)
df = (
    pd.read_csv("data.csv")
    .dropna(subset=["amount"])
    .rename(columns={"amt": "amount_usd"})
    .sort_values("date")
    .reset_index(drop=True)
)

# With inplace (verbose and fragile — can't chain)
df = pd.read_csv("data.csv")
df.dropna(subset=["amount"], inplace=True)
df.rename(columns={"amt": "amount_usd"}, inplace=True)
df.sort_values("date", inplace=True)
df.reset_index(drop=True, inplace=True)
```

Third, and most importantly, **`inplace=True` is being soft-deprecated.** The pandas development team has signaled that `inplace=True` will be removed in a future version. The official guidance (as of pandas 2.x) is to stop using it entirely. If you build habits around `inplace=True` now, your code will break when that happens.

**The rule is simple:** always reassign — `df = df.operation()`. It’s explicit, chainable, and future-proof.

### A Quick Reference of Anti-Patterns to Avoid

**Avoid iterating with `for` loops over rows.** `for index, row in df.iterrows()` is almost always the wrong approach — it’s hundreds of times slower than vectorized operations. If you find yourself writing a loop, ask: “Is there a vectorized way to do this?” (`apply`, `np.where`, arithmetic, `.str` methods).

**Avoid chained indexing.** `df["col"][0] = value` is chained indexing and produces the `SettingWithCopyWarning`. Use `df.loc[0, "col"] = value` instead.

**Avoid storing DataFrames with `object` dtype string columns longer than needed.** If you’ve finished filtering and aggregating, cast string columns you still need to `category` or extract what you need into simpler structures.

**Always reset the index after filtering/sorting if you use integer-based lookups.** Call `.reset_index(drop=True)` to avoid the `loc` vs `iloc` drift bug described in the indexing section.

-----

*This guide was designed to grow with you — from your first `read_csv` to optimizing production pipelines. The best way to internalize these concepts is to apply them to real data. Start with something you care about, ask questions of it with pandas, and the patterns will click into place.*

*Happy wrangling! 🐼*
