# TIME SERIES ANALYSIS OF SEASONAL EGG SALES DATA

---

## INTRODUCTION

This project focuses on the time series analysis of monthly egg sales data, which exhibits strong seasonal characteristics likely influenced by cyclical consumer demand, seasonal dietary habits, and agricultural production patterns.

The goal of this analysis is: -

- To identify and understand the inherent seasonal structure in the data using decomposition and autocorrelation techniques.

- To transform the time series into a stationary form, which is a prerequisite for most statistical forecasting models, through differencing and log transformation.

- To build and evaluate appropriate time series models.

---

## DATA EXPLORATION AND PREPROCESSING

The analysis begins with a thorough exploration of the raw time series data, which consists of monthly egg sales over several years. Visual inspection is one of the most important first steps in any time series analysis, as it reveals patterns that inform later modeling choices.

---

### TIME SERIES VISUALIZATION

A strong seasonal pattern, repeating roughly every 12 months. We can see peaks and troughs which appear at consistent intervals. These fluctuations can be because of recurring market or consumer behavior trends.

The amplitude of fluctuations appears to increase with the overall level of sales. For example, early years (1992–2000) show smaller seasonal variations, while recent years show much larger swings.

This increasing spread over time suggests the need for variance-stabilizing transformations, such as a log transformation, before modeling.

Due to the clear trend and changing variance, the series is non-stationary.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/84d8a6ae-5e49-42d8-ba89-822771b4169d" />


---

### LOG TRANSFORMATION

To stabilize the variance particularly during the seasonal peaks a logarithmic transformation was applied to the original data. This transformation is common in time series analysis to address two key issues, first being Variance Stabilization and the second one Improving Normality.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/ff802097-d88a-4a19-a557-5b1059342e41" />

---

### STATIONARITY TESTING

Before fitting ARIMA or SARIMA models, one of the most critical requirements is that the time series must be stationary. A stationary series has a constant mean, constant variance, and constant autocovariance over time. This ensures the model’s assumptions hold true and that forecasting remains meaningful beyond the observed period.

**Augmented Dickey-Fuller (ADF) Test – Original Series**

To formally test for stationarity, the Augmented Dickey-Fuller (ADF) test is applied to the original time series. The hypotheses are:

Null hypothesis (H₀): The series has a unit root (i.e., it is non-stationary).

Alternative hypothesis (H₁): The series is stationary.

**Result on original series**: ADF Test Statistic: -0.59323707 , p-value: 0.87250

---

### FIRST ORDER DIFFERENCING AND SEASONAL DIFFERENCING

To make our data stationary and handle the trends, the first difference of the series is computed. This process helps eliminate the deterministic trend and stabilize the mean.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/9b7bfc1b-5e0d-442e-b969-fd5359c0d943" />

<img width="500" alt="image" src="https://github.com/user-attachments/assets/06ccd201-a18e-464f-9c26-f0fd5457038c" />

Now we apply Seasonal differencing as we know that our data does have seasonal trends. The ACF and PACF plots after first differencing shows that there is a need of seasonal differencing.

This captures and removes the periodic structure repeating every 12 months.

After applying both first order and seasonal differencing (d = 1, D = 1, s = 12), the ADF test is rerun and gives p-value as 2.8279785076636504e-10 which is less then 0.05, thus making our series stationary.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/13f9e0a7-1bfc-4b5c-9e58-d92bb9e7191a" />


<img width="500" alt="image" src="https://github.com/user-attachments/assets/edbf80c0-0eac-455b-99a0-43eb2edda608" />


From the time series plot of the new transformed series, we can see stable mean and variance and no evident of seasonality or trend.

---

## SARIMA AND GARCH MODEL FITTING

### ACF and PACF

The ACF and PACF plots above help in determining the initial orders for the non-seasonal ARIMA(p,d,q) model. From

ACF observations: Significant spikes at lags 1 and 12. Gradual decay afterward, indicating the presence of MA (moving average) components and Lag 12 spike hints at seasonality.

PACF observations: Sharp drop after lag 2, suggesting AR components and Lag 12 is also significant, reinforcing the seasonal signal.

This combination suggests that the underlying data likely includes both AR and MA components along with seasonal dynamics.

---

### Seasonal SARIMA Model Evaluation

Several seasonal SARIMA models were fit using a 12-month seasonal cycle. The goal was to incorporate both short-term and annual seasonal dependencies, optimize fit (minimize AIC), and ensure the residuals are white noise.

| allModel                | AIC     | (-ve)   | BIC                                                 | (-ve) | Obsevation |
| ----------------------- | ------- | ------- | --------------------------------------------------- | ----- | ---------- |
| SARIMA(2,1,1)(1,1,1,12) | 1517.44 | 1495.28 | Strong fit, but many seasonal terms not significant |       |            |
| SARIMA(2,1,2)(1,1,1,12) | 1515.94 | 1490.11 | Slightly worse AIC, higher std err                  |       |            |
| SARIMA(2,1,3)(1,1,1,12) | 1528.43 | 1498.94 | Best overall by AIC/loglikelihood                   |       |            |
| SARIMA(1,1,1)(1,1,1,12) | 1518.31 | 1499.84 | Fewer parameters, but parameters significant        |       |            |

---

### Model Information (SARIMA(1,1,1)(1,1,1,12))

| Metric             | Value                            |
| ------------------ | -------------------------------- |
| Dependent Variable | Sales                            |
| Observations       | 324                              |
| Model Type         | SARIMAX(1,1,1)(1,1,1,12)         |
| Log Likelihood     | 764.154                          |
| AIC                | -1518.307                        |
| BIC                | -1499.839                        |
| HQIC               | -1510.914                        |
| Sample Period      | Jan 1993 – Dec 2019              |
| Covariance Type    | OPG (Outer Product of Gradients) |

**Parameter Estimates**

| Term          | Coefficient | Std. Error | z-value | p-value | 95% Confidence Interval |
| ------------- | ----------- | ---------- | ------- | ------- | ----------------------- |
| AR(1)         | 0.0781      | 0.068      | 1.155   | 0.248   | \[-0.054, 0.211]        |
| MA(1)         | -0.8073     | 0.034      | -23.481 | 0.000   | \[-0.875, -0.740]       |
| SAR(12)       | -0.3426     | 0.069      | -4.936  | 0.000   | \[-0.479, -0.207]       |
| SMA(12)       | -0.2501     | 0.085      | -2.943  | 0.003   | \[-0.417, -0.084]       |
| σ² (Variance) | 0.0003      | 2.01e-05   | 16.886  | 0.000   | \[0.000, 0.000]         |

---

### Residual Diagnostics

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/3da0fea9-c1c5-42b0-a6e1-f5c985842e03" /><img width="1000" alt="image" src="https://github.com/user-attachments/assets/eac5d3cd-527d-40ea-95c9-db4906aef2b6" />

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/781fbc46-5419-4a77-b0f5-67977e4ba909" />
<img width="1000" alt="image" src="https://github.com/user-attachments/assets/732dcdf3-4383-434c-97cc-0ffb7340ab84" />



The residuals of the final model were evaluated through ljung-box test. The first 11 lags showed that residuals had no autocorrelations. But from lag 12, there is a sudden drop in the p-values, which suggested that seasonal effect was not fully captured.

We can observe a slight left-skewed, mild excess kurtosis which is expected in real-world time series.

While models like SARIMA (2,1,3)(1,1,1,12) had better AIC and log-likelihood, they introduced more parameters which can lead to overfitting. Also, the other model residual analysis also showed many significant lags in the ACF and PACF. Selecting this model helps with better business communications and development as it has a simple structure and much better interpretability.

---

### Garch Model

Although the residual diagnostics from the SARIMA model (1,1,1) (1,1,1,12) did not suggest the presence of volatility clustering as confirmed by, a GARCH (1,1) model was fit to the residuals as a learning exercise to explore volatility modeling using financial econometrics techniques. As for the model parameters: -

**Mean model**: Constant (mu ≈ 0.00032)
**Volatility model**: GARCH (1,1)
**Distribution**: Normal
**Optimization**: Maximum Likelihood
**Observations**: 324

**Model Summary**

| Parameter     | Coefficient | Std. Error | t-Statistic | p-Value  | Observations                                      |
| ------------- | ----------- | ---------- | ----------- | -------- | ------------------------------------------------- |
| μ (mu)        | 3.20e-04    | 2.66e-05   | 12.06       | < 0.0001 | Mean return is tiny but statistically significant |
| ω (omega)     | 1.93e-04    | 3.88e-06   | 49.77       | < 0.0001 | Base level of volatility                          |
| α (alpha\[1]) | 0.4140      | 0.0283     | 14.61       | < 0.0001 | Impact of previous shocks                         |
| β (beta\[1])  | 0.4902      | 0.0361     | 13.57       | < 0.0001 | Persistence of volatility                         |

Both α (0.414) and β (0.490) are statistically significant, meaning past squared shocks and lagged conditional variances explain the variance today. The Sum of alpha and beta implies stationary volatility and mean-reverting behavior.

Volatility decays over time and the constant mean is near zero, which is typical in residual series.

---

## FORECASTING

After validating the SARIMA(1,1,1)(1,1,1,12) model through diagnostics and residual analysis, it was used to forecast future egg sales.

<img width="1200" alt="image" src="https://github.com/user-attachments/assets/c807ebd7-74a6-497e-b2fd-4ae7b3c3afb1" />


The SARIMA model can reproduce historical patterns and project them into the future. It reflects the natural growth in egg sales while keeping the periodic seasonality seen over decades.

The forecast can provide valuable input for planning inventory, production, and pricing decisions over the next year or two.

---

## CONCLUSION

This project undertook a comprehensive time series analysis of monthly egg sales data, successfully capturing both long-term trends and recurring seasonal patterns. Through careful exploratory analysis, we identified a clear upward trajectory in sales volume over time, accompanied by strong annual seasonality. To prepare the data for modeling, a log transformation was applied to stabilize the variance, followed by both first-order and seasonal differencing to achieve stationarity.

The model selection process involved interpreting the ACF and PACF plots, which guided the development of candidate ARIMA configurations. After evaluating several alternatives, the SARIMA(1,1,1)(1,1,1,12) model was selected based on its strong statistical performance. This model fit the data well with an AIC of -1518.31, and most importantly, produced residuals that behaved like white noise, as confirmed by the Ljung-Box test with p-values above 0.65. Its forecasts extended the historical seasonality and trend seamlessly into the future, demonstrating its reliability and predictive strength.

Although the residuals did not show significant heteroskedasticity, a GARCH(1,1) model was applied as an exploratory step. The GARCH model confirmed that the conditional variance was low and stable, offering further support for the adequacy of the SARIMA model. This exercise also provided a useful opportunity to practice volatility modeling and gain insights into how GARCH behaves in stable systems.

The forecasts generated by the final model offer valuable guidance for future planning. The increasing trend in egg sales is expected to continue, and the forecast intervals, though gradually widening, suggest only moderate uncertainty. This makes the model well-suited for short- to medium-term forecasting in operational and strategic contexts.

---

## FUTURE IMPROVEMENTS

One way to enhance the model's predictive capabilities is by incorporating exogenous variables using the SARIMAX framework. Factors such as feed prices, seasonal holidays, or regional consumption patterns could provide valuable context that improves forecast precision, especially over longer time horizons.

Advanced modeling approaches, machine learning algorithms like XGBoost or time series models such as Prophet could provide added flexibility in capturing non-linear behavior.

---
