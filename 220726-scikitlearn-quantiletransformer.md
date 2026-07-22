# QuantileTransformer in scikit-learn

Part of the `mlops-learning-journey` series — Week 6. Covers `QuantileTransformer`, how it differs from `StandardScaler`, and why it's robust to outliers.

---

## 1. Why scaling matters

Most ML models are sensitive to the *scale* of numeric features — a feature ranging from 0–1,000,000 can dominate one ranging from 0–1, even if the smaller one is more predictive. Scaling puts features on comparable footing before fitting a model.

---

## 2. StandardScaler — recap

`StandardScaler` scales each value using:

```
z = (x - mean) / std_dev
```

- Centers data around 0 with unit variance.
- **Problem:** mean and standard deviation are both pulled by outliers. One extreme value can distort the scale for every other point in the column.

---

## 3. QuantileTransformer

### What it does

Instead of using mean/std, `QuantileTransformer`:

1. Ranks every value by its **quantile (percentile) position** in the data.
2. Maps that percentile rank onto a **target distribution** — either:
   - `output_distribution="normal"` (Gaussian-shaped), or
   - `output_distribution="uniform"` (spread evenly between 0 and 1)

This makes it a **non-linear** transformation — it doesn't just shift and rescale like `StandardScaler`; it reshapes the actual distribution of the data.

### Why it's robust to outliers

Because the transform works on **rank**, not raw magnitude, an extreme value simply lands at the top (or bottom) percentile — it doesn't drag the rest of the scale with it the way a huge value would distort a mean.

| | StandardScaler | QuantileTransformer |
|---|---|---|
| Basis | Mean & standard deviation | Quantile / percentile rank |
| Transformation type | Linear | Non-linear |
| Outlier sensitivity | High — one outlier skews the whole scale | Low — outliers pushed to distribution edges, rest unaffected |
| Best used when | Data is roughly normal, no major outliers | Data is skewed or outlier-heavy |

### Example

```python
from sklearn.preprocessing import QuantileTransformer, StandardScaler
import numpy as np

X = np.array([[1], [2], [3], [4], [5], [100]])  # 100 is an outlier

scaler = StandardScaler()
print(scaler.fit_transform(X))
# Outlier drags the mean/std — every other value gets compressed near the bottom

qt = QuantileTransformer(output_distribution="normal", n_quantiles=6, random_state=0)
print(qt.fit_transform(X))
# Outlier is pushed to the far right of the normal distribution
# The other 5 values keep sensible, evenly-spread positions relative to each other
```

### Key parameters

| Parameter | Purpose |
|---|---|
| `output_distribution` | Target shape: `"normal"` or `"uniform"` |
| `n_quantiles` | Number of quantiles used to estimate the mapping (default 1000; reduce for small datasets) |
| `random_state` | For reproducibility when subsampling large data |

---

## 4. When to choose which

- **`StandardScaler`** → default choice when data is roughly normal and you have no strong reason to expect outliers.
- **`QuantileTransformer`** → when data is skewed, has heavy tails, or contains outliers you can't (or don't want to) remove — it normalizes the shape without needing to hand-pick a cutoff.

---

## 5. Learning in progress

- **Metrics** — currently studying evaluation metrics (e.g. for regression/classification) to compare model performance against a baseline. Notes to be expanded as this topic develops.

---

*Next up: continuing through remaining `sklearn.preprocessing` transformers, and metrics.*
