# Maths in Quant Repository Study

[All studies](../../README.md) · [Trading](../README.md)

> Source:
>
> - [Maths in Quant — 量化金融数学基础教程](https://github.com/pcman-zhang/maths-in-quant)

> Table of Contents
>
> - [0. Overview](#0-overview)
> - [1. Core Mental Model 核心思维框架](#1-core-mental-model-核心思维框架)
> - [2. Chapter Breakdown 章节结构](#2-chapter-breakdown-章节结构)

## 0. Overview

> A repository for learning quantitative-finance mathematics through explanations and Python examples.

- These study notes follow the mathematical framework and chapter sequence of the Maths in Quant repository.

## 1. Core Mental Model 核心思维框架

> Remember this framework first so that every mathematical topic has a clear purpose.

1. Core problem
   - Question: How can we make the best capital-allocation decision now when the future is uncertain?
   - Central trade-off: Return versus risk.
2. Core formula 核心公式
   - Quantitative model 量化模型 = Mathematical hypothesis 数学假设 + Data structure 数据结构 + Parameter estimation 参数估计 + Validation method 验证方法
     - Mathematical hypothesis 数学假设: The relationship assumed by the model.
     - Data structure 数据结构: The data used and how observations are organized.
     - Parameter estimation 参数估计: The model values learned from the data.
     - Validation method 验证方法: The method used to test whether the model works.
3. Major areas of quantitative finance 量化金融的主要领域
   - Asset allocation 资产配置
   - Factor investing 因子投资
   - Derivatives pricing 衍生品定价
   - Statistical arbitrage and high-frequency trading 统计套利与高频交易
4. Layers of mathematical knowledge

   | Layer 层级 | Mathematical Topics 数学主题 |
   | --- | --- |
   | Mathematical Foundations 数学基础 | <ul><li>Functions and limits</li><li>Derivatives and differentials</li><li>Integrals and accumulation</li><li>Logarithms and exponentials</li><li>Series and Taylor expansions</li></ul> |
   | Probability, Statistics, and Linear Algebra 概率统计与线性代数 | <ul><li>Probability fundamentals</li><li>Random variables and distributions</li><li>Expected value, variance, and covariance</li><li>Law of large numbers and central limit theorem</li><li>Hypothesis testing and confidence intervals</li><li>Regression analysis</li><li>Linear algebra</li></ul> |
   | Advanced Methods 进阶方法 | <ul><li>Optimization</li><li>Time-series analysis</li><li>Stochastic analysis</li><li>Monte Carlo simulation</li></ul> |
   | Machine Learning 机器学习 | <ul><li>Supervised learning</li><li>PCA dimensionality reduction</li><li>Clustering</li></ul> |

5. Relationship between the areas and mathematics
   - Different areas depend on different combinations of mathematical tools.
6. Learning sequence
   - Progress from foundational concepts to advanced quantitative methods.
7. Visual map
   - [Interactive Mathematics Knowledge Graph 交互式数学知识图谱](00-maths-in-quant.html) — download and open this HTML file in a browser to interact with it

## 2. Chapter Breakdown 章节结构

1. Phase 1: Foundation Toolbox 第一阶段：基础工具箱

   > Objective: Build the ability to describe financial phenomena using foundational mathematics.

   - [Chapter 2 — Functions, Limits, and Continuity 函数、极限与连续性](02-functions-limits-and-continuity.md)
     - Function 函数 — $f(x)$: Maps an input to an output.
     - Limit 极限 — $\lim_{x\to a}f(x)=L$: Describes the value a function approaches.
     - Continuity 连续性 — $\lim_{x\to a}f(x)=f(a)$: Describes a function that changes without a hole or jump.
   - [Chapter 3 — Derivatives and Differentials 导数与微分](03-derivatives-and-differentials.md)
     - Derivative 导数
       - First Derivative 一阶导数 — $f'(x)$: Measures the instantaneous rate of change.
       - Second Derivative 二阶导数 — $f''(x)$: Measures how the first derivative changes.
     - Differential 微分 — $df$: Estimates the output change caused by a small input change.
   - [Chapter 4 — Integrals and Accumulation 积分与累积](04-integrals-and-accumulation.md)
     - Definite Integral 定积分 — $\int_a^b f(x)\,dx$: Calculates the total area under a graph between two points.
       - Fundamental Theorem of Calculus 微积分基本定理 — $\int_a^b f(x)\,dx=F(b)-F(a)$: Calculates the integral by finding a function whose derivative is $f(x)$.
       - Example
         1. Function: $f(x)=2x$.
         2. Antiderivative: $F(x)=x^2$.
         3. Integral: $\int_0^3 2x\,dx=[x^2]_0^3=9$.
       - Improper Integral 反常积分 — $\int_a^\infty f(x)\,dx$: Calculates the total area when an endpoint is infinity or the graph becomes infinite.
   - [Chapter 5 — Logarithms, Exponentials, and Compound Interest 对数、指数与复利](05-logarithms-exponentials-and-compound-interest.md)
     - Compound Interest 复利: Earns returns on both the original principal and previously accumulated returns.
       - Discrete Compounding 离散复利 — $FV=PV\left(1+\frac{r}{n}\right)^{nt}$: Grows money at fixed compounding intervals.
       - Continuous Compounding 连续复利 — $FV=PV\cdot e^{rt}$: Grows money by compounding continuously.
     - Logarithm 对数函数 — $\ln(x)$: Reverses exponential growth and converts multiplication into addition.
     - Return 收益率
       - Simple Return 简单收益率 — $R_t=\frac{P_t}{P_{t-1}}-1$: Measures the percentage price change between two points in time.
       - Log Return 对数收益率 — $r_t=\ln\left(\frac{P_t}{P_{t-1}}\right)$: Measures growth in an additive form across time.
   - Chapter 6 — Series and Taylor Expansions 级数与泰勒展开
     - Taylor Expansion 泰勒展开: Approximates a smooth function near a chosen point with a polynomial.
       - First-Order Expansion 一阶展开 — $f(x)\approx f(x_0)+f'(x_0)(x-x_0)$: Uses the function value and slope to form a straight-line approximation.
       - Second-Order Expansion 二阶展开 — $f(x)\approx f(x_0)+f'(x_0)(x-x_0)+\frac{f''(x_0)}{2}(x-x_0)^2$: Adds curvature to improve the approximation.
     - Maclaurin Series 麦克劳林级数: A Taylor expansion centred at zero.
     - Remainder 余项: Measures the part not captured by the approximation.
     - Financial Interpretation 金融含义
       - First order: Duration for bonds and Delta for options.
       - Second order: Convexity for bonds and Gamma for options.
2. Phase 2: Probability and Statistics 第二阶段：概率统计

   > Objective: Build a framework for describing uncertainty with probability—the central language of quantitative finance.

   - Chapter 7 — Probability Fundamentals 概率论基础
     - Core ideas: Sample spaces, events, probability axioms, conditional probability, and Bayesian updating.
     - Core formulas: Law of total probability and Bayes' theorem.
   - Chapter 8 — Random Variables and Distributions 随机变量与分布
     - Core ideas: Discrete and continuous random variables and the distributions used to model financial outcomes.
     - Core formulas: PDF, CDF, and normal-distribution density.
   - Chapter 9 — Expected Value, Variance, and Covariance 期望、方差与协方差
     - Expected Value 期望 — $E[X]$: Calculates the probability-weighted average outcome.
     - Variance 方差 — $\operatorname{Var}(X)$: Measures how widely outcomes vary around the expected value.
     - Covariance 协方差 — $\operatorname{Cov}(X,Y)$: Measures whether two assets move together.
       - Correlation 相关系数 — $\rho_{XY}$: Standardizes covariance between $-1$ and $1$.
       - Covariance Matrix 协方差矩阵 — $\Sigma$: Organizes the variances and covariances of multiple assets.
     - Portfolio Model 投资组合模型
       - Expected return — $E[R_p]=w^T\mu$: Combines asset returns according to their portfolio weights.
       - Portfolio variance — $\sigma_p^2=w^T\Sigma w$: Combines portfolio weights with asset co-movement to calculate total risk.
     - Diversification 分散化: Reduces portfolio risk when assets do not move perfectly together.
   - Chapter 10 — Laws of Large Numbers, Central Limit Theorem, and Statistical Inference 大数定律、中心极限定理与统计推断
     - Law of Large Numbers 大数定律: Makes the sample average approach the true average as the sample grows.
     - Central Limit Theorem 中心极限定理: Makes the distribution of the sample average approach a normal distribution.
     - Statistical Inference 统计推断
       - Standard Error 标准误 — $\operatorname{SE}=\frac{\sigma}{\sqrt{n}}$: Measures the uncertainty of an estimate.
       - Confidence Interval 置信区间: Constructs a range around an estimate using its sampling uncertainty.
     - Resampling and Adjustment 重采样与修正
       - Bootstrap 自助法: Estimates uncertainty by repeatedly resampling the observed data.
       - Block Bootstrap 分块自助法: Preserves short-term dependence when resampling time-series data.
       - Effective Sample Size 有效样本量: Estimates how much independent information the data actually contains.
     - Financial Limitation 金融局限: Historical estimates remain useful only while the data-generating process is sufficiently stable.
   - Chapter 11 — Hypothesis Testing and p-Values 假设检验与 p 值
     - Hypothesis Testing 假设检验: Tests whether observed performance provides enough evidence of a real effect.
       - Null Hypothesis 原假设 — $H_0$: Assumes that no real effect exists.
       - Alternative Hypothesis 备择假设 — $H_1$: Assumes that a real effect exists.
     - Test Statistic 检验统计量 — $t=\frac{\bar{x}-\mu_0}{s/\sqrt{n}}$: Measures how far the observed result is from the null hypothesis relative to its uncertainty.
     - p-Value: Measures how unusual the observed result would be if the null hypothesis were true.
     - Testing Risk 检验风险
       - Type I Error 第一类错误: Mistakes noise for a real effect.
       - Type II Error 第二类错误: Fails to detect a real effect.
       - Statistical Power 检验功效: Measures the ability to detect a real effect.
     - Multiple Testing 多重检验: Increases false discoveries when many strategies or factors are tested.
       - Bonferroni Correction 邦费罗尼校正 — $\alpha^*=\frac{\alpha}{m}$: Makes the acceptance threshold stricter as the number of tests increases.
     - Practical Validation 实际验证
       - Statistical significance asks whether the result is distinguishable from noise.
       - Economic significance asks whether the effect remains valuable after costs and risk.
   - Chapter 12 — Regression Analysis 回归分析
     - Core ideas: Linear relationships, coefficient significance, explanatory power, and multicollinearity.
     - Core formulas: OLS estimator, standard error, R², t-statistic, and F-test.
3. Phase 3: Linear Algebra 第三阶段：线性代数

   > Objective: Use vectors and matrices to represent and analyse portfolios containing multiple assets.

   - Chapter 13 — Vectors and Matrices 向量与矩阵
     - Core ideas: Portfolio weights, return vectors, dot products, similarity, and matrix representations of asset histories.
     - Core formulas: Vector operations, dot product, norm, cosine similarity, and matrix–vector multiplication.
   - Chapter 14 — Matrix Operations and Eigen Decomposition 矩阵运算与特征分解
     - Core ideas: Portfolio risk structure, covariance estimation, eigen decomposition, PCA, and eigenportfolios.
     - Core formulas: Matrix multiplication, matrix inverse, eigen equation, eigen decomposition, shrinkage estimation, and condition number.
   - Chapter 15 — Linear Systems and Least Squares 线性方程组与最小二乘法
     - Core ideas: Linear-system solutions, least squares, projection geometry, and residual orthogonality.
     - Core formulas: Normal equation, matrix-form OLS, projection matrix, and geometric interpretation of R².
4. Phase 4: Optimization Theory 第四阶段：优化理论

   > Objective: Express and solve return-maximization and risk-minimization problems under constraints.

   - Chapter 16 — Multivariable Optimization 多元函数优化
     - Core ideas: Directional change, gradients, curvature, convexity, gradient descent, and Newton's method.
     - Core formulas: Gradient, Hessian matrix, gradient-descent update, and Newton update.
   - Chapter 17 — Constrained Optimization and Lagrange Multipliers 带约束优化与拉格朗日乘子
     - Core ideas: Finding optimal portfolios while satisfying equality and inequality constraints.
     - Core formulas: Lagrangian function and KKT conditions.
   - Chapter 18 — Convex Optimization and Numerical Methods 凸优化与数值方法
     - Core ideas: Convex sets, convex functions, global optima, and numerical portfolio optimization.
     - Core formulas: Standard quadratic-programming form.
5. Phase 5: Time-Series Analysis 第五阶段：时间序列分析

   > Objective: Model how financial variables depend on their own history and evolve through time.

   - Chapter 19 — Time-Series Foundations 时间序列基础
     - Core ideas: Stationarity, autocorrelation, and partial autocorrelation.
     - Core formulas: Autocovariance, autocorrelation coefficient, and Ljung–Box test.
   - Chapter 20 — Moving-Average and Autoregressive Models 移动平均与自回归模型
     - Core ideas: Predicting a series from past values and shocks and using differencing to reach stationarity.
     - Core formulas: AR(1), MA(1), and ARIMA(p,d,q).
   - Chapter 21 — GARCH Models GARCH 模型
     - Core ideas: Volatility clustering and the recursive evolution of conditional variance.
     - Core formulas: GARCH(1,1) mean and variance equations.
   - Chapter 22 — Cointegration and Pairs Trading 协整与配对交易
     - Core ideas: Spurious regression, unit roots, long-run equilibrium, cointegration, and error correction.
     - Core formulas: ADF statistic, Johansen tests, and the VECM error-correction equation.
6. Phase 6: Stochastic Analysis 第六阶段：随机分析

   > Objective: Build the mathematical foundation for derivatives pricing and continuous-time financial models.

   - Chapter 23 — From Random Walks to Brownian Motion 从随机游走到布朗运动
     - Core ideas: Moving from discrete random walks to continuous stochastic processes.
     - Core formulas: Brownian-motion definition and Itô isometry.
   - Chapter 24 — Itô's Lemma 伊藤引理
     - Core ideas: Why ordinary calculus fails for Brownian motion and why second-order terms matter.
     - Core formulas: One-variable and multivariable forms of Itô's lemma.
   - Chapter 25 — Black–Scholes Model Black–Scholes 模型
     - Core ideas: Risk-neutral pricing, no-arbitrage reasoning, and the Black–Scholes PDE.
     - Core formulas: European call and put pricing formulas and Greeks.
   - Chapter 26 — Monte Carlo Simulation and Numerical Methods 蒙特卡洛模拟与数值方法
     - Core ideas: Simulation-based pricing, path-dependent derivatives, and variance reduction.
     - Core formulas: Risk-neutral pricing expectation and Monte Carlo estimator.
7. Phase 7: Factor Models and Machine Learning 第七阶段：因子模型与机器学习

   > Objective: Integrate statistics and linear algebra into factor research and extend them to high-dimensional and nonlinear problems.

   - Chapter 27 — Multi-Factor Models 多因子模型
     - Core ideas: Factor construction, factor testing, factor combination, and the separation of alpha and risk models.
     - Core formulas: IC, IR, Fama–MacBeth regression, and factor-based performance attribution.
   - Chapter 28 — Supervised Learning and Regularization 监督学习与正则化
     - Core ideas: Classification, regularization, factor selection, bias–variance trade-off, and cross-validation.
     - Core formulas: Sigmoid function, cross-entropy loss, Lasso, Ridge, Elastic Net, and K-fold cross-validation estimator.
   - Chapter 29 — Clustering Analysis 聚类分析
     - Core ideas: Distance, similarity, asset grouping, market-regime identification, and hierarchical portfolio construction.
     - Core formulas: K-means objective, Ward distance, and silhouette score.
8. Phase 8: From Research to Live Trading 第八阶段：从研究到实盘

   > Objective: Bridge research and live deployment through honest evaluation, cost modelling, and risk discipline.

   - Chapter 30 — The Science of Backtesting 回测的科学
     - Core ideas: Survivorship bias, multiple-testing bias, honest validation, and inflated backtest performance.
     - Core formulas: Expected maximum Sharpe ratio and deflated Sharpe ratio.
   - Chapter 31 — Costs, Capacity, and Execution 成本、容量与执行
     - Core ideas: Trading costs, market impact, strategy capacity, turnover, and execution constraints.
     - Core formulas: Square-root impact law and capacity scaling relation.
   - Chapter 32 — From Research to Live Trading 从研究到实盘
     - Core ideas: Position sizing, drawdown controls, live monitoring, and strategy decay.
     - Core formulas: Kelly criterion and Bayesian state updates.
