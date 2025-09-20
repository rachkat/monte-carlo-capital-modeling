# Monte Carlo Simulation for Capital Modeling

![Method: Monte Carlo](https://img.shields.io/badge/Method-Monte%20Carlo-blue)
![Model: Poisson×Lognormal](https://img.shields.io/badge/Model-Poisson%C3%97Lognormal-purple)
![Language: R](https://img.shields.io/badge/Language-R-276DC3)
![License: MIT](https://img.shields.io/badge/License-MIT-green)

**Author:** Rachel Goldsbury  
**Course:** SNHU DAT 610 Optimization & Risk Assessment · **Instructor:** Kyle Camac  
**Date:** September 22, 2024

---

## Quick Links
- **Full paper (PDF):** [monte_carlo_capital_modeling.pdf](./monte_carlo_capital_modeling.pdf)

---

## Executive Summary
This project estimates **annual collision loss** via **Monte Carlo simulation** with:
- **Event Frequency** ~ Poisson(λ)  
- **Loss Severity** ~ Lognormal(μ, σ)

For each simulated year we draw the number of events and **sum the per-event severities** (compound Poisson). The **99.9th percentile** of simulated annual losses is used as a **capital proxy**. A classroom shortcut (frequency × one severity draw) is also shown for comparison.

---

## Method Overview
1. **Data prep (R):** Load IIHS data, drop NAs, inspect.  
2. **Frequency (Poisson):** λ = average number of events per year (`mean(table(df$YEAR))`).  
3. **Severity (Lognormal):** From sample mean `m` and sd `s`, derive:
   - `sigma = sqrt(log(1 + (s^2 / m^2)))`  
   - `mu    = log(m / sqrt(1 + (s^2 / m^2)))`
4. **Monte Carlo (N ≥ 10,000):**
   - Draw `N_events ~ Poisson(λ)`  
   - Draw `N_events` severities from Lognormal(μ, σ) and **sum** them  
5. **Capital estimate:** `quantile(annual_loss, 0.999)`

# Capital proxy at 99.9%
cap_99_9 <- unname(quantile(annual_loss, probs = 0.999))
cap_99_9
Why this matters: In a compound Poisson model, annual loss is the sum of severities across all events in a year. Multiplying event count by a single severity draw can understate tail risk (e.g., the 99.9% quantile).

---

Files
monte_carlo_capital_modeling.pdf — full write-up (methods, results, references)

---

# Notes & Pitfalls
Match the severity column name (severity_col) to your dataset and ensure it is per-event loss.

Use larger N (e.g., 50k–100k) for more stable extreme quantiles.

Keep filenames lowercase_with_underscores to avoid broken README links.

Set a seed when reporting numeric results.

---

## ▶️ Reproducible R Snippet — **Compound Poisson (recommended)**
```r
# --- CONFIG ---
set.seed(610)  # reproducibility
path <- "iihs_data_for_EX_8.csv"   # <-- replace with your path
severity_col <- "Collision."       # <-- replace if your column differs
N <- 10000                         # simulated years

# --- LOAD & CLEAN ---
df <- read.csv(path, check.names = FALSE)
df <- na.omit(df); rownames(df) <- NULL

# --- FREQUENCY: estimate lambda from events per year ---
lambda <- mean(table(df$YEAR))

# --- SEVERITY: derive lognormal parameters from sample mean/sd ---
m <- mean(df[[severity_col]])
s <- sd(df[[severity_col]])
sigma <- sqrt(log(1 + (s^2 / m^2)))
mu    <- log(m / sqrt(1 + (s^2 / m^2)))

# --- MONTE CARLO (compound Poisson) ---
n_events <- rpois(N, lambda)

annual_loss <- sapply(n_events, function(k) {
  if (k == 0) 0 else sum(rlnorm(k, meanlog = mu, sdlog = sigma))
})```

---
