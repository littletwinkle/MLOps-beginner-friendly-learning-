# human-learn: Outlier Detection — Resolved

**Status:** Resolved
**Author:** K
**Related:** sklearn-learnings.md (human-learn section)

## Summary

`human-learn` handles outliers through a dedicated outlier-detection module that follows scikit-learn's own outlier API: `.fit()` then `.predict()`, returning `1` for inlier and `-1` for outlier (same convention as `IsolationForest` / `OneClassSVM`).

## Two approaches

- **`FunctionOutlierDetector`** — wraps a plain function that returns `True`/`False` for whether a point looks normal. Anything that fails the function is flagged `-1`.
- **`InteractiveOutlierDetector`** — lets you draw a region on a chart marking what counts as "normal" data; anything outside the drawn boundary is an outlier.

## Applied to the penguin example

Define a function checking whether a penguin's measurements (length, etc.) fall inside the expected range for its region, wrap it in `FunctionOutlierDetector`, `.fit()` it on known data, then `.predict()` on new penguins — anything outside the expected pattern returns `-1`.

## Assignment: try it locally

**Goal:** build a `FunctionOutlierDetector` for the penguin dataset and test it against new, unseen measurements.

1. Install locally:
```bash
pip install human-learn pandas scikit-learn
```

2. Load a penguin dataset (e.g. `seaborn`'s built-in one, or any CSV with `species`/`region`, `bill_length_mm`, `flipper_length_mm`).

3. Write a rule function that returns `True` for "normal" penguins — e.g. bill length and flipper length both within a plausible range for their region — and `False` otherwise.

4. Wrap it in `FunctionOutlierDetector`, fit it on the known data, then predict on a few new/edited rows (including at least one you deliberately make unrealistic, like a tiny flipper length) to confirm it correctly returns `-1` for the outlier.

### Starter code (fill in the threshold logic yourself first)
```python
import pandas as pd
from hulearn.outlier_detection import FunctionOutlierDetector

def is_normal_penguin(dataf):
    # TODO: return a boolean Series - True for "normal", False for outlier
    # e.g. dataf["bill_length_mm"].between(30, 60)
    pass

df = pd.read_csv("penguins.csv")  # or load from seaborn

detector = FunctionOutlierDetector(is_normal_penguin)
detector.fit(df)

predictions = detector.predict(df)
print(predictions)  # 1 = inlier, -1 = outlier
```

### Solution (reference, once you've tried it yourself)
```python
import pandas as pd
from hulearn.outlier_detection import FunctionOutlierDetector

def is_normal_penguin(dataf):
    return dataf["bill_length_mm"].between(30, 60) & dataf["flipper_length_mm"].between(170, 235)

df = pd.read_csv("penguins.csv")

detector = FunctionOutlierDetector(is_normal_penguin)
detector.fit(df)

predictions = detector.predict(df)
print(predictions)

# Test with a deliberately unrealistic row
test_row = pd.DataFrame([{"bill_length_mm": 5, "flipper_length_mm": 50}])
print(detector.predict(test_row))  # expect -1
```

Adjust the threshold ranges to match whatever penguin dataset you're actually using - the ranges above are illustrative, not verified against a specific dataset.

## Interactive outlier detection (drawing polygons)

`InteractiveOutlierDetector` builds the outlier model from polygons you draw on charts, instead of from a function.

- `InteractiveCharts(df, labels="<column>")` builds the charts. The label column colors the points while you draw.
- You can use an existing column (region, species) or create your own, e.g. `df["group"] = ...` with values like "group 1" / "group 2", and pass its name as `labels`.
- `charts.add_chart(x=..., y=...)` shows a chart to draw on. Run each `add_chart` in its own cell.
- `InteractiveOutlierDetector(json_desc=charts.data(), threshold=1)` turns the drawing into a model. `.fit(X, y)` is only a formality; `.predict(X)` returns `1` inside the polygons and `-1` for outliers.
- `threshold` is the minimum number of polygons a point must fall in to not be an outlier (default 1).
- Drawing needs a mouse in a Jupyter notebook (uses Bokeh), so use local Jupyter rather than a phone. matplotlib is only for viewing results: scatter the features and color by `preds`.

```python
from hulearn.experimental.interactive import InteractiveCharts
from hulearn.outlier import InteractiveOutlierDetector
import numpy as np
import matplotlib.pyplot as plt

df["group"] = np.where(df["bill_length_mm"] < 45, "group 1", "group 2")

charts = InteractiveCharts(df, labels="group")
charts.add_chart(x="bill_length_mm", y="bill_depth_mm")      # own cell, draw polygons
charts.add_chart(x="flipper_length_mm", y="body_mass_g")     # own cell, draw polygons

X, y = df.drop(columns=["group"]), df["group"]
model = InteractiveOutlierDetector(json_desc=charts.data(), threshold=1)
preds = model.fit(X, y).predict(X)  # 1 = inside, -1 = outlier

plt.scatter(X["bill_length_mm"], X["bill_depth_mm"], c=preds)
plt.xlabel("bill_length_mm")
plt.ylabel("bill_depth_mm")
plt.show()
```

### Missing values (course takeaway)

From the last part of the freeCodeCamp scikit-learn crash course: when a new row is missing one feature (e.g. `bill_length_mm`) but the others are present, prediction is still possible using the polygons on charts of the other columns.

Not yet verified on the installed version: how `threshold` interacts with this. With `threshold=1`, one chart accepting the point is likely enough; with `threshold=2`, the chart using the missing column may not be able to count, so the point could come back `-1`. Test it:

```python
test = X.iloc[[0]].copy()
test["bill_length_mm"] = np.nan
print(model.predict(test))
```
