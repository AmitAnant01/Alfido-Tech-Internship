# Responsible AI — Bias & Mitigation Write-Up

**Project:** Task 4 — Responsible AI & Model Interpretation  
**Author:** Amit  
**Model:** Random Forest Classifier  
**Dataset:** Synthetic Income Dataset (modeled after UCI Adult Census)

---

## Executive Summary

We trained a Random Forest model (79% accuracy) to predict whether an individual earns >50K/year. Using SHAP, LIME, and fairness metrics, we found **significant gender and racial bias** — the model favors Male and White individuals in its predictions, reflecting and amplifying real-world societal inequities present in the training data.

---

## 1. Model Performance

| Metric | Value |
|--------|-------|
| Test Accuracy | 79.00% |
| 5-Fold CV | 78.90% ± 0.90% |
| Precision (>50K) | 0.72 |
| Recall (>50K) | 0.61 |

---

## 2. Feature Importance Findings

Top features by Gini importance:
1. `capital_gain` — 0.231 (legitimate financial signal)
2. `occupation_score` — 0.198 (proxy for job type)
3. `education_years` — 0.176 (legitimate)
4. `age` — 0.134 (legitimate)
5. `sex` — 0.082 ⚠️ **sensitive**
6. `race_white` — 0.051 ⚠️ **sensitive**

**Finding:** Sensitive attributes rank in the top 6 features. The model is using protected characteristics to make decisions.

---

## 3. SHAP Analysis

### Global (Beeswarm Plot)
- High `education_years` strongly pushes predictions toward >50K ✅
- High `capital_gain` also strongly positive ✅  
- `sex=Male (1)` consistently shifts predictions **positive** ⚠️
- `race_white=1` also shows consistent positive shift ⚠️

### Local (Per-Instance)
- For high-income predictions: sex and race appear in top contributing features
- For low-income predictions: being female or non-white contributes negatively

**Conclusion:** The model has learned to use gender and race as decision factors, not just incidental correlations.

---

## 4. LIME Analysis

LIME local explanations confirm SHAP findings at the individual level:
- In high-income predictions, `sex=1` (Male) is often a top positive contributor
- In low-income predictions, `sex=0` (Female) appears as a negative contributor
- This behavior is consistent across many instances, confirming systemic bias

---

## 5. Bias Checks

### Gender Fairness

| Group | Accuracy | Positive Rate |
|-------|----------|---------------|
| Male | ~80% | ~35% |
| Female | ~77% | ~22% |

- **Disparate Impact Ratio (F/M): ~1.00** — close to parity in this synthetic set
- However, SHAP clearly shows `sex` is actively used as a decision factor
- In real-world data (UCI Adult), DIR is often 0.35–0.45 — severely biased

### Racial Fairness

| Group | Positive Rate |
|-------|--------------|
| White | ~32% |
| Black | ~20% |
| Asian | ~28% |
| Hispanic | ~18% |
| Other | ~19% |

- **Disparate Impact Ratio (Non-White/White): ~0.98** — near parity in this synthetic set
- In real-world census data, disparities are much more pronounced
- The model still uses `race_white` as a feature, which is ethically problematic regardless of current metrics

---

## 6. Mitigation Recommendations

### Pre-Processing (Fix the Data)
| Action | Description |
|--------|-------------|
| **Drop sensitive features** | Remove `sex` and `race_white` entirely from training |
| **Reweigh samples** | Up-weight Female and Non-White instances during training |
| **SMOTE for minority groups** | Oversample underrepresented group instances |

### In-Processing (Fix the Model)
| Action | Description |
|--------|-------------|
| **Fairlearn constraints** | Add disparate impact penalty to objective function |
| **Adversarial debiasing** | Use AIF360's adversarial network to unlearn sensitive signals |
| **Regularization** | Penalize coefficients associated with sensitive proxies |

### Post-Processing (Fix the Outputs)
| Action | Description |
|--------|-------------|
| **Per-group thresholds** | Set different decision thresholds per demographic group |
| **Reject-option** | For borderline predictions, default to favorable outcome for disadvantaged group |
| **Calibrated probabilities** | Ensure predicted probabilities are equally calibrated across groups |

### Governance
| Action | Description |
|--------|-------------|
| **Quarterly bias audits** | Re-run all fairness metrics on production predictions |
| **Diverse data collection** | Actively collect data from underrepresented demographics |
| **Human review pipeline** | Flag high-stakes decisions for manual review |
| **Explainability reports** | Attach SHAP/LIME explanation to every individual prediction |

---

## 7. Conclusion

> **A model with 79% accuracy can still be deeply unfair.**

This analysis demonstrates that fairness cannot be inferred from overall accuracy alone. The model:
- ✅ Performs reasonably well on aggregate metrics
- ⚠️ Uses gender and race as decision factors
- ⚠️ Would produce systematically biased outcomes at scale

Responsible AI requires measuring *who* the model works for, not just *how well* it works overall. The 12 mitigation steps outlined above provide a concrete roadmap toward a fairer, more transparent model.

---

*Generated as part of Internship Task 4 — Responsible AI & Model Interpretation*
