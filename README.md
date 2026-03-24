# Observational Causal Inference: PSM & IPTW

Causal inference methods applied to observational data, using the classic **LaLonde (1986) job training dataset** to estimate the Average Treatment Effect (ATE) of a job training program on 1978 earnings (`re78`).

## Dataset

The LaLonde dataset (`lalonde` from the `cobalt`/`MatchIt` packages) contains 614 subjects with the following variables:

| Variable | Description |
|---|---|
| `treat` | Treatment indicator (1 = received job training) |
| `age` | Age in years |
| `educ` | Years of education |
| `race` | Race (black, hispanic, other) |
| `married` | Married (1/0) |
| `nodegree` | No high school degree (1/0) |
| `re74` | Real earnings in 1974 (pre-treatment) |
| `re75` | Real earnings in 1975 (pre-treatment) |
| `re78` | Real earnings in 1978 (outcome) |

The treated (n=185) and control (n=429) groups are substantially imbalanced at baseline — most notably on race (SMD=1.67), married status (SMD=0.72), and pre-treatment earnings.

## Methods

### 1. Propensity Score Matching (PSM)

Propensity scores are estimated via logistic regression on 8 confounders: age, education, race (black/hispanic dummies), married, nodegree, re74, re75.

Two matching strategies are compared:
- **No caliper** — nearest-neighbor 1:1 matching without replacement (185 pairs)
- **Caliper = 0.1** — restricts matches within 0.1 PS units (111 pairs), improving balance at the cost of sample size

Balance is assessed with standardized mean differences (SMD) before and after matching, and visualized with a Love plot (`cobalt`).

### 2. Inverse Probability of Treatment Weighting (IPTW)

Each subject is weighted by the inverse probability of their observed treatment:
- Treated: `w = 1 / PS`
- Control: `w = 1 / (1 - PS)`

This creates a pseudo-population where confounders are balanced across treatment groups. The causal effect is estimated via a weighted regression (`svyglm`).

**Truncated IPTW** caps weights at the 1st/99th percentiles (reducing from range 1.0–40.1 to 1.0–12.6) to limit the influence of extreme weights.

### 3. Sensitivity Analysis

Rosenbaum bounds (`rbounds`) assess how strong unmeasured confounding would need to be to invalidate the estimated effect.

## Results

| Method | Estimate (re78) | 95% CI |
|---|---|---|
| Unadjusted | −$635 | — |
| PSM (no caliper) | +$927 | (−422, 2,277) |
| PSM (caliper=0.1) | +$1,247 | (−420, 2,914) |
| IPTW | +$225 | (−1,563, 2,012) |
| IPTW (truncated) | +$487 | (−1,094, 2,068) |

The unadjusted estimate is negative due to confounding (the training program enrolled lower-earning individuals). After adjustment, all methods point to a positive effect of job training on 1978 earnings, though confidence intervals are wide and include zero.

## R Libraries

```r
library(tableone)   # Table 1 with SMD
library(Matching)   # PSM via Match()
library(MatchIt)    # lalonde dataset
library(cobalt)     # Love plots, balance diagnostics
library(survey)     # Weighted regression (svyglm)
library(ipw)        # IPTW utilities
library(rbounds)    # Rosenbaum sensitivity bounds
```

## Notebook

[`causality_from_observations_psm_iptw.ipynb`](causality_from_observations_psm_iptw.ipynb)
