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
