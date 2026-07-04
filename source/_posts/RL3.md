---
title: 强化学习3——贝尔曼最优
categories:
  - RL
  - 技术
tags: [强化学习, 数学原理]
date: 2026-06-28
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  强化学习Chapter3——贝尔曼最优
</div>

本节的核心概念：optimal state value、optimal policy；基本工具：Bellman Optimality Equation（BOE贝尔曼最优方程）

- 当我们计算State Value时，我们实际上是在评估一个策略的好坏。我们希望找到一个最优策略，使得在每个状态下，采取该策略所获得的期望回报最大化。
  - 如果有$$v_\pi(s) \geq v_{\pi'}(s) \quad\text{ for all } s \in \mathcal{S}$$那么我们就说策略$\pi$优于策略$\pi'$。
# Bellman Optimality Equation（BOE贝尔曼最优方程）
贝尔曼最优方程是强化学习中用于描述最优策略的核心方程。它基于动态规划的思想，定义了在每个状态下，采取最优策略所能获得的最大期望回报。
## 推导
- 我们先回顾Bellman Eqaution：
  $$v_\pi(s) = \sum_{a} \pi(a|s) (\sum_{r} p(r | s, a) r + \gamma \sum_{s'} p(s' | s, a) v_\pi(s'))$$
- 对于最优策略$\pi^*$，我们希望在每个状态$s$下，选择一个动作$a$，使得期望回报最大化。因此，我们可以将Bellman Equation中的策略$\pi$替换为最优策略$\pi^*$，并引入最大化操作：
$$
\begin{aligned}
v(s) &= \max_{\pi} \sum_{a} \pi(a|s) \left( \sum_{r} p(r|s, a)r + \gamma \sum_{s'} p(s'|s, a)v(s') \right), \quad \forall s \in \mathcal{S} \\
&= \max_{\pi} \sum_{a} \pi(a|s) q(s, a) \quad s \in \mathcal{S}
\end{aligned}
$$
  - 我们已知$p(r|s, a)$和$p(s'|s, a)$
  - $v(s),v(s')$是我们想要求解的最优状态值函数
- **matrix-vector形式**表示为
$$\mathbf{v} = \max_{\pi} \left( \mathbf{r} + \gamma \mathbf{P} \mathbf{v} \right)$$
## 最优化问题
$$\max_{\pi} \sum_{a} \pi(a|s) q(s, a) \quad s \in \mathcal{S}$$
- 考虑到
  - $\sum_{a} \pi(a|s) = 1$
  - 我们可以得到$$\max_{\pi} \sum_{a} \pi(a|s) q(s, a) = \max_{a \in \mathcal{A}(s)} q(s, a)$$此时最优的目标被达到当$$\pi^*(a|s) = \begin{cases} 1, & a = \arg\max_{a' \in \mathcal{A}(s)} q(s, a') \\ 0, & \text{otherwise} \end{cases}$$
- 我们将BOE的形式写为函数形式：
$$f(v) := \max_{\pi} \left( \mathbf{r} + \gamma \mathbf{P} \mathbf{v} \right)$$
  - **所以我们只需要求解**$$v = f(v)$$
  对应单个状态$s$的形式为$$[f(v)]_s := \max_{\pi} \sum_{a} \pi(a|s) q(s, a)$$

## Contractive Mapping（收缩映射）：求解Bellman Optimality Equation的核心数学工具
不动点与收缩映射的定义如下：
![](/blog-img/RL3/image.png)
- Theorem定理
  - **Existence and Uniqueness Theorem**：如果一个映射是收缩映射，那么它在该空间中有唯一的不动点，并且通过迭代该映射可以收敛到这个不动点。
  - **Algorithm**：Banach Fixed Point Iteration（Banach不动点迭代法）
    1. 初始化一个任意的向量$x_0$。
    2. 对于每个迭代步骤$t$，计算$x_{t+1} = f(x_t)$。
    3. 重复步骤2，直到$x_t$收敛到不动点$x^*$。
    - ***此外，对于矩阵/向量函数，该结论也适用***。
    - ***Bellman Optimality Equation满足收缩映射的条件，即***
      - 对于$$v = f(v) = \max_{\pi} \left( \mathbf{r} + \gamma \mathbf{P} \mathbf{v} \right)$$总存在一个唯一的不动点$v^*$，并且通过迭代$f$可以收敛到$v^*$，即
      $$v_{k+1} = f(v_k) = \max_{\pi} \left( \mathbf{r} + \gamma \mathbf{P} \mathbf{v}_k \right)$$
      - **给定任意初始猜测 $v_0$，该序列 $\{v_k\}$ 会以指数级速度收敛至 $v^*$。收敛速度由 $\gamma$ 决定。**
## 解的最优性
- 假设我们已经找到了最优状态值函数$v^*$，他满足
$$v^* = \max_{\pi} \left( \mathbf{r} + \gamma \mathbf{P} \mathbf{v}^* \right)$$
假设我们有一个策略$\pi^*$，有$$\pi^* = \arg\max_{\pi} \left( \mathbf{r}_\pi + \gamma \mathbf{P}_\pi \mathbf{v}^* \right)$$
所以有$v^* = \mathbf{r}_{\pi^*} + \gamma \mathbf{P}_{\pi^*} \mathbf{v}^*$，这也是一个贝尔曼方程。
  - **此贝尔曼方程就是贝尔曼最优方程，所以可知贝尔曼最优公式是特殊的贝尔曼方程。那么自然这里的$v^*$就是$v_{\pi^*}$对应的State Value**，且此为最优的状态值函数（这一步存在严格的数学证明）。
  - 那么对于任意状态$s\in \mathcal{S}$，对于一个**确定性贪心策略** $\pi^*$，该策略在该状态下选择的动作$a^*$是概率为1的。
  - ***记住，Optimal State Value Function $v^*$是唯一的，但 Optimal Policy $\pi^*$可能不唯一。***
# 分析最优策略
$$v(s) = \max_{\pi} \sum_{a} \pi(a|s) \left( \sum_{r} {\color{red}p(r|s, a)}r + {\color{red}\gamma} \sum_{s'} {\color{red}p(s'|s, a)}v(s') \right)$$
> Q:如果我们希望得到最优策略，我们要设计什么？
- Reward Design $r$
- Discount Factor $\gamma$
- System Model $p(s'|s, a)$,$p(r|s, a)$
- **上述公式中红色部分是我们设计用于求解最优策略的关键因素**。我们可以通过调整奖励函数、折扣因子以及系统模型来影响最优策略的选择。
- $v(s),v(s'),\pi(a | s)$ are unknown to be calculated