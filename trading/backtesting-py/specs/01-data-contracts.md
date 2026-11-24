# Data Contracts

[All studies](../../../README.md) · [Trading](../../README.md) · [Backtesting.py overview](../README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. Simulation Objects](#1-simulation-objects)
> - [2. Trading Objects](#2-trading-objects)

## 0. Overview

> Python object types and data shapes used by the documented backtest flow.

> Sources: [backtesting.py](https://github.com/kernc/backtesting.py/blob/ca2e2611621e472542ba90f7243a1fa06a7d7108/backtesting/backtesting.py) and [_util.py](https://github.com/kernc/backtesting.py/blob/ca2e2611621e472542ba90f7243a1fa06a7d7108/backtesting/_util.py).

- <code><a href="#11-backtest">Backtest</a></code>: Object holding the data, strategy class, and simulation settings for a test.
  - `run()` creates:
    - <code>data: <a href="#12-_data">_Data</a></code>: Runtime market data wrapping the input <code><a href="#12-_data">DataFrame</a></code>.
    - <code>broker: <a href="#13-_broker">_Broker</a></code>: Simulated account and trading state.
      - <code>orders: List[<a href="#21-order">Order</a>]</code>: Pending.
      - <code>trades: List[<a href="#22-trade">Trade</a>]</code>: Open.
      - <code>closed_trades: List[<a href="#22-trade">Trade</a>]</code>: Completed.
      - <code>position: <a href="#23-position">Position</a></code>: Combined view of open [trades](#22-trade).
    - <code>strategy: <a href="#14-strategy">Strategy</a></code>: Your strategy instance, connected to `broker` and `data`.
      - <code>_data: <a href="#12-_data">_Data</a></code>: Reference to the same runtime data.
      - <code>_broker: <a href="#13-_broker">_Broker</a></code>: Reference to the same broker.

## 1. Simulation Objects

### 1.1 Backtest

- Fields

  | Field | Type | Meaning | Example |
  | --- | --- | --- | --- |
  | `_data` | <code><a href="#12-_data">pd.DataFrame</a></code> | Input historical candles | OHLCV frame |
  | `_strategy` | <code>Type[<a href="#14-strategy">Strategy</a>]</code> | Class to instantiate | <code><a href="#14-strategy">MyStrategy</a></code> |
  | `_broker` | `functools.partial` | Broker factory holding simulation settings | <code><a href="#13-_broker">_Broker</a></code> with configured cash and costs |
  | `_finalize_trades` | `bool` | Whether to close remaining trades after the last candle | `False` |
  | `_results` | <code>Optional[<a href="#11-backtest">pd.Series</a>]</code> | Latest results and statistics | `None` before the first run |

- Method output: `plot()` returns a Bokeh chart layout.

### 1.2 _Data

- Exposed data

  | Property | Type | Meaning | Example |
  | --- | --- | --- | --- |
  | `Open` | <code><a href="#12-_data">_Array</a></code>, a NumPy array subclass | Opening prices in the visible history; required input | `data.Open[-1] == 98.0` |
  | `High` | <code><a href="#12-_data">_Array</a></code> | Highest prices in the visible history; required input | `data.High[-1] == 102.0` |
  | `Low` | <code><a href="#12-_data">_Array</a></code> | Lowest prices in the visible history; required input | `data.Low[-1] == 97.0` |
  | `Close` | <code><a href="#12-_data">_Array</a></code> | Closing prices in the visible history; required input | `data.Close[-1] == 100.0` |
  | `Volume` | <code><a href="#12-_data">_Array</a></code> | Optional candle volume; NaN when omitted | `data.Volume[-1] == 1_000` |
  | `index` | <code><a href="#12-_data">pd.Index</a></code> | Candle timestamps or period labels | `pd.Timestamp("2026-10-07")` |
  | `df` | <code><a href="#12-_data">pd.DataFrame</a></code> | Full history in `init()`; through the current candle in `next()` | OHLCV frame; additional input columns allowed |

### 1.3 _Broker

- Account state and calculated properties

  | Field / property | Type | Meaning | Example |
  | --- | --- | --- | --- |
  | `_cash` | Numeric scalar | Cash balance | `10_000` |
  | `equity` | Numeric property | Cash plus unrealized profit/loss: `_cash + _position_unrealized_pl` | `10_000 + 200 = 10_200` |
  | `_equity` | <code><a href="#13-_broker">np.ndarray</a></code> | Equity history, with one slot per candle | `np.array([np.nan, 10_000.0, 10_200.0])` |
  | `_leverage` | Numeric scalar | Leverage derived from `1 / margin` | `1.0` |
  | `margin_available` | Numeric property | Equity minus used margin, floored at zero | `10_200 - 1_000 / 1 = 9_200` |

- Trading state

  | Field | Type | Meaning | Initial value |
  | --- | --- | --- | --- |
  | `orders` | <code>List[<a href="#21-order">Order</a>]</code> | Pending | `[]` |
  | `trades` | <code>List[<a href="#22-trade">Trade</a>]</code> | Open | `[]` |
  | `closed_trades` | <code>List[<a href="#22-trade">Trade</a>]</code> | Completed | `[]` |
  | `position` | <code><a href="#23-position">Position</a></code> | Combined view of open [trades](#22-trade) | Empty (`size=0`) |

### 1.4 Strategy

- Fields and public properties

  | Field / property | Type | Meaning | Example |
  | --- | --- | --- | --- |
  | `_broker` | <code><a href="#13-_broker">_Broker</a></code> | Account and execution state | Current run's broker |
  | `_data` | <code><a href="#12-_data">_Data</a></code> | Visible market history | Current run's data |
  | `_params` | `dict` | Parameter overrides passed through `run(...)` | `{}` when no overrides are passed |
  | `_indicators` | `list` of <code><a href="#14-strategy">_Indicator</a></code> arrays | Indicators registered through `I(...)` | `[]` before registration |
  | `data` | <code><a href="#12-_data">_Data</a></code> | Public access to `_data` | `self.data.Close[-1]` |
  | `position` | <code><a href="#23-position">Position</a></code> | Combined view of open [trades](#22-trade) | `self.position.size` |
  | `orders` | <code>Tuple[<a href="#21-order">Order</a>, ...]</code> | Pending instructions | `()` when empty |
  | `trades` | <code>Tuple[<a href="#22-trade">Trade</a>, ...]</code> | Open | `()` when empty |
  | `closed_trades` | <code>Tuple[<a href="#22-trade">Trade</a>, ...]</code> | Completed | `()` when empty |
  | `equity` | `float` | Current account value from the [broker](#13-_broker) | `10_200.0` |

## 2. Trading Objects

### 2.1 Order

- Properties

  | Property | Type | Meaning | Example |
  | --- | --- | --- | --- |
  | `size` | Numeric scalar | Signed quantity, or a liquidity fraction when its absolute value is below `1` | `10` units or `0.5` of available liquidity |
  | `limit` | `Optional[float]` | Limit-entry price | `None` for market execution |
  | `stop` | `Optional[float]` | Stop-entry trigger price | `None` |
  | `sl` | `Optional[float]` | Stop-loss price for the resulting [trade](#22-trade) | `75.0` |
  | `tp` | `Optional[float]` | Take-profit price for the resulting [trade](#22-trade) | `110.0` |
  | `parent_trade` | <code>Optional[<a href="#22-trade">Trade</a>]</code> | Target of closing or protective instructions | `None` for an independent entry |
  | `tag` | `object` | Optional tracking label | `"entry-1"` |

- Order Types: One [Order](#21-order) class, selected by `limit` / `stop` (buy examples at price `100`).

  | Type | `limit` | `stop` | Execution |
  | --- | --- | --- | --- |
  | Market | `None` | `None` | Fill at the next candle's Open by default (`trade_on_close=False`). |
  | Limit | `95` | `None` | Buy at `95` or lower, before trading costs. |
  | Stop-market | `None` | `105` | Trigger at `105` or higher, then execute at market price; the fill can be above `105`. |
  | Stop-limit | `106` | `105` | Trigger at `105` or higher, then use the limit of `106`; can remain unfilled. |

### 2.2 Trade

- Properties

  | Property | Type | Meaning | Example |
  | --- | --- | --- | --- |
  | `size` | `int` | Signed asset quantity in standard [Backtest](#11-backtest) | `10` shares |
  | `entry_price` | `float` | Filled entry price | `80.0` |
  | `exit_price` | `Optional[float]` | Filled exit price; absent while open | `None` |
  | `entry_bar` | `int` | Entry candle index | `12` |
  | `exit_bar` | `Optional[int]` | Exit candle index; absent while open | `None` |
  | `value` | Numeric property | Absolute quantity multiplied by current or exit price | `1_000.0` at price `100.0` |
  | `pl` | Numeric property | Profit/loss; commissions are reflected after closing | `200.0` with zero commission |
  | `sl` | `Optional[float]` | Current stop-loss price | `75.0` |
  | `tp` | `Optional[float]` | Current take-profit price | `110.0` |
  | `tag` | `object` | Tracking label inherited from the entry [order](#21-order) | `"entry-1"` |

### 2.3 Position

- Calculated properties

  | Property | Type | Meaning | Example |
  | --- | --- | --- | --- |
  | `size` | `int` in standard [Backtest](#11-backtest) | Net asset quantity across open [trades](#22-trade) | `10` shares at RM100 means RM1,000 market value |
  | `pl` | Numeric property | Unrealized profit/loss across open [trades](#22-trade) | `200.0` |
  | `pl_pct` | Numeric property | Unrealized profit/loss as a percentage of entry value | `25.0` for entry `80`, current price `100` |
  | `is_long` | `bool` | Whether net size is positive | `True` |
  | `is_short` | `bool` | Whether net size is negative | `False` |
