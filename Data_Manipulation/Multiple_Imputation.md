
# Multiple Imputation

## Introduction

Multiple imputation is a method for handling missing data by replacing each missing value with several plausible values instead of only one. This produces multiple completed datasets. The analyst then estimates the same model on each completed dataset and combines the estimates using standard pooling rules. The main advantage is that multiple imputation reflects uncertainty about the missing values instead of pretending that the imputed values are known with certainty.

In applied econometrics, multiple imputation is useful when dropping incomplete observations would substantially reduce sample size or create concerns about selection. It is often used in household surveys, labor-force data, education data, and administrative microdata, where variables such as income, consumption, test scores, and parental background may be missing for part of the sample.

## Keep in Mind

- Multiple imputation is usually justified when the missing-data process is plausibly missing at random (MAR), meaning that after conditioning on observed variables, missingness does not depend on the unobserved value itself.
- A good imputation model should include variables that predict both the missingness and the missing values.
- In applied work, the imputation model should usually be at least as rich as the final analysis model. If the final regression includes transformations, interactions, or nonlinear terms, those should be handled carefully in the imputation step as well.
- Do not treat imputed values as if they were true observed values. The purpose of multiple imputation is to carry uncertainty through to the final estimates.
- Multiple imputation is not a cure-all. If data are missing not at random (MNAR), multiple imputation under MAR may still produce biased results.

## Why Econometricians Use It

Suppose we want to estimate the relationship

$$
wage_i = \beta_0 + \beta_1 education_i + \beta_2 experience_i + \beta_3 female_i + u_i
$$

But some observations are missing `wage` or `education`. If we simply drop all incomplete cases, we may lose many observations and change the composition of the sample. Multiple imputation replaces the missing values several times using models based on the observed data, runs the wage regression on each completed dataset, and then combines the results.

This is often preferable to ad hoc single-imputation methods because the final standard errors incorporate both within-imputation variation and between-imputation variation.

## A Small Dataset Example 

Below is a small dataset example for illustration.

| id | wage | education | experience | female | urban |
|---:|----:|----------:|-----------:|-------:|------:|
| 1  | 4200 | 16 | 5  | 1 | 1 |
| 2  | 3800 | 14 | 7  | 0 | 1 |
| 3  |      | 12 | 10 | 1 | 0 |
| 4  | 5100 | 18 | 6  | 0 | 1 |
| 5  | 2950 |    | 12 | 1 | 0 |
| 6  | 4700 | 16 | 8  | 0 | 1 |
| 7  | 3300 | 13 |    | 1 | 0 |
| 8  |      | 15 | 9  | 0 | 1 |
| 9  | 2800 | 11 | 14 | 1 | 0 |
| 10 | 5600 | 18 | 7  | 0 | 1 |

In this dataset, `wage`, `education`, and `experience` contain missing values. A reasonable imputation model might use the other observed variables to predict the missing ones.

## General Workflow

A practical workflow for multiple imputation is:

1. Inspect the amount and pattern of missingness.
2. Decide which variables belong in the imputation model.
3. Choose the number of imputations \(m\).
4. Generate \(m\) completed datasets.
5. Estimate the same econometric model on each dataset.
6. Pool the estimates and standard errors.

In many applications, chained equations are used. Under this approach, each incomplete variable is imputed with its own conditional model, such as linear regression for continuous variables or logistic regression for binary variables.

## Choosing Variables for the Imputation Model

A common mistake is to impute using only a few variables. In applied econometrics, the imputation model should usually include:

- All variables in the final estimating equation,
- Variables that predict missingness,
- Variables strongly correlated with incomplete variables,
- Grouping or design variables when relevant.

For example, if the final model regresses wages on education and experience, then those variables should appear in the imputation model. If urban residence predicts whether wages are missing, then `urban` should also be included.

## Interpreting the Output

After imputation, you do not interpret the imputed values themselves as substantive results. The key outputs are the pooled regression coefficients, pooled standard errors, confidence intervals, and test statistics. These reflect uncertainty from both sampling variation and missing-data imputation.

## Implementation

### Python

The example below uses `statsmodels`' MICE implementation. It creates an artificial dataset directly in code, imputes the missing values, estimates a wage regression, and pools the results.

```python
# Install if needed:
# pip install pandas statsmodels numpy

import numpy as np
import pandas as pd
import statsmodels.api as sm
from statsmodels.imputation.mice import MICEData, MICE

# Create a small artificial dataset
data = pd.DataFrame({
    "wage":       [4200, 3800, np.nan, 5100, 2950, 4700, 3300, np.nan, 2800, 5600],
    "education":  [16,   14,   12,     18,   np.nan, 16,   13,   15,    11,   18],
    "experience": [5,    7,    10,     6,    12,     8,    np.nan, 9,   14,   7],
    "female":     [1,    0,    1,      0,    1,      0,    1,    0,     1,    0],
    "urban":      [1,    1,    0,      1,    0,      1,    0,    1,     0,    1]
})

# Inspect missingness
print(data.isna().sum())

# Set up the MICE data object
imp_data = MICEData(data)

# Estimate the analysis model on multiply imputed datasets
# Formula: wage on education, experience, female, and urban
model = MICE(
    "wage ~ education + experience + female + urban",
    sm.OLS,
    imp_data
)

# Fit using 10 burn-in iterations and 5 imputations
results = model.fit(n_burnin=10, n_imputations=5)

print(results.summary())
```

### What this code does

- MICEData(data) prepares the incomplete dataset for chained-equation imputation.

- MICE(...) specifies the econometric model to estimate after imputation.

- fit(n_burnin=10, n_imputations=5) carries out the iterative imputation process and pools the final regression results.

### R
In R, the mice package is the standard implementation for multiple imputation by chained equations. The example below uses predictive mean matching for continuous variables, which is often attractive in applied work because it preserves plausible observed values.

```
install.packages("mice")
library(mice)

# Create a small artificial dataset
data <- data.frame(
  wage = c(4200, 3800, NA, 5100, 2950, 4700, 3300, NA, 2800, 5600),
  education = c(16, 14, 12, 18, NA, 16, 13, 15, 11, 18),
  experience = c(5, 7, 10, 6, 12, 8, NA, 9, 14, 7),
  female = c(1, 0, 1, 0, 1, 0, 1, 0, 1, 0),
  urban = c(1, 1, 0, 1, 0, 1, 0, 1, 0, 1)
)

# Inspect missingness
colSums(is.na(data))

# Run multiple imputation
# m = number of imputations
# method = pmm for continuous variables
imp <- mice(data, m = 5, method = "pmm", seed = 123)

# Estimate the regression on each imputed dataset
fit <- with(imp, lm(wage ~ education + experience + female + urban))

# Pool results using Rubin's rules
pooled <- pool(fit)

summary(pooled)
```

### What this code does

- mice(...) creates multiple completed datasets.

- with(...) fits the same linear model to each imputed dataset.

- pool(...) combines the estimates and standard errors.
 
