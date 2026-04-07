# 🗺️ The Data Scientist’s Blueprint: Mastering Pandas & NumPy

### A Plain-English Guide from Raw Memory to Production-Ready Data

> *“Before you can model data, you have to understand it. Before you can understand it, you have to be able to hold it, shape it, and clean it. That’s what this guide is for.”*

-----

## Table of Contents

1. [The Foundation — Memory, Arrays, and the Engine Under the Hood](#chapter-1-the-foundation--memory-arrays-and-the-engine-under-the-hood)
1. [The Structure — Series, DataFrames, and Labels](#chapter-2-the-structure--series-dataframes-and-labels)
1. [The First Look — Reading Data and Inspecting It](#chapter-3-the-first-look--reading-data-and-inspecting-it)
1. [The Selection — Reaching Into Your Data Precisely](#chapter-4-the-selection--reaching-into-your-data-precisely)
1. [The Cleanup — Filtering, Missing Data, and Duplicates](#chapter-5-the-cleanup--filtering-missing-data-and-duplicates)
1. [The Transformation — Strings, Dates, and New Features](#chapter-6-the-transformation--strings-dates-and-new-features)
1. [The Combination — Merging and Stacking Datasets](#chapter-7-the-combination--merging-and-stacking-datasets)
1. [The Performance — Optimization for ML Pipelines](#chapter-8-the-performance--optimization-for-ml-pipelines)
1. [Summary Reference Table](#summary-reference-table)

-----

## Chapter 1: The Foundation — Memory, Arrays, and the Engine Under the Hood

### The Analogy: A Spreadsheet vs. a Block of Frozen Memory

Imagine you have two ways to store a list of 1,000 temperatures. The first way is to write each temperature on a separate sticky note and toss them all in a box. To add them up, you pull out each note one at a time, read it, do a little arithmetic, and put it back. That’s essentially what a Python `list` does — each item is its own Python object, stored wherever memory happens to be free, with no guarantee they’re sitting next to each other.

The second way is to write all 1,000 temperatures in a single, unbroken row on a long strip of graph paper, one number per box, every box the same size. To add them up, you just run your finger across the strip — no hunting, no jumping around. That is exactly what a **NumPy array** does. It reserves one single, contiguous block of memory, stores every value as the same type (say, 64-bit floating point), and lets the CPU process all of them with a single instruction. This is called **vectorization**, and it is why NumPy operations run in milliseconds while equivalent Python loops take seconds.

This matters enormously for you as a data scientist, because **Pandas is built entirely on top of NumPy**. Every column in a Pandas DataFrame is, at its core, a NumPy array with some labels attached. When you ask Pandas to multiply a column by two, it hands that job to NumPy, which hands it to optimized C code, which runs at near-hardware speed. Pandas gives you the labels and convenience; NumPy supplies the muscle.

### Understanding the `ndarray` — NumPy’s Core Object

```python
import numpy as np

# Create a simple 1D NumPy array — all float64, stored in a single memory block
temperatures = np.array([98.6, 99.1, 97.8, 100.2, 98.9], dtype=np.float64)

# 'shape' tells you the dimensions of your data
print(temperatures.shape)   # (5,) — one dimension, five elements

# 'dtype' tells you what type each value is
print(temperatures.dtype)   # float64 — each value takes 8 bytes of memory

# 'strides' reveals how many bytes to jump to reach the next element
# For float64, that's 8 bytes — which confirms contiguous storage
print(temperatures.strides)  # (8,)

# This 2D array is like a table: 3 rows, 4 columns
matrix = np.array([[1, 2, 3, 4],
                   [5, 6, 7, 8],
                   [9, 10, 11, 12]], dtype=np.float32)

print(matrix.shape)    # (3, 4)
# To move one row down: jump 4 values × 4 bytes = 16 bytes
# To move one column right: jump 1 value × 4 bytes = 4 bytes
print(matrix.strides)  # (16, 4)
```

The key insight here is **dtype homogeneity** — every element in an `ndarray` is the same type. This is what makes vectorized operations possible. If you have a mix of types (say, integers and strings), NumPy cannot apply a single low-level instruction to every element. That forces a fallback to Python’s slower object system, which is exactly the trap we’ll warn you about with Pandas’ `object` dtype in the next chapter.

### Why Vectorization Changes Everything for ML

```python
import numpy as np

large_array = np.random.rand(1_000_000)  # One million random floats

# --- The slow way: a Python loop ---
# Python calls its interpreter overhead once per iteration — 1 million times
result_slow = []
for val in large_array:
    result_slow.append(val * 2)

# --- The fast way: vectorized NumPy operation ---
# NumPy sends the entire array to a C routine in a single call
result_fast = large_array * 2  # Runs roughly 100x faster than the loop above

# You can also apply conditions across an entire array at once
# This returns a boolean array: True where the condition is met
above_average = large_array > large_array.mean()
print(above_average[:5])  # e.g., [False True False True True]
```

### Know the Limits

NumPy arrays live entirely in your computer’s RAM. There is no disk-spill, no streaming, no lazy evaluation by default — the moment you create an array, that memory is allocated. A float64 array with 10 million elements takes 80 MB; a 2D array of the same dtype with 10 million rows and 50 columns takes roughly 4 GB. Before you load a large dataset, it’s worth doing a quick mental multiplication: `rows × columns × bytes_per_value`. For float64, bytes_per_value is 8. For float32, it’s 4. This single habit will save you from a lot of out-of-memory crashes.

-----

## Chapter 2: The Structure — Series, DataFrames, and Labels

### The Analogy: The Filing Cabinet

Picture a large metal filing cabinet. Each **drawer** is labeled — “Customer Info,” “Sales Records,” “Product Data.” Inside each drawer, the papers are arranged in a specific order, and every paper has a row number printed on it. You can find any piece of paper either by its label (the drawer name + row label) or by its physical position (third drawer, fifteenth sheet from the top).

A Pandas **DataFrame** is that filing cabinet. Each column is a drawer (with a name), and each row has both an index label (which could be a number, a date, a customer ID — whatever makes sense) and a physical position (0, 1, 2, …). A **Series** is a single drawer pulled out and laid flat — it’s one column with its row labels still attached.

This distinction between *labels* and *positions* is the single most important concept in Pandas. Almost every confusing behavior, every unexpected error, and every subtle bug traces back to mixing up labels and positions. We’ll return to this idea throughout the guide.

### Series — A Labeled Column

```python
import pandas as pd
import numpy as np

# A Series is a 1D array with an index (labels on the left)
scores = pd.Series([88, 92, 75, 61, 95],
                   index=["alice", "bob", "carol", "dave", "eve"])

print(scores)
# alice    88
# bob      92
# carol    75
# dave     61
# eve      95
# dtype: int64

# The NumPy array is still there underneath — you can always reach it
print(scores.to_numpy())   # array([88, 92, 75, 61, 95])
print(scores.dtype)        # int64 — NumPy's integer type

# Arithmetic operates on the values, alignment happens on the labels
# This is critically different from a Python list
bonus = pd.Series([5, 10, 5], index=["bob", "carol", "alice"])
print(scores + bonus)
# alice    93.0   ← aligned by label: 88 + 5
# bob     102.0   ← aligned by label: 92 + 10
# carol    80.0   ← aligned by label: 75 + 5
# dave      NaN   ← no matching label in 'bonus'
# eve       NaN   ← no matching label in 'bonus'
# dtype: float64
```

Notice that last result carefully. When Pandas couldn’t find a matching label in `bonus` for “dave” and “eve,” it filled those positions with `NaN` (Not a Number) rather than raising an error. This **automatic label alignment** is one of Pandas’ most powerful features and, at the same time, one of its most common sources of surprise.

### DataFrame — The Full Filing Cabinet

```python
import pandas as pd
import numpy as np

# A DataFrame is a collection of Series that share the same index
df = pd.DataFrame({
    "name":   ["Alice", "Bob", "Carol", "Dave"],
    "age":    np.array([25, 32, 41, 28], dtype=np.int32),
    "salary": np.array([50000.0, 72000.0, 95000.0, 61000.0], dtype=np.float64),
    "dept":   ["Engineering", "Marketing", "Engineering", "HR"],
})

print(df)
#     name  age   salary         dept
# 0  Alice   25  50000.0  Engineering
# 1    Bob   32  72000.0    Marketing
# 2  Carol   41  95000.0  Engineering
# 3   Dave   28  61000.0           HR

# The index (0, 1, 2, 3) was assigned automatically
# You can replace it with something more meaningful
df_indexed = df.set_index("name")
print(df_indexed.index)  # Index(['Alice', 'Bob', 'Carol', 'Dave'], dtype='object')

# Each column is backed by a NumPy array of the appropriate dtype
print(df.dtypes)
# name       object   ← strings stored as Python objects (be careful here)
# age         int32
# salary    float64
# dept       object
```

### The `object` Dtype — A Performance Warning

> **Watch Out: The `object` Dtype Trap**
> 
> When Pandas sees a column of strings (like “Engineering” or “Marketing”), it stores it as `object` dtype. This means instead of a clean NumPy array of fixed-size values, Pandas is storing an array of *pointers* — each one pointing to a separate Python string object somewhere in memory. This defeats vectorization entirely. String operations on `object` columns are essentially Python loops in disguise, and they’re 10 to 100 times slower than numeric operations. The fix for low-cardinality string columns (columns with only a few unique values) is the `category` dtype, which we cover in Chapter 8.

### Know the Limits

By default, when you print a large DataFrame, Pandas will only show the first and last few rows and columns, hiding the middle ones with `...`. This is to protect your screen from being overwhelmed, but it also means **you can easily miss structural problems** (like a block of missing values in the middle of the dataset) if you only look at the printed output. Always use `.info()` and `.describe()` (covered in the next chapter) as your true inspection tools, not just the print output.

-----

## Chapter 3: The First Look — Reading Data and Inspecting It

### The Analogy: The Doctor’s Intake Form

When a patient arrives at a clinic, the doctor doesn’t immediately start running tests. They start with an intake form — name, age, known allergies, current medications. This gives them a quick, structured overview before diving into specifics. Loading a dataset should follow the same discipline. `read_csv()` is how you admit the patient; `head()`, `info()`, and `describe()` are your intake form.

### Reading Data with `read_csv()`

```python
import pandas as pd

# The simplest possible read — let Pandas figure everything out
df = pd.read_csv("customers.csv")

# --- The smarter approach for larger datasets ---
df = pd.read_csv(
    "customers.csv",

    # Tell Pandas the type of each column upfront
    # This prevents 'int64' columns from being read as 'float64' just because
    # of a few missing values, and avoids string columns loading as 'object'
    dtype={
        "customer_id": "int32",
        "age":         "int16",
        "income":      "float32",
        "region":      "category",    # Low-cardinality string → huge memory saving
    },

    # Automatically convert date strings to proper datetime objects
    parse_dates=["signup_date", "last_purchase"],

    # Tell Pandas which strings in the file mean "this value is missing"
    na_values=["N/A", "unknown", "--", ""],

    # Only load the columns you actually need — saves memory and time
    usecols=["customer_id", "age", "income", "region", "signup_date"],
)
```

### Reading Excel and Writing Back

```python
# Reading from Excel works the same way — just slower due to XML parsing
df_excel = pd.read_excel("report.xlsx",
                          sheet_name="Q1_Data",
                          engine="openpyxl")

# Writing back to CSV — always set index=False unless your index is meaningful
df.to_csv("customers_cleaned.csv", index=False)

# Writing to Excel with a specific sheet name
with pd.ExcelWriter("output.xlsx", engine="openpyxl") as writer:
    df.to_excel(writer, sheet_name="Cleaned", index=False)
```

### Your Three Intake Tools: `head()`, `info()`, and `describe()`

These three commands should be the first things you run on any new dataset. Think of them as answering three questions: “What does it look like?”, “What is it made of?”, and “What are the numbers saying?”

```python
# head() — "What does my data look like?"
# Shows the first 5 rows by default; pass a number to see more
print(df.head(10))

# tail() — check the end of the file for summary rows or junk data
print(df.tail())

# --- info() — "What is my data made of?" ---
# Shows column names, non-null counts, and dtypes
# This is your missing-value radar and dtype checker in one
df.info()
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 10000 entries, 0 to 9999
# Data columns (total 5 columns):
#  #   Column       Non-Null Count  Dtype
# ---  ------       --------------  -----
#  0   customer_id  10000 non-null  int32
#  1   age          9850 non-null   float64   ← 150 missing values!
#  2   income       9900 non-null   float32
#  3   region       10000 non-null  category
#  4   signup_date  10000 non-null  datetime64[ns]
# dtypes: category(1), datetime64[ns](1), float32(1), float64(1), int32(1)
# memory usage: 469.0 KB

# The true memory usage of object columns is underestimated by info()
# Use this for an accurate picture:
print(df.memory_usage(deep=True).sum() / 1e6, "MB")

# --- describe() — "What are the numbers saying?" ---
# For numeric columns: count, mean, std, min, quartiles, max
# For all columns (including strings): use include='all'
print(df.describe())
print(df.describe(include="all"))  # Also shows category and string stats

# A few extra quick-inspection commands worth knowing
print(df.shape)          # (rows, columns) — the dimensions of your table
print(df.columns.tolist()) # All column names as a plain Python list
print(df.nunique())      # How many unique values per column — great for spotting IDs vs. categories
print(df.isnull().sum()) # Count of missing values per column
```

> **Watch Out: `info()` Lies About Memory for `object` Columns**
> 
> `df.info()` shows an estimated memory usage that can be 5 to 10 times too low for `object`-dtype columns. This is because it counts only the size of the pointer array, not the Python string objects those pointers point to. Always use `df.memory_usage(deep=True).sum()` for an honest picture of how much RAM your DataFrame is actually consuming. This matters the moment your data gets anywhere near the size of your available RAM.

### Know the Limits

`read_csv()` loads the entire file into memory at once. If your CSV is 10 GB and you only have 8 GB of RAM, the read will crash. For files larger than memory, use the `chunksize` parameter:

```python
# chunksize turns read_csv into an iterator — each chunk is a small DataFrame
# You process each chunk and accumulate only the results, not the raw data
aggregated_results = []

for chunk in pd.read_csv("huge_file.csv", chunksize=100_000):
    # Do your aggregation on each chunk
    chunk_summary = chunk.groupby("region")["revenue"].sum()
    aggregated_results.append(chunk_summary)

# Combine the per-chunk summaries
final = pd.concat(aggregated_results).groupby(level=0).sum()
```

Keep in mind that `chunksize` is not parallelism — chunks are processed one at a time. For truly parallel, out-of-core processing, look at the `Dask` library, which mimics the Pandas API but runs lazily across chunks.

-----

## Chapter 4: The Selection — Reaching Into Your Data Precisely

### The Analogy: Two Ways to Find a Book in a Library

Imagine you’re in a library. You can find a book in two ways. The first way is by its catalog information — you know the title is “Data Science Basics” and it’s in the “Technology” section. You find it by its *label*. The second way is by its physical location — “third shelf from the left, second row from the top, fourth book from the left end.” You find it by its *position*.

Pandas gives you exactly these two systems. **`loc`** finds data by its label. **`iloc`** finds data by its integer position. They look almost identical, which is why they confuse almost everyone at first, but the rule is simple: if you’re thinking in terms of names, use `loc`. If you’re thinking in terms of numbers (like “the first three rows”), use `iloc`.

### The Confusing Cousins: `loc` vs. `iloc`

```python
import pandas as pd

df = pd.DataFrame({
    "score":  [88, 92, 75, 61, 95],
    "grade":  ["B", "A", "C", "D", "A"],
    "passed": [True, True, True, False, True],
}, index=["alice", "bob", "carol", "dave", "eve"])

# --- loc: LABEL-based selection ---
# Syntax: df.loc[row_label, column_label]

# Select a single row by its label
print(df.loc["alice"])

# Select a range of rows by label — IMPORTANT: both ends are INCLUSIVE
# This gets alice, bob, AND carol (not like Python slices!)
print(df.loc["alice":"carol"])

# Select specific rows and specific columns
print(df.loc[["alice", "eve"], ["score", "grade"]])

# Boolean arrays work perfectly with loc
high_scorers = df.loc[df["score"] > 80]
print(high_scorers)

# --- iloc: POSITION-based selection ---
# Syntax: df.iloc[row_integer, column_integer]

# Select by integer position (like regular Python indexing)
print(df.iloc[0])      # First row (alice)
print(df.iloc[-1])     # Last row (eve)

# Select a range — IMPORTANT: end is EXCLUSIVE, just like Python slices
# This gets rows 0 and 1 (alice and bob), NOT row 2
print(df.iloc[0:2])

# Select specific rows and specific column positions
print(df.iloc[[0, 4], [0, 1]])  # First and last rows, first two columns
```

The single most common mistake is forgetting the difference in how slices work. With `loc["alice":"carol"]`, you get carol. With `iloc[0:2]`, you do not get the row at position 2. This inconsistency is a historical quirk of Pandas, and it trips up even experienced users. Develop the habit of mentally flagging any slice operation and asking yourself: “Am I using loc or iloc right now, and do I want the end included?”

### `at` and `iat` — For When You Need Just One Cell

When you need to read or write a single cell, `loc` and `iloc` do the job but carry unnecessary overhead from building a full result structure. For single-cell access — especially inside loops — use `at` (label-based) or `iat` (position-based):

```python
# Single cell access — much faster than loc/iloc for individual values
print(df.at["bob", "score"])    # 92 — by label
print(df.iat[1, 0])             # 92 — by position

# Useful when iterating (though loops over DataFrames should be a last resort)
for student in ["alice", "carol"]:
    current_score = df.at[student, "score"]
    df.at[student, "score"] = current_score + 5  # Give a 5-point bonus
```

### The SettingWithCopyWarning — The Most Misunderstood Error in Pandas

> **Watch Out: The SettingWithCopyWarning**
> 
> This warning fires when you try to modify data in what Pandas suspects might be a *copy* of your DataFrame rather than the original. When you filter a DataFrame, Pandas sometimes returns a view (a window into the same memory) and sometimes returns a completely new copy — and it doesn’t always tell you which one you have. If you set a value on a view, it modifies the original. If you set a value on a copy, your change is silently discarded. This is one of the most dangerous silent bugs in Pandas.
> 
> The fix is simple and should become a habit: **always call `.copy()` when you intend to create a new, independent DataFrame from a slice.** And when you want to modify the original, **always use `.loc` directly on the original DataFrame.**

```python
# ❌ DANGEROUS — Pandas isn't sure if 'high_scorers' is a view or a copy
high_scorers = df[df["score"] > 80]
high_scorers["grade"] = "Pass"  # SettingWithCopyWarning — may not do what you think

# ✅ SAFE Option 1 — Call .copy() to declare "I want a new, independent DataFrame"
high_scorers = df[df["score"] > 80].copy()
high_scorers["grade"] = "Pass"  # Safe — we own this DataFrame

# ✅ SAFE Option 2 — Modify the original DataFrame in place using .loc
df.loc[df["score"] > 80, "grade"] = "Pass"  # Safe — direct modification
```

### Know the Limits

`loc` and `iloc` are not designed for high-frequency single-cell access inside large loops. Every call to `loc` inside a loop constructs a result object, which adds up quickly over thousands of iterations. If you absolutely must loop (and we’ll show you why you usually don’t need to), use `.at` or `.iat` for individual cells, or better yet, extract the underlying NumPy array with `.to_numpy()` and loop over that — it removes all the Pandas overhead.

-----

## Chapter 5: The Cleanup — Filtering, Missing Data, and Duplicates

### The Analogy: The Pasta Colander

When you drain pasta, you hold a colander over the sink and pour everything through it. The water (what you don’t want) falls through the holes; the pasta (what you want) stays behind. Pandas filtering works exactly the same way. You create a **boolean mask** — an array of `True` and `False` values, one per row — and the rows marked `True` are the pasta that stays, while the `False` rows drain away.

### Boolean Masking — The Core of Filtering

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "age":    [25, 34, 19, 45, 30, 22, 51],
    "income": [40000, 85000, 22000, 120000, 67000, 31000, 95000],
    "region": ["North", "South", "North", "East", "South", "East", "North"],
    "active": [True, True, False, True, True, False, True],
})

# Step 1: Create the mask — this is just a Series of True/False values
age_mask = df["age"] > 30
print(age_mask)
# 0    False
# 1     True
# 2    False
# 3     True
# 4    False
# 5    False
# 6     True
# dtype: bool

# Step 2: Apply the mask to filter the DataFrame
older_customers = df[age_mask]

# You don't need to store the mask separately — you can do it in one step
older_customers = df[df["age"] > 30]

# --- Combining conditions ---
# CRITICAL: use & (AND), | (OR), ~ (NOT) — NEVER use Python's 'and', 'or', 'not'
# ALWAYS wrap each condition in its own parentheses

# People over 30 AND earning over 60,000
high_value = df[(df["age"] > 30) & (df["income"] > 60000)]

# People from North OR South
regional = df[(df["region"] == "North") | (df["region"] == "South")]

# Active customers only (inverting the boolean column)
inactive = df[~df["active"]]

# isin() — cleaner than chaining OR conditions for membership tests
target_regions = ["North", "South"]
regional_clean = df[df["region"].isin(target_regions)]
```

> **Watch Out: Never Use `and`, `or`, `not` for Pandas Filtering**
> 
> If you write `df[df["age"] > 30 and df["income"] > 60000]`, Python will try to evaluate `df["age"] > 30` as a single True/False value for the `and` operator, which doesn’t make sense for a Series of values. It raises a `ValueError`. The correct operators are `&`, `|`, and `~`, and every individual condition must be wrapped in parentheses because of Python’s operator precedence rules.

### `df.query()` — A More Readable Alternative for Complex Filters

For complex conditions, `query()` lets you write filter expressions as readable strings, which can also be faster on large DataFrames when the `numexpr` library is installed.

```python
# The query() version is often easier to read for complex conditions
result = df.query("age > 30 and income > 60000")

# You can reference external Python variables with the @ symbol
min_age = 30
income_floor = 60000
result = df.query("age > @min_age and income > @income_floor")

# isin() has a clean equivalent in query
result = df.query("region in ['North', 'South']")
```

### Handling Missing Data — The “Holes” in Your Dataset

Missing data in Pandas comes in two flavors that look the same but behave differently: `np.nan` (a real floating-point value defined by the IEEE 754 standard) and `None` (a Python object). The important practical difference is that `np.nan` keeps a column’s dtype as `float64`, while `None` in an integer column forces the whole column to `object` dtype — which kills performance.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "feature_a": [1.0, np.nan, 3.0, np.nan, 5.0],
    "feature_b": [10,  20,    np.nan, 40,   50],
    "category":  ["A", "B", None, "A", "C"],
})

# Always start by counting your missing values
print(df.isnull().sum())
# feature_a    2
# feature_b    1
# category     1

# What fraction of each column is missing?
print(df.isnull().mean().round(3))

# --- Strategy 1: dropna() — Remove rows with any missing value ---
# Simple, but dangerous for small datasets or non-random missingness
df_dropped = df.dropna()
# Only drop rows where a specific column is missing
df_dropped_specific = df.dropna(subset=["feature_a"])
# Drop columns that are more than 50% missing
df_dropped_cols = df.dropna(axis=1, thresh=int(len(df) * 0.5))

# --- Strategy 2: fillna() — Replace missing values ---

# Fill with a constant (safe for tree-based models; distorts linear models)
df_zero_filled = df.fillna(0)

# Fill with the column median (robust to outliers — recommended for ML)
for col in ["feature_a", "feature_b"]:
    median_val = df[col].median()  # Compute this on TRAINING data only
    df[col] = df[col].fillna(median_val)

# Forward fill — propagates the last known value forward
# Only valid for time-ordered data (sensor readings, stock prices, etc.)
df_ffilled = df.fillna(method="ffill")

# Fill categorical missingness with the most frequent value
most_common = df["category"].mode()[0]
df["category"] = df["category"].fillna(most_common)
```

> **Watch Out: `dropna()` Can Introduce Bias Into Your ML Model**
> 
> If the missingness in your data is not random — for example, if lower-income customers are more likely to skip the “income” field — then dropping all rows with missing income values removes a specific demographic from your training data. Your model will then perform poorly on exactly those customers it never saw. Before dropping any rows, always ask yourself: “Is the fact that this value is missing itself informative?” If yes, consider creating a binary `is_missing` indicator column before imputing, so the model can learn from the pattern of missingness.

> **Watch Out: Fit Imputation on Training Data Only**
> 
> When you fill missing values with the column median, that median must be computed from the training set only — not from the full dataset. If you compute the median on all your data (including the test set) and use that to fill missing values in the training set, you’re leaking information about the test set into your training process. This is called **data leakage**, and it leads to models that look accurate on your evaluation but fail in production.

### Removing Duplicates

```python
# Check for fully duplicate rows
print(df.duplicated().sum())

# Remove them — keep the first occurrence by default
df_clean = df.drop_duplicates()

# Remove duplicates based on a meaningful natural key
# (e.g., a customer can only appear once per date)
df_deduped = df.drop_duplicates(subset=["customer_id", "event_date"], keep="last")
```

### Know the Limits

Pandas’ missing value handling does not know anything about your model or train/test split. It is just a data transformation tool. The responsibility for preventing data leakage falls entirely on you. Using `sklearn.pipeline.Pipeline` with `sklearn.impute.SimpleImputer` is the correct way to handle imputation in a leak-safe way, because the pipeline fits the imputer on training data and applies the fitted parameters to test data automatically.

-----

## Chapter 6: The Transformation — Strings, Dates, and New Features

### The Analogy: The Assembly Line

Think of feature engineering as a factory assembly line. Raw ingredients (your original columns) come in at one end, and at each station, a worker applies a specific transformation — cleaning a string, extracting a date component, computing a ratio — until a finished product (ML-ready features) comes out the other end. Pandas’ `.str` accessor and `.dt` accessor are those specialized workstations.

### Type Casting with `astype()`

Before any transformation, you often need to tell Pandas what type a column should be. Loading data from a CSV frequently produces columns with the wrong type — a numeric column stored as a string, or an integer column that became float because of a few missing values.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "age_text":   ["25", "32", "41", "28"],
    "price":      [19.99, 49.99, 9.99, 99.99],
    "category":   ["cat", "dog", "cat", "bird"],
    "event_date": ["2024-01-15", "2024-02-20", "2024-03-05", "2024-04-10"],
})

# Cast a string column to integer
df["age"] = df["age_text"].astype("int8")   # int8 saves memory (range: -128 to 127)

# Downcast a float64 to float32 — halves memory, sufficient precision for ML
df["price_f32"] = df["price"].astype("float32")

# Convert low-cardinality strings to category — covered in depth in Chapter 8
df["category"] = df["category"].astype("category")

# Safe numeric conversion — won't crash on bad values, returns NaN instead
df["safe_age"] = pd.to_numeric(df["age_text"], errors="coerce")

# Parse dates from strings
df["event_date"] = pd.to_datetime(df["event_date"])
```

### Renaming Columns — Keeping Your Data Clean

```python
# Rename specific columns with a dictionary
df = df.rename(columns={"age_text": "age_raw", "price": "price_usd"})

# Bulk-normalize all column names: lowercase, spaces replaced with underscores
# This should be one of the first things you do on any new dataset
df.columns = df.columns.str.lower().str.replace(" ", "_", regex=False)
```

### Vectorized String Operations — The `.str` Accessor

The `.str` accessor is a powerful tool for cleaning messy text data. It applies string methods across an entire column, handling `NaN` gracefully (returning `NaN` for missing values instead of crashing). However, it is important to understand what’s happening under the hood: `.str` operations are **not** truly NumPy-vectorized. They iterate over Python string objects in a loop. This makes them far more convenient than writing loops yourself, but also roughly 10 to 100 times slower than equivalent numeric operations.

```python
df = pd.DataFrame({
    "full_name": ["  Alice Smith ", "BOB JONES", "carol brown  ", None],
    "email":     ["alice@company.com", "bob@email.org", "carol@work.net", None],
    "city_code": ["NYC-001", "LAX-042", "CHI-007", "SFO-099"],
})

# Strip whitespace, convert to lowercase, capitalize properly
df["name_clean"] = df["full_name"].str.strip().str.lower().str.title()

# Check for pattern containment (na=False prevents NaN from propagating as NaN)
df["is_company_email"] = df["email"].str.contains("@company.com", na=False)

# Split on a delimiter and extract a specific piece
df["city"] = df["city_code"].str.split("-").str[0]   # "NYC", "LAX", etc.
df["code"] = df["city_code"].str.split("-").str[1]   # "001", "042", etc.

# Extract with a regular expression (named groups become columns)
df["code_number"] = df["city_code"].str.extract(r"-(\d+)$")

# Replace patterns
df["city_code_clean"] = df["city_code"].str.replace(r"-\d+", "", regex=True)
```

### Using `.map()` for Dictionary-Based Translation

For categorical encoding and label mapping, `.map()` with a dictionary is the most efficient approach. It performs a hash-table lookup rather than a Python function call per element, making it faster than `.apply()` for this use case.

```python
# Encode categorical labels for ML models
animal_to_code = {"cat": 0, "dog": 1, "bird": 2}
df["animal_code"] = df["category"].map(animal_to_code)

# .map() with a function is like .apply() on a Series — acceptable for simple ops
df["price_log"] = df["price_usd"].map(lambda x: np.log1p(x))

# But for numeric transformations, NumPy ufuncs are much faster:
df["price_log_fast"] = np.log1p(df["price_usd"])  # Goes straight to C code
```

### Datetime Features — The `.dt` Accessor

Time-related features (hour of day, day of week, days since an event) are among the most valuable in ML. The `.dt` accessor gives you convenient access to all datetime components.

```python
df = pd.DataFrame({
    "purchase_time": pd.to_datetime([
        "2024-01-15 08:30:00",
        "2024-02-20 14:45:00",
        "2024-03-05 22:10:00",
        "2024-03-07 09:00:00",
    ]),
    "amount": [50.0, 120.0, 30.0, 200.0],
})

# Extract individual components as new ML features
df["year"]        = df["purchase_time"].dt.year
df["month"]       = df["purchase_time"].dt.month
df["day_of_week"] = df["purchase_time"].dt.dayofweek   # 0=Monday, 6=Sunday
df["hour"]        = df["purchase_time"].dt.hour
df["is_weekend"]  = df["purchase_time"].dt.dayofweek >= 5
df["is_evening"]  = df["purchase_time"].dt.hour >= 18

# Compute time since a reference point (as a numeric feature)
reference_date = pd.Timestamp("2024-01-01")
df["days_since_jan1"] = (df["purchase_time"] - reference_date).dt.days

# Lag and lead features for time-series forecasting
df = df.set_index("purchase_time").sort_index()
df["amount_yesterday"] = df["amount"].shift(1)   # Previous period's value
df["amount_tomorrow"]  = df["amount"].shift(-1)  # Next period's value (use for targets only)
```

### Know the Limits

`.str` and `.dt` operations silently return `NaN` for any missing values in the source column. This is usually the right behavior, but it means that after a chain of string operations, you may have introduced new `NaN` values where the original strings were `None` or `NaN`. Always check `df.isnull().sum()` after a batch of string transformations. Additionally, `.str` operations on very large string columns (millions of rows) can be slow enough to be a bottleneck — for high-volume production pipelines, consider whether a more efficient format or tool (like Polars or DuckDB) is appropriate.

-----

## Chapter 7: The Combination — Merging and Stacking Datasets

### The Analogy: Combining Stacks of Cards

Imagine you have two stacks of index cards. The first stack has customer names and their IDs. The second stack has purchase records, each one also labeled with a customer ID. You want to combine them to see each purchase alongside the customer’s name.

You could do it in two ways. The first way is to find matching IDs across the two stacks and staple those cards together, side by side — this is a **join** (what `pd.merge()` does). The second way is to simply stack one pile on top of the other, growing the stack taller — this is a **concatenation** (what `pd.concat()` does).

### The Three Cousins: `merge`, `concat`, and `join`

Understanding when to use each is critical because choosing the wrong one can produce silently wrong data — extra rows, missing rows, or a combinatorial explosion of duplicates.

|Operation    |Think of it as…                                       |When to use it                                            |
|-------------|------------------------------------------------------|----------------------------------------------------------|
|`pd.merge()` |SQL JOIN — combine side-by-side on matching keys      |Combining two tables that share a key column              |
|`pd.concat()`|Stacking — combine rows or columns                    |Adding more rows from a similar dataset, or adding columns|
|`df.join()`  |Index-based merge — shorthand for merging on the index|When your join key is already the DataFrame index         |

### `pd.merge()` — SQL-Style Joining

```python
import pandas as pd

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "name":        ["Alice", "Bob", "Carol", "Dave"],
    "region":      ["North", "South", "North", "East"],
})

orders = pd.DataFrame({
    "customer_id": [1, 2, 2, 5],   # Note: 5 doesn't exist in customers; 3 and 4 have no orders
    "product":     ["Laptop", "Phone", "Tablet", "Monitor"],
    "amount":      [1200, 800, 600, 400],
})

# INNER join — only rows where customer_id exists in BOTH tables
# Result has 3 rows: customers 1, 2, and 2 (Bob has two orders)
inner = pd.merge(customers, orders, on="customer_id", how="inner")

# LEFT join — all customers, NaN for any who have no orders
# Result has 4 rows — Dave gets NaN for product and amount
left = pd.merge(customers, orders, on="customer_id", how="left")

# OUTER join — every row from both tables, NaN where there's no match
# Result has 5 rows — Dave from customers, and the mystery customer 5 from orders
outer = pd.merge(customers, orders, on="customer_id", how="outer")

# When the join key has different names in each table
# use left_on and right_on
orders_renamed = orders.rename(columns={"customer_id": "cust_id"})
merged = pd.merge(customers, orders_renamed,
                  left_on="customer_id", right_on="cust_id", how="left")
```

> **Watch Out: The Silent Row-Count Explosion in Many-to-Many Joins**
> 
> If customer 2 (Bob) appears twice in the `customers` table and also has three orders, a merge will produce 2 × 3 = 6 rows for Bob — silently. Pandas does not warn you about many-to-many relationships. Before every merge, check your join key for duplicates: `df["customer_id"].duplicated().any()`. If that returns True and you weren’t expecting it, investigate before proceeding. A good habit is to always check `len(merged)` after a merge and ask yourself if that number makes sense.

### `pd.concat()` — Stacking DataFrames

```python
# Vertical stacking (axis=0) — adding more rows of the same kind of data
df_q1 = pd.DataFrame({"sales": [100, 200], "region": ["North", "South"]})
df_q2 = pd.DataFrame({"sales": [150, 250], "region": ["North", "South"]})

# ignore_index=True resets the index to a clean 0, 1, 2, 3...
# Without it, you'd have index [0, 1, 0, 1] — two rows labeled 0, two labeled 1
combined = pd.concat([df_q1, df_q2], axis=0, ignore_index=True)

# Horizontal stacking (axis=1) — adding new columns
# Pandas aligns on the index — make sure they match!
df_features = pd.DataFrame({"age": [25, 32, 41]})
df_labels   = pd.DataFrame({"outcome": [0, 1, 0]})
full_dataset = pd.concat([df_features, df_labels], axis=1)
```

> **Watch Out: The Hidden Index Alignment Trap in `concat()`**
> 
> `pd.concat()` always aligns on the index before combining. If you concat two DataFrames that have been filtered separately, their integer indexes might not match — for example, one might have index `[0, 2, 4]` and the other `[0, 1, 2]`. Pandas will dutifully align on these indexes, introducing `NaN` values where the labels don’t match. The fix is almost always to call `reset_index(drop=True)` on both DataFrames before concatenating, or to use `ignore_index=True` in the `concat()` call itself.

### Know the Limits

Repeated concatenation inside a loop is one of the most common performance mistakes in Pandas:

```python
# ❌ VERY SLOW — creates a new DataFrame object in every iteration
results = pd.DataFrame()
for chunk in data_chunks:
    results = pd.concat([results, process(chunk)])  # Gets slower every loop

# ✅ FAST — collect first, concat once at the end
results_list = []
for chunk in data_chunks:
    results_list.append(process(chunk))
results = pd.concat(results_list, ignore_index=True)
```

The reason the first approach is so slow is that every call to `pd.concat()` allocates a brand-new block of memory for the result, copies all the existing data into it, then copies the new chunk in. As the accumulator grows, those copies get progressively more expensive. By collecting everything in a plain Python list and concatenating once at the end, you only pay the copy cost one time.

-----

## Chapter 8: The Performance — Optimization for ML Pipelines

### The Analogy: The Difference Between a Rucksack and a Suitcase

Imagine you’re packing for a trip. A disorganized rucksack where everything is stuffed in randomly is like an unoptimized DataFrame — it takes up more space than it needs to, and finding anything takes effort. A well-packed suitcase with clothes sorted by type, rolled tightly, and placed in the right compartments is like an optimized DataFrame — smaller, faster to access, and ready for the journey ahead. This chapter is about packing your data suitcase correctly before handing it to a model.

### Why `.apply()` Is Slow — And What to Do Instead

`apply()` is Pandas’ most dangerous convenience feature. It looks like it should be fast because it feels like a vectorized operation, but what it’s actually doing is calling a Python function once per row (or per element), constructing a full `pd.Series` for each call, and then reassembling the results. For a million-row DataFrame, that’s a million Python function calls — each carrying overhead.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "a": np.random.rand(500_000),
    "b": np.random.rand(500_000),
})

# ❌ SLOW — Python called 500,000 times, once per row
df["result_slow"] = df.apply(
    lambda row: np.sqrt(row["a"] ** 2 + row["b"] ** 2),
    axis=1
)

# ✅ FAST — NumPy operates on entire arrays in a single C call
df["result_fast"] = np.sqrt(df["a"] ** 2 + df["b"] ** 2)

# ✅ ALSO FAST — np.hypot is the purpose-built function for this calculation
df["result_best"] = np.hypot(df["a"].to_numpy(), df["b"].to_numpy())
```

The rule of thumb is: if your logic can be expressed as a combination of standard arithmetic, comparison, and NumPy functions operating on whole columns, it should be. Reserve `.apply()` for logic that genuinely cannot be vectorized — like calling an external API, running a custom parser on malformed text, or applying logic that depends on conditional branching across multiple columns in a way that resists vectorization.

### `np.where()` and `np.select()` — Vectorized Conditionals

```python
# np.where is the vectorized equivalent of a conditional assignment
# It says: "where this condition is True, use value_a; otherwise use value_b"
df["category"] = np.where(df["a"] > 0.5, "high", "low")

# np.select handles multiple conditions cleanly — like a vectorized if/elif/else
conditions = [
    (df["a"] > 0.7) & (df["b"] > 0.7),
    (df["a"] > 0.5),
]
choices = ["both_high", "a_high"]
df["label"] = np.select(conditions, choices, default="neither")
```

### The `category` Dtype — Memory Savings and Faster GroupBy

The `category` dtype is the single most impactful optimization for datasets with low-cardinality string columns (columns where the same few values repeat many times). Instead of storing the string “Engineering” in every single row of a million-row DataFrame, Pandas stores a lookup table with the unique values and an integer array of codes. A million-row column of short strings might take 60 MB as `object` dtype; as `category`, it can take less than 2 MB.

```python
import pandas as pd
import numpy as np

n = 1_000_000
df = pd.DataFrame({
    "department": np.random.choice(
        ["Engineering", "Marketing", "HR", "Finance", "Sales"],
        size=n
    ),
    "status": np.random.choice(["active", "inactive", "pending"], size=n),
    "salary": np.random.uniform(40000, 150000, size=n).astype(np.float32),
})

print("Before optimization:")
print(df.memory_usage(deep=True).sum() / 1e6, "MB")

# Convert low-cardinality string columns to category
df["department"] = df["department"].astype("category")
df["status"]     = df["status"].astype("category")

print("\nAfter optimization:")
print(df.memory_usage(deep=True).sum() / 1e6, "MB")

# category dtype also makes groupby operations 2-5x faster
# because it groups on integer codes rather than comparing strings
result = df.groupby("department")["salary"].mean()
```

### Downcasting Numeric Types

By default, Pandas uses `int64` for integers and `float64` for floats. For most ML applications, `int32` or even `int16` for bounded integers, and `float32` for floats, is entirely sufficient. Halving the precision halves the memory and can meaningfully speed up NumPy operations due to better CPU cache utilization.

```python
def downcast_dataframe(df: pd.DataFrame) -> pd.DataFrame:
    """
    Automatically downcast all numeric columns to their smallest
    safe representation without losing data. Always apply this after
    loading and before saving intermediate pipeline files.
    """
    df = df.copy()  # Don't modify the original

    for col in df.select_dtypes(include="integer").columns:
        df[col] = pd.to_numeric(df[col], downcast="integer")

    for col in df.select_dtypes(include="float").columns:
        df[col] = pd.to_numeric(df[col], downcast="float")

    return df

df_optimized = downcast_dataframe(df)
print(df_optimized.dtypes)
```

### Saving Files: Why Parquet Beats CSV

A CSV file is plain text. Every number, every date, every category is converted to characters and back every time you read or write. There is no type information, no compression, and no structure — just characters. For ML pipelines where you read the same preprocessed data many times, this is wasteful.

```python
# --- Saving ---
# CSV: human-readable but slow, large, no dtype info
df.to_csv("data.csv", index=False)

# Parquet: compressed columns, preserves dtypes, 10-50x faster to read
# Best for long-term storage and sharing between systems
df.to_parquet("data.parquet", engine="pyarrow", compression="snappy", index=False)

# Feather: extremely fast read/write, no compression overhead
# Best for intermediate pipeline files you read repeatedly in one session
df.to_feather("data.feather")

# --- Loading ---
df_parquet = pd.read_parquet("data.parquet")  # Dtypes are perfectly preserved
df_feather  = pd.read_feather("data.feather")
```

### A Complete Optimization Pipeline

```python
def prepare_for_ml(df: pd.DataFrame,
                   category_threshold: int = 50) -> pd.DataFrame:
    """
    Apply all memory and performance optimizations to a DataFrame
    before it enters an ML training pipeline.

    category_threshold: columns with fewer unique values than this
    will be converted to 'category' dtype.
    """
    df = df.copy()

    for col in df.columns:
        dtype = df[col].dtype

        if dtype == "object":
            n_unique = df[col].nunique()
            if n_unique <= category_threshold:
                # Low-cardinality strings become memory-efficient categories
                df[col] = df[col].astype("category")
            # High-cardinality strings are left as-is for manual handling

        elif dtype in ["int64", "int32", "int16"]:
            df[col] = pd.to_numeric(df[col], downcast="integer")

        elif dtype in ["float64", "float32"]:
            df[col] = pd.to_numeric(df[col], downcast="float")

    return df

# Apply the optimization
df_ready = prepare_for_ml(df)

# Extract as a C-contiguous NumPy array for scikit-learn / PyTorch
feature_cols = ["salary", "age"]
X = np.ascontiguousarray(df_ready[feature_cols].to_numpy(dtype=np.float32))
y = df_ready["outcome"].to_numpy()
```

> **Watch Out: `float16` Is Not Always Safe**
> 
> Downcasting all floats to `float16` to save the maximum memory can backfire badly. `float16` has a maximum value of only ~65,504 and loses precision rapidly for values with several significant digits. For normalized features (values between -1 and 1), it is fine. For raw monetary values, ages, distances, or counts, it can introduce rounding errors that corrupt your model’s learning. Prefer `float32` as your default downcast target — it offers 8-decimal-place precision, is supported natively by all major ML frameworks, and uses half the memory of `float64`.

### Know the Limits

Even with all optimizations applied, Pandas is an in-memory, single-machine tool. If your fully optimized DataFrame still doesn’t fit in RAM, you need a different tool. For moderately large data (up to a few hundred GB), **Dask** provides a parallel, distributed Pandas-compatible API. For SQL-like operations on very large flat files without loading them into memory, **DuckDB** is remarkably fast and integrates directly with Pandas. For truly large-scale distributed processing, **Apache Spark** (with the PySpark API) is the standard choice.

-----

## Summary Reference Table

### Quick Command Reference

|Task                   |Command                                           |Key Note                                               |
|-----------------------|--------------------------------------------------|-------------------------------------------------------|
|**Load CSV**           |`pd.read_csv("f.csv", dtype={...})`               |Always provide `dtype` for large files                 |
|**Load Parquet**       |`pd.read_parquet("f.parquet")`                    |Preserves dtypes; preferred over CSV                   |
|**Save CSV**           |`df.to_csv("f.csv", index=False)`                 |Set `index=False` to avoid saving the index as a column|
|**Save Parquet**       |`df.to_parquet("f.parquet", compression="snappy")`|Best for persistent storage                            |
|**Save Feather**       |`df.to_feather("f.feather")`                      |Best for fast intermediate pipeline files              |
|**First look**         |`df.head()`                                       |Shows first 5 rows                                     |
|**Structure audit**    |`df.info()`                                       |Column names, dtypes, non-null counts                  |
|**True memory**        |`df.memory_usage(deep=True).sum()`                |Always use `deep=True` for honest numbers              |
|**Statistics**         |`df.describe()`                                   |Count, mean, std, min, quartiles, max                  |
|**Missing values**     |`df.isnull().sum()`                               |Per-column missing value count                         |
|**Unique counts**      |`df.nunique()`                                    |Cardinality per column                                 |
|**Select by label**    |`df.loc[rows, cols]`                              |End of slice is INCLUSIVE                              |
|**Select by position** |`df.iloc[rows, cols]`                             |End of slice is EXCLUSIVE (like Python)                |
|**Single cell (fast)** |`df.at[label, col]` / `df.iat[i, j]`              |Use in loops; much faster than loc/iloc                |
|**Boolean filter**     |`df[(df["a"] > 1) & (df["b"] < 5)]`               |Parentheses required; use `&`, `                       |
|**Query filter**       |`df.query("a > 1 and b < 5")`                     |More readable; faster with numexpr                     |
|**Membership filter**  |`df[df["col"].isin([1, 2, 3])]`                   |Cleaner than chained OR conditions                     |
|**Drop missing rows**  |`df.dropna(subset=["col"])`                       |Use `subset` to avoid dropping everything              |
|**Impute with median** |`df["col"].fillna(df["col"].median())`            |Fit on training data only                              |
|**Forward fill**       |`df.fillna(method="ffill")`                       |Time-series and sensor data only                       |
|**Drop duplicates**    |`df.drop_duplicates(subset=["key"])`              |Specify `subset` for natural key dedup                 |
|**Cast type**          |`df["col"].astype("float32")`                     |Downcast to save memory                                |
|**Safe numeric cast**  |`pd.to_numeric(df["col"], errors="coerce")`       |Returns NaN on parse failure                           |
|**Parse dates**        |`pd.to_datetime(df["col"])`                       |Do at ingestion time                                   |
|**Rename columns**     |`df.rename(columns={"old": "new"})`               |Dict-based, explicit renaming                          |
|**Normalize names**    |`df.columns.str.lower().str.replace(" ", "_")`    |Clean column names at the start                        |
|**String contains**    |`df["col"].str.contains("pat", na=False)`         |`na=False` prevents NaN propagation                    |
|**String split**       |`df["col"].str.split("-").str[0]`                 |Extract a piece after splitting                        |
|**Regex extract**      |`df["col"].str.extract(r"(\d+)")`                 |Captured group becomes a column                        |
|**Map values**         |`df["col"].map({"a": 1, "b": 2})`                 |Hash-lookup; fastest for encoding                      |
|**NumPy ufunc**        |`np.log1p(df["col"])`                             |Always faster than `.apply()`                          |
|**Conditional assign** |`np.where(cond, val_if_true, val_if_false)`       |Vectorized if/else                                     |
|**Multi-condition**    |`np.select([c1, c2], [v1, v2], default=v0)`       |Vectorized if/elif/else                                |
|**Date component**     |`df["col"].dt.month`, `.dt.dayofweek`             |Requires datetime dtype                                |
|**Lag feature**        |`df["col"].shift(1)`                              |Requires DatetimeIndex and sorted order                |
|**Rolling mean**       |`df["col"].rolling(7).mean()`                     |NaN introduced at start of window                      |
|**GroupBy aggregate**  |`df.groupby("key").agg({"col": "sum"})`           |Use string agg names for best speed                    |
|**Named aggregation**  |`df.groupby("k").agg(result=("col", "sum"))`      |Pandas 0.25+ clean syntax                              |
|**Group transform**    |`df.groupby("k")["col"].transform("mean")`        |Returns same-length Series for new feature             |
|**Merge (SQL join)**   |`pd.merge(left, right, on="key", how="left")`     |Check row count after; watch for duplicates            |
|**Concat (stack rows)**|`pd.concat([df1, df2], ignore_index=True)`        |`ignore_index=True` resets to clean index              |
|**Category dtype**     |`df["col"].astype("category")`                    |Huge memory saving for low-cardinality strings         |
|**Downcast numeric**   |`pd.to_numeric(df["col"], downcast="float")`      |float64→float32; int64→int8/16/32                      |
|**To NumPy (safe)**    |`np.ascontiguousarray(df.to_numpy())`             |Guarantees C-contiguous layout for ML                  |
|**Avoid copy warning** |`df_sub = df[mask].copy()`                        |Always `.copy()` when creating a subset                |
|**Large file reading** |`pd.read_csv("f.csv", chunksize=100_000)`         |Process in chunks to avoid RAM overflow                |

-----

*Guide version 1.0 — Written for data scientists using Python 3.9+, Pandas 1.5+, and NumPy 1.23+. The best way to master these tools is not to memorize them but to use them on real data, make mistakes, and understand why those mistakes happened. Every warning in this guide exists because someone, somewhere, lost hours to that bug.*

-----
