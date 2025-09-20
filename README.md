# Monte Carlo Simulation for Capital Modeling

**Author:** Rachel Goldsbury  
**Course:** SNHU DAT 610 · **Instructor:** Kyle Camac  
**Date:** September 22, 2024

---

## 📄 Quick Links
- **Full paper (PDF):** [monte_carlo_capital_modeling.pdf](./monte_carlo_capital_modeling.pdf)

---

## 🧭 Executive Summary
This project estimates **maximum annual collision loss** using a **Monte Carlo simulation** that combines:
- **Frequency** ~ Poisson(λ) for number of loss events per year
- **Severity** ~ Lognormal(μ, σ) for loss size per event

We simulate many years, compute **annual total loss = frequency × severity**, and read off an extreme quantile (e.g., **99.9%**) as a **capital requirement proxy**. The paper shows a worked example using IIHS data and reports an illustrative **99.9% annual loss ~ 3963.137** (from the exercise’s inputs and draws).

---

## 🔬 Method (at a glance)
1. **Data prep (R):** load CSV → remove NAs → inspect.  
2. **Frequency (Poisson):** `λ = mean(events per year)` (e.g., `mean(table(df$YEAR))`).  
3. **Severity (Lognormal):** derive `μ, σ` from sample mean `m` and sd `s`:
   - `sigma = sqrt(log(1 + (s^2 / m^2)))`  
   - `mu = log(m / sqrt(1 + (s^2 / m^2)))`
4. **Monte Carlo:** repeat N times (e.g., **1000+**):
   - draw `N_events ~ Poisson(λ)`
   - draw `severity ~ Lognormal(μ, σ)` (per-event or average, depending on design)
   - compute **annual total** (e.g., `N_events * severity` for a simple model)
5. **Capital estimate:** take the **99.9th percentile** (or sort and pick top).

---

## ▶️ Reproducible R Snippet
```r
# --- CONFIG ---
set.seed(610)  # for reproducibility
path <- "iihs_data_for_EX_8.csv"  # replace with your file path

# --- LOAD ---
df <- read.csv(path)
df <- na.omit(df)
rownames(df) <- NULL

# --- FREQUENCY (Poisson) ---
yearMean <- mean(table(df$YEAR))  # λ

# --- SEVERITY (Lognormal) ---
m <- mean(df$Collision.)     # replace with your severity column
s <- sd(df$Collision.)
sigma <- sqrt(log(1 + (s^2 / m^2)))
mu <- log(m / sqrt(1 + (s^2 / m^2)))

# --- MONTE CARLO ---
N <- 10000
annual_loss <- replicate(N, rpois(1, yearMean) * rlnorm(1, mu, sigma))

# 99.9% capital estimate
cap_99_9 <- quantile(annual_loss, probs = 0.999, names = FALSE)
cap_99_9
