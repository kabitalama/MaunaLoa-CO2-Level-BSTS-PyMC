# MaunaLoa-Co2-Level-BSTS-PyMC

This repository contains a collection of Bayesian Structural Time Series (BSTS) models for analyzing the Mauna Loa CO₂ data using PyMC. The project demonstrates how to decompose the CO₂ time series into interpretable components—trend, seasonality, and residuals—using both a simple local level model and an enhanced local linear trend model. It also covers model checking (posterior predictive checks, diagnostic trace plots) and forecasting on a held-out test set.

## Table of Contents

- [Overview](#overview)
- [Data](#data)
- [Modeling](#modeling)
  - [Simple BSTS Model](#simple-bsts-model)
  - [Local Linear BSTS Model](#local-linear-bsts-model)
- [Forecasting](#forecasting)
- [Results and Diagnostics](#results-and-diagnostics)
- [Installation](#installation)

## Overview

Atmospheric CO₂ measurements at the Mauna Loa Observatory have been collected since the late 1950s, providing one of the longest continuous climate records. This project employs Bayesian methods to model and forecast the Mauna Loa CO₂ time series by:

- **Decomposing** the time series into trend, seasonality, and noise.
- **Estimating** model parameters using Markov Chain Monte Carlo (MCMC) sampling.
- **Forecasting** future CO₂ concentrations by simulating from the posterior distribution.
- **Validating** model performance via posterior predictive checks and diagnostic plots.

## Data

The dataset is sourced from [DataHub CO₂ PPM](https://datahub.io/core/co2-ppm) and specifically uses the `co2-mm-mlo.csv` file, which contains monthly measurements of atmospheric CO₂ (in parts per million) at Mauna Loa. This project uses the "Interpolated" column, which provides a complete time series with missing values filled via interpolation.

The data is partitioned into a training set (all data except the last 10 years) and a test set (the final 10 years).

## Modeling

Two types of BSTS models are implemented in this project:

### Simple BSTS Model

***NOTE: This model is intended for initial exploration and understanding of the data. It decomposes the time series using a basic local level (Gaussian random walk) for the trend and a seasonal component with 12 discrete parameters (one per month).  While useful for exploratory analysis, this model does not capture the dynamic evolution of the trend as effectively as the local linear approach.***

- **Components:**
  - **Local Level (Trend):** Modeled as a Gaussian random walk.
  - **Seasonality:** Modeled with 12 discrete parameters (one for each month) constrained to sum to zero.
- **Usage:** Useful for basic decomposition of the time series.

### Local Linear BSTS Model

- **Components:**
  - **Local Linear Trend:** Introduces an initial level and an evolving slope (modeled via random walk increments). The latent level is computed as the cumulative sum of the slope added to the initial level.
  - **Seasonality:** As above, with a 12-month seasonal component.
- **Usage:** Better captures systematic growth or acceleration in CO₂ levels compared to a simple random walk model.

## Forecasting

The project demonstrates forecasting by:

- Fitting the model on the training set.
- Simulating future latent levels and seasonal effects from the posterior.
- Generating forecasts for the held-out test period (last 10 years) and comparing them with actual observations.
- Visualizing an ensemble of forecast trajectories, the forecast mean, and the 95% credible interval.

## Results and Diagnostics

The repository includes scripts to:

- Plot the estimated latent trend and seasonal components (both discrete and continuous views).
- Generate posterior predictive checks (PPC) to assess model fit.
- Display diagnostic trace plots and summary statistics (e.g., effective sample sizes and R-hat values) to confirm good MCMC convergence.


## Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/yourusername/MaunaLoa-BSTS-PyMC.git
   cd MaunaLoa-BSTS-PyMC

2. **Create a virtual environment and install dependencies:**

``` bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```