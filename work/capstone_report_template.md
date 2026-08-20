# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Sushmita Mishra
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/06sushmita/flyrank-ML-Internship.git
- **Date:** August 20, 2026

---

## 1. Problem framing

### Research question

Can search performance, content characteristics, freshness, and engagement signals be used to identify and rank content that is likely to be declining?

### Decision supported

This project supports the decision of **which content pages should be reviewed first** when editorial review capacity is limited.

The unit of analysis is an individual content page.

The final model produces a predicted probability of the positive `is_declining_label` and uses that score to create a ranked content-review queue.

A human editor can use the ranking to decide which pages deserve investigation first.

The possible actions include:

- Review first
- Investigate for a possible refresh
- Investigate performance signals
- Monitor

A false positive can result in editorial time being spent reviewing content that does not require immediate attention. A false negative can result in declining content receiving lower review priority.

Machine learning is useful here because reviewing every page manually does not scale. A ranking model can help concentrate limited review capacity on the pages that appear most relevant for investigation.

The model is therefore a **decision-support tool**, not an automatic content-refresh system.

---

## 2. Data safety

This analysis uses the anonymized FlyRank ML Internship dataset and follows the public-safe workflow provided by the starter repository.

The analysis uses aggregated search-performance, content, freshness, and engagement signals.

### Numeric features

The final model uses:

- `search_volume`
- `competition`
- `cpc`
- `word_count`
- `char_count`
- `log_impressions_90d`
- `log_clicks_90d`
- `log_sessions_90d`
- `log_ai_sessions_90d`
- `days_with_impressions`
- `days_with_sessions`
- `content_age_days`
- `days_since_last_update`
- `ctr`
- `avg_position`
- `engagement_rate`
- `scroll_rate`
- `ai_traffic_pct`

### Categorical features

- `competition_level`
- `content_type`
- `main_intent`
- `age_tier`
- `freshness_tier`
- `word_count_tier`
- `impression_tier`
- `position_tier`

### Excluded fields and leakage checks

Pseudonymous identifiers such as `client_id` and `content_id` were not used as predictive features. They are identifiers for grouping or reporting only.

`trend_direction` and `trend_pct` were considered potential leakage risks because they are directly related to performance movement. They were not included in the final model feature list.

Client names, domains, URLs, private search queries, credentials, and raw private exports were excluded from the public-facing work.

No client-identifying information is intentionally included in the capstone artifacts.

---

## 3. Baseline

The project first evaluated the transparent baseline rule provided by the starter workflow.

The baseline provides a simple and interpretable reference for determining whether machine learning improves the prioritization of pages for review.

The baseline and learned models were evaluated on the same held-out test set.

### Baseline results

| Metric | Baseline |
|---|---:|
| Precision@50 | 0.24 |
| ROC AUC | 0.627 |
| Average Precision | 0.468 |
| Accuracy | 0.609 |
| F1 | 0.274 |
| Recall | 0.189 |

The test-set positive rate was approximately **54.2%**.

Precision@50 is the primary decision metric because the intended use case is prioritizing a small number of pages for review.

The baseline achieved **24% Precision@50**.

---

## 4. Model / analysis

### Target

The target variable is:

`is_declining_label`

The target represents whether a content item satisfies the predefined decline criteria used by the feature-preparation pipeline.

### Models evaluated

Three supervised classification models were compared:

1. Logistic Regression
2. Decision Tree
3. Random Forest

The **Random Forest** was selected as the final model because it achieved the strongest overall ranking performance.

### Final model features

The model combines search-demand, content, historical performance, freshness, visibility, engagement, and AI-traffic signals.

The numeric features are:

`search_volume`, `competition`, `cpc`, `word_count`, `char_count`, `log_impressions_90d`, `log_clicks_90d`, `log_sessions_90d`, `log_ai_sessions_90d`, `days_with_impressions`, `days_with_sessions`, `content_age_days`, `days_since_last_update`, `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`, `ai_traffic_pct`.

The categorical features are:

`competition_level`, `content_type`, `main_intent`, `age_tier`, `freshness_tier`, `word_count_tier`, `impression_tier`, `position_tier`.

Pseudonymous IDs and label-derived trend fields were deliberately excluded from the predictive feature set.

---

## 5. Evaluation

### Validation design

The model uses a **client-holdout split**.

- Training rows: **27,675**
- Test rows: **2,325**
- Test positive rate: **54.2%**

The client-holdout design evaluates whether the learned patterns generalize to held-out clients rather than simply memorizing patterns from the same clients.

### Model comparison

All models were evaluated on the same held-out test set.

| Model | Precision@50 | ROC AUC | Average Precision |
|---|---:|---:|---:|
| Baseline | 0.24 | 0.627 | 0.468 |
| Logistic Regression | 0.40 | 0.700 | 0.522 |
| Decision Tree | 0.58 | 0.742 | 0.575 |
| **Random Forest** | **0.74** | **0.750** | **0.618** |

Random Forest achieved the strongest result.

Its Precision@50 was **0.74**, compared with **0.24** for the baseline.

This is an absolute improvement of **0.50** in Precision@50.

### Error analysis

The main errors are false positives and false negatives.

A false positive means a page receives a high predicted decline probability but does not satisfy the decline label.

A false negative means a declining page receives a lower ranking score and may therefore receive less review priority.

Because the practical objective is prioritization, Precision@50 is emphasized over accuracy alone.

The model is not expected to classify every page perfectly. Its purpose is to improve the ordering of limited editorial review capacity.

---

## 6. Interpretation

The Random Forest model showed measurable predictive signal across the search, content, freshness, visibility, and engagement features used in the analysis.

The final model achieved:

- **Precision@50: 0.74**
- **ROC AUC: 0.750**
- **Average Precision: 0.618**

Compared with the baseline Precision@50 of 0.24, the model concentrated substantially more positive examples near the top of the review ranking on the held-out test set.

### Model interpretation

The final Random Forest produces a probability associated with the positive `is_declining_label`.

The evaluation pipeline then uses this model output to create a ranked recommendation queue containing:

- `final_rank`
- `best_model_probability`
- `confidence`
- `suggested_action`
- `final_reason_codes`

These fields make the model output more useful for human review.

The model identifies associations in the available data. These associations should not be interpreted as causal effects.

In particular, the results do not establish that changing a specific feature, refreshing a page, or changing metadata will improve future search performance.

---

## 7. Recommendation

The final Random Forest can be used as a **content-review prioritization system**.

### Action framework

#### Review first

Pages with high predicted decline probability should receive higher review priority.

#### Refresh investigation

Pages with high decline probability and evidence of stale content can be investigated for a possible refresh.

#### Performance investigation

Pages with high decline probability and weakening search or engagement signals can be investigated further.

#### Monitor

Lower-risk pages can receive lower immediate review priority.

### How an editor could use the queue

A FlyRank editor could begin with the highest-ranked pages, inspect the associated reason codes and underlying signals, and then decide whether further investigation or a refresh is appropriate.

The model score should not be treated as an automatic action.

A high score does not prove that a refresh will improve future performance.

The recommendations are therefore **decision-support**, with final editorial decisions remaining with a human reviewer.

---

## 8. Reproducibility

The capstone follows the workflow provided by the FlyRank ML Internship starter repository.

The analysis consists of:

1. Feature preparation and label definition.
2. Baseline scoring.
3. Training Logistic Regression, Decision Tree, and Random Forest models.
4. Evaluation using the client-holdout split.
5. Generation of the ranked content-review queue.
6. Generation of comparison tables and charts.
7. Publication of the final research paper.

### Important artifacts

The analysis produces:

```text
outputs/model_results.json
outputs/refresh_queue_sample.csv
outputs/model_comparison.csv
outputs/ranked_recommendations.csv
outputs/charts/precision_at_50.png
outputs/charts/roc_auc.png
outputs/charts/average_precision.png
outputs/charts/suggested_actions.png
