
# CAPM Analysis

## Introduction

In this assignment, you will explore the foundational concepts of the Capital Asset Pricing Model (CAPM) using historical data for AMD and the S&P 500 index. This exercise is designed to provide a hands-on approach to understanding how these models are used in financial analysis to assess investment risks and returns.

## Background

The CAPM provides a framework to understand the relationship between systematic risk and expected return, especially for stocks. This model is critical for determining the theoretically appropriate required rate of return of an asset, assisting in decisions about adding assets to a diversified portfolio.

## Objectives

1. **Load and Prepare Data:** Import and prepare historical price data for AMD and the S&P 500 to ensure it is ready for detailed analysis.
2. **CAPM Implementation:** Focus will be placed on applying the CAPM to examine the relationship between AMD's stock performance and the overall market as represented by the S&P 500.
3. **Beta Estimation and Analysis:** Calculate the beta of AMD, which measures its volatility relative to the market, providing insights into its systematic risk.
4. **Results Interpretation:** Analyze the outcomes of the CAPM application, discussing the implications of AMD's beta in terms of investment risk and potential returns.

## Instructions

### Step 1: Data Loading

- We are using the `quantmod` package to directly load financial data from Yahoo Finance without the need to manually download and read from a CSV file.
- `quantmod` stands for "Quantitative Financial Modelling Framework". It was developed to aid the quantitative trader in the development, testing, and deployment of statistically based trading models.
- Make sure to install the `quantmod` package by running `install.packages("quantmod")` in the R console before proceeding.

```r
# Set start and end dates
start_date <- as.Date("2019-05-20")
end_date <- as.Date("2024-05-20")

# Load data for AMD, S&P 500, and the 1-month T-Bill (DTB4WK)
amd_data <- getSymbols("AMD", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
gspc_data <- getSymbols("^GSPC", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
rf_data <- getSymbols("DTB4WK", src = "FRED", from = start_date, to = end_date, auto.assign = FALSE)

# Convert Adjusted Closing Prices and DTB4WK to data frames
amd_df <- data.frame(Date = index(amd_data), AMD = as.numeric(Cl(amd_data)))
gspc_df <- data.frame(Date = index(gspc_data), GSPC = as.numeric(Cl(gspc_data)))
rf_df <- data.frame(Date = index(rf_data), RF = as.numeric(rf_data[,1]))  # Accessing the first column of rf_data

# Merge the AMD, GSPC, and RF data frames on the Date column
df <- merge(amd_df, gspc_df, by = "Date")
df <- merge(df, rf_df, by = "Date")
```

#### Data Processing 
```r
# Data processing is conducted to account for the non-daily reporting of the RF rate. Processing used to fill down missed dates
colSums(is.na(df))
# Fill N/A RF data
df <- df %>%
  fill(RF, .direction = "down")
```

### Step 2: CAPM Analysis

The Capital Asset Pricing Model (CAPM) is a financial model that describes the relationship between systematic risk and expected return for assets, particularly stocks. It is widely used to determine a theoretically appropriate required rate of return of an asset, to make decisions about adding assets to a well-diversified portfolio.

#### The CAPM Formula
The formula for CAPM is given by:

$$
E(R_i) = R_f + \beta_i (E(R_m) - R_f)
$$

Where:

- $E(R_i)$ is the expected return on the capital asset,
- $R_f$ is the risk-free rate,
- $\beta_i$ is the beta of the security, which represents the systematic risk of the security,
- $E(R_m)$ is the expected return of the market.



#### CAPM Model Daily Estimation

- **Calculate Returns**: First, we calculate the daily returns for AMD and the S&P 500 from their adjusted closing prices. This should be done by dividing the difference in prices between two consecutive days by the price at the beginning of the period.
  
$$
\text{Daily Return} = \frac{\text{Today's Price} - \text{Previous Trading Day's Price}}{\text{Previous Trading Day's Price}}
$$

```r
# Initialise columns for daily return of AMD and the S&P500 index based on adjusted closing price
df$AMD_DailyReturns <- NA
df$SP500_DailyReturns <- NA

# Calculate daily return for AMD and the S&P500 Index using a for loop function
for (i in 2:nrow(df)) {
  df$AMD_DailyReturns[i] <- (df$AMD[i] - df$AMD[i-1])/df$AMD[i-1]
  df$SP500_DailyReturns[i] <- (df$GSPC[i] - df$GSPC[i-1])/df$GSPC[i-1]
}
```

- **Calculate Risk-Free Rate**: Calculate the daily risk-free rate by conversion of annual risk-free Rate. This conversion accounts for the compounding effect over the days of the year and is calculated using the formula:
  
$$
\text{Daily Risk-Free Rate} = \left(1 + \frac{\text{Annual Rate}}{100}\right)^{\frac{1}{360}} - 1
$$

```r
# Initialise column for Daily Risk-Free Rate
df$Daily_RF <- NA

# Calculate Daily Risk-Free Rate using a for loop function
for (i in 1:nrow(df)) {
  df$Daily_RF[i] <- (1+(df$RF[i]/100))^(1/360)-1
}
```


- **Calculate Excess Returns**: Compute the excess returns for AMD and the S&P 500 by subtracting the daily risk-free rate from their respective returns.

```r
# Initialise columns for excess returns of AMD and S&P500 Index over the daily risk-free rate
df$AMD_ExcessReturn <- NA
df$SP500_ExcessReturn <- NA

# Calculate Daily Excess Returns for AMD and the S&P500 index using a for loop function
for (i in 2:nrow(df)) {
  df$AMD_ExcessReturn[i] <- df$AMD_DailyReturn[i] - df$Daily_RF[i] 
  df$SP500_ExcessReturn[i] <- df$SP500_DailyReturn[i] - df$Daily_RF[i]
}
```


- **Perform Regression Analysis**: Using linear regression, we estimate the beta (\(\beta\)) of AMD relative to the S&P 500. Here, the dependent variable is the excess return of AMD, and the independent variable is the excess return of the S&P 500. Beta measures the sensitivity of the stock's returns to fluctuations in the market.

```r
# Fit linear model to data
CAPM_model <- lm(df$AMD_ExcessReturn ~ df$SP500_ExcessReturn, data = df)
summary(CAPM_model)

# Extract beta
beta <- coef(CAPM_model)["df$SP500_ExcessReturn"]

# Print beta using concatenate function 
cat("AMD 5yr beta is equal to:", round(beta, 2), "\n")

```


#### Interpretation

What is your \(\beta\)? Is AMD more volatile or less volatile than the market?

AMD's beta is approximately 1.57. This result means that AMD is more volatile than the S&P500 index. A stock with a beta of 1.00 means that it's price moves in exact alignment with the market index. Therefore, a stock with a beta of 1.57 means that for every 1.00% of movement in the market index, the stock will move 1.60% in the same direction. 

This high beta is expected, as AMD operates in the technology sector, which is known for its high volatility due to rapid innovation, changing consumer preferences and significant competition. AMD competes against major players such as NVIDIA and Intel. Furthermore, institutional investors place emphasis on growth when valuing technology stocks such as AMD. Therefore, when the market is volatile and investor sentiment sours, the price of stock's such as AMD tend to decline rapidly, or vice-versa.

AS AMD is more volatile than the market index, an investor can expect higher potential returns as compensation. This is illuminated by the CAPM model, which suggests that an investors expected return can be calculated as a product of beta and the excess return of the market over the risk-free rate.


#### Plotting the CAPM Line
Plot the scatter plot of AMD vs. S&P 500 excess returns and add the CAPM regression line.

```r
# Create dataframe to be plotted which omits missing values (NA); the first values in the df dataframe
df_plot <- na.omit(df)

# Plot S&P500 excess returns against AMD excess returns. Add smooth line to reflect intercept
ggplot(mapping = aes(x = df_plot$SP500_ExcessReturn, y = df_plot$AMD_ExcessReturn), date = df) + 
  geom_point() + 
  geom_smooth(method = "lm", se = FALSE) +
  labs (title = "S&P500 vs AMD excess returns", x = "S&P500 excess returns", y = "AMD excess returns")
```

### Step 3: Predictions Interval
Suppose the current risk-free rate is 5.0%, and the annual expected return for the S&P 500 is 13.3%. Determine a 90% prediction interval for AMD's annual expected return.



**Answer:**

```r
# Calculate number of observations in df dataframe
n <- nrow(df)

# Take mean of S&P500 returns
SP500_MeanExcessReturn <- mean(df$SP500_ExcessReturn, na.rm = TRUE)

# Find standard error of estimate
se <- sqrt(sum(residuals(CAPM_model)^2)/(n-2))

# Take sum of squares of the S&P500 excess returns
SSX <- sum((df$SP500_ExcessReturn-SP500_MeanExcessReturn)^2, na.rm = TRUE)

# Find expected S&P500 excess returns
SP500_ExpExcessReturn <- 0.133 - 0.05

# Take standard error of the forecast
sf <- se*sqrt(1+1/n+(SP500_ExpExcessReturn-SP500_MeanExcessReturn)^2/SSX)

# Calculate the expected return using the CAPM model
rf_current <- 0.05
SP500_ExpReturn <- 0.133
AMD_ExpReturn <- rf_current + beta*(SP500_ExpReturn-rf_current)

# Calculate t-value for 90% confidence interval
alpha <- 0.10
t_value <- qt(1-alpha/2, df = n-2)

# Calculate prediction interval
lower_bound <- AMD_ExpReturn-t_value*sf*sqrt(252)
upper_bound <- AMD_ExpReturn+t_value*sf*sqrt(252)

# Print results using concatenate function

cat("A 90% prediction interval for AMD's annual expected return: \n")
cat("Lower Bound:", round(lower_bound, 4), "\n")
cat("Upper Bound:", round(upper_bound,4), "\n")
```
