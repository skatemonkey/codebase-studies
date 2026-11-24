# Derivatives and Differentials 导数与微分

[All studies](../../README.md) · [Trading](../README.md) · [Maths in Quant overview](README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. Financial Sensitivity](#1-financial-sensitivity)

## 0. Overview

> A derivative measures sensitivity; a differential uses that sensitivity to estimate the resulting change.<br>导数衡量敏感度；微分利用该敏感度估算实际变化。

- Derivative 导数
   - First Derivative 一阶导数 — $f'(x)$
      - Measures the instantaneous rate of change.
      - Original function: $f(x)=x^2$ gives the output.
      - Derivative function: $f'(x)=2x$ gives the rate of change.
      - Example: $f'(2)=4$ is the rate of change at $x=2$.
   - Second Derivative 二阶导数 — $f''(x)$
      - Measures how the first derivative changes.
      - Example
         1. Original function: $f(x)=x^2$.
         2. First derivative: $f'(x)=2x$.
         3. Second derivative: $f''(x)=2$.
- Differential 微分 — $df$
   - Estimates the output change caused by a small input change.
   - It is not the second derivative: $f''(x)$ measures how the rate changes, while $df$ estimates the output change.
   - Formula: $df\approx f'(x)\,dx$.
   - Example:
      - Rate at $x=2$: $f'(2)=4$.
      - Input change: $dx=0.1$, so $x$ changes from $2$ to $2.1$.
      - Estimated output change: $df\approx4\times0.1=0.4$.
- Differentiation Rules 求导法则
   - Core calculation rules covered in this chapter.
   - Power Rule 幂函数法则: $\frac{d}{dx}x^n=nx^{n-1}$.
   - Chain Rule 链式法则: $\frac{d}{dx}f(g(x))=f'(g(x))g'(x)$.
   - Product Rule 乘积法则: $\frac{d}{dx}[u(x)v(x)]=u'(x)v(x)+u(x)v'(x)$.
   - Sum, Difference, and Constant Rules 和差与常数倍法则
      - $\frac{d}{dx}[f(x)\pm g(x)]=f'(x)\pm g'(x)$.
      - $\frac{d}{dx}[cf(x)]=cf'(x)$.
- [Financial Sensitivity 金融敏感度](#1-financial-sensitivity)
   - Applies derivatives and differentials to financial values.
   - Examples: bond duration, bond convexity, and option Greeks.

## 1. Financial Sensitivity

> Applies derivatives and differentials to financial values.

- Bond Duration 债券久期
   - Uses the first derivative to estimate how a bond's price responds to a yield change.
   - Formula: $D_{mod}=-\frac{P'(y)}{P(y)}$.
   - Price estimate: $\frac{\Delta P}{P}\approx-D_{mod}\,\Delta y$.
- Bond Convexity 债券凸性
   - Uses the second derivative to improve the duration estimate when the yield change is larger.
   - Formula: $C=\frac{P''(y)}{P(y)}$.
   - Corrected estimate: $\frac{\Delta P}{P}\approx-D_{mod}\,\Delta y+\frac{1}{2}C(\Delta y)^2$.
- Option Greeks 期权希腊字母
   - Use derivatives to measure how an option's value responds to different market variables.
   - Delta: $\Delta=\frac{\partial V}{\partial S}$.
   - Gamma: $\Gamma=\frac{\partial^2V}{\partial S^2}$.
   - Vega: $\frac{\partial V}{\partial\sigma}$.
   - Theta: $\Theta=\frac{\partial V}{\partial t}$.
   - Rho: $\rho=\frac{\partial V}{\partial r}$.
