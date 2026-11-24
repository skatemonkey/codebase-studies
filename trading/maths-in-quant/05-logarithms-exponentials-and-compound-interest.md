# Logarithms, Exponentials, and Compound Interest 对数、指数与复利

[All studies](../../README.md) · [Trading](../README.md) · [Maths in Quant overview](README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)

## 0. Overview

> Exponential and logarithmic functions describe compound financial growth and returns.<br>指数函数和对数函数描述金融中的复合增长与收益。

- Compound Interest 复利
   - Earns returns on both the original principal and previously accumulated returns.
   - Discrete Compounding 离散复利 — $FV=PV\left(1+\frac{r}{n}\right)^{nt}$
      - Grows money at fixed compounding intervals.
   - Continuous Compounding 连续复利 — $FV=PV\cdot e^{rt}$
      - Grows money by compounding continuously.
- Logarithm 对数函数 — $\ln(x)$
   - Reverses exponential growth and converts multiplication into addition.
- Return 收益率
   - Simple Return 简单收益率 — $R_t=\frac{P_t}{P_{t-1}}-1$
      - Measures the percentage price change between two points in time.
   - Log Return 对数收益率 — $r_t=\ln\left(\frac{P_t}{P_{t-1}}\right)$
      - Measures growth in an additive form across time.
