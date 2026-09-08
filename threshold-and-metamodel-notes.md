# Decision Threshold & Meta-Models

## Decision Threshold

Classifiers like `LogisticRegression` don't output a label directly — they output a probability.

```python
y_proba = model.predict_proba(X_test)[:, 1]   # probability of positive class
threshold = 0.5
y_pred = (y_proba >= threshold).astype(int)   # probability -> label
```

- Default threshold is 0.5, but it's an arbitrary choice, not a fixed rule.
- **Lower threshold** → model flags more cases as positive → recall ↑, precision ↓
- **Higher threshold** → model flags fewer cases as positive → recall ↓, precision ↑
- `threshold` is a plain Python variable — there is nothing to import. Compare it to `if age >= 18:` — same kind of comparison, just applied to a model's probability output instead.
- Recall/precision are computed **after** applying the threshold, and are aggregate scores over the whole test set — not a per-row property.

## Meta-Models (Stacking)

A meta-model is any ordinary classifier assigned to `final_estimator` inside `StackingClassifier`. It learns from the *predictions* of the base models, not the raw features.

```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, StackingClassifier
from sklearn.neighbors import KNeighborsClassifier

base_estimators = [
    ("logreg", LogisticRegression(max_iter=1000)),
    ("rf", RandomForestClassifier(n_estimators=100, random_state=42)),
    ("knn", KNeighborsClassifier(n_neighbors=5))
]

stacking_model = StackingClassifier(
    estimators=base_estimators,
    final_estimator=LogisticRegression(max_iter=1000),   # this IS the meta-model
    cv=5
)

stacking_model.fit(X_train_scaled, y_train)
y_proba = stacking_model.predict_proba(X_test_scaled)[:, 1]   # threshold logic applies the same way here
```

**Key distinctions:**

| Concept | What it does | Import needed? |
|---|---|---|
| `StackingClassifier` | The mechanism that trains base models + a meta-model on top | Yes — real sklearn class |
| Meta-model (e.g. `LogisticRegression` passed to `final_estimator`) | Any regular model, playing the "combiner" role | Yes, but the import is for the model itself, not for "meta" |
| Threshold | A plain number you choose to convert probability → label | No — it's just a variable |
| VotingClassifier (contrast) | Combines base models using fixed, hand-set weights | — |
| StackingClassifier's meta-model (contrast) | *Learns* how to combine base models from data | — |

## Full example: threshold + meta-model together

```python
threshold = 0.5
y_pred_at_threshold = (y_proba >= threshold).astype(int)

import numpy as np
from sklearn.metrics import recall_score, precision_score

for t in np.linspace(0.1, 0.9, 9):
    y_pred_t = (y_proba >= t).astype(int)
    print(f"threshold={t:.1f} -> recall={recall_score(y_test, y_pred_t):.3f}, "
          f"precision={precision_score(y_test, y_pred_t, zero_division=0):.3f}")
```
