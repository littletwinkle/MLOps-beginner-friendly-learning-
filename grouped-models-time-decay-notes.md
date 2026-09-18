# Grouped Models & Time Decay

## 1. Feature pipeline (diet + time columns)

Standard preprocessing before any grouping/decay logic — encode the categorical column, scale numeric ones:

```python
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

categorical_features = ["diet"]
numeric_features = ["time_month", "feature1", "feature2"]

preprocessor = ColumnTransformer(transformers=[
    ("cat", OneHotEncoder(handle_unknown="ignore"), categorical_features),
    ("num", StandardScaler(), numeric_features),
])

clf = Pipeline(steps=[
    ("preprocess", preprocessor),
    ("model", LogisticRegression()),
])
```

## 2. Grouped modeling — one model per category (e.g. per diet)

**Manual approach:**

```python
models_by_diet = {}

for diet_val in df["diet"].unique():
    subset = df[df["diet"] == diet_val]
    X = subset[["time_month", "feature1", "feature2"]]
    y = subset["target"]

    pipe = Pipeline(steps=[("scaler", StandardScaler()), ("model", LogisticRegression())])
    pipe.fit(X, y)
    models_by_diet[diet_val] = pipe

# Prediction routes to the matching model
def predict_for_sample(row, models_by_diet):
    diet_val = row["diet"]
    model = models_by_diet.get(diet_val, list(models_by_diet.values())[0])  # fallback if unseen
    X_new = row[["time_month", "feature1", "feature2"]].values.reshape(1, -1)
    return model.predict(X_new)[0]
```

**Automated approach — `GroupedPredictor` (scikit-lego, a separate library):**

```python
# pip install scikit-lego
from sklego.meta import GroupedPredictor

grouped_model = GroupedPredictor(estimator=clf, groups=["diet"])
grouped_model.fit(X, y, groups=df[["diet"]])
grouped_model.predict(X_new, groups=groups_new)
```

## 3. Time decay — weighting recent data more

**Not `DummyClassifier`.** `DummyClassifier` is a baseline model (`strategy="most_frequent"`, `"stratified"`, `"uniform"`, `"constant"`) with no decay parameter — it's unrelated to time-weighting.

**Manual sample_weight approach:**

```python
import numpy as np

T = df["time_idx"].max()
decay_factor = 0.9   # closer to 1 = gentler decay, closer to 0 = aggressive

df["sample_weight"] = decay_factor ** (T - df["time_idx"])

pipe.fit(X, y, sample_weight=df["sample_weight"])
```

**Automated — `DecayEstimator` (scikit-lego):**

```python
from sklego.meta import DecayEstimator

df_sorted = df.sort_values("time_idx")   # must sort by time first
decay_model = DecayEstimator(LogisticRegression(), decay=0.9)
decay_model.fit(df_sorted[["feature1", "feature2"]].values, df_sorted["target"].values)
```

## 4. Grouping by month vs. time decay — different strategies

| Approach | What it does | Drawback |
|---|---|---|
| Group by month (`groups=["diet", "month"]`) | Separate model per diet+month combo | Many small models, possible data scarcity |
| Time decay (`sample_weight` or `DecayEstimator`) | One model per diet, recent rows count more | Risk of distribution shift if train/test periods mismatch |

## 5. Train/test distribution risk

If decay is aggressive, the model is effectively trained on mostly recent data. If your evaluation set spans older periods too, performance may look worse than expected — not because the model is bad, but because train and test distributions no longer match.

**Best practice:** use time-based train/test splits (train on older data, test on newer data) and tune the decay factor using out-of-time validation, rather than guessing a value.

## Quick mental model

- **Grouping** → different models for different categories
- **Decay** → recent rows matter more within a model
- **DummyClassifier** → a naive baseline, has nothing to do with either of the above
- Both grouping and decay can be combined: a `GroupedPredictor` where each group's base model is a `DecayEstimator`
