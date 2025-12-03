# auto-glm-analysis
## Poisson frequency, Tweedie severity, and GLM model diagnostics

## 📘 1. Project Overview

Auto insurers commonly model loss cost using Generalized Linear Models (GLMs). This repository demonstrates how to build, evaluate, and visualize the core components of a pricing 
model:

### Claim Frequency

Modeled using a Poisson GLM with an exposure offset.

E[N]=exp(Xβ)

### Claim Severity
Modeled using a Tweedie GLM, ideal for continuous, right-skewed claims with a mass at zero.

E[Y]=exp(Xβ)

## 🧮 2. Models Included
### 🔢 Frequency Model (Poisson GLM)

``` python
  formula_freq = """
    claim_count ~ C(vehicle_make) + C(insured_occupation)
""" 

freq_model = smf.glm(
    formula=formula_freq,
    data=df,
    family=sm.families.Poisson(),
    offset=np.log(df["exposure"])
).fit(cov_type='HC0')
```

### 💥 Severity Model (Tweedie GLM with Power Search)

The model automatically tests Tweedie variance powers from p = 1.1 → 2.0 and selects the lowest-AIC model.

```python
var_powers = np.arange(1.1, 2.0, 0.1)

best_aic = np.inf
best_model = None

for p in var_powers:
    model = smf.glm(
        formula=formula_sev,
        data=claims,
        family=sm.families.Tweedie(var_power=p, link=sm.families.links.log())
    ).fit()

if model.aic < best_aic:
    best_aic = model.aic
    best_model = model
```
## 📊 3. Model Evaluation Charts
The repository includes code to generate standard GLM diagnostics:
* ✔ Predicted vs Residuals
* ✔ Deviance Residuals
* ✔ Predicted vs Actual (severity & frequency)
* ✔ Frequency deciles
* ✔ Severity deciles
* ✔ Lift charts for model ranking

These visualizations help validate model fit and highlight over/under-prediction patterns.

## 📈 4. Decile & Lift Analytics

Deciling is a core actuarial technique for evaluating segmentation strength.


Policies are sorted into 10 equal-sized groups based on predicted risk:


* Decile 1: lowest predicted risk
* Decile 10: highest predicted risk

For each decile we compute:

* Mean actual loss
* Mean predicted loss
* Lift relative to portfolio average

This helps assess pricing accuracy and identify where the model provides meaningful differentiation.

## 🗂 5. Repository Structure
``` python
auto-insurance-glm-analysis/
│
├── frequency_model.py
├── severity_model.py
├── model_diagnostics.py
├── decile_charts.py
├── glm_auto_notebook.ipynb      # optional combined notebook
├── requirements.txt
└── README.md
```
Optional additions:
``` python
/plots/                # example output charts  
/data/                 # anonymized or synthetic dataset
``` 
## 🛠 6. Requirements

Install dependencies:
``` python
pip install -r requirements.txt
```
Major packages:
* pandas
* numpy
* statsmodels
* matplotlib
* seaborn (optional)

## ▶ 7. Running the Project
Option A — Notebook
``` python
jupyter notebook glm_auto_notebook.ipynb
```
Option B — Run Scripts Directly
``` python
python frequency_model.py
python severity_model.py
python decile_charts.py
```
## 🔗 8. Connect & Feedback

If you have feedback, questions, or want to discuss GLMs, actuarial methods, Tweedie distributions, or insurance analytics—feel free to reach out!

I can also help you:

* Add your name + LinkedIn badge
* Insert example charts
* Write a more formal “Actuarial Case Study”
* Expand this into a full portfolio project

Just tell me!
