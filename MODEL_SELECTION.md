# Model selection

## The question
The model predicts whether a customer will churn. A false negative could mean we miss a customer at risk of leaving; a false positive could lead to an unnecessary retention offer.

## Candidates
Results are from MLflow batch `20261121233402`, using the same train/test split and data hash `76a6447e764b`.

| Candidate | Why I tried it | AUC | F1 | F1 per slice (lowest) | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---:|---:|---|---:|---:|---|
| Logistic Regression | Fast, compact linear baseline | 0.798 | 0.495 | one-year 0.250; two-year 0.250; month-to-month 0.543 (lowest: 0.250) | 2.7 | 4 | `d5b7a4f6f8134dccb1d96585214ce164` |
| Random Forest | Tests nonlinear patterns with multiple trees | 0.759 | 0.454 | one-year 0.222; two-year 0.364; month-to-month 0.493 (lowest: 0.222) | 9.9 | 3160 | `d220706ddeff4af19b3c45b66fd15974` |
| Gradient Boosting | Tests sequential trees that correct earlier errors | 0.748 | 0.417 | one-year 0.444; two-year 0.222; month-to-month 0.436 (lowest: 0.222) | 2.7 | 142 | `9e4d1c65b37a47ea90f4b69a4b6f2b17` |

## The champion
I picked Logistic Regression. It had the best overall F1 (0.495) and AUC (0.798), and its weakest slice F1 (0.250) was higher than the other candidates' weakest slice (0.222). It also had the lowest latency (2.7 ms) and smallest size (4 KB). I accepted that Random Forest did better for two-year contracts (0.364 vs 0.250), because it was worse overall, worse on its weakest slice, slower, and much larger.

## My promotion rules
- Latency budget (p95): 50 ms, because the slowest local candidate measured 9.9 ms and the five-times headroom allows for slower CI runners while keeping the API responsive.
- Minimum gain to replace the champion (F1 points): 0.005, because a replacement must improve absolute F1 by at least half a percentage point rather than promoting on a negligible difference.

## What I would check next
I would check how many customers are in each contract slice and measure performance on a later time period. A small slice or changing customer behavior could make these test results less representative of production.
