# MevEngine Grid and DCA trading strategy

***

### Table of Contents

1. Introduction
2. Strategy Overview
3. Front End Interface
   * Interface Components
4. Grid Class
   * Initialization Parameters
   * Key Attributes
   * Methods
5. Grid Calculation Methods
   * Price Determination Methods
   * Quantity Determination Methods
6. DCA Strategy
   * Concept
   * Implementation
   * Configuration Options
7. Risk Management
   * Stop Loss and Take Profit
   * Trailing Stop Loss
   * ATR-Based Grid Spacing
   * Risk Profiles
8. Market Options
   * Spot Market
   * Futures Market
9. Error Handling
   * Error Detection and Notification
   * Common Errors and Resolutions
10. Usage Example
    * Grid Strategy Example
    * DCA Strategy Example
11. Performance Monitoring
    * Metrics to Track
    * Adjustments and Optimization
12. Conclusion

***

### Introduction

This documentation provides a detailed overview of a combined grid and Dollar-Cost Averaging (DCA) trading strategy, implemented using Python. The strategy is designed to operate on cryptocurrency markets through the Binance exchange, using a PyQt5 graphical user interface for configuration and control. The documentation covers the strategy's components, configuration, and usage, catering to both futures and spot markets, with specific considerations for risk management and error handling.

### Strategy Overview

The strategy comprises two main components:

1. **Grid Trading**: This component places a series of buy and sell orders at predefined price levels (grids). The goal is to capitalize on market volatility by executing trades as the price oscillates within the set grid levels. The strategy can be configured to operate in both long and short or both  positions, depending on the market type.
2. **Dollar-Cost Averaging (DCA)**: DCA is a long-term investment strategy where a fixed amount of capital is invested at regular intervals, regardless of market conditions. This approach reduces the impact of volatility by averaging the purchase cost over time.

These components can be used individually or in combination, allowing for a flexible approach to market conditions.

### Front End Interface

***

### Interface Components

The strategy is controlled through a PyQt5-based graphical user interface, implemented in the `PyqtMain.py` script. The interface provides a user-friendly way to configure and execute the strategy. Key components include:

1. **Market Type Selector:** Choose between spot and futures markets.
2. **Trade Profile Selector:** Allows users to select between buying, selling, or both.
3. **Capital Allocation Input:** Specifies the percentage of total capital to be used for trading.
4. **Grid Quantity Selector:** Choose between different grid quantity profiles (e.g., vanilla).
5. **Grid Method Selector:** Determines the method for grid placement (e.g., automatic).
6. **Risk Profile Selector:** Allows users to choose from conservative, moderate, or aggressive risk profiles.
7. **Strategy Selector:** Select the trading strategy to be used (e.g., grid).
8. **Symbol Input:** Specifies the trading pair (e.g., BTC/USDT).
9. **Timeframe Selector:** Sets the timeframe for grid adjustments (e.g., 1 minute).
10. **Number of Grids Selector:** Sets the number of grid levels.

These components allow users to customize their trading strategy parameters easily, ensuring flexibility and control over the trading process Which are accessible to the user

### Strategy Parameters

This document outlines the parameters for the trading strategy which are used

### Parameters

* **timeframe**: The timeframe of the strategy, e.g. `'15m'` for 15 minutes.
* **df\_long**: The timeframe of the longer dataframe used for the strategy.
* **df\_short**: The timeframe of the shorter dataframe used for the strategy.
* **symbol**: The trading symbol for which to execute the strategy.
* **capital**: The initial capital to invest (in USD).
* **num\_grids**: The number of grids to use in the strategy (typically a positive integer).
* **grid\_method\_price**: The method to determine grid prices (`'vanilla'` for equally spaced grids, `'auto'` for automatically determined prices, `'sr'` for support/resistance determined prices).
* **grid\_method\_quantity**: The method to determine grid quantities (`'vanilla'` for equally spaced quantities, `'sequential'` for sequential quantities, `'martingle'` for martingale quantities, `'reverse_martingle'` for reverse martingale quantities).
* **punch\_limit**: The number of punches to allow before exiting the strategy (typically a positive integer); leave as `0` if you don't want any limit.
* **trailing\_sl**: The trailing stop loss percentage (in percentage).
* **tp**: The take profit percentage (in percentage).
* **grid\_method**: The type of grid strategy (`'both'`, `'short'`, `'long'`).
* **futures**: Whether to use the futures or spot market (`True` for futures, `False` for spot market).

### Grid Class

The core functionality of the grid trading component is encapsulated in the `Grid` class.

#### Initialization Parameters

* **df**: `pandas.DataFrame` containing historical price data. This data is used to calculate grid levels and perform market analysis.
* **symbol**: The trading symbol (e.g., 'BTCUSDT') for which the strategy is applied.
* **type**: Specifies the type of trading ('buy' for long positions, 'sell' for short positions).
* **num\_grids**: The number of grid levels to be created. More grids allow for finer price level differentiation.
* **amount**: The total capital allocated for the strategy. This amount is distributed across the grid levels.
* **grid\_method\_price**: The method used to determine the price levels for the grids. Options include 'auto' and 'SR' (Support and Resistance).
* **tp**: Take profit percentage, defining the price increase needed to trigger a sell order.
* **grid\_method\_quantity**: The method used to determine the quantity of assets to buy/sell at each grid level. Options include 'vanilla', 'sequential', 'martingle', and 'reverse\_martingle'.
* **punches**: The number of adverse trades (e.g., trades resulting in losses) allowed before the strategy halts to prevent further losses.

#### Key Attributes

* **volume\_step\_size**: The minimum increment for order volumes, ensuring compliance with exchange rules.
* **price\_tick\_size**: The minimum increment for order prices, aligned with market conventions.
* **min\_notional**: The minimum order value required by the exchange to execute trades.

#### Methods

The `Grid` class includes methods for calculating grid levels, placing orders, and managing positions. Key methods include:

* **calculate\_grid\_prices**: Determines the price levels for the grid based on the selected method.
* **calculate\_grid\_quantities**: Calculates the quantity of assets to be traded at each grid level.

### Grid Calculation Methods

#### Price Determination Methods

1. **Support and Resistance (SR) Based**:
   * Uses historical price data to identify key support and resistance levels. These levels are used as grid boundaries.
   * Falls back to ATR-based calculations if insufficient SR levels are identified.
   *   Example:

       ```python
       def calculate_grid_prices(self, market_price=None):
           # Calculate SR levels and set as grid prices
       ```
2. **Auto**:
   * Combines SR levels with Fibonacci retracements to dynamically determine grid spacing.
   * This method adapts to recent market conditions, providing a more responsive grid structure.
   *   Example:

       ```python
       def auto_grid_prices(self):
           # Use Fibonacci retracements and SR levels for grid pricing
       ```

#### Quantity Determination Methods

1. **Vanilla**:
   * Distributes the total capital equally across all grid levels.
   *   Example:

       ```python
       def vanilla_quantities(self):
           # Equal distribution of capital across grids
       ```
2. **Sequential**:
   * Increases the investment amount for each successive grid level, typically used to average down/up.
   *   Example:

       ```python
       def sequential_quantities(self):
           # Incrementally increasing quantities for each grid level
       ```
3. **Martingale**:
   * Increases the investment size exponentially after each loss, aiming to recover losses more quickly.&#x20;
   * The algorithm automatically calculates appropriate grid Quantities for each level and a suitable exponential in order to stay within the limits
   *   Example:

       ```python
       def martingale_quantities(self):
           # Exponential increase in quantities after each loss
       ```
4. **Reverse Martingale**:
   * Reduces the investment size after each loss by half.
   * The algorithm automatically calculates appropriate grid Quantities for each level and a suitable exponential in order to stay within the limits
   *   Example:

       ```python
       def reverse_martingale_quantities(self):
           # Reducing quantities after each gain to protect profits
       ```

### DCA Strategy

#### Concept

The Dollar-Cost Averaging (DCA) strategy involves investing a fixed amount at regular intervals, regardless of the asset's price. This method reduces the impact of volatility and prevents emotional decision-making. Over time, it averages the purchase cost, potentially lowering the average price paid for an asset.

Appart from this The Dca strategy only initializes if there is an RSI divergernce that has occured recently along with EMA crossover in order to get better entry.

#### Implementation

The DCA strategy is implemented In a similar fashion as the Grid Class with appropriate changes. The interface allows users to configure key parameters, while the backend handles the execution of periodic investments.

NOTE: DCA is long only or short only, selecting both would mean the algorithm would take whichever signal is recieved.

#### Configuration Options

* **Investment Amount**: The fixed amount of capital to be invested as a percent of your capital
* **Investment Interval**: The time period between each investment (e.g., daily, weekly, monthly).
* **Market**: Choice between futures and spot markets. Note that the DCA strategy in the spot market is long-only.
* **Risk Profile**: Determines the aggressiveness of the investment approach, impacting the stop-loss and take-profit levels.
* **Grid quantity methods : Same as Grid Trading**
* **Grid price Methods : Same as Grid Trading**
* **No. of Grids**&#x20;

### Risk Management

#### Stop Loss and Take Profit

Stop-loss and take-profit levels are crucial for managing risk and securing profits. These levels are determined based on the selected risk profile:

#### Risk Profiles

The strategy offers three predefined risk profiles, each with specific settings for stop-loss, take-profit, and grid spacing:

* **Conservative**: Low risk with tight stop-loss and take-profit levels, suitable for risk-averse traders.
  * **Conservative**: Stop loss and take profit are set at 0.5% to minimize risk.
* **Moderate**: Balanced risk and reward, with moderate stop-loss and take-profit levels.
  * **Moderate**: Stop loss and take profit are set at 1%, balancing risk and reward.
* **Aggressive**: Higher risk with wider stop-loss and take-profit levels, targeting higher returns.
  * **Aggressive**: Stop loss and take profit are set at 2%, allowing for higher potential returns with increased risk.

#### Trailing Stop Loss

A trailing stop loss dynamically adjusts the stop-loss level based on market movements. As the market price moves in favor of the trade, the trailing stop loss follows, locking in profits and minimizing potential losses.

#### ATR-Based Grid Spacing

If the number of grids calculated are less than the requested number of grids we'll shift to equally spaced grids.

ATR) to determine the distance between grid levels. This adaptive method makes sure that we still adjust to market volatility, ensuring appropriate spacing based on current conditions.

### Market Options

#### Spot Market

* **Long-Only**: The strategy supports only long positions in the spot market for both the grid and dca
* **No Shorting**: Due to the nature of spot trading, shorting is not possible.

#### Futures Market

* **Long and Short Positions**: The strategy can take both long and short positions, allowing for profit opportunities in both rising and falling markets.

### Error Handling

#### Error Detection and Notification

The strategy includes robust error handling mechanisms to detect and respond to various issues, such as:

* **Order Execution Errors**: Issues related to insufficient balance, minimum order size, etc
* **Network Errors**: Loss of connection to the exchange or data provider.
* **Data Errors**: Inaccuracies in market data or historical prices.

#### Common Errors and Resolutions

* **Insufficient Balance**: Adjust the capital allocation or reduce the number of grids.

### Usage Example

{% file src=".gitbook/assets/MevEnginedoc.mp4" %}

### Conclusion

This documentation provides a comprehensive guide to implementing a combined grid and DCA trading strategy. The strategy offers flexibility in terms of market selection (spot and futures), risk management, and execution. By leveraging the an Easy interface, users can easily configure and monitor their strategies, ensuring they are well-equipped to navigate the complexities of financial markets. For further information and technical support, please contact Sales@MevEngine.com

***
