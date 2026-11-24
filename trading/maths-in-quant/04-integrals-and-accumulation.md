# Integrals and Accumulation 积分与累积

[All studies](../../README.md) · [Trading](../README.md) · [Maths in Quant overview](README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. Financial Applications](#1-financial-applications)

## 0. Overview

> An integral continuously adds small pieces to obtain a total.<br>积分将连续的小量累积为总量。

- Definite Integral 定积分 — $\int_a^b f(x)\,dx$
   - Calculates the total area under a graph between two points.
   - Fundamental Theorem of Calculus 微积分基本定理 — $\int_a^b f(x)\,dx=F(b)-F(a)$
      - Calculates the integral by finding a function whose derivative is $f(x)$.
      - Example
         1. Function: $f(x)=2x$.
         2. Antiderivative: $F(x)=x^2$.
         3. Integral: $\int_0^3 2x\,dx=[x^2]_0^3=9$.
   - Improper Integral 反常积分 — $\int_a^\infty f(x)\,dx$
      - Calculates the total area when an endpoint is infinity or the graph becomes infinite.
- [Financial Applications 金融应用](#1-financial-applications)
   - Uses integrals to accumulate continuous financial values.

## 1. Financial Applications

> Uses integrals to accumulate continuous financial values.

- Continuous Cash-Flow Discounting 连续现金流贴现
   - Adds discounted cash flows from time $0$ to $T$.
   - Formula: $PV=\int_0^T C(t)e^{-rt}\,dt$.
- Perpetuity 永续年金
   - Calculates the present value of a continuous cash flow that continues forever.
   - Integral: $PV=\int_0^\infty Ce^{-rt}\,dt$.
   - Result: $PV=\frac{C}{r}$.
- Expected Option Payoff 期权预期收益
   - Adds every possible payoff weighted by its probability.
   - Formula: $E[\max(S_T-K,0)]=\int_K^\infty(S_T-K)f(S_T)\,dS_T$.
- Tail Risk 尾部风险
   - VaR: Identifies the loss threshold exceeded with probability $1-\alpha$.
      - Formula: $\int_{\mathrm{VaR}_\alpha}^{\infty}f_L(l)\,dl=1-\alpha$.
   - CVaR: Calculates the average loss after VaR is exceeded.
      - Formula: $\mathrm{CVaR}_\alpha=\frac{\int_{\mathrm{VaR}_\alpha}^{\infty}l f_L(l)\,dl}{1-\alpha}$.
