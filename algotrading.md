
## Algorithmic Trading Strategy

## Introduction

In this assignment, you will develop an algorithmic trading strategy by incorporating financial metrics to evaluate its profitability. This exercise simulates a real-world scenario where you, as part of a financial technology team, need to present an improved version of a trading algorithm that not only executes trades but also calculates and reports on the financial performance of those trades.

## Background

Following a successful presentation to the Board of Directors, you have been tasked by the Trading Strategies Team to modify your trading algorithm. This modification should include tracking the costs and proceeds of trades to facilitate a deeper evaluation of the algorithm’s profitability, including calculating the Return on Investment (ROI).

After meeting with the Trading Strategies Team, you were asked to include costs, proceeds, and return on investments metrics to assess the profitability of your trading algorithm.

## Objectives

1. **Load and Prepare Data:** Open and run the starter code to create a DataFrame with stock closing data.

2. **Implement Trading Algorithm:** Create a simple trading algorithm based on daily price changes.

3. **Customize Trading Period:** Choose your entry and exit dates.

4. **Report Financial Performance:** Analyze and report the total profit or loss (P/L) and the ROI of the trading strategy.

5. **Implement a Trading Strategy:** Implement a trading strategy and analyze the total updated P/L and ROI. 

6. **Discussion:** Summarise your finding.


## Instructions

### Step 1: Data Loading

Start by running the provided code cells in the "Data Loading" section to generate a DataFrame containing AMD stock closing data. This will serve as the basis for your trading decisions. First, create a data frame named `amd_df` with the given closing prices and corresponding dates. 

```r
# Load data from CSV file
amd_df <- read.csv("AMD.csv")
# Convert the date column to Date type and Adjusted Close as numeric
amd_df$date <- as.Date(amd_df$Date)
amd_df$close <- as.numeric(amd_df$Adj.Close)
amd_df <- amd_df[, c("date", "close")]
```

#### Plotting the Data
Plot the closing prices over time to visualize the price movement.
```r
# AMD share price data is plotted do that price growth and market cycles can be analysed. Dat
e is plotted on the horizontal axis, closing price is plotted on the vertical axis.
plot(amd_df$date, amd_df$close,'l')
```

### Step 2: Trading Algorithm
Implement the trading algorithm as per the instructions. You should initialize necessary variables, and loop through the dataframe to execute trades based on the set conditions.

- Initialize Columns: Start by ensuring dataframe has columns 'trade_type', 'costs_proceeds' and 'accumulated_shares'.
- Change the algorithm by modifying the loop to include the cost and proceeds metrics for buys of 100 shares. Make sure that the algorithm checks the following conditions and executes the strategy for each one:
  - If the previous price = 0, set 'trade_type' to 'buy', and set the 'costs_proceeds' column to the current share price multiplied by a `share_size` value of 100. Make sure to take the negative value of the expression so that the cost reflects money leaving an account. Finally, make sure to add the bought shares to an `accumulated_shares` variable.
  - Otherwise, if the price of the current day is less than that of the previous day, set the 'trade_type' to 'buy'. Set the 'costs_proceeds' to the current share price multiplied by a `share_size` value of 100.
  - You will not modify the algorithm for instances where the current day’s price is greater than the previous day’s price or when it is equal to the previous day’s price.
  - If this is the last day of trading, set the 'trade_type' to 'sell'. In this case, also set the 'costs_proceeds' column to the total number in the `accumulated_shares` variable multiplied by the price of the last day.



```r
# trade-type, costs_proceeds and accumulated_shares columns were initialized so that the algo
rithm could be implemented.
# Initialize columns for trade type, cost/proceeds, and accumulated shares in amd_df
amd_df$trade_type <- NA
amd_df$costs_proceeds <- NA # Corrected column name
amd_df$accumulated_shares <- 0 # Initialize if needed for tracking
# A variable for share_size was created and set to 100 so that each transaction is of a stand
ard size and according to the algorithms outline. A start_date and end_date variable was crea
ted so that the trading period could easily be adjusted. Initially, start_date and end_date a
re set so that the whole dataset is captured.
share_size <- 100
accumulated_shares <- 0
start_date <- 1
end_date <- nrow(amd_df)
# A for loop is used to ensure that each of the scenario's outlined in the trading algorithm
brief are captured by the algorithm. The for loop using a variable 'i', which loops for each
datapoint between the start_date and end_date.
for (i in start_date:end_date) {

 # AMD shares should always be bought on the first data point of the trading period, i = st
art_date. As such, trade_type is set to buy, and costs_proceeds is set to the clos
ing price multiplied by the share_size of 100 (as a negative to reflect money flowing out). A
ccumulated_shares starts at the value of the share_size, 100.
 if (i == start_date) {
 amd_df$trade_type[i] <- "Buy"
 amd_df$costs_proceeds[i] <- -amd_df$close[i] * share_size
 amd_df$accumulated_shares[i] <- share_size

 # In contrast, all AMD shares should be sold on the last data point of the trading perio
d, i = end_date. As such, trade_type is set to sell. It is assumed that no more shares will b
e bought on the final day. Hence, the value of accumulated_shares is set to that of the previ
ous day. This value is multiplied by AMD_df's closing price to return the proceeds from the s
ale.
 } else if (i == end_date) {

 amd_df$trade_type[i] <- "Sell"
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1]
 amd_df$costs_proceeds[i] <- amd_df$close[i] * amd_df$accumulated_shares[i]

 # If the closing price of the current data point is less than the closing price of the pr
evious data point, i.e. AMD's share price has fallen since the previous day, then a Buy trade
is executed. 100 shares are bought at the current closing price (a negative of the current cl
osing price is used to reflect money flowing out) and the accumulated_shares value increases
by 100.
 } else if (amd_df$close[i] < amd_df$close[i - 1]) {
 amd_df$trade_type[i] <- "Buy"
 amd_df$costs_proceeds[i] <- -amd_df$close[i] * 100
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1] + 100

 # If all other conditions are not met, trade_type and costs_proceeds are not updated. Fur
ther, as no more shares have been purchased, the accumulated share price is left at the same
value of the previous data point.
 } else {
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1]
 }
}

```


### Step 3: Customize Trading Period
- Define a trading period you wanted in the past five years 
```r
# Data-frame values were reset so that data not included in the specific trading period do no
t impact future ROI and profitability calculations.
amd_df$trade_type <- NA
amd_df$costs_proceeds <- NA
amd_df$accumulated_shares <- NA
# The selected start and end dates were selected. The index values for the start and end date
s were found using the which function so that they could be used within the for loop.
start_date <- which(amd_df$date == "2023-01-31")
end_date <- which(amd_df$ date == "2024-05-17")
# No changes were made to the original trading algorithm.
for (i in start_date:end_date) {

 if (i == start_date) {
 amd_df$trade_type[i] <- "Buy"
 amd_df$costs_proceeds[i] <- -amd_df$close[i] * 100
 amd_df$accumulated_shares[i] <- 100

 } else if (i == end_date) {

 amd_df$trade_type[i] <- "Sell"
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1]
 amd_df$costs_proceeds[i] <- amd_df$close[i] * amd_df$accumulated_shares[i]

 } else if (amd_df$close[i] < amd_df$close[i - 1]) {
 amd_df$trade_type[i] <- "Buy"
 amd_df$costs_proceeds[i] <- -amd_df$close[i] * 100
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1] + 100

 } else {
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1]
 }
}

```


### Step 4: Run Your Algorithm and Analyze Results
After running your algorithm, check if the trades were executed as expected. Calculate the total profit or loss and ROI from the trades.

- Total Profit/Loss Calculation: Calculate the total profit or loss from your trades. This should be the sum of all entries in the 'costs_proceeds' column of your dataframe. This column records the financial impact of each trade, reflecting money spent on buys as negative values and money gained from sells as positive values.
- Invested Capital: Calculate the total capital invested. This is equal to the sum of the 'costs_proceeds' values for all 'buy' transactions. Since these entries are negative (representing money spent), you should take the negative sum of these values to reflect the total amount invested.
- ROI Formula: $$\text{ROI} = \left( \frac{\text{Total Profit or Loss}}{\text{Total Capital Invested}} \right) \times 100$$

```r
# Total profitability was calculated by taking the sum of the costs_proceeds column - noting
that all BUY trades were denoted as negative values to reflect as cash outflows.
profit <- sum(amd_df$costs_proceeds, na.rm = TRUE)
print("Profitability:")

# ROI was calculated as the net return from investment divided by the cost of investment
InvCost <- sum(amd_df$costs_proceeds[start_date: end_date - 1], na.rm = TRUE)
Net_Return <- sum(amd_df$costs_proceeds[end_date])
ROI <- (Net_Return/(InvCost*-1) - 1)*100
print("ROI:")
```

### Step 5: Profit-Taking Strategy or Stop-Loss Mechanisum (Choose 1)
- Option 1: Implement a profit-taking strategy that you sell half of your holdings if the price has increased by a certain percentage (e.g., 20%) from the average purchase price.
- Option 2: Implement a stop-loss mechanism in the trading strategy that you sell half of your holdings if the stock falls by a certain percentage (e.g., 20%) from the average purchase price. You don't need to buy 100 stocks on the days that the stop-loss mechanism is triggered.


```r
# A stop_loss_limit variable was created to set the maximum day-on-day loss before half of th
e accumulated shares were to be sold.
stop_loss_limit <- -0.03
for (i in start_date:end_date) {

 if (i == start_date) {
 amd_df$trade_type[i] <- "Buy"
 amd_df$costs_proceeds[i] <- -amd_df$close[i] * 100
 amd_df$accumulated_shares[i] <- 100
 } else if (i == end_date) {
 amd_df$trade_type[i] <- "Sell"
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1]
 amd_df$costs_proceeds[i] <- amd_df$close[i] * amd_df$accumulated_shares[i]

 # An additional column to implement the stop-loss mechanism was added. If the % drop of t
he current closing price from the previous closing price was greater than 3.00%, then hal
f of AMD's shares were sold. The accumulated_shares were also halved to reflect this change i
n holdings.
 } else if (amd_df$close[i]/amd_df$close[i-1] - 1 <= stop_loss_limit) {
 amd_df$trade_type[i] <- "Sell"
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i-1]/2
 amd_df$costs_proceeds[i] <- amd_df$close[i]*(amd_df$accumulated_shares[i-1]/2)

 } else if (amd_df$close[i] < amd_df$close[i - 1]) {
 amd_df$trade_type[i] <- "Buy"
 amd_df$costs_proceeds[i] <- -amd_df$close[i] * 100
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1] + 100

 } else {
 amd_df$accumulated_shares[i] <- amd_df$accumulated_shares[i - 1]
 }
}

```


### Step 6: Summarize Your Findings
- Did your P/L and ROI improve over your chosen period?
- Relate your results to a relevant market event and explain why these outcomes may have occurred.


```r
# Calculation of Profitability - sum of costs proceeds column
profit <- sum(amd_df$costs_proceeds, na.rm = TRUE)
print("Profitability:")
print(profit)

# Calculation of ROI - SALE revenue divided by share BUY costs.
InvCost <- sum(amd_df$costs_proceeds[start_date: end_date - 1], na.rm = TRUE)
Net_Return <- sum(amd_df$costs_proceeds[end_date])
ROI <- (Net_Return/(InvCost*-1) - 1)*100
print("ROI:")
print(ROI)

```

Using a stop-less mechanism, setting the maximum day-on-day percentage loss to -3.00% increased ROI by
188.63%, at a profit of $73,459. Between the 11/03/2024 and the final data point of 17/05/2024, AMD
experienced a significant amount of growth due to industry tailwinds and performance. However, its position as
a high-growth tech stock also means that its share price is more susceptible to volatility and market concerns.
A -3.00% stop-loss mechanism ensures that the trading strategy is robust and can manage this volatility. For
instance, on the 15th of February 2023, Federal Reserve Chairman Jerome Powell indicated that the central
bank would need more evidence of easing inflation before cutting interest rates. Accordingly, AMD’s share price
dropped by 6.00% from $85.18 to $80.08 on the 16th of February. Its share price did not recover and move
back to ~ $85 until the 8th of March. The stop-loss mechanism’s ‘SELL’ trade on the 16th of February as a
result of the 6.00% decline allowed the algorithm to cash in on the share price before it declined further, whilst
also enabling it to buy at the lower price.
A day-on-day percentage loss was also used rather than a rolling average % loss due to AMD’s growth over
this period. Appling a rolling average was ultimately ineffective, as it would lead to significant sales early (as the
share price was increasing strongly), and no sales late in the period due to previously weaker share price
pulling down the average.




