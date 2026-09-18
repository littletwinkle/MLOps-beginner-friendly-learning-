# Learning Summary — Grouped Models (per-diet) & Time Decay (Personal Reference)

**Goal:** one model per category (e.g. per diet type), where recent data matters more than old data.

**Feature pipeline (unrelated to grouping/decay — just standard preprocessing):**
- `OneHotEncoder` on the diet column (categorical → binary columns)
- `StandardScaler` on numeric/time features
- Combined via `ColumnTransformer`, wrapped in a `Pipeline`

**Grouped modeling (one model per diet) — two approaches:**
1. Manual: loop over each unique diet value, filter the data, fit a separate pipeline per group, store in a dict keyed by diet value. At prediction time, look up the right model by the new row's diet value.
2. `GroupedPredictor` from **scikit-lego** (a separate library, not core sklearn — `pip install scikit-lego`) — does the same thing automatically: pass `groups=["diet"]`, it fits one estimator per unique group value and routes new predictions automatically.

**Time decay (recent data weighted more) — corrected concept:**
- This is NOT done via `DummyClassifier`. `DummyClassifier` is a baseline model with simple strategies (`most_frequent`, `stratified`, `uniform`, `constant`) — it has no decay parameter, and no "DK" setting.
- Correct approach: compute a `sample_weight` array where weight decays the further back in time a sample is, using something like `weight = decay_factor ** (latest_time - sample_time)`. Pass this into `.fit(X, y, sample_weight=weight)`.
- Or use `DecayEstimator` from scikit-lego, which does this automatically — just sort data by time first, then wrap: `DecayEstimator(base_model, decay=0.9)`.
- Decay factor closer to 1 (e.g. 0.95–0.99) = gentle decay, keeps more historical influence. Closer to 0 = aggressive, model mostly sees only recent data.

**Grouping by month vs. time decay — two different strategies, not the same thing:**
- Grouping by month = a separate model per month (or per diet+month combo) — can lead to too many small models with sparse data.
- Time decay = one model per diet, but recent rows count more during training — usually the more practical choice.

**Risk I identified myself, confirmed as valid:** if decay is too aggressive, the model is trained mostly on recent data. If evaluation also uses only recent data, that's consistent — but if train/test time periods don't match, you get distribution shift and misleading performance. Fix: use time-based train/test splits (train on older data, test on newer), and tune the decay factor using an out-of-time validation set rather than guessing.

**Key correction to remember:** grouping (per diet/month) and time-weighting (recency) are two separate, combinable techniques — neither one is "the metamodel" on its own, and `DummyClassifier` is unrelated to either.
