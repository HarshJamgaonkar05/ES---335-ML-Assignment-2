# Question 3: Working with Autoregressive Modeling [2 Marks]

### Overview

I implemented autoregressive (AR) models using linear regression to forecast daily minimum temperatures from the dataset. The approach involves using previous temperature values as features to predict the next day's temperature. This is auto regressive modeling.

### Autoregressive Model Implementation

#### Model Architecture
I implemented three autoregressive models:
- **AR(1):** Uses previous 1 day's temperature → T(t) = β₀ + β₁×T(t-1) + ε
- **AR(2):** Uses previous 2 days' temperatures → T(t) = β₀ + β₁×T(t-1) + β₂×T(t-2) + ε  
- **AR(3):** Uses previous 3 days' temperatures → T(t) = β₀ + β₁×T(t-1) + β₂×T(t-2) + β₃×T(t-3) + ε

- **Train-Test Split:** 80% training (2,919 samples), 20% testing (730 samples) [Without changing the order to maintain temporal order as it is time series data]


### Results and Performance

#### Model Performance (RMSE Values)
Based on the implementation:
- **AR(1):** Train RMSE ≈ 2.6191, Test RMSE ≈ 2.3767
- **AR(2):** Train RMSE ≈ 2.6118, Test RMSE ≈ 2.3689  
- **AR(3):** Train RMSE ≈ 2.5649, Test RMSE ≈ 2.3224

**Best Model:** AR(3) with the lowest test RMSE of approximately 2.3224

#### Model Coefficients Analysis
The AR models learned meaningful relationships:
- **Positive coefficients** for recent temperature lags indicate temperature persistence
- **Decreasing influence** as lag increases (recent temperatures matter more)
- **Reasonable intercepts** close to the mean temperature

### Visualization Analysis

#### Forecasting Plots
1. **Comparison Plot:** Shows all three AR models' performance side by side
2. **Main Forecasting Plot:** Displays the best model (AR(3)) with:
   - Training data (light blue background)
   - True test values (blue line)
   - Predicted values (red line)  
   - Clear train/test separation

#### Key Observations from Plots
- **Good fit:** Predictions closely follow true temperature patterns
- **Captures trends:** Model successfully captures both short-term fluctuations and longer patterns

### Inferences

#### AR(3) performed the best as:
1. **More context:** Uses 3 previous days providing richer temporal information
2. **Better pattern recognition:** Captures short-term weather patterns better
3. **Balanced complexity:** Neither simple nor overly complex

#### Model Strengths
- **Simplicity:** Linear regression is interpretable and computationally efficient
- **Temporal variation:** Successfully captures day-to-day temperature correlations

#### One limitation might be that:
- This model onl takes into consideration the temperatur values of the previous days to predict. Other factors affecting temp are lost.

### Conclusion

The autoregressive approach successfully demonstrates temperature forecasting using historical data. AR(3) achieved the best performance with RMSE of 2.3224°C, proving that recent temperature history contains valuable predictive information. 
