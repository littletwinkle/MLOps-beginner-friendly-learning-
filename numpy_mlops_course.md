# NumPy for Testers & MLOps Engineers — Beginner Course

> **Who this is for:** Software testers moving into ML, and beginners learning MLOps fundamentals.
> **Environment:** Kaggle Notebooks / Jupyter (NumPy pre-installed)
> **Goal:** Understand NumPy as the data layer that sits under every ML pipeline.

---

## Table of Contents
1. [Why NumPy Matters in MLOps](#1-why-numpy-matters-in-mlops)
2. [Creating Arrays](#2-creating-arrays)
3. [Array Properties](#3-array-properties)
4. [Reshaping Arrays](#4-reshaping-arrays)
5. [Indexing and Slicing](#5-indexing-and-slicing)
6. [Boolean Filtering and Masking](#6-boolean-filtering-and-masking)
7. [Math Operations and Broadcasting](#7-math-operations-and-broadcasting)
8. [Saving and Loading Arrays](#8-saving-and-loading-arrays)
9. [Common Errors and How to Fix Them](#9-common-errors-and-how-to-fix-them)
10. [Mini Project: Data Preprocessing Pipeline](#10-mini-project-data-preprocessing-pipeline)

---

## 1. Why NumPy Matters in MLOps

In an ML pipeline, data flows through multiple stages — ingestion, preprocessing, model training, prediction. NumPy is the foundation of all of these.

| Role | How NumPy is Used |
|------|------------------|
| **Tester** | Validate data shapes, check for nulls/outliers, compare expected vs actual arrays |
| **MLOps Engineer** | Preprocess raw data, build feature arrays, save/load model inputs/outputs |

```python
import numpy as np

# Every ML model receives and returns NumPy arrays
model_input = np.array([5.1, 3.5, 1.4, 0.2])  # one sample, 4 features
print(type(model_input))   # <class 'numpy.ndarray'>
print(model_input.shape)   # (4,)
```

---

## 2. Creating Arrays

### Basic creation
```python
import numpy as np

# From a list
scores = np.array([85, 90, 78, 92, 88])

# Zeros — used to initialise empty results containers
predictions = np.zeros(5)
print(predictions)   # [0. 0. 0. 0. 0.]

# Ones — used in weight initialisation
weights = np.ones((3, 3))

# Full — fill with a constant value
placeholder = np.full((2, 4), -1)

# Range of values — useful for batch indices
batch_ids = np.arange(0, 100, 10)
print(batch_ids)   # [ 0 10 20 30 40 50 60 70 80 90]

# Evenly spaced values — for thresholds, learning rates
thresholds = np.linspace(0.1, 0.9, 5)
print(thresholds)   # [0.1  0.3  0.5  0.7  0.9]
```

### Tester perspective
```python
# Check that model output has the right number of predictions
expected_count = 100
actual_output = np.zeros(100)  # simulating model output
assert len(actual_output) == expected_count, "Output size mismatch!"
```

### MLOps perspective
```python
# Initialise a results array before filling in a loop
n_samples = 1000
results = np.zeros(n_samples)
for i in range(n_samples):
    results[i] = i * 0.5  # simulating predictions
```

---

## 3. Array Properties

Properties (not methods) — no parentheses needed.

```python
data = np.array([[1, 2, 3],
                 [4, 5, 6]])

print(data.shape)    # (2, 3) — 2 rows, 3 columns
print(data.ndim)     # 2 — number of dimensions
print(data.dtype)    # int64 — data type
print(data.size)     # 6 — total elements
```

### Why this matters for testers
```python
# Validate pipeline output dimensions
model_output = np.random.rand(100, 5)   # 100 samples, 5 class probabilities

assert model_output.shape == (100, 5), f"Unexpected shape: {model_output.shape}"
assert model_output.dtype == np.float64, f"Wrong dtype: {model_output.dtype}"
print("Shape and dtype checks passed.")
```

### Why this matters for MLOps
```python
# Before feeding data into a model, confirm shape is correct
feature_matrix = np.random.rand(500, 10)
print(f"Feeding {feature_matrix.shape[0]} samples with {feature_matrix.shape[1]} features")
```

---

## 4. Reshaping Arrays

```python
flat = np.arange(12)
print(flat.shape)   # (12,)

matrix = flat.reshape(3, 4)   # reshape is a METHOD — use parentheses
print(matrix.shape)   # (3, 4)

# -1 lets NumPy calculate the missing dimension
auto = flat.reshape(-1, 4)
print(auto.shape)   # (3, 4)
```

### MLOps use case
```python
# ML models often expect 2D input (samples, features)
single_sample = np.array([5.1, 3.5, 1.4, 0.2])
print(single_sample.shape)   # (4,)  — 1D, will cause errors in sklearn

# Fix: reshape to (1, 4)
ready_for_model = single_sample.reshape(1, -1)
print(ready_for_model.shape)   # (1, 4)
```

### Tester use case
```python
# Verify reshape does not change total data
original = np.arange(24)
reshaped = original.reshape(4, 6)
assert original.size == reshaped.size, "Data lost during reshape!"
print("Reshape integrity check passed.")
```

---

## 5. Indexing and Slicing

```python
data = np.array([10, 20, 30, 40, 50, 60, 70, 80])

print(data[0])      # 10 — first element
print(data[-1])     # 80 — last element
print(data[2:5])    # [30 40 50] — slice
print(data[::2])    # [10 30 50 70] — every other element
```

### 2D slicing
```python
matrix = np.array([[1,  2,  3,  4],
                   [5,  6,  7,  8],
                   [9, 10, 11, 12]])

print(matrix[0, :])    # [1 2 3 4]  — first row
print(matrix[:, 1])    # [2 6 10]   — second column
print(matrix[1:, 2:])  # [[7 8] [11 12]]  — submatrix
```

### MLOps use case — batch slicing
```python
dataset = np.random.rand(1000, 10)   # 1000 samples, 10 features
batch_size = 32

batch_1 = dataset[0:32]
batch_2 = dataset[32:64]
print(f"Batch shape: {batch_1.shape}")   # (32, 10)
```

### Tester use case — spot check rows
```python
# Verify specific rows contain expected values
data = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
assert data[1, 2] == 6, "Row 1, Col 2 should be 6"
print("Spot check passed.")
```

---

## 6. Boolean Filtering and Masking

```python
scores = np.array([45, 78, 92, 55, 88, 34, 67])

# Boolean mask — returns True/False for each element
mask = scores > 60
print(mask)   # [False  True  True False  True False  True]

# Apply mask to filter values
passing = scores[mask]
print(passing)   # [78 92 88 67]

# Shorthand (same result)
print(scores[scores > 60])
```

### np.where() — conditional replacement
```python
# Replace values conditionally (like IF in Excel)
labels = np.where(scores >= 60, "PASS", "FAIL")
print(labels)   # ['FAIL' 'PASS' 'PASS' 'FAIL' 'PASS' 'FAIL' 'PASS']
```

### Combining conditions — brackets are required
```python
data = np.array([10, 25, 45, 60, 80, 95])

# Between 30 and 70
filtered = data[(data >= 30) & (data <= 70)]
print(filtered)   # [45 60]

# Below 20 OR above 80
extremes = data[(data < 20) | (data > 80)]
print(extremes)   # [10 95]
```

### Tester use case — validate model output range
```python
predictions = np.array([0.1, 0.85, 1.2, -0.05, 0.7])   # simulated probabilities

out_of_range = predictions[(predictions < 0) | (predictions > 1)]
assert len(out_of_range) == 0, f"Invalid probabilities found: {out_of_range}"
```

### MLOps use case — flag anomalies
```python
response_times = np.array([120, 135, 980, 142, 128, 1200, 131])
threshold = 500

anomalies = np.where(response_times > threshold, "ALERT", "OK")
print(anomalies)   # ['OK' 'OK' 'ALERT' 'OK' 'OK' 'ALERT' 'OK']
```

---

## 7. Math Operations and Broadcasting

```python
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

print(a + b)    # [11 22 33 44]
print(a * b)    # [10 40 90 160]
print(b / a)    # [10. 10. 10. 10.]
print(a ** 2)   # [ 1  4  9 16]
```

### Aggregate functions
```python
data = np.array([4, 7, 2, 9, 1, 5])

print(np.sum(data))    # 28
print(np.mean(data))   # 4.666...
print(np.max(data))    # 9
print(np.min(data))    # 1
print(np.std(data))    # standard deviation
```

### Broadcasting example
```python
# Apply a discount to every product in every shop
# Shape: (3 shops, 4 products)
revenue = np.array([[200, 150, 300, 100],
                    [180, 210, 270, 130],
                    [250, 190, 310, 160]])

# Discount rates per shop (shape: (3,) broadcasts to (3,4))
discounts = np.array([0.10, 0.15, 0.20])
savings = revenue * discounts[:, np.newaxis]
print(savings.round(1))
```

### MLOps use case — normalise features
```python
features = np.array([[1, 200, 0.5],
                     [2, 150, 1.0],
                     [3, 300, 0.8]])

# Min-max normalisation across each column
min_vals = features.min(axis=0)
max_vals = features.max(axis=0)
normalised = (features - min_vals) / (max_vals - min_vals)
print(normalised.round(3))
```

---

## 8. Saving and Loading Arrays

```python
# Save a single array
data = np.array([1, 2, 3, 4, 5])
np.save('/kaggle/working/data.npy', data)

# Load it back
loaded = np.load('/kaggle/working/data.npy')
print(loaded)   # [1 2 3 4 5]
```

### Save multiple arrays (MLOps pipeline handoff)
```python
X_train = np.random.rand(800, 10)
X_test  = np.random.rand(200, 10)
y_train = np.random.randint(0, 2, 800)
y_test  = np.random.randint(0, 2, 200)

# Save all in one file
np.savez('/kaggle/working/split_data.npz',
         X_train=X_train, X_test=X_test,
         y_train=y_train, y_test=y_test)

# Load back by name
loaded = np.load('/kaggle/working/split_data.npz')
print(loaded['X_train'].shape)   # (800, 10)
print(loaded['y_test'].shape)    # (200,)
```

### Tester use case — verify saved data integrity
```python
original = np.array([10, 20, 30, 40, 50])
np.save('/kaggle/working/test_data.npy', original)

reloaded = np.load('/kaggle/working/test_data.npy')
np.testing.assert_array_equal(original, reloaded)
print("Data integrity check passed.")
```

---

## 9. Common Errors and How to Fix Them

| Error | Cause | Fix |
|-------|-------|-----|
| `NameError: name 'np' is not defined` | Forgot to import | Add `import numpy as np` |
| `ValueError: cannot reshape array` | Size mismatch | Check `.size` before reshaping |
| `IndexError: index out of bounds` | Index too large | Check `.shape` first |
| `TypeError: ufunc 'add' did not contain a loop` | Mixed types | Use `.astype(float)` |
| `SyntaxError` with `&` or `\|` | Missing brackets | Wrap each condition in `()` |

```python
# Fix: reshape error
arr = np.arange(10)
try:
    arr.reshape(3, 4)
except ValueError as e:
    print(f"Error: {e}")
    print(f"Array has {arr.size} elements — try reshape(2,5) or reshape(5,2)")

# Fix: boolean condition brackets
scores = np.array([45, 78, 92, 55])
# Wrong:  scores[scores > 50 & scores < 90]
# Correct:
result = scores[(scores > 50) & (scores < 90)]
print(result)   # [78 55]  — wait, 55 is not > 50 ✓ and < 90 ✓ → [78]
```

---

## 10. Mini Project: Data Preprocessing Pipeline

> **Scenario:** You receive raw sensor data. Preprocess it and save it ready for a model.

```python
import numpy as np

# --- Step 1: Simulate raw incoming data ---
np.random.seed(42)
raw_data = np.random.randint(0, 200, size=(100, 5)).astype(float)

# Inject some missing values (represented as -1)
raw_data[5, 2]  = -1
raw_data[20, 0] = -1
raw_data[55, 4] = -1

print(f"Raw data shape: {raw_data.shape}")

# --- Step 2: Detect and report anomalies (Tester role) ---
missing_mask = raw_data == -1
missing_count = np.sum(missing_mask)
print(f"Missing values found: {missing_count}")
assert missing_count < raw_data.size * 0.05, "Too many missing values — pipeline halted."

# --- Step 3: Replace missing values with column mean (MLOps role) ---
for col in range(raw_data.shape[1]):
    col_data = raw_data[:, col]
    valid_mean = np.mean(col_data[col_data != -1])
    col_data[col_data == -1] = valid_mean

print("Missing values replaced with column means.")

# --- Step 4: Normalise (min-max scaling) ---
min_vals = raw_data.min(axis=0)
max_vals = raw_data.max(axis=0)
normalised = (raw_data - min_vals) / (max_vals - min_vals)

assert normalised.min() >= 0.0, "Normalisation failed — values below 0"
assert normalised.max() <= 1.0, "Normalisation failed — values above 1"
print(f"Normalised range: {normalised.min():.3f} to {normalised.max():.3f}")

# --- Step 5: Train/test split ---
split = int(0.8 * len(normalised))
X_train = normalised[:split]
X_test  = normalised[split:]
print(f"Train: {X_train.shape}, Test: {X_test.shape}")

# --- Step 6: Save pipeline output ---
np.savez('/kaggle/working/preprocessed.npz', X_train=X_train, X_test=X_test)
print("Saved to /kaggle/working/preprocessed.npz")

# --- Step 7: Verify saved data (Tester role) ---
loaded = np.load('/kaggle/working/preprocessed.npz')
np.testing.assert_array_almost_equal(loaded['X_train'], X_train)
print("Pipeline output verified successfully.")
```

---

## Quick Reference Card

```python
import numpy as np

np.array([1,2,3])          # create from list
np.zeros((3,4))            # zeros array
np.ones((3,4))             # ones array
np.full((2,3), 99)         # fill with value
np.arange(0, 10, 2)        # range with step
np.linspace(0, 1, 5)       # evenly spaced

arr.shape                   # dimensions (property)
arr.ndim                    # number of dimensions (property)
arr.dtype                   # data type (property)
arr.reshape(3, 4)           # reshape (method)

arr[0]                      # first element
arr[-1]                     # last element
arr[1:4]                    # slice
arr[arr > 5]                # boolean filter
np.where(arr > 5, 'Y','N')  # conditional values

np.sum(arr)                 # sum
np.mean(arr)                # mean
np.std(arr)                 # standard deviation
np.min(arr) / np.max(arr)   # min / max

np.save('file.npy', arr)           # save single array
np.load('file.npy')                # load single array
np.savez('file.npz', a=arr1)       # save multiple
np.load('file.npz')['a']           # load by name
```

---

*Built as part of a 6-week MLOps portfolio roadmap.*
*Next: pandas_mlops_course.md*
