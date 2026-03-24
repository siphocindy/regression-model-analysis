# regression-model-analysis
YNB Regression Model


# Time-Based Applicant Volume Modeling Using Linear Regression (R)

## Overview

This project implements a time-series–inspired linear regression model to quantify how job application volume changes over time following a job posting. The objective is to statistically evaluate whether applicant engagement decays, stabilizes, or increases as time progresses.

Rather than relying on descriptive summaries alone, this analysis formalizes the relationship using Ordinary Least Squares (OLS) regression.

---

## Problem Framing

Recruitment teams often assume applicant volume declines after initial posting, but this assumption is rarely quantified.

This project answers:

> **Does application volume significantly change as a function of time since posting?**

Formally:

[
Applications_t = \beta_0 + \beta_1(Time_t) + \epsilon_t
]

Where:

* ( Applications_t ) = number of applications received on day *t*
* ( Time_t ) = days since initial posting
* ( \beta_1 ) measures the rate of change in applicant volume
* ( \epsilon_t ) = stochastic error term

---

## Dataset

Source: Excel export of applicant submissions

Raw features included:

* `Created Time` (timestamp)
* Job title
* Application stage
* Geographic information

Feature engineering steps:

* Converted timestamps to Date objects
* Aggregated observations to daily counts
* Engineered numeric time index (`days_since_posting`)

Unit of analysis: **Daily aggregated application count**

---

## Data Preparation Pipeline

```r
library(readxl)
library(dplyr)
library(lubridate)

applications <- read_excel("Applications_1770488361399.xlsx")

applications_clean <- applications %>%
  mutate(created_time = ymd_hms(`Created Time`)) %>%
  mutate(post_date = as.Date(created_time)) %>%
  group_by(post_date) %>%
  summarise(applications_per_day = n()) %>%
  ungroup() %>%
  mutate(days_since_posting = as.numeric(post_date - min(post_date)))
```

Key transformation decisions:

* Aggregation to daily level reduces timestamp noise.
* Numeric time index enables parametric regression.
* First posting date serves as baseline reference (Day 0).

---

## Model Specification

```r
model <- lm(applications_per_day ~ days_since_posting,
            data = applications_clean)

summary(model)
```

### OLS Assumptions Evaluated

1. **Linearity** — Assessed via scatter plot + fitted regression line
2. **Independence** — Time-based aggregation; note potential autocorrelation risk
3. **Homoscedasticity** — Residual vs fitted plot
4. **Normality of residuals** — Q-Q plot

Diagnostic checks:

```r
par(mfrow = c(2,2))
plot(model)
```

---

## Interpretation of Coefficients

* **β₀ (Intercept):** Estimated application volume at posting launch
* **β₁ (Slope):** Average change in daily application volume per additional day

If:

* β₁ < 0 → Applicant decay effect
* β₁ > 0 → Growing momentum
* β₁ ≈ 0 → Stable engagement

Statistical significance evaluated via p-value for β₁.

---

## Model Evaluation Metrics

Key outputs examined:

* R² and Adjusted R²
* F-statistic for overall model significance
* Standard error of estimate
* Confidence intervals for coefficients

Example extraction:

```r
confint(model)
```

---

## Potential Statistical Considerations

Because application counts are discrete and non-negative, future iterations may explore:

* **Poisson regression**
* **Negative binomial regression**
* Autoregressive time-series models (ARIMA)
* Nonlinear decay models (exponential decay)

This baseline OLS model provides an interpretable starting point before transitioning to count-based modeling frameworks.

---

## Visualization

```r
library(ggplot2)

ggplot(applications_clean,
       aes(x = days_since_posting, y = applications_per_day)) +
  geom_point() +
  geom_smooth(method = "lm", se = TRUE) +
  labs(
    title = "Daily Application Volume vs Time Since Posting",
    x = "Days Since Posting",
    y = "Applications per Day"
  )
```

The visualization supports model interpretability and slope direction assessment.

---

## Technical Skills Demonstrated

* Data wrangling with `dplyr`
* Time parsing and transformation with `lubridate`
* Feature engineering for regression modeling
* OLS regression with `lm()`
* Model diagnostics and assumption testing
* Statistical interpretation for applied business contexts

---

## Business Translation

Quantifying applicant volume dynamics enables:

* Optimal job repost timing
* Resource planning for recruitment teams
* Predictive workforce pipeline modeling
* Data-driven posting lifecycle management

---

## Future Extensions

* Multivariate regression incorporating:

  * Job category
  * Location (dummy encoding)
  * Posting platform
* Cross-posting comparative modeling
* Logistic regression for hire probability
* Survival analysis for time-to-hire modeling
* Deployment via Shiny dashboard

---

## Author

Sipho Mwamba
Data Analytics | Workforce & Health Data Modeling



