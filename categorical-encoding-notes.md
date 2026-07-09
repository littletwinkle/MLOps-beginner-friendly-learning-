# Categorical Encoding in scikit-learn

Part of the `mlops-learning-journey` series. Covers `OrdinalEncoder` and `OneHotEncoder`, focusing on how they handle rare categories, unknown categories, and missing values.

---

## 1. OrdinalEncoder

Encodes categories as integers (0, 1, 2, ...). Useful when categories have a natural order, or as a simpler alternative to one-hot encoding.

### Key parameters

| Parameter | Purpose |
|---|---|
| `handle_unknown="use_encoded_value"` | Don't error on unseen categories at transform time |
| `unknown_value` | The integer code assigned to categories never seen during `fit()` |
| `encoded_missing_value` | The integer code assigned to `NaN` / missing values |
| `min_frequency` | Absolute cutoff — categories occurring fewer than this many times become "infrequent" |
| `max_categories` | Cap on total distinct codes — keeps the top `(max_categories - 1)` most frequent categories individually, merges the rest into one "infrequent" bucket |

### Example

```python
X_train = np.array(
    [["a"] * 5 + ["b"] * 20 + ["c"] * 10 + ["d"] * 3 + [np.nan]],
    dtype=object).T

enc = preprocessing.OrdinalEncoder(
    handle_unknown="use_encoded_value", unknown_value=3,
    max_categories=3, encoded_missing_value=4)
enc.fit(X_train)

X_test = np.array([["a"], ["b"], ["c"], ["d"], ["e"], [np.nan]], dtype=object)
enc.transform(X_test)

# array([[2.],
#        [0.],
#        [1.],
#        [2.],
#        [3.],
#        [4.]])
```

### Reading the output

Training frequencies: b=20, c=10, a=5, d=3.

With `max_categories=3`, sklearn keeps the top 2 most frequent as individual codes and merges everything else into one shared "infrequent" code:

| Category | Code | Reason |
|---|---|---|
| b | 0 | frequent, own code |
| c | 1 | frequent, own code |
| a | 2 | infrequent → merged bucket |
| d | 2 | infrequent → same merged bucket as a |
| e | 3 | never seen in training → `unknown_value` |
| NaN | 4 | missing → `encoded_missing_value` |

### `min_frequency` vs `max_categories`

Two different rules for handling rare categories — can be combined.

- **`min_frequency`**: absolute cutoff. "Anything below this count is infrequent." (e.g. `min_frequency=5` → anything occurring < 5 times is merged)
- **`max_categories`**: relative/ranked cutoff. "Keep only the top N-1 most frequent individually, merge the rest."

If both are set, sklearn applies `min_frequency` first, then further merges by rank if the category count still exceeds `max_categories`.

---

## 2. OneHotEncoder

Encodes each category as its own binary column (1 if present, 0 otherwise). No implied order between categories.

### Handling unknown categories

With `handle_unknown="ignore"`, an unseen category at transform time doesn't error — it produces a row of **all zeros** (no column fires), rather than being assigned a specific code like OrdinalEncoder does.

### Example

```
# Training data / matching test data
[[1. 0. 0.]
 [0. 1. 0.]
 [0. 0. 1.]]

# Unseen category at transform time
[[0. 0. 0.]]
```

### Key contrast: OrdinalEncoder vs OneHotEncoder for unknowns

| Encoder | Unknown category handling |
|---|---|
| `OrdinalEncoder` (`unknown_value=N`) | Assigned one specific integer code |
| `OneHotEncoder` (`handle_unknown="ignore"`) | Row of all zeros — no column fires |

---

## 3. NumPy slicing refresher (used throughout)

`X[:, -1]` — all rows, last column only.

- `:` → all rows
- `-1` → last column

```python
X = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

X[:, -1]     # array([3, 6, 9])  → last column
X[:, 0]      # first column
X[:, :-1]    # all columns EXCEPT the last (start-inclusive, end-exclusive slicing)
```

`:-1` excludes only the very last index — it does **not** exclude middle columns. E.g. for 4 columns (indices 0,1,2,3), `X[:, :-1]` includes indices 0, 1, 2.

### Features vs. target

- **Features (X)**: input variables fed to the model (independent variables / predictors)
- **Target (y)**: the value being predicted (dependent variable / label)

```python
X = data[:, :-1]   # all columns except last → features
y = data[:, -1]    # last column only → target
```

Model learns the mapping `features → target` during `.fit(X, y)`, then predicts new `y` from new `X` during `.predict(X)`.

---

## 4. `plt.scatter` refresher

Used to visualize the relationship between two numeric variables — each point is one data sample.

### Basic syntax

```python
import matplotlib.pyplot as plt

plt.scatter(X, y)
plt.xlabel("Feature")
plt.ylabel("Target")
plt.title("Feature vs Target")
plt.show()
```

### Common parameters

| Parameter | Purpose |
|---|---|
| `x`, `y` | Data for horizontal/vertical axes (arrays of equal length) |
| `c` | Color — can be a single color, or an array to color-code by category/value |
| `s` | Marker size (single value or array to vary per point) |
| `alpha` | Transparency (0 = invisible, 1 = solid) — useful when points overlap |
| `cmap` | Colormap used when `c` is an array of values |
| `marker` | Marker style (`"o"`, `"x"`, `"^"`, etc.) |
| `label` | Legend label for this set of points |

### Example: comparing actual vs predicted values

A common ML use case — check how close predictions are to the true values:

```python
plt.scatter(y_test, y_pred, alpha=0.6)
plt.xlabel("Actual values")
plt.ylabel("Predicted values")
plt.title("Actual vs Predicted")
plt.plot([y_test.min(), y_test.max()],
         [y_test.min(), y_test.max()],
         color="red", linestyle="--")  # perfect-prediction reference line
plt.show()
```

If predictions were perfect, every point would fall exactly on the red diagonal line.

### Example: color-coding by category

```python
plt.scatter(X[:, 0], X[:, 1], c=y, cmap="viridis")
plt.colorbar(label="Target class")
plt.show()
```

Here, points are colored according to their target/class label (`y`), useful for visually inspecting whether classes are separable based on two features.

---

*Next up: continuing through remaining `sklearn.preprocessing` transformers.*
