# Pair Trading Strategy Study

[All studies](../../README.md) · [Trading](../README.md) · [quant-trading overview](README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. Potential Problems](#1-potential-problems)

## 0. Overview

> Trade deviations from a statistically tested relationship between two assets.
>
> Buy the relatively cheap asset and short the relatively expensive asset, expecting their price relationship to return to normal.

1. Input
   - `A1`, `A2`: two assets' daily closing prices.
   - History: the previous 250 trading days, excluding today.
2. Calculate
   - After each daily close, test the relationship using that history.
     - Cointegration: a fitted price gap tends to return toward its historical average.
   - When the test changes from fail to pass
     - Fit `A2`'s price from `A1`'s price.
     - Keep this fitted relationship while the daily tests pass.
   - Measure today's gap
     - `Spread = A2 − fitted A2`.
     - `Z`: distance of `Spread` from its historical average, measured in standard deviations.
   - Set bounds when the test changes from fail to pass
     - `U`: initial `Z` + historical spread standard deviation.
     - `L`: initial `Z` − historical spread standard deviation.
     - Keep these bounds while the daily tests pass.
3. Decide
   - Apply the rules

     | Relationship test | Gap | `A1` position | `A2` position |
     | --- | --- | --- | --- |
     | Pass | `Z > U` | Buy/hold. | Short/hold. |
     | Pass | `Z < L` | Short/hold. | Buy/hold. |
     | Pass | `L ≤ Z ≤ U` | Close/stay out. | Close/stay out. |
     | Fail | Any | Close/stay out. | Close/stay out. |

   - Short: sell borrowed shares; buy them back when closing.
   - Same positions: hold; no extra trade. Opposite positions: close and reverse both.
   - Repeat at the next daily close.

## 1. Potential Problems

- Relationship breaks
  - The author warns that a historical relationship can disappear.
  - Prices may keep separating after entry, causing losses.
- Backtest limitations
  - Bounds mix units: unitless `Z` with spread variation measured in price units.
  - Share counts use each asset's highest price across the full test, including future prices.
  - The reported result needs revalidation after fixing these issues.
