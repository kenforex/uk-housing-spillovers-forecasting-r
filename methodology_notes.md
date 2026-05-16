# Methodology Notes

This document summarises the estimation methodology used in this repository. The empirical workflow compares three forecasting models for local housing-market dynamics:

1. a homogeneous autoregressive model;
2. a heterogeneous autoregressive model;
3. a network autoregressive model with spatial spillovers.

The purpose of this document is to make the modelling choices transparent and to explain how the code connects to the empirical strategy.

---

## 1. Outcome Variable

Let

$$
y_{i,t}
=
\log\left(\frac{h_{i,t}}{CPI_t}\right)
-
\log\left(\frac{h_{i,t-1}}{CPI_{t-1}}\right)
$$

denote the log real house price return for Local Authority District (LAD) $i = 1, \ldots, N$ at month $t = 1, \ldots, T$.

Here:

- $h_{i,t}$ is the nominal average house price in LAD $i$ at time $t$;
- $CPI_t$ is the UK Consumer Price Index at time $t$;
- $y_{i,t}$ measures the monthly real house price return.

Let

$$
Y_t = (y_{1,t}, \ldots, y_{N,t})'
$$

denote the $N \times 1$ vector of log real house price returns across all LADs.

The analysis focuses on house price returns rather than house price levels. This choice is useful because returns are more suitable for short-horizon forecasting and are less affected by non-stationarity concerns than price levels.

---

## 2. Homogeneous Autoregressive Model

The homogeneous autoregressive model assumes that all LADs share the same dynamic coefficients. In other words, the slope coefficients are common across local housing markets.

A simple one-lag homogeneous AR specification is:

$$
Y_t = \beta_0 + \beta_1 Y_{t-1} + \gamma_1 X_{t-1} + \varepsilon_t
$$

where:

- $\beta_0$ is a common intercept;
- $Y_{t-1}$ is the vector of one-month lagged real house price returns;
- $X_{t-1}$ is the vector of one-month lagged changes in housing sales volume;
- $\beta_1$ measures the common autoregressive effect;
- $\gamma_1$ measures the common effect of lagged sales-volume changes;
- $\varepsilon_t = (\varepsilon_{1,t}, \ldots, \varepsilon_{N,t})'$ is the vector of error terms.

This model can be estimated using pooled ordinary least squares (OLS). The pooled estimator treats the panel as one combined dataset and estimates a common vector of coefficients using observations across both LADs and time.

The pooled OLS estimator is:

$$
\hat{\beta}_{pool} = (Z'Z)^{-1}Z'Y
$$

where $Z$ is the matrix of regressors and $Y$ is the stacked dependent variable.

For a homogeneous AR model without additional explanatory variables, the regressor matrix can be written as:

$$
Z = (I_N, Y_{t-1}, \ldots, Y_{t-l})
$$

For a homogeneous AR model with explanatory variables, the regressor matrix can be written as:

$$
Z =
(I_N, Y_{t-1}, \ldots, Y_{t-l},
X_{1,t-1}, \ldots, X_{1,t-l},
\ldots,
X_{n,t-1}, \ldots, X_{n,t-l})
$$

### Interpretation

The homogeneous AR model is useful when the researcher assumes that all local housing markets follow a similar dynamic process. It is relatively parsimonious and can be efficient when the degree of heterogeneity across LADs is limited.

---

## 3. Heterogeneous Autoregressive Model

The heterogeneous autoregressive model relaxes the assumption of common slope coefficients. Instead, it allows the dynamic relationship to vary across LADs.

The one-lag heterogeneous AR model is:

$$
y_{i,t}
=
\beta_{0,i}
+
\beta_{1,i} y_{i,t-1}
+
\gamma_{1,i} x_{i,t-1}
+
\varepsilon_{i,t}
$$

where:

- $\beta_{0,i}$ is an LAD-specific intercept;
- $y_{i,t-1}$ is the one-month lagged real house price return for LAD $i$;
- $\beta_{1,i}$ is the LAD-specific autoregressive coefficient;
- $x_{i,t-1}$ is the one-month lagged change in sales volume for LAD $i$;
- $\gamma_{1,i}$ is the LAD-specific coefficient on sales-volume changes;
- $\varepsilon_{i,t}$ is the idiosyncratic error term.

This model assumes that each LAD is influenced only by its own lagged outcome and its own covariates.

The coefficients can be estimated separately for each LAD using OLS:

$$
\hat{\beta}_i
=
(z'_{i,t}z_{i,t})^{-1}z'_{i,t}y_{i,t}
$$

where:

$$
z_{i,t} = (1, y_{i,t-1}, \ldots, y_{i,t-l})
$$

and

$$
\hat{\beta}_i
=
(\hat{\beta}_{0,i}, \hat{\beta}_{1,i,1}, \ldots, \hat{\beta}_{1,i,l})
$$

More generally, the heterogeneous AR model can be extended to an autoregressive model with exogenous regressors, denoted $AR-X(p)$:

$$
y_{i,t}
=
\beta_{0,i}
+
\sum_{l=1}^{p} \beta_{1,i,l} y_{i,t-l}
+
\sum_{l=1}^{p}
\left(
\gamma_{1,i,l}x_{1,i,t-l}
+
\cdots
+
\gamma_{n,i,l}x_{n,i,t-l}
\right)
+
\varepsilon_{i,t}
$$

where:

- $p$ is the lag order;
- $\beta_{1,i,l}$ measures the LAD-specific autoregressive effect at lag $l$;
- $x_{k,i,t-l}$ denotes explanatory variable $k$ for LAD $i$ at lag $l$;
- $\gamma_{k,i,l}$ is the corresponding LAD-specific coefficient.

This specification is closely related to an autoregressive distributed lag model.

---

## 4. Mean Group Estimator

The heterogeneous AR model can also be summarised using the Mean Group estimator. This estimator first estimates the model separately for each LAD and then averages the LAD-level coefficient estimates.

The Mean Group estimator is:

$$
\hat{\beta}_{MG}
=
\frac{1}{N}
\sum_{i=1}^{N}
\hat{\beta}_i
$$

where:

- $\hat{\beta}_i$ is the LAD-specific coefficient vector;
- $\hat{\beta}_{MG}$ is the cross-sectional average of all LAD-level estimates.

### Interpretation

The Mean Group estimator is useful when the researcher expects meaningful heterogeneity across local housing markets. However, when the underlying data-generating process is relatively homogeneous, pooled estimators may perform better in forecasting because they use information more efficiently across the panel.

---

## 5. Network Autoregressive Model

The network autoregressive model extends the homogeneous AR model by incorporating spatial or network interaction terms. This allows the model to capture whether neighbouring housing markets help predict local house price returns.

The network autoregressive model with $p$ lags is:

$$
Y_t
=
\beta_0
+
\sum_{l=1}^{p} \beta_{1,l}Y_{t-l}
+
\sum_{l=1}^{p} \beta_{2,l}WY_{t-l}
+
\varepsilon_t
$$

where:

- $Y_t = (y_{1,t}, \ldots, y_{N,t})'$ is the vector of real house price returns;
- $\beta_0$ is a homogeneous intercept;
- $Y_{t-l}$ captures own-market autoregressive dynamics;
- $WY_{t-l}$ captures spatially lagged house price returns;
- $W$ is an $N \times N$ spatial weights matrix;
- $w_{ij}$ measures the strength of interaction between LAD $i$ and LAD $j$;
- $w_{ii}=0$ for all $i$, so that an LAD is not treated as its own neighbour;
- $\varepsilon_t$ is the vector of idiosyncratic error terms.

A version with additional explanatory variables can be written as:

$$
Y_t
=
\beta_0
+
\sum_{l=1}^{p} \beta_{1,l}Y_{t-l}
+
\sum_{l=1}^{p} \beta_{2,l}WY_{t-l}
+
\sum_{l=1}^{p}
\left(
\gamma_{1,l}X_{1,t-l}
+
\cdots
+
\gamma_{n,l}X_{n,t-l}
\right)
+
\varepsilon_t
$$

where:

- $X_{1,t-l}, \ldots, X_{n,t-l}$ are lagged explanatory variables;
- $\gamma_{1,l}, \ldots, \gamma_{n,l}$ are homogeneous coefficients.

The least-squares type estimator for the NAR model can be written as:

$$
\hat{\beta}_{NAR}
=
(Z_W'Z_W)^{-1}Z_W'Y
$$

where $Z_W$ is the matrix containing the intercept, own lag terms, spatial lag terms, and any additional explanatory variables.

For a NAR model without explanatory variables:

$$
Z_W
=
(I_N,
Y_{t-1}, \ldots, Y_{t-l},
WY_{t-1}, \ldots, WY_{t-l})
$$

For a NAR model with explanatory variables:

$$
Z_W
=
(I_N,
Y_{t-1}, \ldots, Y_{t-l},
WY_{t-1}, \ldots, WY_{t-l},
X_{1,t-1}, \ldots, X_{1,t-l},
\ldots,
X_{n,t-1}, \ldots, X_{n,t-l})
$$

### Interpretation

The NAR model is useful when local housing markets may be affected not only by their own past dynamics, but also by the past dynamics of neighbouring markets. In this project, the spatial weights matrix is used to represent the strength of cross-market interaction.

---

## 6. Why Compare These Models?

The comparison between homogeneous AR, heterogeneous AR, and NAR models is designed to evaluate three competing sources of predictive information:

1. **Own-market persistence**  
   Whether past house price returns in the same LAD help predict future returns.

2. **Market heterogeneity**  
   Whether each LAD has its own distinct dynamic process.

3. **Spatial spillovers**  
   Whether neighbouring LADs contain additional predictive information.

This comparison helps assess whether local housing markets behave as independent units, as heterogeneous individual markets, or as interconnected markets influenced by spatial dependence.

---

## 7. Practical Implementation in the Repository

The repository implements this methodology through a sequence of R scripts. The workflow can be summarised as follows:

1. clean and transform local house price data;
2. construct real house price returns using CPI-adjusted prices;
3. construct housing sales-volume changes;
4. build spatial weights matrices for LADs;
5. estimate homogeneous AR, heterogeneous AR, and NAR models;
6. compare forecasting performance across models;
7. generate tables and figures for the paper.

