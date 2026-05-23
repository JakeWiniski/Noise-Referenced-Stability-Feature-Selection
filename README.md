# Noise-Referenced Stability Feature Selection (v0.1.0)
[![DOI](https://zenodo.org/badge/1136348057.svg)](https://doi.org/10.5281/zenodo.20358453)

### Summary

**Noise-Referenced Stability Feature Selection** is a practical workflow for identifying reliable predictors in small-to-moderate numerical datasets where sample size is limited, features are correlated, and interpretability matters. The method combines bootstrap resampling, Random Forest permutation importance, and an explicit noise baseline to separate true signal from spurious associations. Rather than selecting features solely by raw importance, the workflow evaluates how *consistently* each feature outperforms noise, producing a stability metric that is robust to sampling variation. The result is an interpretable, defensible feature set accompanied by diagnostics that explain how much of the target is realistically explainable.

This workflow is designed for applied R&D settings—process data, experimental studies, materials science, and sensor analytics—where datasets are often noisy, modest in size, and rich in engineered but redundant predictors.

## Novelty and Scope

This repository presents an applied feature-selection workflow for noisy experimental and process datasets. It does **not** claim to introduce a fundamentally new statistical theory or algorithmic class.

Related ideas already exist across methods such as:

- stability selection
- permutation importance
- null importance testing
- bootstrap feature selection
- Boruta-style shadow-feature methods
- model-based variable screening

The contribution of this project is an **applied synthesis**: a transparent, reproducible workflow that combines bootstrap stability, permutation importance, and an explicit noise reference into a practical diagnostic pipeline.

The method is intended to help answer applied questions such as:

- Which predictors are consistently stronger than noise?
- Which features are likely to be redundant proxies?
- How stable are selected predictors across resampling?
- Is the target meaningfully explainable with the available features?
- Which relationships are useful enough to support further experimentation?

This repository should be interpreted as:

- a practical workflow
- a technical prototype
- a diagnostic feature-selection tool
- a foundation for further refinement and benchmarking

It should not currently be interpreted as:

- a validated production package
- a causal inference method
- a guarantee of feature importance under all model classes
- a claim of novelty over existing feature-selection literature
- 
---

## Technical Walkthrough

The workflow proceeds in four structured stages:

**1) Stability-Based Feature Selection**

- A Random Forest model is fit repeatedly on bootstrap samples of the data.  
- For each bootstrap, permutation importance is computed for all features.  
- A parallel “null model” is created by shuffling the target variable, establishing a per-bootstrap noise floor.  
- Each feature receives a stability score (**freq_sig**) representing the fraction of bootstraps where its importance exceeds the null baseline.  
- Features are selected using a two-stage rule:  
  - retain those with `freq_sig` above a user-defined threshold  
  - rank the survivors by mean importance and keep the top N  

This step identifies predictors that are *consistently* useful rather than occasionally lucky.

**2) Explanatory Model Comparison**

The selected feature set is evaluated using three model classes of increasing flexibility:

- **Elastic Net (linear)** – to assess linear structure  
- **Generalized Additive Model (smooth nonlinear)** – to test for smooth nonlinear effects  
- **Random Forest (nonlinear + interactions)** – to capture complex relationships  

Cross-validated RMSE, MAE, and R² are compared to determine whether the target is primarily linear, nonlinear, or weakly explainable overall.

**3) Elastic Net Coefficient Stability**

To interpret linear relationships, the workflow reconstructs coefficients across cross-validation folds and reports:

- how often each coefficient is non-zero  
- mean magnitude and variability  
- directional stability  

This reveals which features are consistently used by a regularized linear model and how redundant predictors share weight.

**4) Random Forest Permutation Importance**

Finally, model-agnostic permutation importance is computed as the change in RMSE when each feature is shuffled. This provides a clear, unit-based measure of how much predictive performance depends on each variable.

Together, these steps form a coherent diagnostic pipeline:  
**detect stable signal → evaluate explainability → interpret model structure.**

---

## Practical Use Case

This workflow is most useful when:

- datasets contain **30–500 observations**  
- predictors are primarily **numeric and engineered**  
- features are **correlated or redundant**  
- the goal is to understand the data rather than to maximize predictive performance  
- interpretability is essential for decision making  

Typical applications include laboratory experiments, bioprocess data, material characterization studies, sensor arrays, and other domains where collecting new samples is expensive.

The method answers practical questions such as:

- *Which features consistently matter?*  
- *Which predictors are redundant proxies for the same signal?*  
- *Is the target primarily linear or nonlinear?*  
- *How much of the variation is realistically explainable?*  

Importantly, the workflow is diagnostic rather than purely predictive. It is intended to support insight generation, hypothesis refinement, and data-driven decision making.

---

## Demonstration Synthetic Dataset

To illustrate the method, the repository includes a purpose-built synthetic dataset designed to mimic the structure of real experimental data.

### Feature Types

The dataset contains several classes of variables:

- **Primary linear drivers**  
  - `strong_linear_1`, `strong_linear_2`  
  - Large, stable linear effects on the target  

- **Secondary linear signal**  
  - `moderate_linear`  
  - Weaker but real linear influence  

- **True nonlinear effect**  
  - `nonlinear_1`  
  - A smooth nonlinear relationship  

- **True interaction**  
  - `interaction_a`  
  - Captures a genuine multiplicative effect  

- **Redundant confounder proxies**  
  - `confound_proxy_1`, `confound_proxy_2`, `confound_proxy_3`  
  - Correlated measurements of an underlying latent factor  

- **Correlated but non-causal feature**  
  - `interaction_b`  
  - A proxy related to `interaction_a` but not directly causal  

- **Noise variables**  
  - `pure_noise_*`  
  - Completely unrelated to the target  

### How the Dataset Demonstrates the Workflow

This structure allows the workflow to highlight several realistic behaviors:

- Stability selection correctly identifies the truly informative features while rejecting noise.  
- Correlated proxies appear as useful but non-unique predictors, demonstrating redundancy handling.  
- Elastic Net coefficients show how linear models distribute weight among redundant features.  
- Model comparison reveals whether nonlinear or interaction effects materially improve explanation.  
- Permutation importance quantifies how much predictive performance actually depends on each feature.

The dataset therefore acts as a controlled sandbox that mirrors the kinds of patterns commonly encountered in real scientific and engineering data.

---

### Bottom Line

Noise-Referenced Stability Feature Selection provides a transparent, statistically cautious way to extract meaningful signal from noisy tabular data—prioritizing **reliability and interpretability** over raw predictive performance.

---

### License

This project is released under the Apache License 2.0.

The intent is to support open exploration, reproducibility, and extension of noise-referenced feature-selection workflows while preserving a permissive license structure suitable for research, education, and applied development.
