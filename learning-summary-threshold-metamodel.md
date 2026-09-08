# Learning Summary — Threshold & Meta-Models (Personal Reference)

**Decision Threshold**
- A classifier like LogisticRegression outputs a *probability*, not a direct label.
- `predict_proba(X)[:, 1]` gives the probability of the positive class (e.g. "diabetic").
- The threshold is the cutoff you choose to convert that probability into a label:
  `y_pred = (y_proba >= threshold).astype(int)`
- Default is 0.5, but it's just a number you pick — not a library object, not something to import.
- Lowering the threshold → catches more positives (higher recall), but more false alarms (lower precision). Raising it does the opposite.
- Correction made this session: recall_score is **not** computed per patient — it's one aggregate score across the whole test set, computed *after* labels are assigned via the threshold.

**Meta-Model (Stacking)**
- A meta-model is just a regular model (e.g. LogisticRegression) assigned to `final_estimator=` inside `StackingClassifier`.
- It doesn't see the raw features — it learns from the *predictions* of the base models (e.g. LogisticRegression, RandomForest, KNN).
- Different from VotingClassifier (Week 7): voting uses fixed, hand-set weights; stacking's meta-model *learns* how much to trust each base model.
- Correction made this session: there is no `import meta` — "meta-model" is a role, not a class. Whatever model you plug into `final_estimator` needs its own normal import (e.g. `from sklearn.linear_model import LogisticRegression`), same as any base model.
- Threshold logic applies identically on top of a meta-model's `predict_proba()` output — no difference in mechanics, just applied to the stacked model's final probability instead of a single model's.

**General import rule confirmed twice this session:**
Import when it's a real sklearn class/function (`LogisticRegression`, `StackingClassifier`, `recall_score`). Don't import when it's a plain value you're choosing yourself (`threshold = 0.5`) or a conceptual role rather than an object (`meta-model`).
