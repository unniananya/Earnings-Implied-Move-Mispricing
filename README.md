# Earnings Straddle Mispricing: A Cross-Sectional Alpha Study

This project studies whether pre-earnings option and liquidity features predict which earnings straddles are relatively overpriced or underpriced. The research focuses on at-the-money straddles around earnings announcements and evaluates whether implied earnings moves exceed or fall short of realized post-earnings stock moves.

The main finding is that earnings straddles are expensive on average, but the degree of overpricing is predictable. A simple out-of-sample linear or ridge regression model ranks future straddle edge effectively. The predicted-best quintile loses much less than buying all straddles, while the predicted-worst quintile performs much worse. A theoretical long-best/short-worst spread remains positive after simple bid/ask transaction-cost assumptions.

## Research Question

Can pre-earnings option and liquidity features predict which earnings straddles are relatively overpriced or underpriced?

The project does not assume that earnings straddles are always profitable or always unprofitable. Instead, it studies the cross-section of earnings events and ranks which straddles are most overpriced versus relatively less overpriced.

## Core Idea

A straddle consists of one call option and one put option with the same underlying stock, strike price, and expiration date. Around earnings, an at-the-money straddle approximates the options market's expected move.

The key target variable is:

```text
Straddle Edge = Realized Move - Implied Move
```

Interpretation:

```text
Edge > 0: the stock moved more than implied, so the straddle was relatively cheap.
Edge < 0: the stock moved less than implied, so the straddle was expensive.
```

## Data Construction

Each row in the dataset represents one stock earnings event.

For each earnings event, the dataset links:

- Earnings announcement date
- Stock ticker
- Stock price before earnings
- Stock price after earnings
- Option prices before earnings
- Nearest option expiration after earnings
- At-the-money call option price
- At-the-money put option price

The main dataset uses the simple after-expiry straddle method. The option quote is observed before earnings, while the selected option expiration is the first expiration after the earnings announcement.

Example:

```text
Earnings date: May 10
Option quote date: May 9
Option expiration date: May 17
```

The quote is known before earnings, and the option expires after the earnings event. This allows the straddle price to capture the market's expected earnings move.

## Dataset Sizes

The final simple dataset sizes are:

```text
Base dataset:   9,804 events
Loose dataset:  9,022 events
Strict dataset: 7,412 events
```

The loose dataset is used for main modeling. The strict dataset applies stronger liquidity filters and is used as a robustness check.

## Implied Move

The main implied move variable is:

```text
implied_move_simple = after_straddle_mid / stock_price
```

The straddle midpoint is:

```text
after_straddle_mid = after_call_mid + after_put_mid
```

The call and put midpoints are:

```text
call_mid = (call_bid + call_ask) / 2
put_mid  = (put_bid + put_ask) / 2
```

Example:

```text
Stock price: $100
ATM call midpoint: $3.00
ATM put midpoint: $2.70

Straddle midpoint = $3.00 + $2.70 = $5.70
Implied move = $5.70 / $100 = 5.70%
```

This means the options market is pricing an expected earnings move of approximately 5.70%.

## Realized Move

The main realized move uses the stock's next opening price relative to the previous close:

```text
realized_move_open_to_prevclose =
abs(next_open_price - previous_close_price) / previous_close_price
```

This captures the immediate market reaction to earnings.

A close-to-close version is also created as a secondary measure:

```text
realized_move_close_to_prevclose
```

The main target uses the open reaction because it is closer to the earnings event response and less affected by later intraday trading.

## Target Variable

The main target variable is:

```text
simple_straddle_edge_open =
realized_move_open_to_prevclose - implied_move_simple
```

Examples:

```text
Implied move = 6%
Realized move = 4%
Edge = 4% - 6% = -2%

The straddle was expensive.
```

```text
Implied move = 5%
Realized move = 8%
Edge = 8% - 5% = +3%

The straddle was cheap.
```

## Liquidity Filters

Liquidity filters are applied because option prices can be noisy when contracts are illiquid. The key liquidity variables are:

- after_call_spread_pct
- after_put_spread_pct
- after_call_oi
- after_put_oi
- after_call_vol
- after_put_vol

The bid-ask spread percentage is:

```text
spread_pct = (ask - bid) / midpoint
```

### Loose Dataset Filters

```text
implied move > 0 and < 100%
realized move >= 0 and < 100%
call and put spread percentage < 75%
call and put open interest >= 1
```

### Strict Dataset Filters

```text
implied move > 0 and < 100%
realized move >= 0 and < 100%
call and put spread percentage < 50%
call and put open interest >= 10
call and put volume >= 1
```

The loose and strict datasets produce similar average results, showing that the main conclusion is not driven only by poor option quotes.

## Features

The model uses pre-earnings option and liquidity features.

### Implied Move

```text
implied_move_simple
```

This is the market-implied move from the ATM straddle. It is the most important feature because high implied-move straddles tend to have worse future edge.

### Total Spread Percentage

```text
total_spread_pct =
(after_call_spread_pct + after_put_spread_pct) / 2
```

This measures the average bid-ask spread of the call and put.

### Total Open Interest

```text
total_open_interest =
after_call_oi + after_put_oi
```

This measures the total number of outstanding call and put contracts.

### Total Volume

```text
total_volume =
after_call_vol + after_put_vol
```

This measures total option trading activity on the quote date.

### Log Open Interest

```text
log_total_open_interest =
log(1 + total_open_interest)
```

The log transformation reduces the effect of extreme values.

### Log Volume

```text
log_total_volume =
log(1 + total_volume)
```

The log transformation reduces scale differences across contracts.

### Log Stock Price

```text
log_stock_price =
log(stock_price)
```

Stock price is included because lower-priced stocks tend to have worse straddle edge.

### Earnings Month

```text
earnings_month
```

This captures earnings-season effects.

## Baseline Quintile Tests

Before regression modeling, the project tests one-variable signals by sorting each signal into quintiles.

The strongest single-variable signal is `implied_move_simple`.

Loose dataset:

```text
Q1 average edge = -0.56%
Q5 average edge = -2.54%
Q5 - Q1 = -1.98%
```

Strict dataset:

```text
Q1 average edge = -0.50%
Q5 average edge = -2.46%
Q5 - Q1 = -1.96%
```

This shows that high implied-move straddles are much more overpriced than low implied-move straddles.

The implied-move effect is stable by year:

```text
2018: Q5 - Q1 = -1.49%
2019: Q5 - Q1 = -1.39%
2020: Q5 - Q1 = -3.56%
2021: Q5 - Q1 = -1.82%
2022: Q5 - Q1 = -2.85%
2023: Q5 - Q1 = -1.11%
2024: Q5 - Q1 = -1.20%
```

Other baseline findings:

```text
Higher total spread percentage -> worse edge
Higher volume -> better edge
Open interest alone -> weak relationship
Higher stock price -> slightly better edge
```

Loose dataset correlations with future edge:

```text
implied move correlation: -0.239
spread correlation:       -0.073
open interest correlation: 0.001
volume correlation:        0.090
stock price correlation:   0.024
```

## Combined Score

A hand-built overpriced score is created using the baseline findings:

```text
overpriced_score =
rank_implied_move
+ rank_spread
- rank_volume
- rank_stock_price
```

Higher implied move and higher spread are treated as signs of overpricing. Higher volume and higher stock price are treated as signs of relatively better edge.

Loose dataset:

```text
Q1 edge = -0.61%
Q2 edge = -0.94%
Q3 edge = -1.25%
Q4 edge = -1.83%
Q5 edge = -2.58%

Q5 - Q1 = -1.97%
```

Strict dataset:

```text
Q1 edge = -0.64%
Q5 edge = -2.48%

Q5 - Q1 = -1.84%
```

The combined score produces a clear monotonic pattern.

## Regression Modeling

The project uses simple, interpretable models:

- Linear regression
- Ridge regression

Complex machine learning models are not used because the goal is interpretability and reduced overfitting risk.

The target is:

```text
simple_straddle_edge_open
```

The regression features are:

```text
implied_move_simple
total_spread_pct
log_total_open_interest
log_total_volume
log_stock_price
earnings_month
```

## Out-of-Sample Validation

The models are tested using expanding-window validation:

```text
Train 2018-2020 -> Test 2021
Train 2018-2021 -> Test 2022
Train 2018-2022 -> Test 2023
Train 2018-2023 -> Test 2024
```

This avoids look-ahead bias because the model only uses past data to predict future earnings events.

Linear regression performance:

```text
2021: RMSE = 3.62%, MAE = 2.27%, R² = 0.0588
2022: RMSE = 4.10%, MAE = 2.93%, R² = 0.1319
2023: RMSE = 4.10%, MAE = 2.81%, R² = -0.0153
2024: RMSE = 4.41%, MAE = 2.98%, R² = 0.0233
```

The ridge regression results are nearly identical.

The predictions have positive out-of-sample correlation with actual edge:

```text
linear regression correlation = 0.248
ridge regression correlation  = 0.248
```

## Predicted Edge Quintile Results

Sorting predicted edge into quintiles gives:

```text
Q1 actual edge = -2.54%
Q2 actual edge = -1.62%
Q3 actual edge = -1.35%
Q4 actual edge = -0.79%
Q5 actual edge = -0.51%
```

The model successfully ranks straddles from worst to best.

The Q5 minus Q1 spread is:

```text
linear regression: +2.03%
ridge regression:  +2.03%
implied-move-only baseline: +1.74%
```

The ranking is positive in every test year:

```text
2021: Q5 - Q1 = +2.38%
2022: Q5 - Q1 = +3.02%
2023: Q5 - Q1 = +1.29%
2024: Q5 - Q1 = +1.29%
```

Ridge coefficient summary:

```text
implied_move_simple:     -0.010130
log_total_open_interest: -0.003469
total_spread_pct:        -0.001829
log_stock_price:         -0.000014
earnings_month:           0.001077
log_total_volume:         0.004578
```

The signs are intuitive:

```text
Higher implied move -> worse edge
Wider spreads -> worse edge
Higher volume -> better edge
```

## Transaction-Cost-Aware Strategy Evaluation

The Part 3 predictions are merged with option bid/ask data to evaluate transaction-cost-aware edge measures.

The following implied move measures are created:

```text
mid_implied_move = after_straddle_mid / stock_price
ask_implied_move = (call_ask + put_ask) / stock_price
bid_implied_move = (call_bid + put_bid) / stock_price
```

The edge measures are:

```text
mid_edge      = realized_move - mid_implied_move
long_ask_edge = realized_move - ask_implied_move
short_bid_edge = bid_implied_move - realized_move
```

Interpretation:

```text
mid_edge: research midpoint result
long_ask_edge: realistic measure for buying straddles at the ask
short_bid_edge: simplified proxy for selling straddles at the bid
```

Average transaction-cost summary:

```text
average midpoint implied move = 5.79%
average ask implied move      = 6.08%
average bid implied move      = 5.49%
average realized move         = 4.43%
average midpoint edge         = -1.36%
average long-at-ask edge      = -1.66%
average short-at-bid edge     = +1.06%
```

Average straddle bid-ask spread:

```text
0.59% of stock price
10.59% of midpoint straddle value
```

Bid-ask costs are meaningful and materially affect results.

## Strategy Results

Main strategy results:

```text
Buy all straddles - midpoint:       -1.36%
Buy all straddles - at ask:         -1.66%

Buy Q5 predicted-best - midpoint:   -0.48%
Buy Q5 predicted-best - at ask:     -0.60%

Buy Q1 predicted-worst - midpoint:  -2.43%
Short Q1 predicted-worst - at bid:  +1.89%

Long Q5 / Short Q1 midpoint spread: +1.95%
Long Q5 at ask / Short Q1 at bid:   +1.29%
```

Buying all earnings straddles is unattractive on average. Buying only the predicted-best quintile loses much less, but it remains slightly negative after ask costs. The strongest result is the cross-sectional spread between predicted-best and predicted-worst straddles.

Year-by-year transaction-cost-aware spread:

```text
2021: midpoint spread = +2.38%, bid/ask spread = +1.61%
2022: midpoint spread = +3.02%, bid/ask spread = +2.16%
2023: midpoint spread = +1.29%, bid/ask spread = +0.79%
2024: midpoint spread = +1.29%, bid/ask spread = +0.76%
```

## Top-Decile Results

The strongest signal is concentrated in the extremes:

```text
D1 midpoint edge = -3.08%
D10 midpoint edge = -0.33%
D10 - D1 midpoint spread = +2.75%

D10 long ask edge = -0.43%
D1 short bid edge = +2.49%

Long D10 ask / Short D1 bid spread = +2.07%
```

## Main Findings

The project produces four main findings.

### 1. Earnings Straddles Are Expensive on Average

Loose dataset:

```text
average implied move = 5.71%
average realized move = 4.27%
average edge = -1.44%
```

Strict dataset:

```text
average implied move = 5.73%
average realized move = 4.37%
average edge = -1.36%
```

The similar loose and strict results show that average overpricing is not driven only by illiquid option quotes.

### 2. High Implied-Move Straddles Are Especially Overpriced

In the loose dataset:

```text
lowest implied-move quintile average edge = -0.56%
highest implied-move quintile average edge = -2.54%
```

This pattern appears in the strict dataset and in every year from 2018 to 2024.

### 3. Combining Implied Move and Liquidity Features Improves Ranking

The out-of-sample regression model produces:

```text
Regression Q5 - Q1 spread: +2.03%
Implied-move-only Q5 - Q1 spread: +1.74%
```

The combined model improves the ranking over the implied-move-only baseline.

### 4. The Relative-Value Spread Remains Positive After Bid/Ask Costs

The transaction-cost-aware spread is:

```text
Long Q5 at ask / Short Q1 at bid = +1.29%
```

The spread is positive in every test year from 2021 to 2024.

## Interpretation

The model is not a naive long-straddle profit machine. Buying all straddles loses money on average, and even buying the predicted-best quintile at the ask remains slightly negative.

The model is best interpreted as a cross-sectional ranking and filtering tool. It identifies which earnings straddles are most overpriced and should be avoided by long-volatility traders. It also supports a relative-value volatility strategy: long the predicted-best straddles and short the predicted-worst straddles.

The short side requires careful risk management. Short straddles have tail risk, margin requirements, and large downside exposure if the stock moves sharply. The short-bid measure used here is a simplified move-based proxy, not a full options PnL calculation.

## Limitations

This project uses move-based edge rather than full option payoff PnL. A future extension should compute actual straddle returns based on option payoff at expiration or immediately after earnings.

The transaction-cost assumptions use bid and ask prices. Actual execution varies depending on order size, liquidity, and market conditions.

The feature set is intentionally simple. Additional features such as realized volatility, prior earnings moves, analyst forecast dispersion, market volatility, sector controls, market capitalization, implied volatility skew, and volatility term structure can extend the project.

The long-short strategy should be interpreted as evidence of relative mispricing, not as a fully implementable trading strategy without additional risk controls.

## Technologies Used

- Python
- pandas
- NumPy
- DuckDB
- scikit-learn
- statsmodels
- matplotlib
- WRDS
- OptionMetrics
- CRSP

## Conclusion

This project develops a cross-sectional earnings straddle mispricing model using pre-earnings option prices and liquidity features.

Earnings straddles are expensive on average, with implied moves exceeding realized moves by roughly 1.4 percentage points in the main clean sample. The overpricing is not uniform. High implied-move and wide-spread straddles are especially expensive, while higher-volume straddles tend to have better edge.

A simple out-of-sample linear or ridge regression model ranks future straddle edge effectively. The predicted-best quintile outperforms the predicted-worst quintile by about 2.0 percentage points at midpoint and about 1.3 percentage points after simple bid/ask assumptions.

The final interpretation is that the model is most useful as a relative-value ranking and trade-filtering tool. It helps identify the most overpriced long-volatility trades to avoid and supports further research into long-best/short-worst volatility spreads. It should not be interpreted as proof that buying earnings straddles is profitable on average.

