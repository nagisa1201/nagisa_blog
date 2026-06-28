---
title: 强化学习2——贝尔曼公式
categories:
  - RL
  - 技术
tags: [强化学习, 数学原理]
date: 2026-06-28
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  强化学习Chapter2——贝尔曼公式
</div>

**本章核心为 State Value 与核心工具 Bellman Equation**。

# 如何计算 Return

- Option1：直接法
$$v_1 = r_1 + \gamma(r_2 + \gamma r_3 + \gamma^2 r_4 + \cdots)$$
- Option2：迭代法
$$v_1 = r_1 + \gamma v_2$$其中$$v_2 = r_2 + \gamma v_3$$以此类推。
通过 Option2 的迭代法，我们可以将 Return 的计算转化为一个递归问题，这就是贝尔曼方程的核心思想。

$$
\underbrace{\begin{bmatrix} v_1 \\\\ v_2 \\\\ v_3 \\\\ v_4 \end{bmatrix}}_{\mathbf{v}} 
= \begin{bmatrix} r_1 \\\\ r_2 \\\\ r_3 \\\\ r_4 \end{bmatrix} 
+ \begin{bmatrix} \gamma v_2 \\\\ \gamma v_3 \\\\ \gamma v_4 \\\\ \gamma v_1 \end{bmatrix} 
= \underbrace{\begin{bmatrix} r_1 \\\\ r_2 \\\\ r_3 \\\\ r_4 \end{bmatrix}}_{\mathbf{r}} 
+ \gamma \underbrace{\begin{bmatrix} 0 & 1 & 0 & 0 \\\\ 0 & 0 & 1 & 0 \\\\ 0 & 0 & 0 & 1 \\\\ 1 & 0 & 0 & 0 \end{bmatrix}}_{\mathbf{P}} 
\underbrace{\begin{bmatrix} v_1 \\\\ v_2 \\\\ v_3 \\\\ v_4 \end{bmatrix}}_{\mathbf{v}}
$$

*which can be rewritten as*

$$
\mathbf{v} = \mathbf{r} + \gamma \mathbf{P} \mathbf{v}
$$

## 对于一个单步过程而言：

$$S_t \xrightarrow{A_t} R_{t+1}, S_{t+1}$$

- **这些变化的步骤均由概率决定**
- $S_t \rightarrow A_t$ is governed by policy $\pi(A_t = a \mid S_t = s)$
- $S_t, A_t \rightarrow R_{t+1}$ is governed by reward probability $p(R_{t+1} = r \mid S_t = s, A_t = a)$
- $S_t, A_t \rightarrow S_{t+1}$ is governed by state transition probability $p(S_{t+1} = s' \mid S_t = s, A_t = a)$

## 而对于 multi-step process 而言：

$$S_t \xrightarrow{A_t} R_{t+1}, S_{t+1} \xrightarrow{A_{t+1}} R_{t+2}, S_{t+2} \xrightarrow{A_{t+2}} R_{t+3}, S_{t+3} \cdots$$

- discount return 计算为
$$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$$
其中 $\gamma \in [0, 1)$。

# State Value

实际上 State Value 就是 $G_t$ 即 return 的 Expected Value，记为 $v_\pi(s)$，即在给定状态 $s$ 下，按照策略 $\pi$ 采取动作后，未来所有奖励的折扣累积和的期望值，公式记作

$$v_\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]$$

- $v_\pi(s)$ 是关于状态 $s$ 的函数，当前状态取一个具体的值。
- $v_\pi(s)$ 基于 Policy，在不同的策略下，$v_\pi(s)$ 的值也不同。

> **Q: State Value 和 Return 的区别是什么？**
> A: Return 是一个随机变量，State Value 是 Return 的期望值。**Return 是具体的一次采样结果，而 State Value 是对所有可能结果的期望**，表示在给定状态下按照策略采取动作后，未来所有奖励的折扣累积和的期望值。
>
> *若从一个 State 出发，策略和环境都是确定性的（有且仅有一种 trajectory），则 Return 和 State Value 数值相等。若存在多种可能的 trajectory，则 Return 是随机变量，State Value 是其期望值*。

# Bellman Equation

Bellman Equation 描述了不同状态的 state value 之间的关系。它将一个状态的价值表示为该状态下采取某个动作后，未来所有奖励的折扣累积和的期望值。

## 公式推导与含义

同样考虑一个 random trajectory：
$$S_t \xrightarrow{A_t} R_{t+1}, S_{t+1} \xrightarrow{A_{t+1}} R_{t+2}, S_{t+2} \xrightarrow{A_{t+2}} R_{t+3}, S_{t+3} \cdots$$

- return $G_t$ 的计算为
$$
\begin{aligned}
G_t &= R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \dots, \\\\
    &= R_{t+1} + \gamma(R_{t+2} + \gamma R_{t+3} + \dots), \\\\
    &= R_{t+1} + \gamma G_{t+1}
\end{aligned}
$$

- 代入 State Value 的定义，得到中间项
$$
\begin{aligned}
v_\pi(s) &= \mathbb{E}_\pi[G_t \mid S_t = s],\\\\
         &= \mathbb{E}_\pi[R_{t+1} + \gamma G_{t+1} \mid S_t = s],\\\\
         &= \mathbb{E}_\pi[R_{t+1} \mid S_t = s] + \gamma \mathbb{E}_\pi[G_{t+1} \mid S_t = s]
\end{aligned}
$$

- **First, calculate the first term** $\mathbb{E}[R_{t+1} \mid S_t = s]$
$$
\begin{aligned}
\mathbb{E}[R_{t+1} \mid S_t = s] &= \sum_{a} \pi(a|s) \mathbb{E}[R_{t+1} \mid S_t = s, A_t = a] \\\\
                                  &= \sum_{a} \pi(a|s) \sum_{r} p(r|s, a) r
\end{aligned}
$$
  - This is the mean of **immediate rewards**，体现了在状态 $s$ 下，按照策略 $\pi$ 采取动作后的**即时奖励**

- **Next, calculate the second term** $\mathbb{E}[G_{t+1} \mid S_t = s]$
$$
\begin{aligned}
\mathbb{E}[G_{t+1} \mid S_t = s] &= \sum_{s'} \mathbb{E}[G_{t+1} \mid S_t = s, S_{t+1} = s'] \, p(s'|s) \\\\
                                  &= \sum_{s'} \mathbb{E}[G_{t+1} \mid S_{t+1} = s'] \, p(s'|s) \\\\
                                  &= \sum_{s'} v_{\pi}(s') \, p(s'|s) \\\\
                                  &= \sum_{s'} v_{\pi}(s') \sum_{a} p(s'|s, a) \pi(a|s)
\end{aligned}
$$
  - This is the mean of **future rewards**，体现了在状态 $s$ 下，按照策略 $\pi$ 采取动作后，未来所有奖励的折扣累积和的期望值。
    - note that：第一步到第二步依据 **Markov Property**，即未来状态只依赖于当前状态和当前动作，而与过去的状态和动作无关——在已知下一状态为 $s'$ 的情况下，$S_t$ 对 $G_{t+1}$ 没有额外信息。

- Finally, combine the two terms to get the ***Bellman Equation***:
$$
\begin{aligned}
v_\pi(s) &= \mathbb{E}[R_{t+1} \mid S_t = s] + \gamma \mathbb{E}[G_{t+1} \mid S_t = s], \\\\
         &= \underbrace{\sum_a \pi(a|s) \sum_r p(r|s, a) r}_{\text{mean of immediate rewards}} + \gamma \underbrace{\sum_a \pi(a|s) \sum_{s'} p(s'|s, a) v_\pi(s')}_{\text{mean of future rewards}}, \\\\
         &= \sum_a \pi(a|s) \left[ \sum_r p(r|s, a) r + \gamma \sum_{s'} p(s'|s, a) v_\pi(s') \right], \quad \forall s \in \mathcal{S}.
\end{aligned}
$$

  - $v_\pi(s)$ 与 $v_\pi(s')$ 就是我们希望计算的 state value。
  - $\pi(a|s)$ 是已知的策略，***计算上述 State Value 的过程，实际上就是在评估一个已知策略的好坏***，若策略已知，则 State Value 是可以计算的。
  - $p(r|s, a)$ 和 $p(s'|s, a)$ 代表动态环境模型，若知道了环境模型，则为已知环境问题（Model-Based），反之为 Model-Free 问题。
    - 此处尤其对刚开始学习强化学习的同学做概念辨析：
    ① $\pi(a|s)$ 指的是策略概率，是由 agent 决定的，在处于状态 $s$ 下，agent 采取动作 $a$ 的概率；
    ② $p(r|s, a)$ 和 $p(s'|s, a)$ 指的是环境奖励、状态转移概率，是由环境决定的，在处于状态 $s$ 下，采取动作 $a$ 后，环境给出奖励 $r$ 和转移到下一状态 $s'$ 的概率。

## Bellman Equation 的矩阵形式

> **Q: What is the significance of the Bellman Equation?**
> A: 这个式子对状态空间中的每一个状态 $s$ 都成立，体现了不同状态的 state value 之间的关系。**若有 n 个状态，则有 n 个方程，组成一个线性方程组，解这个方程组即可得到所有状态的 state value**。

- 首先，我们需要简化 Bellman Equation 的形式
$$v_\pi(s) = \sum_a \pi(a|s) \left[ \sum_r p(r|s, a) r + \gamma \sum_{s'} p(s'|s, a) v_\pi(s') \right]$$
$$v_\pi(s) = r_\pi(s) + \gamma \sum_{s'} \mathbf{p}_\pi(s, s') \, v_\pi(s')$$

$$ r_\pi(s) = \sum_a \pi(a|s) \sum_r p(r|s, a) r \qquad ①$$
①式代表我从状态 $s$ 出发，按照策略 $\pi$ 采取动作后，环境给出的即时奖励的期望值。

$$ \mathbf{p}_\pi(s, s') = \sum_a \pi(a|s) \, p(s'|s, a) \qquad ②$$
②式代表从状态 $s$ 出发，按照策略 $\pi$ 采取动作后，转移到下一状态 $s'$ 的概率。

- 所以，对于所有状态 $s \in \mathcal{S}$，我们可以将 Bellman Equation 写成矩阵形式：
$$\mathbf{v}_\pi = \mathbf{r}_\pi + \gamma \mathbf{P}_\pi \mathbf{v}_\pi$$
  - $\mathbf{v}_\pi = [v_\pi(s_1), v_\pi(s_2), \dots, v_\pi(s_n)]^T \in \mathbb{R}^n$
  - $\mathbf{r}_\pi = [r_\pi(s_1), r_\pi(s_2), \dots, r_\pi(s_n)]^T \in \mathbb{R}^n$
  - $\mathbf{P}_\pi$ 是一个 $n \times n$ 的状态转移概率矩阵。

- 以一个题目作为例子：
![](/blog-img/RL2/image.png)

# 求解 State Value

利用矩阵形式的 Bellman Equation，我们可以通过线性代数的方法求解 State Value。
$$\mathbf{v}_\pi = \mathbf{r}_\pi + \gamma \mathbf{P}_\pi \mathbf{v}_\pi$$

- ① 求逆法：但实际不会使用，因为当状态空间很大时，求逆的计算量非常大，且数值不稳定。
$$\mathbf{v}_\pi = (\mathbf{I} - \gamma \mathbf{P}_\pi)^{-1} \mathbf{r}_\pi$$

- ② 迭代法：通过不断迭代更新 State Value，直到收敛。
$$ \mathbf{v}_{k+1} = \mathbf{r}_\pi + \gamma \mathbf{P}_\pi \mathbf{v}_{k}$$
  - 随机给定一个初始值 $\mathbf{v}_0$，然后不断迭代更新，直到收敛到一个稳定的值 $\mathbf{v}_\pi$。
  - **可以得到一个序列** $\{\mathbf{v}_k\}$，当 $k \to \infty$ 时，$\mathbf{v}_k \to \mathbf{v}_\pi$。

# Action Value

指 agent 在状态 $s$ 下，采取动作 $a$ 后，其可获得的平均 return 期望值：

$$q_\pi(s, a) = \mathbb{E}_\pi[G_t \mid S_t = s, A_t = a]$$

- 其依赖于：
  - 当前状态 $s$，即 agent 所处的环境状态；
  - 当前动作 $a$，即 agent 在当前状态下采取的动作；
  - 策略 $\pi$，即 agent 在当前状态下采取动作的概率分布。
- action value 与 state value 的联系：
$$
\underbrace{\mathbb{E}[G_t \mid S_t = s]}_{v_\pi(s)} = \sum_{a} \underbrace{\mathbb{E}[G_t \mid S_t = s, A_t = a]}_{q_\pi(s,a)} \, \pi(a|s) \qquad ③
$$
  - namely：
$$
{\color{red}v_\pi(s)} = \sum_{a} \pi(a|s) \, {\color{red}q_\pi(s,a)}
$$
  - 将 $v_\pi(s)$ 的结果式代入③式可得：
$$
{\color{red}q_\pi(s, a)} = \sum_{r} p(r|s, a) r + \gamma \sum_{s'} p(s'|s, a) \, {\color{red}v_\pi(s')}
$$
