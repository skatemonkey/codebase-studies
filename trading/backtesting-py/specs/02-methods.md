# Methods

[All studies](../../../README.md) · [Trading](../../README.md) · [Backtesting.py overview](../README.md)

> Table of Contents
>
> - [Strategy](#strategy)
> - [Backtest](#backtest)
> - [_Broker](#broker)
> - [Trading Objects](#trading-objects)

- <a name="strategy"></a>**[Strategy](01-data-contracts.md#14-strategy)** (`self`: your strategy instance)
  - `init()`: Engine calls once before the candle loop.
  - `next()`: Engine calls for each candle in the loop.
  - `self.I(...)`: Register and return an [_Indicator](01-data-contracts.md#14-strategy) array.
  - `self.buy(...)`: Submit and return a positive-size [Order](01-data-contracts.md#21-order).
  - `self.sell(...)`: Submit and return a negative-size [Order](01-data-contracts.md#21-order).
- <a name="backtest"></a>**[Backtest](01-data-contracts.md#11-backtest)**
  - <code>bt = <a href="01-data-contracts.md#11-backtest">Backtest</a>(data, strategy, **settings)</code>: Create the configured test.
  - `bt.run(**parameters)`: Simulate and return statistics as [pd.Series](01-data-contracts.md#11-backtest).
  - `bt.plot(...)`: Plot results and return a [Bokeh chart layout](01-data-contracts.md#11-backtest).
  - `bt.optimize(...)`: Find best parameters; return [pd.Series](01-data-contracts.md#11-backtest) by default.
- <a name="broker"></a>**[_Broker](01-data-contracts.md#13-_broker)**
  - <code><a href="01-data-contracts.md#13-_broker">broker</a>.new_order(...)</code>: Create, queue, and return an [Order](01-data-contracts.md#21-order).
  - <code><a href="01-data-contracts.md#13-_broker">broker</a>.next()</code>: Process pending [Orders](01-data-contracts.md#21-order) and record equity for this candle.
- <a name="trading-objects"></a>**Trading Objects**
  - **[Position](01-data-contracts.md#23-position)**
    - <code>self.<a href="01-data-contracts.md#23-position">position</a>.close(portion=1.0)</code>: Queue closing [Orders](01-data-contracts.md#21-order) for `portion` of every open [Trade](01-data-contracts.md#22-trade).
  - **[Trade](01-data-contracts.md#22-trade)**
    - <code><a href="01-data-contracts.md#22-trade">trade</a>.close(portion=1.0)</code>: Queue a closing [Order](01-data-contracts.md#21-order) for `portion`.
  - **[Order](01-data-contracts.md#21-order)**
    - <code><a href="01-data-contracts.md#21-order">order</a>.cancel()</code>: Cancel this pending instruction.
