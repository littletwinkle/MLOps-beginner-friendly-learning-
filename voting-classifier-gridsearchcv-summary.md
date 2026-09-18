# VotingClassifier + GridSearchCV — Summary

*Notes from debugging a VotingClassifier + decision boundary plotting session — churn model portfolio*

---

## VotingClassifier

- An **ensemble model** that combines predictions from multiple already-trained classifiers (e.g., `clf1`, `clf2`) into one combined model (`clf3`)
- `weights` parameter controls how much influence each individual model has on the final combined prediction
- `voting='hard'` — majority vote on predicted class labels (default; works with any classifier)
- `voting='soft'` — averages predicted probabilities (requires all estimators to support `predict_proba()`)

```python
clf3 = VotingClassifier(
    estimators=[('clf1', clf1), ('clf2', clf2)],
    weights=[0.5, 0.5],
    voting='soft'
)
clf3.fit(x, Y)
```

## GridSearchCV

- A **hyperparameter tuning tool** — doesn't create models, searches across parameter combinations for a model you already built, using cross-validation
- Can tune a `VotingClassifier`'s `weights` and `voting` settings automatically, instead of manually trying combinations by hand

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'weights': [[0.1, 0.9], [0.5, 0.5], [0.9, 0.1]],
    'voting': ['soft', 'hard']
}

grid = GridSearchCV(clf3, param_grid, cv=5)
grid.fit(x, Y)
print(grid.best_params_)
```

## How VotingClassifier, Pipeline, and GridSearchCV relate

- `VotingClassifier` = the model itself (an ensemble of fixed, already-chosen models)
- `Pipeline` = bundles steps (preprocessing → model) into one object
- `GridSearchCV` = tunes parameters — works **together with** a pipeline, not as a replacement for one. It can tune a `VotingClassifier` whether that classifier sits inside a pipeline or stands alone.

**What GridSearchCV actually searches:** the weight given to each already-selected model, and settings like `voting` — not new model types. The models (KNN, Logistic Regression, etc.) are fixed once defined; only their combination is tuned.

## Why this matters for MLOps pipelines

Automating the search for the best ensemble configuration (via `GridSearchCV`) instead of hardcoding weights based on guesswork fits naturally into an automated training/retraining pipeline — the same principle that applies to any hyperparameter tuning step in a production ML workflow.

---

## Decision boundary visualization (for reference)

Two-feature-only limitation: decision boundary plots require exactly 2 input features. If your data has more, either slice to 2 columns or reduce with PCA before fitting/plotting.

**Simplified plotting (mlxtend, matches common tutorial style):**
```python
from mlxtend.plotting import plot_decision_regions
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 3, figsize=(15, 4))
for clf, title, ax in zip([clf1, clf2, clf3],
                           ['Classifier 1', 'Classifier 2', 'Voting Classifier'],
                           axes):
    plot_decision_regions(x, Y, clf=clf, ax=ax)
    ax.set_title(title)
plt.show()
```
