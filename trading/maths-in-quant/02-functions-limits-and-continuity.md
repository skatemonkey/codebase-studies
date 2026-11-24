# Functions, Limits, and Continuity 函数、极限与连续性

[All studies](../../README.md) · [Trading](../README.md) · [Maths in Quant overview](README.md)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. Function](#1-function)
> - [2. Limit](#2-limit)
> - [3. Continuity](#3-continuity)
> - [4. Core Formulas](#4-core-formulas)

## 0. Overview

- [Function](#1-function)
   - Maps an input to an output.
   - Example: Interest rate → bond price.
- [Limit](#2-limit)
   - Describes the value a function approaches.<br>极限描述“趋势”而非“到达”。
   - Example: Yield approaching 0 → bond price approaching ¥125.
- [Continuity](#3-continuity)
   - Describes a function that changes without a hole or jump.
   - Market models often assume continuity, but real prices can jump.
- Python demonstrations
   - Plots the bond price–yield curve to show higher yield → lower bond price.
   - Reduces yield toward 0 to show bond price approaching ¥125.
   - Increases compounding frequency to show discrete compounding approaching continuous compounding `e^(rt)`.

## 1. Function

> Maps an input to an output.

- How to read a function
   - Input: the value given to the function.
   - Function: the rule that uses the input.
   - Output: the result produced by the rule.
   - Example: `yield → bond-pricing rule → bond price`.

- Financial mappings

  | Financial Scenario | Input | Output | Function |
  | --- | --- | --- | --- |
  | Bond pricing 债券定价 | Yield `y` | Bond price `P` | `P(y)` |
  | Option pricing 期权定价 | Underlying asset price `S` | Option value `C` | `C(S)` |
  | Compound interest 复利计算 | Time `t` | Future value `A` | `A(t)` |
  | Risk measurement 风险度量 | Asset-weight vector `w` | Portfolio volatility `σₚ` | `σₚ(w)` |

- Bond price–yield example
   - Relationship: `yield → bond-pricing rule → bond price`.
   - Formula: $P(y)=\sum_{t=1}^{T}\frac{C}{(1+y)^t}+\frac{F}{(1+y)^T}$.
   - A higher yield discounts every future payment more heavily, producing a lower bond price.
   - Characteristics
      - Monotonicity 单调性: yield increases → bond price always decreases.
      - Nonlinearity 非线性: equal changes in yield do not produce equal changes in price.
      - Convexity 凸性: the curve bends, so a yield decrease raises the price more than an equal yield increase lowers it.

## 2. Limit

> Describes the value a function approaches.<br>极限描述“趋势”而非“到达”。
>
> - Example: `4.9 → 4.99 → 4.999` approaches `5`.

- How to read a limit
   - $\lim_{x \to a} f(x)=L$ means `x approaches a → f(x) approaches L`.

- Financial intuitions 金融直觉
   - These are three examples of the same limit concept, not three types of limits.
   1. Yield Approaching Zero 收益率趋近于零
      - Input: yield $y\to0$.
      - Output: bond price $P(y)\to¥125$.
      - Meaning: no discounting, so five ¥5 coupons plus the ¥100 principal approach ¥125.
   2. Compounding Frequency Approaching Infinity 复利频率趋近于无穷
      - Input: compounding frequency $n\to\infty$.
      - Output: $\left(1+\frac{r}{n}\right)^{nt}\to e^{rt}$.
      - Meaning: increasingly frequent discrete compounding approaches continuous compounding.
   3. Market-Opening Gap 市场开盘跳空
      - Input: time approaches the opening from before and after.
      - Output: price approaches ¥100 from the left and ¥105 from the right.
      - Meaning: the two sides disagree, so one shared limit does not exist.

## 3. Continuity

> Describes a function that changes without a hole or jump.
>
> - Continuous: `$1 → $2 → $3`
> - Not continuous: `$1 → $2 → $10`

- Three conditions at $x=a$
   1. $f(a)$ exists.
   2. $\lim_{x\to a}f(x)$ exists.
   3. $\lim_{x\to a}f(x)=f(a)$.
- Financial Meaning 金融意义
   1. Continuous-Price Assumption 连续价格假设
      - Point: models treat price as a continuous path $S(t)$.
      - Why: this enables calculus and models such as Black–Scholes.
   2. Real-Market Jumps 真实市场跳跃
      - Point: opening gaps, price adjustments, and sudden news can make prices jump.
      - Why: real price movement does not always follow the continuous assumption.
   3. Model Risk 模型风险
      - Point: continuous models can underestimate losses caused by jumps.
      - Why: sudden price changes are excluded from the assumption.

## 4. Core Formulas

<table>
  <thead>
    <tr>
      <th>Category</th>
      <th>Formula</th>
      <th>Expression</th>
      <th>Related Concept</th>
      <th>Role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">Mathematical Foundation<br>数学基础</td>
      <td>Function Limit Definition 函数极限定义</td>
      <td>$\lim_{x\to a}f(x)=L \iff \forall\varepsilon>0,\exists\delta>0:0&lt;\lvert x-a\rvert&lt;\delta\Rightarrow\lvert f(x)-L\rvert&lt;\varepsilon$</td>
      <td>Limit</td>
      <td>Defines a limit precisely.</td>
    </tr>
    <tr>
      <td>One-Sided Limits 单侧极限</td>
      <td>$\lim_{x\to a^-}f(x)=\lim_{x\to a^+}f(x)=L$</td>
      <td>Limit</td>
      <td>Tests whether both directions approach the same value.</td>
    </tr>
    <tr>
      <td>Continuity at a Point 点连续定义</td>
      <td>$\lim_{x\to a}f(x)=f(a)$</td>
      <td>Continuity</td>
      <td>Tests whether the approaching value equals the actual value.</td>
    </tr>
    <tr>
      <td rowspan="2">Financial Application<br>金融应用</td>
      <td>Continuous Compounding Limit 连续复利极限</td>
      <td>$\lim_{n\to\infty}\left(1+\frac{r}{n}\right)^{nt}=e^{rt}$</td>
      <td>Limit</td>
      <td>Converts increasingly frequent compounding into continuous compounding.</td>
    </tr>
    <tr>
      <td>Coupon-Bond Pricing Function 附息债券定价函数</td>
      <td>$P(y)=\sum_{t=1}^{T}\frac{C}{(1+y)^t}+\frac{F}{(1+y)^T}$</td>
      <td>Function</td>
      <td>Maps yield to bond price.</td>
    </tr>
  </tbody>
</table>
