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

## 🛠 5. Requirements

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

## 🧠 6. The Analysis

#### Going in to this project, I wanted to find appropriate frequency and severity models for this auto dataset. This was a dataset pulled off of kaggle that interested me. 

🚗 ## **Frequency Modeling**

I started with a Poisson regression to model claim frequency. The dataset did not include claim counts, so I generated a synthetic claim frequency variable. I began with a fairly saturated model:
``` python
formula_freq = """
claim_count ~ age + months_as_customer + C(policy_state) + 
              C(coverage) + deductible + vehicle_age + C(vehicle_make) +
              C(insured_sex) + C(insured_education_level) + 
              C(insured_occupation) + C(insured_hobbies) + 
              C(insured_relationship) + incident_hour_of_the_day + 
              number_of_vehicles_involved + C(collision_type)
              """
```
After running this model and reviewing the summary outputs, I found many variables that were not statistically significant. I kept variables only if they had at least one category with a p-value < 0.05. I then iterated: removing variables, re-fitting the model, and comparing the AIC values.
Ultimately, I arrived at the simplified model:
``` python
formula_freq = """
claim_count ~  C(vehicle_make) +C(insured_occupation)
              """
```

## 📊 Residual Diagnostics
The first diagnostic plot I examined was the Predicted vs. Raw Residuals plot.
For Poisson models, residuals should form layered horizontal bands, since:
* actual values are discrete, and
* predictions are continuous.

### **Predicted vs Raw Residual Plot**
<img width="689" height="468" alt="image" src="https://github.com/user-attachments/assets/1ffcac77-f891-4bb0-8ae4-57441e98fddf" />

We do see the expected layered pattern. While the curves are not perfectly concave, the overall structure indicates reasonable model performance.
Next, I reviewed the Pearson residual histogram. Ideally, this should resemble a bell curve:
* centered around 0
* relatively symmetric
* minimal extreme outliers

### **Pearson Redidual Histogram**
<img width="695" height="468" alt="image" src="https://github.com/user-attachments/assets/006b2546-07eb-4b73-a1b9-69bf7addf87c" />

This histogram meets expectations. The residuals are centered around zero and exhibit a roughly normal shape. There is slight right-skewness, but this is common in insurance frequency data. Overall, the Poisson frequency model appears effective.


## 🔥 **Severity Modeling**

For severity modeling, I initially attempted a Gamma GLM, but the results were unsatisfactory. I then shifted to a Tweedie distribution, which introduces a power parameter that can flexibly capture data between Poisson (p=1) and Gamma (p=2).
I again began with a saturated formula and iteratively refined it by:
* removing insignificant variables
* testing different Tweedie power parameters
* selecting the combination with the lowest AIC
I settled on the following final formula, with a selected Tweedie power parameter of 1.8:
``` python
formula_sev = """
sev ~ age + 
       bodily_injuries + number_of_vehicles_involved +
        C(vehicle_make) + C(vehicle_model) +
        C(collision_type) + 
       age:C(vehicle_make) 
```
## 📈 **Severity Diagnostic Plots**

The first plot to look at is our predicted vs raw results graph. With this severity data set, we want to see a fairly random spread of residuals with little to no pattern. Below is the result.
### **Predicted vs Raw Residual Plot**
<img width="733" height="468" alt="image" src="https://github.com/user-attachments/assets/e377551c-569b-43d4-8ba8-75e3b10fadaa" />

This residual plot indicates:
* tight clustering for low severities (great predictive accuracy), and
* mild but acceptable widening of the spread as predictions increase.
Overall, the residual pattern supports this model specification.

The next important plot to view is the deviance residual histogram. Like the pearson residual histogram in the frequency case, we want to see a bell shape pattern with a centering around zero. 
### **Deviance Residual Plot**
<img width="687" height="468" alt="image" src="https://github.com/user-attachments/assets/ae8e98f9-e48a-4ceb-86a7-34371f731463" />

This histogram aligns with what we expect:
* roughly bell-shaped
* centered near zero
* limited extreme observations
This is exactly what we want to see with Tweedie deviance residuals.

Now we shift over to a predicted vs actual value plot. This plot relates to our first residual plot that we looked at so we have some hints relating to what we will see. In general, we want to see that our model predicts accurately while not overpredicting or underpredicting values. 
### **Predicted vs Actual Severity Plot**
<img width="749" height="468" alt="image" src="https://github.com/user-attachments/assets/289b3a68-2be3-4064-a7d4-58d3b700a01f" />

Again, the model performs strongly:
* low severities are predicted with high accuracy
* mid-range severities are reasonably accurate
* large values show natural variability
The next two plots we look at are a decile plot and a lift chart. In these two graphs we want to see a constant increase in mean severity and lift respectively as predicted severity decile increase. 
### **Decile Plot**
<img width="868" height="545" alt="image" src="https://github.com/user-attachments/assets/15265acd-8422-48af-a093-f9fbc651fe37" />

The decile chart shows a relatively consistent increase in mean actual severity across predicted deciles — a strong indicator of model effectiveness. 
### **Lift Chart**
<img width="846" height="545" alt="image" src="https://github.com/user-attachments/assets/9aaa3715-309a-41a4-84ba-685735b505eb" />

Our lift chartshows lift increasing with higher deciles which is another indicator of model effectiveness. While the increase is not substantial at each decile, we see the general pattern which is fairly good for this insurance data. 

#### We can conlcude from our summary and plots that our severity model is adequate. 


## 📖 7. What I Learned
#### This project was my first time modeling frequency and severity on an auto insurance dataset. I learned many things working through this project

1. I gained experience using the statsmodeling package which was new to me.
2. I learned how Poisson residuals behave differently, especially the expected layered pattern in residual plots.
3. I built and interpreted decile charts and lift charts for the first time.

