# 🧬 The Data Scientist’s Blueprint: Mastering Pandas & NumPy for AI/ML

> *A definitive, practical guide structured as a Data Journey — from raw memory to production-ready ML pipelines.*

-----

## Table of Contents

1. [The Foundation — NumPy & Core Structures](#1-the-foundation--numpy--core-structures)
1. [Data Ingestion & Inspection](#2-data-ingestion--inspection)
1. [Precision Selection — loc, iloc, at, iat](#3-precision-selection--loc-iloc-at-iat)
1. [The Logic Layer — Filtering & Boolean Masking](#4-the-logic-layer--filtering--boolean-masking)
1. [Data Hygiene — Missing Values & Duplicates](#5-data-hygiene--missing-values--duplicates)
1. [Transformation & String Operations](#6-transformation--string-operations)
1. [Advanced Analytics — Grouping & Aggregation](#7-advanced-analytics--grouping--aggregation)
1. [The Apply Dilemma — Performance-Aware Row Operations](#8-the-apply-dilemma--performance-aware-row-operations)
1. [Merging & Reshaping](#9-merging--reshaping)
1. [Time-Series for ML](#10-time-series-for-ml)
1. [Optimization for Scale](#11-optimization-for-scale)
1. [Cheat Sheet](#12-cheat-sheet)

-----

## 1. The Foundation — NumPy & Core Structures

### What Is NumPy, and Why Does It Matter for ML?

Before you write a single line of Pandas, you need to understand what is actually happening in memory. NumPy’s core object is the **`ndarray`** — an N-dimensional array stored as a single, contiguous block of typed memory. Every value in an `ndarray` is the same dtype, which means the CPU can apply the same operation to every element without branching. This is **vectorization**, and it is the reason array math is orders of magnitude faster than Python loops.

```python
import numpy as np

# A 1D array — all float64, contiguous in memory
arr = np.array([1.0, 2.0, 3.0, 4.0])

# Shape tells you the dimensions; strides tell you how many bytes to step
# to reach the next element along each axis
print(arr.shape)   # (4,)
print(arr.strides) # (8,) — each float64 is 8 bytes

# 2D array — shape and strides reveal memory layout (row-major / C-order by default)
matrix = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.float32)
print(matrix.shape)    # (2, 3)
print(matrix.strides)  # (12, 4) — 3 floats per row × 4 bytes = 12 bytes to next row
```

**Why strides matter for ML:** Most scikit-learn estimators and deep learning frameworks (PyTorch, TensorFlow) expect arrays in C-contiguous order. If your data is Fortran-order or non-contiguous (a common result of slicing), operations can silently slow down. Use `np.ascontiguousarray()` before feeding data to a model.

### Series and DataFrames as Labeled NumPy Wrappers

A Pandas `Series` is essentially a NumPy `ndarray` with an **index** — a labeled axis that allows lookup by name rather than position. A `DataFrame` is a collection of `Series` objects that share the same index, each column backed by its own block of memory.

```python
import pandas as pd
import numpy as np

# Under the hood: a Series wraps a NumPy array
s = pd.Series([10, 20, 30], index=["a", "b", "c"])
print(s.to_numpy())  # array([10, 20, 30]) — the raw NumPy array
print(s.dtype)       # int64

# A DataFrame is a dict of aligned Series
df = pd.DataFrame({
    "age":    np.array([25, 32, 41], dtype=np.int32),
    "salary": np.array([50000.0, 72000.0, 95000.0], dtype=np.float64),
    "name":   ["Alice", "Bob", "Carol"],
})

# Access the underlying NumPy values of the entire DataFrame
# (only works cleanly if all columns share a dtype)
print(df[["age", "salary"]].to_numpy())
```

### Understanding dtypes — And the `object` Type Pitfall

The dtype of a column determines its memory footprint, how fast operations run, and whether NumPy vectorization applies at all.

|dtype                               |Description                         |Bytes per value    |
|------------------------------------|------------------------------------|-------------------|
|`int8` / `int16` / `int32` / `int64`|Signed integers                     |1 / 2 / 4 / 8      |
|`float16` / `float32` / `float64`   |Floating point                      |2 / 4 / 8          |
|`bool`                              |True/False                          |1                  |
|`datetime64[ns]`                    |Nanosecond timestamps               |8                  |
|`category`                          |Encoded categoricals                |varies (small)     |
|**`object`**                        |**Python objects (usually strings)**|**~56+ bytes each**|


> [!WARNING]
> **The `object` dtype is a performance trap.** When Pandas cannot infer a homogeneous numeric type, it falls back to `object`, storing Python object *pointers* rather than raw values. This defeats NumPy vectorization entirely, bloats memory by 5–10×, and makes every operation slow. Always inspect your dtypes after loading data and explicitly cast where possible.

```python
# Detecting and fixing the object type pitfall
df = pd.read_csv("data.csv")
print(df.dtypes)

# A column that looks numeric but loaded as object — common with mixed data
df["price"] = pd.to_numeric(df["price"], errors="coerce")  # NaN on parse failure

# Confirm the fix
print(df["price"].dtype)  # float64
```

### Limits & Constraints

NumPy arrays and Pandas structures are designed for **in-memory, batch processing**. The entire dataset must fit in RAM. For a 64-bit float array with N elements, memory usage is approximately `N × 8` bytes — a 10M-row, 50-column float64 DataFrame consumes ~4 GB. Pandas is **not suited for streaming data** or datasets larger than available RAM unless chunked explicitly (see Section 2).

-----

## 2. Data Ingestion & Inspection

### Reading CSV and Excel Files

`pd.read_csv()` is the workhorse of data ingestion. Its most important parameters go far beyond just specifying a filename.

```python
import pandas as pd

df = pd.read_csv(
    "transactions.csv",
    dtype={
        "user_id": "int32",        # Prevent upcasting to int64
        "amount":  "float32",      # Save memory on wide datasets
        "country": "category",     # Efficient for low-cardinality strings
    },
    parse_dates=["created_at"],    # Auto-parse to datetime64
    na_values=["N/A", "unknown"],  # Treat custom strings as NaN
    usecols=["user_id", "amount", "country", "created_at"],  # Only load needed columns
)

# Excel: similar API, but significantly slower due to XML parsing
df_xl = pd.read_excel("report.xlsx", sheet_name="Q1", engine="openpyxl")
```

### Chunked Reading — Handling Datasets Larger Than RAM

When a file is too large to load at once, `chunksize` turns `read_csv()` into an iterator that yields one `DataFrame` per chunk. This is the primary Pandas strategy for out-of-core processing.

```python
# Pattern: accumulate aggregated results, not raw data
chunk_stats = []

for chunk in pd.read_csv("huge_log.csv", chunksize=100_000):
    # Process each chunk independently — never store the chunks themselves
    stats = chunk.groupby("event_type")["duration_ms"].mean()
    chunk_stats.append(stats)

# Combine the per-chunk summaries (much smaller than the raw data)
final_stats = pd.concat(chunk_stats).groupby(level=0).mean()
```

> [!IMPORTANT]
> **`chunksize` is not parallelism.** Chunks are processed sequentially. For true parallel ingestion, look at `Dask` or `Polars`. Also note that some operations (like sorting or global deduplication) cannot be done correctly on independent chunks without a second pass.

### Inspecting What You Actually Have

`df.info()` gives a quick summary, but it lies about memory usage for `object` columns because it estimates rather than measures pointer sizes.

```python
df = pd.read_csv("data.csv")

# Surface-level summary — fast but approximate for object columns
df.info()

# True memory usage — traverses Python objects to measure actual heap size
print(df.memory_usage(deep=True))          # Per-column, in bytes
print(df.memory_usage(deep=True).sum() / 1e6, "MB")  # Total

# Statistical snapshot
print(df.describe(include="all"))  # numeric + categorical
print(df.nunique())                # Cardinality — critical for deciding 'category' dtype
print(df.isnull().sum())           # Missing value map
```

> [!IMPORTANT]
> **Always use `memory_usage(deep=True)` when auditing large DataFrames.** `df.info()` reports the shallow size of `object` columns (just the pointer array), which can be 5–10× smaller than the real heap footprint. This matters enormously when you are deciding whether a dataset fits in RAM.

### Limits & Constraints

`read_excel` is significantly slower than `read_csv` because it parses XML under the hood. For large Excel files, export to CSV first. `read_csv` with default settings will upcast integer columns to `int64` and float columns to `float64`, doubling memory usage for data that only needs 32-bit precision. Always provide a `dtype` dictionary for wide datasets.

-----

## 3. Precision Selection — loc, iloc, at, iat

### The Four Indexers and When to Use Each

Pandas offers four indexers with subtly different behaviors. Choosing the wrong one is one of the most common sources of bugs in data pipelines.

|Indexer          |Axis Type         |Returns                    |Speed |Best For                          |
|-----------------|------------------|---------------------------|------|----------------------------------|
|`.loc[row, col]` |**Label-based**   |Scalar / Series / DataFrame|Medium|Named rows/columns, boolean arrays|
|`.iloc[row, col]`|**Position-based**|Scalar / Series / DataFrame|Medium|Integer position indexing         |
|`.at[row, col]`  |**Label-based**   |**Scalar only**            |Fast  |Single cell access by label       |
|`.iat[row, col]` |**Position-based**|**Scalar only**            |Fast  |Single cell access by position    |

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "score": [88, 92, 75, 61],
    "grade": ["B", "A", "C", "D"],
}, index=["alice", "bob", "carol", "dave"])

# --- loc: label-based, inclusive on both ends ---
print(df.loc["alice":"carol", "score"])   # Rows alice through carol (INCLUSIVE)
print(df.loc[df["score"] > 80, "grade"])  # Boolean mask with loc

# --- iloc: position-based, exclusive on end (Python slice rules) ---
print(df.iloc[0:3, 0])   # Rows 0, 1, 2 (NOT 3) — column 0

# --- at / iat: single scalar, fastest for loops ---
print(df.at["bob", "score"])   # 92
print(df.iat[1, 0])            # 92 — same cell, positional
```

### The SettingWithCopyWarning — Why It Happens and How to Fix It

This is arguably the most misunderstood warning in Pandas. It fires when you try to set values on what *might* be a copy of a DataFrame slice rather than the original.

```python
# ❌ BAD — triggers SettingWithCopyWarning and may silently fail
high_scorers = df[df["score"] > 80]
high_scorers["grade"] = "Pass"  # Are we modifying df or a copy? Pandas isn't sure.

# ✅ CORRECT — use .copy() to declare intent explicitly
high_scorers = df[df["score"] > 80].copy()
high_scorers["grade"] = "Pass"  # Now we own this DataFrame; no ambiguity

# ✅ ALSO CORRECT — use .loc on the original DataFrame
df.loc[df["score"] > 80, "grade"] = "Pass"
```

> [!WARNING]
> **Never ignore `SettingWithCopyWarning` in an ML pipeline.** The transformation you think you applied to your training data may have been silently dropped if it operated on a copy. This can cause your preprocessing steps to produce inconsistent results between training and inference. Always `.copy()` or use `.loc` directly.

### Limits & Constraints

`.loc` with a label slice is **inclusive on both ends**, which differs from standard Python slicing. This surprises almost everyone the first time. `.iloc` follows standard Python exclusive-end rules. Neither indexer is appropriate for high-frequency single-cell access inside a loop — prefer `.at` or `.iat` there, or better yet, drop to NumPy arrays entirely for loop-heavy logic.

-----

## 4. The Logic Layer — Filtering & Boolean Masking

### Boolean Masking — The NumPy Foundation

Filtering in Pandas is built on NumPy’s boolean array indexing. When you write a comparison like `df["age"] > 30`, Pandas calls the underlying NumPy vectorized comparison, producing a boolean `ndarray` of the same length. That array is then used to index the DataFrame.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "age":    [25, 34, 19, 45, 30],
    "income": [40000, 85000, 22000, 120000, 67000],
    "region": ["North", "South", "North", "East", "South"],
})

# Basic boolean mask — returns a boolean Series (backed by np.bool_ array)
mask = df["age"] > 30
print(mask.to_numpy())  # [False  True False  True False]

# Compound conditions — use & (AND), | (OR), ~ (NOT), never 'and'/'or'/'not'
filtered = df[(df["age"] > 25) & (df["income"] > 60000)]

# isin() for membership testing — cleaner than chained OR conditions
target_regions = ["North", "South"]
regional = df[df["region"].isin(target_regions)]

# Negation with isin
non_regional = df[~df["region"].isin(target_regions)]
```

> [!WARNING]
> **Always wrap compound boolean conditions in parentheses.** Python’s operator precedence means `df["age"] > 25 & df["income"] > 60000` is parsed as `df["age"] > (25 & df["income"]) > 60000`, which is wrong and will raise a confusing error. Each condition must be wrapped: `(df["age"] > 25) & (df["income"] > 60000)`.

### `df.query()` — Readable Syntax With Performance Benefits

`query()` accepts a string expression and evaluates it, optionally using the `numexpr` library for large DataFrames. It is more readable for complex conditions and can be meaningfully faster on wide DataFrames (>200k rows) because `numexpr` avoids creating intermediate boolean arrays.

```python
# Equivalent to the compound mask above, but more readable
result = df.query("age > 25 and income > 60000")

# Reference external Python variables with @
min_age = 25
threshold = 60000
result = df.query("age > @min_age and income > @threshold")

# isin equivalent in query
result = df.query("region in ['North', 'South']")
```

### When to Use What: Filtering

|Approach                    |Best When                                                  |Avoid When                                   |
|----------------------------|-----------------------------------------------------------|---------------------------------------------|
|Boolean mask `df[condition]`|Simple conditions, readable code                           |Very complex nested logic (gets unreadable)  |
|`df.query(expr)`            |Complex multi-condition filters, large DataFrames (numexpr)|Columns with spaces in names (need backticks)|
|`df.isin(values)`           |Membership testing against a list or set                   |Continuous ranges (use `between()` instead)  |
|`df.between(low, high)`     |Continuous range filters                                   |Membership/categorical logic                 |

### Limits & Constraints

Boolean masking always **creates a new DataFrame** (it does not filter in place). For very large DataFrames, each intermediate mask allocation consumes memory. Chain conditions into a single expression rather than filtering step by step. `query()` with `numexpr` requires the `numexpr` package installed separately; without it, `query()` offers no performance advantage over masks.

-----

## 5. Data Hygiene — Missing Values & Duplicates

### NaN vs. None — They Are Not the Same Thing

This distinction matters in ML preprocessing because the two types behave differently in aggregations, type inference, and model input validation.

`np.nan` is a **floating-point value** defined by the IEEE 754 standard. It is a real float64 value — it takes 8 bytes of memory and participates in numeric operations (but propagates: `np.nan + 1 = np.nan`). `None` is a **Python object** — it is a pointer to a singleton, takes ~16 bytes, and forces a column to `object` dtype.

```python
import numpy as np
import pandas as pd

# NaN is a float — numeric columns use it
s_float = pd.Series([1.0, np.nan, 3.0])
print(s_float.dtype)   # float64
print(s_float.isna())  # [False, True, False]

# None forces object dtype when mixed with non-floats
s_obj = pd.Series([1, None, 3])
print(s_obj.dtype)   # object — bad for ML!
print(s_obj.isna())  # [False, True, False] — isna() handles both

# Pandas 1.0+ nullable integer dtype avoids this
s_int = pd.Series([1, pd.NA, 3], dtype="Int32")  # Capital I
print(s_int.dtype)   # Int32 — nullable integer, no object coercion
```

### Strategies for Missing Values — And Their ML Implications

```python
df = pd.DataFrame({
    "feature_a": [1.0, np.nan, 3.0, np.nan, 5.0],
    "feature_b": [10, 20, np.nan, 40, 50],
    "label":     [0, 1, 1, 0, 1],
})

# 1. dropna — simple but dangerous for small datasets
df_dropped = df.dropna()  # Loses rows 1 and 3 entirely

# 2. fillna with a constant
df_zero = df.fillna(0)  # Safe for tree-based models; distorts linear models

# 3. Forward-fill (time-series, sensor data)
df_ffill = df.fillna(method="ffill")  # Propagates last known value

# 4. Median imputation — robust to outliers
for col in ["feature_a", "feature_b"]:
    median = df[col].median()
    df[col] = df[col].fillna(median)

# 5. Using NumPy for fast in-place imputation on the array level
arr = df[["feature_a", "feature_b"]].to_numpy()
col_medians = np.nanmedian(arr, axis=0)  # Ignores NaN in median calculation
# Replace NaN positions with column medians
nan_mask = np.isnan(arr)
arr[nan_mask] = np.take(col_medians, np.where(nan_mask)[1])
```

> [!WARNING]
> **`dropna()` can silently cripple small ML datasets.** If missingness is not random (i.e., certain demographics or sensor failure modes are more likely to produce NaN), dropping those rows introduces bias. Always analyze the missingness pattern before deciding: `df.isnull().mean()` shows the fraction missing per column, and `df[df["feature_a"].isnull()]` shows which rows are affected.

> [!IMPORTANT]
> **Forward-fill and median imputation both introduce model bias** in different ways. `ffill` assumes temporal continuity — valid for sensor readings, invalid for tabular survey data. Median imputation compresses the distribution around the center, which can understate variance for regularized models. Document every imputation choice as a pipeline hyperparameter.

### Handling Duplicates

```python
# Detect duplicates — True where a row is a duplicate of an earlier row
print(df.duplicated().sum())

# Keep the first occurrence, drop subsequent duplicates
df_clean = df.drop_duplicates()

# Deduplicate based on a subset of columns (e.g., a natural key)
df_dedup = df.drop_duplicates(subset=["user_id", "event_date"], keep="last")
```

### Limits & Constraints

Pandas does not have a built-in missing-value imputer that is aware of train/test splits. If you compute `median()` on the full dataset and use it to impute before splitting, you have **data leakage**. Always fit imputation parameters (mean, median, mode) on the **training set only**, then apply them to the test set. Use `sklearn.impute.SimpleImputer` inside a `Pipeline` for leak-safe imputation.

-----

## 6. Transformation & String Operations

### Type Casting with `astype()`

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "age_str":  ["25", "32", "41"],
    "score":    [88.7, 91.2, 74.5],
    "category": ["cat", "dog", "cat"],
})

# Downcast to save memory — critical before feeding to ML models
df["age"]   = df["age_str"].astype("int8")     # 25, 32, 41 fit in int8 (-128 to 127)
df["score"] = df["score"].astype("float32")    # float32 sufficient for most ML
df["category"] = df["category"].astype("category")  # Encodes to integer codes internally

# Check memory impact
print(df.memory_usage(deep=True))
```

### Renaming Columns

```python
# Rename specific columns with a dictionary
df = df.rename(columns={"age_str": "age_raw", "score": "test_score"})

# Bulk rename using a function (e.g., normalize all column names)
df.columns = df.columns.str.lower().str.replace(" ", "_", regex=False)
```

### Vectorized String Operations

Pandas’ `.str` accessor applies Python string methods element-wise using a loop internally — it is **not** NumPy-vectorized in the true sense. For numeric columns, Pandas delegates to NumPy’s C kernels. For string columns, it executes a Python loop over Python string objects. This means string operations are roughly **10–100× slower** than equivalent numeric operations.

```python
df = pd.DataFrame({"name": ["Alice Smith", " Bob Jones ", "Carol Brown"]})

# .str operations — readable but slow for very large string columns
df["name_clean"] = df["name"].str.strip().str.lower()
df["first_name"] = df["name"].str.split(" ").str[0]
df["has_alice"]  = df["name"].str.contains("Alice", case=False, na=False)

# Extract with regex
df["last_name"] = df["name"].str.extract(r"(\w+)$")
```

### Using `.map()` for Dictionary-Based Translation

For categorical encoding and label mapping, `.map()` with a dictionary is both readable and efficient because it avoids Python loops on the Series level.

```python
# Dictionary-based mapping — ideal for encoding categorical labels
label_map = {"cat": 0, "dog": 1, "bird": 2}
df["animal_code"] = df["animal"].map(label_map)

# .map() with a function (similar to .apply() for Series, but faster for simple ops)
df["score_pct"] = df["score"].map(lambda x: round(x / 100, 4))

# For complex multi-column logic, .map() on Series is faster than .apply() on DataFrame
# because it avoids the overhead of constructing a Series per row
```

> [!IMPORTANT]
> **Prefer `.map()` over `.apply()` for single-column transformations.** For dictionary lookups specifically, `.map()` builds a hash-table lookup under the hood. For transformations that can be expressed as NumPy ufuncs (e.g., `np.log`, `np.sqrt`), apply the NumPy function directly to the column — it will use C-level vectorization and run orders of magnitude faster than either `.map()` or `.apply()`.

```python
# ✅ Fast — NumPy ufunc applied directly to the underlying array
df["log_income"] = np.log1p(df["income"])

# ❌ Slow — Python function called once per element
df["log_income"] = df["income"].apply(lambda x: np.log1p(x))
```

### Limits & Constraints

`.str` operations return `NaN` for missing values without raising errors — which is actually useful. However, operations like `.str.split().str[0]` silently return `NaN` for any row where the original value was `NaN`, which can introduce unexpected nulls downstream. Always call `.fillna("")` before string operations if you want consistent non-null output.

-----

## 7. Advanced Analytics — Grouping & Aggregation

### The Split-Apply-Combine Pattern

`groupby()` follows the **Split-Apply-Combine** paradigm, formalized by Hadley Wickham. The DataFrame is split into groups by one or more keys, a function is applied to each group independently, and the results are combined back into a single structure.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "region":   ["North", "South", "North", "East", "South", "East"],
    "product":  ["A", "B", "A", "C", "B", "A"],
    "sales":    [100, 200, 150, 80, 220, 130],
    "units":    [10, 20, 15, 8, 22, 13],
})

# Basic aggregation — returns a DataFrame with MultiIndex if multiple agg functions
summary = df.groupby("region")["sales"].agg(["mean", "sum", "count"])
print(summary)

# Multi-column groupby with named aggregations (Pandas 0.25+)
summary = df.groupby(["region", "product"]).agg(
    total_sales=("sales", "sum"),
    avg_units=("units", "mean"),
    num_transactions=("sales", "count"),
).reset_index()

# Transform — returns a Series with the same index as the original df
# Useful for creating group-normalized features
df["sales_vs_region_mean"] = df.groupby("region")["sales"].transform(
    lambda x: (x - x.mean()) / x.std()
)
```

### `pivot_table` vs `crosstab`

Both functions reshape data into a 2D summary table, but they serve different purposes.

|Feature             |`pivot_table`                                   |`crosstab`                                                 |
|--------------------|------------------------------------------------|-----------------------------------------------------------|
|Primary use         |Aggregate numeric values by two categorical keys|Frequency counts / contingency tables                      |
|Input               |A DataFrame                                     |Two array-like sequences (can be from different DataFrames)|
|Default aggregation |`mean`                                          |`count`                                                    |
|Margins (totals)    |`margins=True`                                  |`margins=True`                                             |
|Handles missing keys|Yes — fills with `fill_value`                   |Yes                                                        |

```python
# pivot_table — "what is the average sales per region × product?"
pivot = pd.pivot_table(
    df,
    values="sales",
    index="region",
    columns="product",
    aggfunc="sum",
    fill_value=0,
    margins=True,   # Adds "All" row and column
)

# crosstab — "how many observations are there for each region × product combination?"
ct = pd.crosstab(df["region"], df["product"], margins=True)
```

### Limits & Constraints

`groupby()` with a custom `lambda` or user-defined function inside `.agg()` or `.apply()` triggers slow Python execution — the group iteration is not vectorized. Always prefer string aggregation shortcuts (`"mean"`, `"sum"`, `"std"`) or NumPy functions (`np.mean`, `np.std`) over lambdas. GroupBy operations on `object`-dtype columns are especially slow; cast to `category` dtype before grouping to get a 2–5× speedup on large DataFrames.

-----

## 8. The Apply Dilemma — Performance-Aware Row Operations

### Why `.apply()` Is a Last Resort

`DataFrame.apply(func, axis=1)` iterates over rows as Series objects, calling `func` once per row as a Python function call. For a DataFrame with 1 million rows, that is 1 million Python function calls — each with the overhead of constructing a `pd.Series`, calling the function, and unpacking the result.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "a": np.random.rand(100_000),
    "b": np.random.rand(100_000),
})

# ❌ Extremely slow — Python loop hidden behind .apply()
df["result_slow"] = df.apply(lambda row: np.sqrt(row["a"] ** 2 + row["b"] ** 2), axis=1)

# ✅ Fast — NumPy operates on full arrays at C speed
df["result_fast"] = np.sqrt(df["a"] ** 2 + df["b"] ** 2)

# ✅ Also fast — direct array arithmetic
arr_a = df["a"].to_numpy()
arr_b = df["b"].to_numpy()
df["result_numpy"] = np.hypot(arr_a, arr_b)
```

### When Apply Is Acceptable

`.apply()` with `axis=0` (column-wise) is reasonable because it iterates over columns (typically tens or hundreds), not rows. `.apply()` with `axis=1` should only be used when the logic genuinely cannot be vectorized — for example, calling an external API, running a custom string parser, or applying a conditional that depends on multiple columns in a way that resists vectorization.

### `np.vectorize` — Better Syntax, Same Speed

`np.vectorize` is syntactic sugar over a Python loop. It makes a non-vectorized function *look* vectorized, but it does not compile or optimize the function. Use it for readability when the logic cannot be rewritten with array operations, but do not expect a speed improvement over `.apply()`.

```python
# np.vectorize wraps a scalar function for array input
def classify(a, b):
    if a > 0.5 and b > 0.5:
        return "high"
    elif a > 0.5:
        return "medium"
    return "low"

vectorized_classify = np.vectorize(classify)

# This is cleaner than apply, but has similar performance
df["class"] = vectorized_classify(df["a"].to_numpy(), df["b"].to_numpy())
```

### The Right Tool for Row-Wise Logic

```python
# Use np.where for simple conditional assignment — fully vectorized
df["flag"] = np.where(df["a"] > 0.5, 1, 0)

# Use np.select for multi-condition assignment
conditions = [
    (df["a"] > 0.5) & (df["b"] > 0.5),
    df["a"] > 0.5,
]
choices = ["high", "medium"]
df["class"] = np.select(conditions, choices, default="low")

# Numba for truly complex, loop-dependent logic (JIT compilation)
# from numba import njit
# @njit
# def complex_row_logic(a, b): ...
```

### Limits & Constraints

Even `np.vectorize` and `.apply()` are orders of magnitude slower than true NumPy vectorization. For production ML preprocessing pipelines with millions of rows, any row-wise Python function is a bottleneck. Profile with `%timeit` in a notebook before committing to a `.apply()` call. If vectorization is impossible, consider Numba’s `@njit` decorator for JIT-compiled loops that approach C speed.

-----

## 9. Merging & Reshaping

### Three Ways to Combine DataFrames

Pandas offers three primary combining operations, and choosing the wrong one is a common source of silent data bugs.

|Operation    |Primary Use                                                  |Key Behavior                                              |
|-------------|-------------------------------------------------------------|----------------------------------------------------------|
|`pd.merge()` |SQL-style joins on one or more key columns                   |Flexible; handles all join types; resets/creates new index|
|`pd.concat()`|Stack DataFrames vertically or horizontally                  |Aligns on index; union or intersection of columns         |
|`df.join()`  |Join on the **index** (or one column against another’s index)|Thin wrapper around merge; cleaner syntax for index joins |

### `pd.merge()` — SQL-Style Joins

```python
import pandas as pd
import numpy as np

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "name": ["Alice", "Bob", "Carol", "Dave"],
})

orders = pd.DataFrame({
    "customer_id": [1, 2, 2, 5],
    "amount": [100, 200, 150, 80],
})

# INNER join — only rows with matching keys in both DataFrames
inner = pd.merge(customers, orders, on="customer_id", how="inner")

# LEFT join — all customers, NaN where no matching order
left = pd.merge(customers, orders, on="customer_id", how="left")

# Joining on columns with different names
orders_renamed = orders.rename(columns={"customer_id": "cust_id"})
merged = pd.merge(customers, orders_renamed,
                  left_on="customer_id", right_on="cust_id", how="left")

# Many-to-many join — Pandas does not warn you; row count can explode
# Always check: assert len(result) <= expected_max_rows
```

### `pd.concat()` — Stacking and Aligning

```python
# Vertical stack (default axis=0) — union of columns, NaN for missing
df1 = pd.DataFrame({"a": [1, 2], "b": [3, 4]})
df2 = pd.DataFrame({"a": [5, 6], "c": [7, 8]})
stacked = pd.concat([df1, df2], axis=0, ignore_index=True)
# Column "b" will be NaN for df2 rows; column "c" will be NaN for df1 rows

# Horizontal stack (axis=1) — must have aligned indexes
df3 = pd.DataFrame({"d": [9, 10]})
wide = pd.concat([df1, df3], axis=1)
```

### Data Alignment — What Happens to the Index

> [!IMPORTANT]
> **Pandas always aligns on the index before combining.** When you `concat` two DataFrames with mismatched indexes, Pandas introduces `NaN` for any index label present in one but not the other. This behavior is powerful but dangerous if your indexes are not meaningful (e.g., default `RangeIndex` after different filtering operations). Always call `reset_index(drop=True)` before concatenating DataFrames whose indexes carry no semantic meaning.

```python
# Index alignment in action — this is intentional Pandas behavior
s1 = pd.Series([1, 2, 3], index=["a", "b", "c"])
s2 = pd.Series([10, 20, 30], index=["b", "c", "d"])

# Addition aligns on index — "a" and "d" have no pair, so they become NaN
print(s1 + s2)
# a     NaN
# b    12.0
# c    23.0
# d     NaN

# To avoid this: align explicitly before arithmetic
s1_aligned, s2_aligned = s1.align(s2, fill_value=0)
print(s1_aligned + s2_aligned)
```

### Reshaping — Melt and Pivot

```python
# Wide to long: melt
df_wide = pd.DataFrame({
    "student": ["Alice", "Bob"],
    "math":    [90, 85],
    "english": [88, 92],
})
df_long = df_wide.melt(id_vars="student", var_name="subject", value_name="score")

# Long to wide: pivot
df_back = df_long.pivot(index="student", columns="subject", values="score")
df_back.columns.name = None  # Clean up the "subject" label from the column axis
df_back = df_back.reset_index()
```

### Limits & Constraints

`pd.merge()` with `how="outer"` on large DataFrames can create output significantly larger than either input if there are many non-matching keys. Always verify the row count after a merge: `assert len(merged) == expected_count`. Many-to-many joins are permitted but will silently produce a combinatorial explosion of rows — Pandas does not raise a warning. Index fragmentation from repeated concatenation (e.g., in a loop) creates a non-contiguous memory layout; call `df.reset_index(drop=True)` and `df = df.copy()` after the loop to defragment.

-----

## 10. Time-Series for ML

### Parsing Dates and the `.dt` Accessor

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "timestamp": ["2024-01-15 08:30:00", "2024-02-20 14:45:00", "2024-03-05 09:00:00"],
    "value":     [100.0, 120.0, 95.0],
})

# Parse strings to datetime64 — do this at ingestion time
df["timestamp"] = pd.to_datetime(df["timestamp"])

# The .dt accessor exposes a rich set of datetime components
df["year"]       = df["timestamp"].dt.year
df["month"]      = df["timestamp"].dt.month
df["day_of_week"] = df["timestamp"].dt.dayofweek  # 0=Monday, 6=Sunday
df["hour"]       = df["timestamp"].dt.hour
df["is_weekend"] = df["timestamp"].dt.dayofweek >= 5

# Set timestamp as index for time-series operations
df = df.set_index("timestamp")
```

### `resample()` — Time-Based Aggregation

`resample()` is the time-series equivalent of `groupby()`, but it groups by time intervals rather than column values. It requires a `DatetimeIndex`.

```python
# Generate a sample time series
dates = pd.date_range("2024-01-01", periods=90, freq="D")
ts = pd.DataFrame({
    "sales": np.random.randint(50, 200, size=90),
    "visits": np.random.randint(100, 1000, size=90),
}, index=dates)

# Resample from daily to weekly (sum sales, mean visits)
weekly = ts.resample("W").agg({
    "sales":  "sum",
    "visits": "mean",
})

# Monthly resampling
monthly = ts.resample("ME").sum()
```

### `shift()` and `diff()` — Lead/Lag Features for Forecasting

Lag features are among the most powerful inputs to time-series ML models. `shift()` displaces values forward or backward in time, while `diff()` computes first differences (a common stationarity transform).

```python
# Create lag features — "what was the value 1, 2, 3 days ago?"
ts["sales_lag_1"] = ts["sales"].shift(1)   # 1-period lag
ts["sales_lag_7"] = ts["sales"].shift(7)   # 1-week lag (seasonal)

# Lead feature — "what will the value be 1 day ahead?" (target leakage risk!)
ts["sales_lead_1"] = ts["sales"].shift(-1)  # Use only for target construction

# Rolling statistics — smoothed signal for feature engineering
ts["sales_rolling_7d"] = ts["sales"].rolling(window=7).mean()
ts["sales_rolling_std"] = ts["sales"].rolling(window=7).std()

# First difference — removes trend for stationary modeling
ts["sales_diff"] = ts["sales"].diff(1)

# Drop NaN rows introduced by shift/diff before modeling
ts_clean = ts.dropna()
```

> [!WARNING]
> **Lag features created with `shift()` introduce `NaN` at the start of the series.** For a lag of 7, the first 7 rows will have `NaN` lag features. If you impute those `NaN` values with a mean or forward-fill, you are creating impossible values — your model will “see” future data in the past. Either drop those rows or use `np.nan` as a sentinel and handle it with a null-aware model.

### Limits & Constraints

`resample()` requires a monotonic `DatetimeIndex`. If your timestamps have duplicates or are unsorted, resample will produce incorrect results. Always call `df.sort_index()` and `df.index.is_monotonic_increasing` before resampling. `shift()` does not extrapolate — it simply moves existing values and fills boundaries with `NaN`. Pandas time-series tools are not designed for **high-frequency tick data** (microsecond resolution) at scale; use `Arctic`, `kdb+`, or `TimescaleDB` for that use case.

-----

## 11. Optimization for Scale

### The `category` Dtype — Memory and Speed

The `category` dtype is one of the most impactful optimizations for low-cardinality string columns. Instead of storing the full string for every row, Pandas stores an integer code array (the index into a fixed category list). For a column with 1 million rows and 50 unique values, this can reduce memory from ~60 MB (`object`) to ~1 MB (`category`).

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "country": np.random.choice(["US", "UK", "DE", "FR", "JP"], size=1_000_000),
    "status":  np.random.choice(["active", "inactive", "pending"], size=1_000_000),
})

# Before optimization
print(df.memory_usage(deep=True).sum() / 1e6, "MB")  # ~65 MB

# Convert low-cardinality string columns to category
df["country"] = df["country"].astype("category")
df["status"]  = df["status"].astype("category")

# After optimization
print(df.memory_usage(deep=True).sum() / 1e6, "MB")  # ~2 MB

# GroupBy on category dtype is also 2-5x faster
```

### Downcasting Numeric Types

```python
def downcast_dataframe(df):
    """
    Automatically downcast numeric columns to the smallest
    dtype that can represent the data without loss.
    """
    for col in df.select_dtypes(include="integer").columns:
        df[col] = pd.to_numeric(df[col], downcast="integer")

    for col in df.select_dtypes(include="float").columns:
        df[col] = pd.to_numeric(df[col], downcast="float")

    return df

df = downcast_dataframe(df)
print(df.dtypes)
print(df.memory_usage(deep=True).sum() / 1e6, "MB")
```

> [!IMPORTANT]
> **Downcasting to `float16` requires caution for ML model inputs.** `float16` has a maximum value of ~65,504 and loses precision for values with many significant digits. It is safe for normalized features (values between -1 and 1) but dangerous for raw income, price, or count data. Prefer `float32` as the general-purpose downcast target — it is supported natively by all major ML frameworks and provides sufficient precision for most applications.

### Using Feather and Parquet Instead of CSV

CSV is a terrible format for ML pipelines. It stores everything as text, requires full re-parsing on every load, and carries no dtype information. Columnar binary formats like **Parquet** and **Feather** preserve dtypes, load 10–50× faster, and compress far better.

```python
import pandas as pd

df = pd.read_csv("large_data.csv", dtype={"category_col": "category"})

# --- Parquet: compressed columnar format, ideal for long-term storage ---
# Preserves dtypes including category; supports predicate pushdown
df.to_parquet("data.parquet", engine="pyarrow", compression="snappy", index=False)
df_loaded = pd.read_parquet("data.parquet")

# --- Feather: fast in-memory format, no compression, ideal for intermediate files ---
# Fastest read/write; not suitable for long-term archival
df.to_feather("data.feather")
df_loaded = pd.read_feather("data.feather")

# --- Benchmark illustration (approximate) ---
# CSV read:     5.2s  (1M rows, 20 columns)
# Parquet read: 0.4s  (same data, ~60% smaller file)
# Feather read: 0.15s (same data, larger file, zero parsing overhead)
```

### Full Optimization Pipeline for ML Preprocessing

```python
def optimize_for_ml(df: pd.DataFrame, cat_threshold: int = 50) -> pd.DataFrame:
    """
    Apply all memory and performance optimizations before model training.
    cat_threshold: columns with fewer unique values than this become 'category'.
    """
    df = df.copy()  # Avoid SettingWithCopyWarning

    for col in df.columns:
        col_dtype = df[col].dtype

        if col_dtype == "object":
            n_unique = df[col].nunique()
            if n_unique < cat_threshold:
                df[col] = df[col].astype("category")
            # else: leave as object or handle as high-cardinality string

        elif col_dtype in ["int64", "int32"]:
            df[col] = pd.to_numeric(df[col], downcast="integer")

        elif col_dtype in ["float64", "float32"]:
            df[col] = pd.to_numeric(df[col], downcast="float")

    return df


# Usage
df_optimized = optimize_for_ml(df)
print(f"Before: {df.memory_usage(deep=True).sum() / 1e6:.1f} MB")
print(f"After:  {df_optimized.memory_usage(deep=True).sum() / 1e6:.1f} MB")

# Convert to NumPy for model input — ensures C-contiguous layout
X = np.ascontiguousarray(df_optimized[feature_cols].to_numpy(dtype=np.float32))
y = df_optimized[target_col].to_numpy()
```

### Limits & Constraints

`category` dtype has an important footgun: if you concatenate two DataFrames whose `category` columns have different category lists, Pandas will raise an error or revert to `object`. Always use `pd.CategoricalDtype(categories=known_categories, ordered=False)` when defining categories ahead of time in a pipeline. Parquet requires either `pyarrow` or `fastparquet` installed — `pyarrow` is recommended as it handles more dtypes (including `pd.NA` nullable integers). Feather v2 (via `pyarrow`) is the recommended format; the older Feather v1 (via `feather-format`) is deprecated.

-----

## 12. Cheat Sheet

### Essential Commands at a Glance

|Task                      |Command                                               |Notes                                      |
|--------------------------|------------------------------------------------------|-------------------------------------------|
|**Load CSV**              |`pd.read_csv("f.csv", dtype={...}, parse_dates=[...])`|Always provide `dtype` for large files     |
|**Load Parquet**          |`pd.read_parquet("f.parquet")`                        |Preserves dtypes; preferred over CSV       |
|**True memory usage**     |`df.memory_usage(deep=True).sum()`                    |`deep=True` required for object columns    |
|**Column dtypes**         |`df.dtypes`                                           |Look for `object` columns to fix           |
|**Cast type**             |`df["col"].astype("float32")`                         |Downcast to save memory                    |
|**To NumPy**              |`df.to_numpy()` or `df["col"].to_numpy()`             |Prefer over `.values`                      |
|**Label select**          |`df.loc[rows, cols]`                                  |Inclusive on both ends                     |
|**Position select**       |`df.iloc[rows, cols]`                                 |Exclusive end, like Python slices          |
|**Single cell (fast)**    |`df.at[label, col]` / `df.iat[i, j]`                  |Use in loops                               |
|**Boolean filter**        |`df[(df["a"] > 1) & (df["b"] < 5)]`                   |Parentheses required                       |
|**Query filter**          |`df.query("a > 1 and b < 5")`                         |Faster with `numexpr` on large df          |
|**isin filter**           |`df[df["col"].isin([1, 2, 3])]`                       |Cleaner than chained OR                    |
|**Missing count**         |`df.isnull().sum()`                                   |Per-column missing tally                   |
|**Drop missing**          |`df.dropna(subset=["col"])`                           |Prefer `subset` to avoid losing all rows   |
|**Impute median**         |`df["col"].fillna(df["col"].median())`                |Fit on train set only in ML pipelines      |
|**Forward fill**          |`df.fillna(method="ffill")`                           |Time-series / sensor data                  |
|**Drop duplicates**       |`df.drop_duplicates(subset=["key"])`                  |Specify `subset` for natural key dedup     |
|**Rename columns**        |`df.rename(columns={"old": "new"})`                   |Or `df.columns = [...]` for bulk rename    |
|**Normalize names**       |`df.columns.str.lower().str.replace(" ", "_")`        |Clean column names                         |
|**String contains**       |`df["col"].str.contains("pat", na=False)`             |`na=False` prevents NaN propagation        |
|**Map values**            |`df["col"].map({"a": 1, "b": 2})`                     |Hash-lookup; faster than `.apply()`        |
|**Vectorized op**         |`np.log1p(df["col"])`                                 |Always prefer NumPy ufuncs over `.apply()` |
|**Conditional assign**    |`np.where(cond, val_true, val_false)`                 |Fast; replaces `.apply()` for binary logic |
|**Multi-condition assign**|`np.select([c1, c2], [v1, v2], default=v0)`           |Replaces chained `np.where`                |
|**GroupBy aggregate**     |`df.groupby("key").agg({"col": "sum"})`               |Use string agg names for speed             |
|**Named agg**             |`df.groupby("k").agg(name=("col", "sum"))`            |Pandas 0.25+ named aggregation             |
|**Group transform**       |`df.groupby("k")["col"].transform("mean")`            |Returns same-length Series                 |
|**Pivot table**           |`pd.pivot_table(df, values, index, columns, aggfunc)` |SQL-like cross-tab with aggregation        |
|**Melt (wide→long)**      |`df.melt(id_vars=[...], var_name, value_name)`        |Reshape for tidy data                      |
|**Merge (SQL join)**      |`pd.merge(left, right, on="key", how="left")`         |All join types: inner/left/right/outer     |
|**Concat (stack)**        |`pd.concat([df1, df2], ignore_index=True)`            |`ignore_index` resets RangeIndex           |
|**To datetime**           |`pd.to_datetime(df["col"])`                           |Do at ingestion time                       |
|**Date components**       |`df["col"].dt.month`, `.dt.dayofweek`                 |Requires DatetimeIndex or datetime Series  |
|**Resample**              |`df.resample("W").sum()`                              |Requires DatetimeIndex                     |
|**Lag feature**           |`df["col"].shift(7)`                                  |NaN introduced at start                    |
|**Rolling mean**          |`df["col"].rolling(7).mean()`                         |NaN introduced at start                    |
|**Category dtype**        |`df["col"].astype("category")`                        |Low-cardinality strings: huge memory saving|
|**Downcast numeric**      |`pd.to_numeric(df["col"], downcast="float")`          |float64→float32; int64→int8/16/32          |
|**Save Parquet**          |`df.to_parquet("f.parquet", compression="snappy")`    |Best for persistent storage                |
|**Save Feather**          |`df.to_feather("f.feather")`                          |Best for intermediate pipeline files       |
|**Contiguous array**      |`np.ascontiguousarray(df.to_numpy())`                 |Required before some ML frameworks         |
|**Copy to avoid warning** |`df_sub = df[mask].copy()`                            |Prevents `SettingWithCopyWarning`          |

-----

*Document version 1.0 — Built for data scientists working in Python 3.9+ with Pandas 1.5+ and NumPy 1.23+.*
