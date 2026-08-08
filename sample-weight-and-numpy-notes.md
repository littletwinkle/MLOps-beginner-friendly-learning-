# sample_weight, class_weight & Custom Scoring — Notes

## 1. sample_weight

Passed into `.fit(X, y, sample_weight=...)` — an array, one value per row, telling the model how much
each individual sample should count during training.

- Example: in a financial transactions dataset, you might give higher weight to rows with larger
  transaction amounts, so the model pays more attention to getting those right.
- **Key insight (and you had this right):** sample_weight doesn't just adjust how you *read* the
  metrics afterward — it changes the training algorithm itself. The loss function is computed as a
  *weighted* sum across samples, so the model's learned parameters (coefficients, splits, etc.) come
  out different depending on the weights. It genuinely reshapes what "best fit" means to the model.

## 2. class_weight — different from sample_weight

This is the one worth separating out clearly, since it's a related but distinct idea:

| | sample_weight | class_weight |
|---|---|---|
| Applies to | Individual rows | Entire classes (e.g. "fraud" vs "not fraud") |
| Set where | Passed into `.fit()` | Usually set on the estimator itself, e.g. `LogisticRegression(class_weight="balanced")` |
| Typical use | "This specific row matters more" | "This whole category is rare/important, don't let the model ignore it" |

Fraud detection is actually the classic `class_weight` example: fraud cases are a tiny minority, so
without reweighting, a model can get high accuracy by just predicting "not fraud" every time.
`class_weight="balanced"` automatically compensates for this imbalance.

## 3. Custom metrics (precision/recall for a specific class) inside GridSearchCV

You're right that you can't just drop a raw metric function into GridSearchCV's `scoring` — it needs
to be wrapped first:

```python
from sklearn.metrics import make_scorer, precision_score, recall_score

# e.g. scoring precision specifically for the "fraud" (positive) class
fraud_precision = make_scorer(precision_score, pos_label=1)
fraud_recall = make_scorer(recall_score, pos_label=1)

grid = GridSearchCV(
    estimator=my_model,
    param_grid=param_grid,
    scoring=fraud_precision,   # or a dict of multiple scorers
    cv=5
)
grid.fit(X, y)   # sample_weight can also be passed here: grid.fit(X, y, sample_weight=weights)
```

`make_scorer` converts a regular metric function into something GridSearchCV knows how to call
consistently across every fold and parameter combination — that's the piece that lets it be reused
inside `.fit()`.

---

## 4. NumPy exploration (as requested)

### `np.linspace(start, stop, num)`

Generates `num` evenly spaced values between `start` and `stop` (inclusive of both ends by default).

```python
import numpy as np
np.linspace(0, 1, 5)
# array([0.  , 0.25, 0.5 , 0.75, 1.  ])
```

Common use: building a smooth range of hyperparameter values to test (e.g. for `param_grid`), or
generating x-values for plotting a smooth curve.

```python
# useful inside a param_grid
param_grid = {"alpha": np.linspace(0.01, 1.0, 10)}
```

### `np.log(x)`

Natural log (base e) transform, applied element-wise to an array.

```python
np.log(np.array([1, np.e, 10]))
# array([0.        , 1.        , 2.30258509])
```

Common use in ML: compressing right-skewed data (e.g. income, transaction amount, population) so
extreme values don't dominate — similar goal to QuantileTransformer, but a simpler, deterministic
formula rather than a distribution-matching transform. Note: `np.log(0)` is undefined (`-inf`), so
data with zeros often uses `np.log1p(x)` (computes `log(1 + x)`) instead.

---

## Quick mental model

- **sample_weight** → "this row matters more" → changes training math directly
- **class_weight** → "this category matters more" → set on the estimator, handles imbalance
- **make_scorer** → wraps a metric so GridSearchCV can reuse it consistently across folds
- **np.linspace** → generate a smooth range of values (great for `param_grid` ranges)
- **np.log / np.log1p** → compress skewed data before modeling
