# Backtesting.py Simulation

[All studies](../../README.md) · [Trading](../README.md) · [Backtesting.py overview](README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. New Order](#1-new-order)

## 0. Overview

> Replays historical candles and simulates [orders](specs/01-data-contracts.md#21-order), [trades](specs/01-data-contracts.md#22-trade), and account changes.

> - Entry point: <code><a href="specs/01-data-contracts.md#11-backtest">Backtest</a>.run()</code>.

1. Setup
   - <code>data: <a href="specs/01-data-contracts.md#12-_data">_Data</a></code>: Runtime wrapper for the input <code><a href="specs/01-data-contracts.md#12-_data">DataFrame</a></code>.
   - <code>broker: <a href="specs/01-data-contracts.md#13-_broker">_Broker</a></code>: Simulated account and trading state.
     - Account state
       - `self._cash = cash`: Numeric cash balance, initialized from the `cash` input.
       - <code>self.position = <a href="specs/01-data-contracts.md#23-position">Position</a>(self)</code>: Combined view of open [trades](specs/01-data-contracts.md#22-trade).
       - `self._equity = np.tile(np.nan, len(index))`: NumPy array recording account equity per candle.
       - `self._leverage = 1 / margin`: Numeric leverage derived from the `margin` input.
     - Calculated account values
       - [equity](specs/01-data-contracts.md#13-_broker): Property returning `self._cash + self._position_unrealized_pl`, the current account value.
       - [margin_available](specs/01-data-contracts.md#13-_broker): Property returning the equity available after subtracting margin used by open [trades](specs/01-data-contracts.md#22-trade), floored at zero.
     - Tracking lists
       - <code>self.orders: List[<a href="specs/01-data-contracts.md#21-order">Order</a>] = []</code>: Pending.
       - <code>self.trades: List[<a href="specs/01-data-contracts.md#22-trade">Trade</a>] = []</code>: Open.
       - <code>self.closed_trades: List[<a href="specs/01-data-contracts.md#22-trade">Trade</a>] = []</code>: Completed.
   - <code>strategy: <a href="specs/01-data-contracts.md#14-strategy">Strategy</a></code>: Your strategy instance, connected to `broker` and `data`.
     - <code><a href="specs/01-data-contracts.md#14-strategy">strategy</a>.init()</code>: Called once before the loop.
2. Candle Loop
   - <code><a href="specs/01-data-contracts.md#13-_broker">broker</a>.next()</code>: Process pending [Orders](specs/01-data-contracts.md#21-order) and update [Trades](specs/01-data-contracts.md#22-trade), cash, and equity.
   - <code><a href="specs/01-data-contracts.md#14-strategy">strategy</a>.next()</code>: Check the current candle and possibly queue new [Orders](specs/01-data-contracts.md#21-order) for the next candle. Examples assume no open [trades](specs/01-data-contracts.md#22-trade).
     - Long: `self.buy(size=10)` → [Order](specs/01-data-contracts.md#21-order) (`size=+10`) → fill → long [Trade](specs/01-data-contracts.md#22-trade) → existing [Position](specs/01-data-contracts.md#23-position) (`size=+10`).
     - Short: `self.sell(size=10)` → [Order](specs/01-data-contracts.md#21-order) (`size=-10`) → fill → short [Trade](specs/01-data-contracts.md#22-trade) → existing [Position](specs/01-data-contracts.md#23-position) (`size=-10`).

   <pre><code>for each candle at index i:
       <a href="specs/01-data-contracts.md#14-strategy">strategy</a>.<a href="specs/01-data-contracts.md#12-_data">data</a>: previous candles + current candle; future candles hidden

       <a href="specs/01-data-contracts.md#13-_broker">broker</a>.next():
           for each pending market <a href="specs/01-data-contracts.md#21-order">Order</a>:
               execute using this candle's Open as base price; cancel if margin insufficient
               <a href="specs/01-data-contracts.md#22-trade">trades</a>: add new / reduce size / remove closed
               <a href="specs/01-data-contracts.md#22-trade">closed_trades</a>: append closed records, including partial closes
               _cash += realized profit/loss - commissions
               <a href="specs/01-data-contracts.md#21-order">orders</a>: remove filled or cancelled
           # <a href="specs/01-data-contracts.md#23-position">Position</a> reads <a href="specs/01-data-contracts.md#22-trade">trades</a> automatically
           equity = _cash + unrealized profit/loss at this candle's Close
           _equity[i] = equity
           if equity &lt;= 0: close all open <a href="specs/01-data-contracts.md#22-trade">trades</a> now; zero cash and remaining equity history; stop

       <a href="specs/01-data-contracts.md#14-strategy">strategy</a>.next():  # new <a href="specs/01-data-contracts.md#21-order">Orders</a> wait for next candle
           # options: limit, stop, sl, tp, tag
           if buy_condition:
               self.buy(size, **options):
                   require positive whole units or a fraction: 0 &lt; size &lt; 1
                   return self.<a href="specs/01-data-contracts.md#13-_broker">_broker</a>.new_order(+size, **options)
           elif sell_condition:
               self.sell(size, **options):
                   require positive whole units or a fraction: 0 &lt; size &lt; 1
                   return self.<a href="specs/01-data-contracts.md#13-_broker">_broker</a>.new_order(-size, **options)
   </code></pre>

## 1. New Order

<pre><code><a href="specs/01-data-contracts.md#13-_broker">broker</a>.new_order(size, limit=None, stop=None, sl=None, tp=None, tag=None):
    reference_price = limit or stop or spread-adjusted current Close
    require sl &lt; reference_price &lt; tp for buy; tp &lt; reference_price &lt; sl for sell
    # skip any missing sl / tp bound
    <a href="specs/01-data-contracts.md#21-order">order</a> = <a href="specs/01-data-contracts.md#21-order">Order</a>(<a href="specs/01-data-contracts.md#13-_broker">broker</a>, size, limit, stop, sl, tp, parent_trade=None, tag=tag)
    if exclusive_orders:
        cancel pending <a href="specs/01-data-contracts.md#21-order">orders</a> except attached stop-loss / take-profit exits
        for <a href="specs/01-data-contracts.md#22-trade">trade</a> in <a href="specs/01-data-contracts.md#13-_broker">broker</a>.<a href="specs/01-data-contracts.md#22-trade">trades</a>:
            <a href="specs/01-data-contracts.md#22-trade">trade</a>.close()  # queue a closing <a href="specs/01-data-contracts.md#21-order">Order</a>
    <a href="specs/01-data-contracts.md#13-_broker">broker</a>.<a href="specs/01-data-contracts.md#21-order">orders</a>.append(<a href="specs/01-data-contracts.md#21-order">order</a>)
    return <a href="specs/01-data-contracts.md#21-order">order</a>  # pending; execution happens in <a href="specs/01-data-contracts.md#13-_broker">broker</a>.next()
</code></pre>
