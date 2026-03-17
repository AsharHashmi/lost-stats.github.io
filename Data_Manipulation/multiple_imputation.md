# Data Manipulation Technique - Multiple Imputation

## Introduction
Multiple imputation is a method for handling missing data by replacing each missing value with several plausible values instead of only one. This produces multiple completed datasets. The analyst then estimates the same model on each completed dataset and combines the estimates using standard pooling rules. The main advantage is that multiple imputation reflects uncertainty about the missing values instead of pretending that the imputed values are known with certainty.

In applied econometrics, multiple imputation is useful when dropping incomplete observations would substantially reduce sample size or create concerns about selection. It is often used in household surveys, labor-force data, education data, and administrative microdata, where variables such as income, consumption, test scores, and parental background may be missing for part of the sample.

## Keep in Mind

- Multiple imputation is usually justified when the missing-data process is plausibly missing at random (MAR), meaning that after conditioning on observed variables, missingness does not depend on the unobserved value itself.
- A good imputation model should include variables that predict both the missingness and the missing values.
- In applied work, the imputation model should usually be at least as rich as the final analysis model. If the final regression includes transformations, interactions, or nonlinear terms, those should be handled carefully in the imputation step as well.
- Do not treat imputed values as if they were true observed values. The purpose of multiple imputation is to carry uncertainty through to the final estimates.
- Multiple imputation is not a cure-all. If data are Missing Not at Random (MNAR), multiple imputation under MAR may still produce biased results.

