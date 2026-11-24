# MACD Oscillator Strategy Study

[All studies](../../README.md) · [Trading](../README.md) · [quant-trading overview](README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. Potential Problems](#1-potential-problems)

## 0. Overview

> Trade crossovers between two simple moving averages: `MA1` (short-term) and `MA2` (long-term).
>
> Buy when `MA1 ≥ MA2`. Sell when `MA1 < MA2`.

1. Input
   - Prices: one asset's daily closes.
   - Moving averages
     - `MA1`: short-term moving average.
     - `MA2`: long-term moving average.
2. Calculate
   - After each daily close
     - Take the latest closing prices in each window, including today's.
     - Add those prices; divide by the number of trading days in the window.
     - Each new day, replace the oldest close with the newest.
   - Compare unrounded `MA1` and `MA2`.
   - Oscillator: `MA1 − MA2`.
     - Positive: `MA1 > MA2`.
     - Negative: `MA1 < MA2`.
     - Zero: `MA1 = MA2`.
3. Decide
   - Apply the rules

     | Already holding? | Comparison | Action |
     | --- | --- | --- |
     | No | `MA1 ≥ MA2` | Buy. |
     | Yes | `MA1 ≥ MA2` | Hold; no extra buy. |
     | Yes | `MA1 < MA2` | Sell the holding. |
     | No | `MA1 < MA2` | Stay out. |

   - Selling closes the holding; no short position.
   - Repeat at the next daily close.

## 1. Potential Problems

- Late entry
  - Prices can start rising before the moving averages respond.
  - The buy signal can arrive after part of the rise has already happened.
- Profit and risk: separate experiment
  - Test: a different MACD version using exponential moving averages versus Awesome Oscillator.
  - The author reported:
    - Higher profit.
    - Larger drops in portfolio value during the test.
