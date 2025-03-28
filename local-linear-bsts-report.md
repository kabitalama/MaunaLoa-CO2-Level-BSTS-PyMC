# Local Linear Bayesian Structural Time Series (BSTS) Model for Mauna Loa CO₂ Analysis

This repository contains an implementation of a local linear Bayesian Structural Time Series (BSTS) model to analyze and forecast the Mauna Loa CO₂ measurements using PyMC. The model decomposes the CO₂ time series into an interpretable trend (with a local linear formulation), a seasonal component, and observation noise.

## Table of Contents

- [Model Formulation](#model-formulation)
- [Model Implementation](#model-implementation)
- [Interpretation of Parameters](#interpretation-of-parameters)
- [Forecasting](#forecasting)
- [Results](#results)
- [Conclusion](#conclusion)

## Model Formulation

Our local linear BSTS model is built on the following components:

### 1. Local Linear Trend

We model the latent level (trend) $L_t$ and its slope $\mu_t$ using a local linear trend formulation. The model is specified as:


```math
\begin{aligned}
L_0 &\sim \mathcal{N}(y_0, \sigma_{L0}^2) \quad &\text{(Initial level)} \\
\mu_0 &\sim \mathcal{N}(0, \sigma_{\mu0}^2) \quad &\text{(Initial slope)} \\
\mu_t &= \mu_{t-1} + \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0, \sigma_{\mu}^2), \quad t=1,\dots,T-1 \\
L_t &= L_{t-1} + \mu_{t-1}, \quad t=1,\dots,T-1 
\end{aligned}
```

Here, 
- $L_t$ is the latent level at time $t $,
- $\mu_t$ is the time-varying slope (trend),
- $\epsilon_t$ represents random walk increments in the slope.

### 2. Seasonal Component

The seasonal effect is modeled as 12 monthly parameters with a sum-to-zero constraint:

```math
\gamma_m \sim \mathcal{N}(0, \sigma_{\gamma}^2) \quad \text{for } m = 1, \dots, 12, \quad \text{with } \sum_{m=1}^{12} \gamma_m = 0.
```

For each observation at time $t$, the seasonal effect is given by $\gamma_{m(t)}$, where $m(t)$ is the month corresponding to time $t$.

### 3. Observation Equation

The observed CO₂ concentration $y_t$ is modeled as the sum of the latent level, the seasonal effect, and observation noise:

```math
y_t = L_t + \gamma_{m(t)} + \nu_t, \quad \nu_t \sim \mathcal{N}(0, \sigma_{\text{obs}}^2).
```

## Model Implementation

In our PyMC implementation, the model is constructed as follows:

- **Local Linear Trend:**  
  - `level0` and `slope0` initialize $L_0$ and $\mu_0$.
  - A Gaussian random walk (`rw_slope`) models the increments $\epsilon_t$, and the full slope is constructed as:
    ```python
    slope = pm.Deterministic("slope", pm.math.concatenate([[slope0], rw_slope]))
    ```
  - The latent level is computed as:
    ```python
    level = pm.Deterministic("level", level0 + pm.math.cumsum(slope))
    ```
- **Seasonal Component:**  
  - The model includes 12 seasonal parameters (`seasonal_raw`), which are adjusted to sum to zero:
    ```python
    seasonal_effect = pm.Deterministic("seasonal_effect", seasonal_raw - pm.math.mean(seasonal_raw))
    ```
- **Observation Model:**  
  - The expected observation is given by:
    ```python
    mu_obs = level + seasonal_effect[month_index]
    ```
  - And the likelihood is:
    ```python
    y_obs = pm.Normal("y_obs", mu=mu_obs, sigma=sigma_obs, observed=y_train)
    ```

## Interpretation of Parameters

- **$\sigma_{\text{level}}$:**  
  Controls the variability in the evolution of the latent level $L_t$.

- **$\sigma_{\mu}$:**  
  Controls the variability in the slope increments $\epsilon_t$. A small value indicates that the slope (trend) changes slowly over time.

- **$\sigma_{\text{obs}}$:**  
  Represents the observation noise, reflecting measurement error.

- **$\text{level0}$ and $\text{slope0}$:**  
  Initial conditions for the trend.

- **Seasonal effects $\gamma_m$:**  
  Capture the periodic (monthly) fluctuations. The sum-to-zero constraint ensures that the seasonal component does not introduce an additional overall trend.

## Forecasting

For forecasting, we use the posterior samples from the model to simulate future values. The forecasting process involves:

1. **Extracting the last latent level and slope** from the training period: $L_{T_{\text{train}}}$ and $\mu_{T_{\text{train}}}$.

2. **Extrapolating the latent level:**  
   We forecast the latent level deterministically using the last estimated slope:
   
```math
L_{T_{\text{train}} + t} = L_{T_{\text{train}}} + \mu_{T_{\text{train}}} \times t, \quad t=1,\dots,n_{\text{forecast}}
```

3. **Adding the seasonal effect:**  
   For each forecasted time point, the corresponding seasonal effect $\gamma_{m(t)}$ (using the test set's month information) is added to produce the forecasted observation:
   
```math
\hat{y}_{T_{\text{train}} + t} = L_{T_{\text{train}} + t} + \gamma_{m(T_{\text{train}} + t)}
```

4. **Summarizing and Visualizing:**  
   The forecast is summarized by calculating the mean forecast, along with a 95% credible interval. Additionally, an ensemble of forecast trajectories is plotted to illustrate uncertainty.

See the forecasting section of the code for a detailed simulation of future latent levels and seasonal effects.

## Results

### Forecast (CO2 levels)

![alt text](viz/local-linear-bsts-forecast.png)

### Trend

![alt text](viz/local-linear-bsts-latent-level-estimate.png)

### Local Linear BSTS Summary

![alt text](viz/local-linear-bsts-summary-statistic.png)


## Conclusion

**Key Observations:**
- The local linear BSTS model decomposes the CO₂ time series into a slowly evolving trend and a clear seasonal pattern.
- The latent level captures the gradual upward trend, while the seasonal component reflects recurring monthly fluctuations.

**Limitations:**
- Forecasts based on a constant slope may not fully capture potential acceleration in the trend.
- The deterministic extrapolation approach may understate uncertainty in long-term predictions.
- Residual autocorrelation and potential non-linear dynamics are not explicitly modeled.

**Further Improvements:**
- Incorporate a dynamic or non-linear trend component to better reflect changing growth rates.
- Add an autoregressive structure to model temporal dependencies in the residuals.
- Introduce external covariates (e.g., ENSO indices) to account for additional sources of variability.
- Explore alternative forecasting methods that more robustly quantify forecast uncertainty.
