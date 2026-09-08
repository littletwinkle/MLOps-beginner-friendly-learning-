# MLOps Learning Journey 🚀

A structured, hands-on roadmap from Python fundamentals to a deployable MLOps project.
Built from real Kaggle notebooks — every example was written and tested in a live environment.

## 👤 About

I'm transitioning into MLOps and Data Science, documenting my learning publicly to build a practical portfolio.
This repo tracks my progress week by week — from NumPy basics to a fully deployed ML pipeline.

## 🗺 Roadmap

| Week | Topic | Status |
|---|---|---|
| 1–2 | NumPy — data layer foundations | ✅ Complete |
| 3–4 | Pandas — structured data & cleaning | ✅ Complete |
| 5 | scikit-learn — preprocessing, scaling, class imbalance & model tuning | ✅ Complete |
| 6 | scikit-learn — QuantileTransformer, GridSearchCV, Pipelines | ✅ Complete |
| 7 | scikit-learn — VotingClassifier (soft voting, weighted estimators) | ✅ Complete |
| 8 | scikit-learn — decision thresholds, StackingClassifier (meta-model) | ✅ Complete |
| 9 | Flask / FastAPI — wrap model in an API | ⏳ Upcoming |
| 10 | Docker — containerise the API | ⏳ Upcoming |
| 11 | CI/CD + Deployment — GitHub Actions + Render/Railway | ⏳ Upcoming |

## 📚 Course Notes

Self-written beginner guides with tester, data analyst, and MLOps perspectives on every topic.

| File | Description |
|---|---|
| `numpy_mlops_course.md` | Array creation, reshaping, boolean filtering, broadcasting, save/load + mini project |
| `pandas_mlops_course.md` | DataFrames, null handling, groupby, merging, encoding + ML-ready pipeline project |
| `categorical-encoding-notes.md` | OrdinalEncoder vs OneHotEncoder, rare/unknown category handling, `min_frequency` vs `max_categories`, NumPy slicing + `plt.scatter` refresher |
| `scaling-and-model-selection-notes.md` | QuantileTransformer (non-linear, rank/percentile-based scaling, robust to outliers vs StandardScaler), GridSearchCV (hyperparameter search + cross-validation against a scoring metric), model selection reasoning for Linear Regression |
| `gridsearchcv-parameters-notes.md` *(new)* | Full GridSearchCV parameter reference: `estimator`, `param_grid`, `cv`, `n_jobs`, `refit`, `verbose`, plus `scoring`, `error_score`, `return_train_score`, `pre_dispatch` |
| `sample-weight-and-numpy-notes.md` | sample_weight vs. class_weight (per-row vs. per-category importance), `make_scorer` for custom metrics inside GridSearchCV, `np.linspace`, `np.log`/`np.log1p` |
| `voting-classifier-notes.md` | VotingClassifier — combining multiple estimators, `voting="soft"` (averaged predicted probabilities) vs `voting="hard"` (majority vote), `weights` parameter for weighting each estimator's influence |
| `threshold-in-scikit-learn.md` | Decision thresholds: `predict_proba` vs `decision_function` defaults, precision/recall trade-off, `precision_recall_curve`, `roc_curve`, `TunedThresholdClassifierCV` |
| `threshold-and-metamodel-notes.md` | Applying decision thresholds on top of a `StackingClassifier` meta-model; meta-model (learns to combine base models) vs. VotingClassifier (fixed, hand-set weights) |

## 🗂 Repo Structure

```
MLOps beginner friendly learning
│
├── README.md
│
├── courses/
│   ├── numpy_mlops_course.md
│   ├── pandas_mlops_course.md
│   ├── categorical-encoding-notes.md
│   ├── scaling-and-model-selection-notes.md
│   ├── gridsearchcv-parameters-notes.md
│   ├── sample-weight-and-numpy-notes.md
│   ├── voting-classifier-notes.md
│   ├── threshold-in-scikit-learn.md
│   └── threshold-and-metamodel-notes.md
│
├── notebooks/
│   ├── numpy_practice.ipynb
│   ├── pandas_practice.ipynb
│   └── scikit-learn practice → see Kaggle link below
│
└── projects/
    └── (deployable project coming in Week 11)
```

## 🛠 Tools & Environment

- Kaggle Notebooks — free GPU, pre-installed libraries
- Python 3 — NumPy, Pandas, scikit-learn, Flask/FastAPI
- Docker — containerisation
- GitHub Actions — CI/CD automation

## 📓 Kaggle Notebooks

- scikit-learn — preprocessing, encoding, scaling, class imbalance handling (sample_weight/class_weight), hyperparameter tuning with GridSearchCV, VotingClassifier (soft voting, weighted estimators), decision thresholds & StackingClassifier meta-models
  🔗 https://www.kaggle.com/code/kannammaivr/scikit-learn

## 🔗 Connect

LinkedIn: https://www.linkedin.com/in/kannammai-v-r-03a498239

*Updated weekly. Follow along if you're on a similar journey.*
