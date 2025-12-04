# Auto GLM Analysis

### Poisson Frequency, Tweedie Severity, and Core GLM Diagnostics for Auto Insurance Pricing

## 📘 1. Project Overview

This project demonstrates how to model auto insurance loss cost using Generalized Linear Models (GLMs). The workflow follows a standard actuarial frequency–severity structure:

* **Frequency Model**: Poisson GLM with an exposure offset
* **Severity Model**: Tweedie GLM with power parameter search
* **Evaluation Tools**: Residual diagnostics, decile analysis, and lift charts

This repository is structured to highlight practical model development while keeping the README concise and recruiter-friendly.

## 🧮 2. Models Included

### Frequency Model (Poisson GLM)

A Poisson model is used to estimate claim count per exposure. The model includes categorical variables selected through iterative AIC comparison and significance testing.

### Severity Model (Tweedie GLM)

The severity model uses a Tweedie GLM, which accommodates right-skewed continuous data with a point mass at zero. A power parameter search from **p = 1.1 → 2.0** is performed to select the model with the lowest AIC.

## 📊 3. Diagnostics & Evaluation

This project includes code for:

* Predicted vs residuals plots
* Deviance and Pearson residual histograms
* Predicted vs actual plots
* Frequency and severity decile charts
* Lift charts

These tools assess model fit, segmentation strength, and prediction accuracy.

## 📚 4. Results Summary

* **Frequency Model**: Residual patterns align with Poisson expectations; simplified variable selection produced a stable, interpretable model.
* **Severity Model**: Tweedie with **p ≈ 1.8** provided the best AIC. Diagnostic plots showed reasonable residual distribution and consistent decile/lift progression.
* **Overall**: Both GLMs demonstrate acceptable fit and provide a meaningful loss-cost framework.

## 🧠 5. Full Technical Analysis

The detailed modeling process—including step-by-step variable selection, p-value filtering, AIC comparison, diagnostic interpretation, and full visual results—is available here:

👉 **See full in-depth analysis:** `full_analysis.ipynb`

## ☑️ 6. Requirements

Use the following command to install dependencies:

```bash
pip install -r requirements.txt
```

Packages used include pandas, numpy, statsmodels, matplotlib, and seaborn.

## 📖 7. What I Learned
#### This project was my first time modeling frequency and severity on an auto insurance dataset. I learned many things working through this project

1. I gained experience using the statsmodeling package which was new to me.
2. I learned how Poisson residuals behave differently, especially the expected layered pattern in residual plots.
3. I built and interpreted decile charts and lift charts for the first time.
