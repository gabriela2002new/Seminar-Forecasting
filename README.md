# README file

### Gabriela Cretu, Daniel Perez Domouso, Berdan Erdem, Sitie Pan, Orestis

### Tziapouras

### April 21, 2025

## Data setup and Variable scenarios

The dataset "Dataset-team10" comprises monthly macroeconomic and financial indicators
extracted from an Excel file. Each series is transformed into logarithmic differences or growth
rates when applicable, followed by standardization (mean = 0, standard deviation = 1) to
ensure comparability.
We establish four variable scenarios, with the first three applied in the frequentist re-
cursive, Bayesian recursive, and non-recursive setups. The fourth scenario, due to the large
number of variables, is only utilized in the Bayesian non-recursive and dynamic factor models.

- Scenario 1 (Baseline):Combines key demand-side and supply-side variables, includ-
    ing energy prices, exchange rate, disposable income, and inflation.
    SVAR Script: SVAR_baseline.py
    SVAR Functions: Functions.py
    Bayesian Recursive Script: recursive_baseline.py
    Bayesian Functions (Recursive): BayesianSVAR.py
    Bayesian Non-Recursive Script: non_recursive_baseline.py
    Bayesian Functions (Non-Recursive): BSVAR_functions.py
- Scenario 2 (Demand-Only):Focuses primarily on demand-driven indicators, omit-
    ting supply-side shocks like oil and gas prices.
    SVAR Script: SVAR_demand_only.py
    SVAR Functions: Functions_D3.py
    Bayesian Recursive Script: recursive_demand_only.py
    Bayesian Functions (Recursive): BayesianSVAR.py
    Bayesian Non-Recursive Script: non_recursive_demand_only.py
    Bayesian Functions (Non-Recursive): BSVAR_functions.py
- Scenario 3 (Extensive Specification): A more comprehensive setup with broader
    macroeconomic indicators including financial conditions and interest differentials.
    SVAR Script: SVAR_large.py
    SVAR Functions: Functions_total.py


```
Bayesian Recursive Script: recursive_seven.py
Bayesian Functions (Recursive): BayesianSVAR.py
Bayesian Non-Recursive Script: non_recursive_seven.py
Bayesian Functions (Non-Recursive): BSVAR_functions.py
```
- Scenario 4 (Full Specification):An even more comprehensive setup with a broader
    set of macroeconomic variables.
    Bayesian Non-Recursive Script: non_recursive_largest_10.py
    Bayesian Functions: Bayesian_functions.py
    DFM script: DFM, Forecasting.Rmd

## 1 Standard VAR

This script performs rolling window forecasting using a Vector Autoregression (VAR)
model for inflation growth rate and evaluates the forecasting performance over multiple
horizons. The script evaluates three different variable scenarios and generates forecasts for
1-month, 3-month, and 6-month ahead horizons, using a rolling window approach to train
the VAR model at each iteration. The model’s performance is evaluated using Root Mean
Squared Error (RMSE).

### Dependencies

The following Python libraries are required:

- pandas: Data manipulation and handling.
- numpy: Core numerical operations.
- matplotlib: Plotting and visualization.
- statsmodels: VAR model estimation.
- sklearn: Preprocessing for scaling and metrics for RMSE calculation.

### Workflow Description

1. Configuration Section
This section defines the file paths, target variable, forecasting horizons, and different variable
scenarios for forecasting.
    file_path = r’C:\Users\George Tziapouras\Desktop\BSc^2 year 4\ Seminar
       in forecasting\RECOVERY\Stationary_data_mine (10) -2Final.xlsx’
    target_var = ’growth rate inflation ’
    forecast_horizons = [1, 3, 6]
    window_size = 95
    lags = 1


```
Three different variable scenarios are defined:
```
- Scenario 1: Includes demand-side and supply-side variables.
- Scenario 2: Reduced model focusing on demand-side variables.
- Scenario 3: Full specification with broader economic indicators.
2. Data Loading Preparation
The script loads the time series data from an Excel file, processes it, and standardizes all
columns. Missing timestamps are handled, and the time series frequency is set to monthly.
df = pd.read_excel(file_path , engine=’openpyxl ’)
df[’Months ’] = pd.to_datetime(df[’Months ’])
df = df[~df[’Months ’]. duplicated(keep=’first’)]
df.set_index(’Months ’, inplace=True)
df = df.asfreq(’MS’) # Monthly frequency
df[df.columns] = scale(df[df.columns ]) # Standardize the data
3. Forecast Function: Rolling Forecast with VAR
A rolling-window approach is used to generate forecasts using a VAR model. In each itera-
tion:
- A VAR model is fit to a rolling window of historical data.
- The model forecasts the target variable (inflation growth rate) for a specified number
of periods ahead.
- The forecasted values are stored for evaluation.
def rolling_var_forecast(data , window_size , lags , forecast_horizon):
results = []
n_obs = data.shape [0]
for start in range(n_obs - window_size - forecast_horizon + 1):
end = start + window_size
window_data = data.iloc[start:end]
try:
model = VAR(window_data)
var_res = model.fit(lags)
last_obs = window_data.values[-lags:]
forecast = var_res.forecast(last_obs , steps=
forecast_horizon)
results.append ({
"forecast": forecast [-1],
"forecast_index": end + forecast_horizon - 1


```
})
except Exception as e:
print(f"Window {start}-{end} failed: {e}")
results.append(None)
return results
```
4. Comparison Function: Forecast vs Actual
The script compares forecasted values against actual observed values to compute the RMSE
(Root Mean Squared Error). The forecasted and actual values are aligned for evaluation.
    def build_forecast_vs_actual(results , original_data):
       forecasts = []
       actuals = []
       indices = []
       for r in results:
          if r is None:
             continue
          idx = r[’forecast_index ’]
          if idx >= len(original_data):
             continue
          forecasts.append(r[’forecast ’])
          actuals.append(original_data.iloc[idx]. values)
          indices.append(original_data.index[idx])
       forecast_df = pd.DataFrame(forecasts , index=indices , columns=
          original_data.columns)
       actual_df = pd.DataFrame(actuals , index=indices , columns=
          original_data.columns)
       df_comparison = pd.concat ({’forecast ’: forecast_df , ’actual ’:
          actual_df}, axis =1)
       return df_comparison
5. RMSE Function
The RMSE between forecasted and actual values is computed using the formula:

```
RM SE=
```
```
vu
ut 1
n
```
```
Xn
i=
```
```
(f orecasti−actuali)^2
def compute_rmse(comp_df):
mse = (( comp_df["forecast"] - comp_df["actual"]) ** 2).mean()
return np.sqrt(mse)
```

6. Forecast Plotting Function
The forecasted and actual inflation growth rate values are plotted for visual comparison.
This helps in assessing the forecasting model’s performance visually.
    def plot_forecast_vs_actual(comp_df , target_col):
       plt.figure(figsize =(10, 5))
       plt.plot(comp_df.index , comp_df["actual"][ target_col], label="
          Actual", marker=’o’)
       plt.plot(comp_df.index , comp_df["forecast"][ target_col], label="
          Forecast", linestyle=’--’, marker=’x’)
       plt.title(f’Forecast vs Actual for {target_col}’)
       plt.xlabel(’Date’)
       plt.ylabel(target_col)
       plt.grid(True)
       plt.legend ()
       plt.tight_layout ()
       plt.show()
7. Execution Loop
The script iterates over the defined variable scenarios and forecasting horizons. For each
combination, it generates rolling forecasts, computes RMSE, and visualizes the results.
    rmse_table = {}
    for scenario_name , variables in all_var_sets.items():
       df_scen = df[variables ]. dropna ()
       rmse_table[scenario_name] = {}
       for horizon in forecast_horizons:
          results = rolling_var_forecast(df_scen , window_size , lags ,
             forecast_horizon=horizon)
          comp_df = build_forecast_vs_actual(results , df_scen)
          inflation_df = comp_df.xs(target_var , axis=1, level =1)
          num_forecasts = inflation_df.dropna ().shape [0]
          rmse = np.sqrt ((( inflation_df[’forecast ’] - inflation_df[’
             actual ’]) ** 2).mean())
          rmse_table[scenario_name ][f"{horizon}-step"] = rmse
          plot_forecast_vs_actual(comp_df , target_col=target_var)
    rmse_summary = pd.DataFrame(rmse_table)


### Function Reference

- rolling_var_forecast(data, window_size, lags, forecast_horizon): Generates
    rolling forecasts using a VAR model.
- build_forecast_vs_actual(results, original_data): Aligns forecasted values with
    actual values for RMSE calculation.
- compute_rmse(comp_df): Computes the RMSE between forecasted and actual values.
- plot_forecast_vs_actual(comp_df, target_col): Plots forecasted vs actual val-
    ues for visual comparison.

### RMSE Summary Table

The script produces a summary of the RMSE values for each forecasting horizon (1-step,
3-step, 6-step) and variable scenario. The results are presented in a table where rows corre-
spond to forecast horizons, and columns represent different variable scenarios.

- The RMSE values are printed for each scenario and horizon.
- Forecasts and actual values for inflation growth rate are plotted for each horizon.

### Advanced Features

- Rolling Forecasting: The model is updated with a new window of historical data
    for each forecast.
- RMSE Evaluation: The model’s forecasting accuracy is evaluated using RMSE for
    each variable scenario and forecast horizon.
- Visualization: The forecasted and actual values for inflation are plotted for each
    horizon.

## 2 Structural VAR

This script implements a Vector Autoregression (VAR) model to analyze economic time
series data and estimate the impact of structural shocks, particularly focusing on inflation’s
response to various economic factors. It includes functionality for:

- Data preparation and stationarity checks.
- Estimation of a reduced-form VAR model.
- Structural decomposition using Cholesky factorization.
- Impulse Response Function (IRF) analysis.
- Rolling-window forecasts and comparison of forecast vs actual values.


### Dependencies

The following Python libraries are required:

- numpy: Core numerical operations.
- pandas: Data manipulation and analysis.
- Functions: Custom module implementing various functions for model estimation and
    forecasting.

### Dependencies(Functions)

The following Python libraries are required:

- numpy: Core numerical computations and array operations.
- pandas: Data manipulation and analysis.
- matplotlib: Plotting and visualization library.
- statsmodels.tsa.api: Time series analysis tools, includingVAR(Vector Autoregres-
    sive models).
- scipy.linalg: Linear algebra operations, includingcholesky(Cholesky decomposi-
    tion) andLinAlgError(linear algebra errors).
- scipy.optimize: Optimization tools, includingminimizeandBounds.
- statsmodels.tsa.stattools: Time series statistical tools, includingadfuller(Aug-
    mented Dickey-Fuller test for stationarity).
- scikit-learn: Machine learning tools, includingStandardScalerfor data normal-
    ization.

### Workflow Description

1. Data Loading
The script begins by loading time series data from an Excel file for analysis.
    file_path = ’/Users/pansitie/Desktop/Stationary_data_mine (10).xlsx’
    df = Functions.load_data(file_path)


2. Filtering and Stationarity Testing
Data is filtered by a specified date range, and stationarity tests are conducted on the series.
    start_date = ’2015 -01’
    end_date = ’2024 -01’
    columns = [’Brent_logdiff ’, ’Gas Prices_logdiff ’, ’growth rate
       inflation ’, ’DSPIC96_log_diff ’, ’EUR -USD_logdiff ’]
    df_estimation = Functions.filter_data(df, start_date , end_date ,
       columns)
    Functions.test_stationarity(df_estimation , columns)
3. Data Standardization
The data is scaled (standardized) to ensure comparability across variables.
    df_estimation = Functions.scale_columns(df_estimation , columns)
4. VAR Model Estimation
A reduced-form VAR model is estimated based on the filtered and scaled data.
    var_results = Functions.estimate_var_model(df_estimation)
5. Structural Decomposition
The residuals, covariance matrix, and long-run matrix are computed, and structural shocks
are identified using Cholesky decomposition.
    residuals , cov_matrix , C_inf = Functions.compute_matrices(var_results
       )
    restrictions = Functions.convert_named_to_index_restrictions(
       Functions.define_named_sign_restrictions (), columns)
    optimized_chol_matrix = Functions.optimize_chol_matrix(cov_matrix ,
       C_inf , restrictions)
    structural_shocks = Functions.compute_structural_shocks(residuals ,
       optimized_chol_matrix)
6. Impulse Response Functions (IRFs)
The IRF analysis computes the response of inflation to various economic shocks.
    num_periods = 100
    all_shocks_indices = [0, 1, 3, 4] # Non -inflation shocks
    inflation_index = 2
    all_shocks_irfs = Functions.compute_side_IRF(columns , num_periods ,
       all_shocks_indices , structural_shocks , inflation_index)
    Functions.plot_side_shocks(all_shocks_indices , columns ,
       all_shocks_irfs)


7. Cumulative Response to Cost-Push Shocks
The cumulative impact of cost-push shocks (Brent and Gas prices) on inflation is computed
and visualized.
    cost_push_indices = [0, 1] # Brent and Gas price shocks
    cost_push_irfs_per_period = Functions.augmented_IRFS(num_periods ,
       structural_irfs , inflation_index , cost_push_indices)
    Functions.plot_augumented_IRFS(cost_push_irfs_per_period , "Sum of
       Cost -Push IRFs on Inflation", "Cumulative Response of Inflation to
          Cost -Push Shocks")
8. Forecasting with Rolling Window VAR
A rolling-window approach is used for recursive out-of-sample forecasting with the VAR
model.
    results = Functions.rolling_svar_forecast(df_estimation , window_size
       =95, lags =1)
9. Forecast Evaluation
Forecasts are compared against actual data, and the model’s accuracy is evaluated using
RMSE.
    comparison_df = Functions.build_forecast_vs_actual_df(results ,
       df_estimation)
    rmse = Functions.compute_rmse(comparison_df)
    print("RMSE by variable :\n", rmse)
10. Forecast Visualization
The forecast vs actual values for inflation are plotted for visual inspection.
    Functions.plot_forecast_vs_actual(comparison_df , column_index =2)

### Function Reference

- load_data(filepath): Load time series data from an Excel file.
- filter_data(df, start, end, columns): Filter the dataset based on the date range
    and selected columns.
- test_stationarity(df, columns): Perform stationarity tests on the selected columns.
- scale_columns(df, columns): Standardize the dataset columns for consistency.
- estimate_var_model(df): Estimate a reduced-form VAR model.


- compute_matrices(var_results): Compute residuals, covariance matrix, and the
    long-run matrix (C).
- optimize_chol_matrix(cov_matrix, C_inf, restrictions): Optimize the Cholesky
    decomposition matrix to satisfy structural restrictions.
- compute_structural_shocks(residuals, chol_matrix): Compute the structural
    shocks using the optimized decomposition matrix.
- compute_side_IRF(columns, num_periods, shock_indices, structural_shocks,
    inflation_index): Compute the IRF of inflation to each shock.
- plot_side_shocks(shock_indices, columns, irfs): Plot the IRFs of inflation to
    various shocks.
- augmented_IRFS(num_periods, structural_irfs, inflation_index, cost_push_indices):
    Compute the cumulative cost-push IRF on inflation.
- plot_augumented_IRFS(irfs, label, title): Plot the augmented IRF for cost-
    push shocks on inflation.
- rolling_svar_forecast(df, window_size, lags): Perform recursive out-of-sample
    forecasts using a rolling-window VAR model.
- build_forecast_vs_actual_df(results, df): Build a DataFrame to compare fore-
    casts with actual values.
- compute_rmse(comparison_df): Compute the Root Mean Squared Error (RMSE) of
    forecasts.
- plot_forecast_vs_actual(df, column_index): Visualize the forecast vs actual com-
    parison for a specific column.

### Advanced Features

- Structural Decomposition: Cholesky decomposition is used to identify structural
    shocks and impose sign restrictions.
- Rolling Window Forecasting: Recursive estimation and forecasting allow for time-
    varying dynamics in the VAR model.
- Impulse Response Function (IRF) Analysis: Detailed IRF analysis of inflation’s
    response to multiple economic shocks.
- Cumulative Impact of Shocks: Track the long-term cumulative effect of cost-push
    shocks on inflation.


### Diagnostics & Visualization

- IRF Plotting: Visualize inflation’s response to different economic shocks over time.
- Forecast vs Actual Plotting: Compare the model’s forecasted values against the
    actual data.
- RMSE Evaluation: Quantitative evaluation of forecast accuracy using RMSE met-
    rics.
Script:SVAR_baseline.py
Functions Module:Functions.py
This baseline scenario estimates a Structural Vector Autoregression (SVAR) model to
examine the effects of various macroeconomic shocks on inflation. It employs sign-restricted
identification, impulse response function (IRF) analysis, and out-of-sample forecasting.

### Key Features

- Data Preparation: Monthly macroeconomic time series are loaded from Excel and
    standardized.
- Variables:
    - Brent_logdiff (Oil price)
    - Gas Prices_logdiff (Gasoline price)
    - Growth rate inflation
    - DSPIC96_log_diff (Real disposable income)
    - EUR-USD_logdiff (Exchange rate)
- Estimation Window:January 2015 to January 2024
- Model:Reduced-form VAR(1), followed by structural decomposition

### Workflow Summary

1. Data Loading:Excel file is read and filtered by date and selected columns.
2. Preprocessing: Time series are checked for stationarity and standardized.
3. VAR Estimation: A reduced-form VAR model is estimated.
4. Structural Identification:
    - Sign restrictions are imposed using an optimized Cholesky matrix.
    - Structural shocks are derived based on the identified structure.
5. Impulse Response Functions (IRFs):


- IRFs of inflation to selected shocks (Brent, Gas, Income, FX)
- Cumulative IRF for cost-push shocks (Brent + Gas)
6. Forecasting:
- Rolling-window VAR forecasting (window size = 95, lag = 1)
- Forecasts are compared with actual data
7. Evaluation: Root Mean Squared Error (RMSE) is computed.

### Output

- Structural matrix (optimized Cholesky decomposition)
- Long-run impact matrixC(∞)·D
- Forecast vs. actual plots for inflation
- RMSE of forecasted inflation

### Related Scenarios

- Scenario 2 (Demand-Only): SVAR_demand_only.pywithFunctions_D3.py
- Scenario 3 (Full Specification):SVAR_called_large.pywithFunctions_total.py

## 3 Recursive Bayesian Structural VAR

This script utilizes theBayesianSVARmodule to estimate Bayesian Structural VAR mod-
els with sign (positivity) restrictions and perform rolling forecasts. The model imposes
structural identification via positive impulse responses over a specified horizon.
Key features include:

- Full Bayesian estimation using PyMC.
- Positive sign restrictions on impulse response functions (IRFs).
- Rolling-window forecast with structural identification.
- Posterior summary and IRF visualization.


### Dependencies

The following Python libraries are required:

- pymc: Probabilistic programming and MCMC.
- numpy: Core numerical operations.
- arviz: MCMC diagnostics and summarization.
- matplotlib: Plotting IRFs and forecasts.
- pandas: Data handling and transformation.
- pytensor: Tensor operations backend for PyMC.
- scikit-learn: Data normalization (StandardScaler).
- BayesianSVAR: Data normalization (StandardScaler).

### Dependencies(BayesianSVAR)

The following Python libraries are required:

- pymc: Probabilistic programming and Bayesian modeling.
- numpy: Core numerical computations and array operations.
- arviz: Diagnostics and visualization for Bayesian models.
- matplotlib: Plotting and visualization library.
- pandas: Data manipulation and analysis.
- pytensor: Tensor computation backend used by PyMC.
- numpy.array_api: API for array operations within thenumpyecosystem.
- numpy.ma.core: Support for masked arrays innumpy.
- scikit-learn: Machine learning tools, includingsklearn.preprocessingfor scaling
    data andsklearn.metricsfor evaluating models.


### Workflow Description

1. Data Loading

```
columns = [
’Brent_logdiff ’,
’Gas Prices_logdiff ’,
’growth_rate_inflation ’,
’DSPIC96_log_diff ’,
’EUR -USD_logdiff ’,
]
file_path = r’C:\Users\acer\PycharmProjects\PreprocessingOfData\
Stationary_data_mine (15).xlsx’
df = BayesianSVAR.load_data(file_path)
```
2. Filtering and Scaling

```
filtered_df = BayesianSVAR.filter_data(df, ’2015 -01’, ’2024 -01’,
columns)
filtered_df = BayesianSVAR.scale_columns(filtered_df , columns)
```
3. Model Specification with Positivity Restrictions

```
data_array = filtered_df.values.astype(np.float64)
horizon = 10
model = BayesianSVAR.define_svar_model(data_array , horizon=horizon ,
impose_all_positive=True)
```
4. MCMC Sampling

```
with model:
trace = pm.sample (500, tune =500, target_accept =0.9,
return_inferencedata=True , random_seed =42)
```
5. Posterior Summary

```
summary = az.summary(trace , var_names =["B", "chol_det"])
print(summary)
```

6. IRF Visualization

```
posterior_B = trace.posterior["B"]. values
posterior_chol = trace.posterior["chol_det"]. values
combined_B = posterior_B.reshape(-1, n_vars , n_vars)
combined_chol = posterior_chol.reshape(-1, n_vars , n_vars)
BayesianSVAR.plot_irfs_combined(n_samples , combined_B , combined_chol ,
2, [0, 1, 3, 4], horizon , columns)
```
7. Rolling Forecast with Bayesian SVAR

```
results = BayesianSVAR.rolling_svar_forecast_bayesian_steps(
filtered_df ,
window_size =94,
steps_ahead =1,
impose_all_positive=True ,
num_samples =500,
num_tune =
)
```
8. Forecast Visualization and Evaluation

```
BayesianSVAR.plot_forecasts_with_ci(results , filtered_df , ’
growth_rate_inflation ’, var_index =2)
rmse = BayesianSVAR.compute_rmse_rolling(results , filtered_df , ’
growth_rate_inflation ’)
print(f"RMSE for ’growth rate inflation ’: {rmse :.4f}")
```
### Function Reference

- load_data(filepath): Load and preprocess data from Excel.
- filter_data(df, start, end, columns): Select a date range and relevant variables.
- scale_columns(df, columns): Standardize data using sklearn’s StandardScaler.
- define_svar_model(data, horizon, impose_all_positive): Create a PyMC model
    with IRF sign restrictions.
- plot_irfs_combined(...): Visualize IRFs across posterior samples.
- rolling_svar_forecast_bayesian_steps(...): Perform rolling-window SVAR es-
    timation and forecasting.
- plot_forecasts_with_ci(...): Plot forecast means and credible intervals.
- compute_rmse_rolling(...): Compute RMSE of rolling forecasts.


### Advanced Features

- All-Positive Restrictions: Ensures all impulse responses are positive up to a defined
    horizon.
- Full Bayesian Inference: MCMC sampling for posterior estimation.
- Rolling Forecasting: Tracks structural changes over time via recursive re-estimation.

### Diagnostics & Visualization

- Posterior Summary: Mean, standard deviation, and credible intervals for structural
    parameters.
- Impulse Response Plots: Posterior sample-based IRFs with imposed restrictions.
- Forecast Charts: Actuals vs predicted trajectories with 95% credible bands.

## 4 Non-recursive Bayesian Structural VAR

The BSVAR package implements Bayesian Structural Vector Autoregression (SVAR)
models with Minnesota priors and sign restrictions. It allows users to:

- Estimate impulse response functions (IRFs)
- Incorporate economic theory through sign restrictions.
- Perform rolling forecasts and assess out-of-sample performance.
This readme outlines dependencies, core functionality, and a basic workflow for using the
BSVAR pipeline.

### Dependencies

The following Python packages are required:

- numpy: Numerical computations.
- pymc: Bayesian inference and MCMC sampling.
- arviz: MCMC diagnostics and posterior visualization.
- pandas: Data manipulation and analysis.
- matplotlib: Visualization.
- scikit-learn: Data standardization.
- pytensor: Computational graph support (used by PyMC).
- BSVAR_functions: a module that provides helper functions required by the main mod-
    ule.


### Dependencies(BSVAR_functions)

The following Python libraries are required:

- numpy: Core numerical computations and array operations.
- pandas: Data manipulation and analysis.
- scipy.stats: Statistical functions, includinginvwishartandmultivariate_normal
    distributions.
- numpy.linalg: Linear algebra operations, includinginv(matrix inversion) andcholesky
    (Cholesky decomposition).
- matplotlib: Plotting and visualization library.
- scikit-learn: Machine learning tools, includingStandardScalerfor data normal-
    ization andmean_squared_errorfor model evaluation.

### Basic Workflow

1. Configuration and Data Import

```
columns = [
’Brent_logdiff ’,
’Gas Prices_logdiff ’,
’growth_rate_inflation ’,
’DSPIC96_log_diff ’,
’EUR -USD_logdiff ’,
]
file_path = "Stationary_data_mine.xlsx"
data , df = BSVAR_functions.load_data(file_path , columns)
```
2. Filtering and Scaling

```
filtered_df = BSVAR_functions.filter_data(df, "2015 -01", "2024 -01",
columns)
filtered_df = BSVAR_functions.scale_columns(filtered_df , columns)
```
3. Prior Setup and Data Preparation

Y, X = BSVAR_functions.prepare_data(filtered_df.to_numpy (), p=1)
B_prior , V_prior = BSVAR_functions.set_minnesota_priors(filtered_df.
to_numpy (), k=len(columns), p=1)


4. Sign Restrictions and MCMC Sampling

```
sign_restrictions = {
’target ’: columns.index("growth_rate_inflation"),
**dict(enumerate(np.ones(len(columns))))
}
B_draws , Sigma_draws , P_draws , irfs , fevd = BSVAR_functions.run_mcmc(
Y, X, B_prior , V_prior , len(columns), p=1, irf_horizon =15,
sign_restrictions=sign_restrictions
)
```
5. Summarization and Visualization

```
mean_B , mean_Sigma , mean_P , mean_irfs , mean_fevd = BSVAR_functions.
summarize_posteriors(
B_draws , Sigma_draws , P_draws , irfs , fevd , columns , p=
)
BSVAR_functions.plot_irfs(mean_irfs , lower_irfs , upper_irfs ,
inflation_idx , k, columns)
BSVAR_functions.plot_fevd(mean_fevd , lower_fevd , upper_fevd ,
inflation_idx , k, columns)
```
6. Rolling Forecast

```
rolling_results = BSVAR_functions.
rolling_svar_forecast_bayesian_steps(
data=filtered_df ,
window_size =90,
steps_ahead =6,
irf_horizon =15,
num_draws =500,
p=1,
columns=columns
)
BSVAR_functions.plot_forecasts_with_ci(rolling_results , filtered_df ,
’growth_rate_inflation ’, var_index =2)
```
### Function Reference

- load_data(filepath, columns): Read and preprocess data from Excel.
- filter_data(df, start, end, columns): Subset and clean the data.


- scale_columns(df, columns): Standardize variables usingStandardScaler.
- prepare_data(data, p): Format data into lagged structures for VAR modeling.
- set_minnesota_priors(data, k, p): Define Bayesian priors on coefficient and vari-
    ance matrices.
- run_mcmc(...): Perform MCMC sampling under sign restrictions.
- summarize_posteriors(...): Compute posterior means and diagnostic summaries.
- rolling_svar_forecast_bayesian_steps(...): Conduct rolling-window forecasts.
- compute_rmse_rolling(...): Evaluate RMSE of forecasts against actuals.

### Advanced Features

- Sign Restrictions: Impose theory-driven response signs during identification of shocks.
- Rolling Forecast: Capture structural shifts and time-varying dynamics with rolling
    estimation.
- Credible Intervals: Compute 95% credible intervals for IRFs and FEVDs using
    posterior draws.
- Posterior Diagnostics: Access draws for B,Σ, and P matrices for deeper inspection.

### Diagnostics & Visualization

- IRF Plotting: Plot posterior mean IRFs with credible bands.
- Forecast Visualization: Overlay forecasts and confidence bands on actual series.

## 5 Dynamic Factor Augmented VAR Model (DFAVAR)

### Overview

The DFM package implements dynamic factor models for multivariate time series analysis.
It enables users to:

- Extract latent factors that drive co-movements in macroeconomic or financial variables.
- Compute and visualize Impulse Response Functions (IRFs).
- Generate forecasts and assess out-of-sample performance.
This readme provides installation instructions, usage examples, and a brief description
of core functions.


### Dependencies

The following packages are required:

- readxl: Import Excel data files.
- dplyr: Data manipulation.
- expm: Matrix exponential and powers (for IRF computation).
- reshape2: Data reshaping for IRF plotting.
- ggplot2: Visualization.
- dfms: Core model functions (this package).

### Basic Workflow

Below is a typical analysis pipeline using DFM.

1. Data Preparation

```
library(readxl)
library(dplyr)
# Load and clean monthly time series data
data <- read_excel ("data/my_series.xlsx")
data$Months <- as.Date(paste0(data$Months , "-01"))
data <- data %>% filter (!is.na(Months)) %>% arrange(Months)
```
2. Train/Test Split

```
# Remove initial observations if needed
data <- data [ -(1:3), ]
train_end <- as.Date ("2018 -12 -01")
train_data <- data %>% filter(Months <= train_end)
test_data <- data %>% filter(Months > train_end)
```
3. Model Estimation

```
library(dfms)
# Fit a dynamic factor model with 2 factors and 2 lags
model <- dfm(
data = train_data ,
factors = 2,
```

```
lags = 2
)
```
4. Impulse Response Functions (IRFs)

```
# Compute IRF: effect of a shock in Brent prices on inflation
irf_res <- irf(
object = model ,
impulse = "Brent_logdiff",
response = "growth rate inflation",
n.ahead = 24
)
# Plot IRF
plot(irf_res)
```
5. Forecasting

```
# Forecast next 24 months
fcst <- forecast(
object = model ,
h = 24
)
plot(fcst)
```
### Function Reference

- dfm(data, factors, lags, ...): Fit a dynamic factor model to your multivariate
    time series.
- irf(object, impulse, response, n.ahead, ...): Compute impulse response func-
    tions for selected shock/response pairs.
- forecast(object, h, ...): Generate multi-step forecasts from the fitted DFM.

### Advanced Features

- Model Selection: Automatic criteria (AIC, BIC) can help choose the optimal number
    of factors and lags via theselect_model()function.
- Custom Factor Loadings: Use theloadings_priorargument indfm()to impose
    sign or scale constraints on specific series.
- Rolling Window Estimation: Evaluate time-varying dynamics withrolling_dfm()
    to fit models on moving sub-samples.


- Parallel Computation: Speed up estimation by settingn.cores > 1in functions
    likedfm()andirf().

### Diagnostics & Visualization

- Model Fit: Examine standardized residuals and Ljung–Box tests viacheck_residuals(model).
- Variance Decomposition: Usefevd(model, n.ahead)to compute forecast error
    variance contributions of each shock.
- Plotting Utilities:
    - plot_factors(model)– visualize estimated latent factors over time.
    - plot_loadings(model)– heatmap of series loadings on each factor.
    - plot_fevd(fevd_res)– bar chart of variance shares at selected horizons.



