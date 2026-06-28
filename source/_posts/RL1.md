---
title: 强化学习1——基本概念
categories:
  - RL
  - 技术
tags: [强化学习, 数学原理]
date: 2026-06-28
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  强化学习Chapter1——基本概念
</div>

**强化学习**（Reinforcement Learning, RL）是一种机器学习方法，它通过与环境的交互来学习最优策略，以最大化累积奖励。强化学习的核心思想是智能体（Agent）在环境（Environment）中采取行动（Action），根据环境的反馈（Reward）调整其行为，从而逐步优化其策略（Policy）。

# 强化学习基本概念

- **State（状态）**：agent 有关的环境的状态（status）。
  - **State space（状态空间）**：所有可能的状态的集合。
- **Action（动作）**：agent 可以采取的行为。
  - **Action space（动作空间）**：所有可能的动作的集合。action 依赖于 state，对于不同的 state，action space 可能不同。
- **Policy（策略）**：agent 在一个 State 下采用何种 Action 的规则。
  - **Deterministic policy（确定性策略）**：在给定状态下，策略总是选择相同的动作，即某项动作被选择的概率为 1，其余动作概率为 0。
  $$\pi(a|s) = P(A_t = a \mid S_t = s) = \begin{cases} 1, & \text{if } a = \mu(s) \\ 0, & \text{if } a \neq \mu(s) \end{cases}$$
  - **Stochastic policy（随机策略）**：在给定状态下，策略以一定概率分布选择不同的动作。
  $$\pi(a|s) = P(A_t = a \mid S_t = s)$$
  - 策略（无论确定还是随机）作为概率分布，满足以下公理：
    - **非负性**：$\pi(a|s) \geq 0, \quad \forall a \in \mathcal{A}, s \in \mathcal{S}$
    - **归一性**：$\sum_{a \in \mathcal{A}} \pi(a|s) = 1, \quad \forall s \in \mathcal{S}$ （对于离散动作空间）
  - 确定性策略与随机策略的关系：确定性策略是随机策略的特例，即对于每个状态 $s$，存在某个动作 $a$ 使得 $\pi(a|s) = 1$；而严格意义上的随机策略则不存在这样的动作——即 $\forall a \in \mathcal{A}, s \in \mathcal{S}$，都有 $\pi(a|s) < 1$。
- **Reward（奖励）**：agent 在采取某个动作后，环境给予的反馈信号，用于评估该动作的好坏。
  - 同样存在确定性和随机性奖励的概念：
    - **Deterministic reward（确定性奖励）**：在给定状态和动作下，奖励总是相同的。
    - **Stochastic reward（随机性奖励）**：在给定状态和动作下，奖励可能不同，具有一定的概率分布。
- **Trajectory（轨迹）**：agent 在环境中经历的一系列状态、动作和奖励的序列（a state-action-reward chain）。
- **Return（回报）**：对于一个 trajectory，回报是从当前时间步开始，未来所有奖励的**折扣累积和**。通常使用折扣因子 $\gamma$ 来计算未来奖励的现值。
  - **Discount rate（折扣因子）** $\gamma \in [0, 1]$：
    - 当 $\gamma$ 趋近于 0 时，远期奖励迅速衰减，return 主要取决于**下一步**的即时奖励（agent 变得"近视" / myopic）。
    - 当 $\gamma$ 趋近于 1 时，远期奖励衰减很慢，return 对远近奖励几乎**同等看重**（agent 变得"远视" / farsighted），更注重长期累积回报。
    $$\text{Return: } G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$$
- **Episode（回合）**：伴随一个 terminal state 的 trajectory 称为一个 episode，也叫一个 trial。**Episode 通常是有限步的**。
  - 有些任务没有终止状态，称为 **continuing tasks**（持续性任务）。
  - 事实上，我们通常会把 episodic tasks 转化成 continuing tasks：
    - **Option 1**：将 **absorbing state 的 reward 设置为 0**，进入该状态后始终停留在其中并获得 0 奖励（相当于将 episode 无限延续下去）。
    - **Option 2**：将 **terminal state 视作一个普通状态**，并且假设在该状态下存在一个较好的策略可以继续收集 reward。

# MDP（Markov Decision Process，马尔可夫决策过程）

强化学习通常使用马尔可夫决策过程（Markov Decision Process, MDP）来建模。MDP 是一个数学框架，用于描述具有随机性和决策性的环境。

## MDP 要素（Sets）

- **状态集**（State Space, $\mathcal{S}$）：所有可能状态的集合。
- **动作集**（Action Space, $\mathcal{A}(s)$）：在状态 $s$ 下所有可能动作的集合。
- **奖励集**（Reward Space, $\mathcal{R}(s,a)$）：在状态 $s$ 下采取动作 $a$ 后所有可能奖励的集合。
- **状态转移概率**（State Transition Probability, $\mathcal{P}(s'|s,a)$）：在状态 $s$ 下采取动作 $a$ 后转移到状态 $s'$ 的概率。
- **奖励概率**（Reward Probability, $\mathcal{P}(r|s,a)$）：在状态 $s$ 下采取动作 $a$ 后获得奖励 $r$ 的概率。
- **策略**（Policy, $\pi(a|s)$）：在状态 $s$ 下采取动作 $a$ 的概率分布。
- **当 Policy 确定时，Markov Decision Process 退化为一个 Markov Process**（此时 "Decision" 消失，因为动作选择已无不确定性）。

## Markov Property（马尔可夫性质）

**马尔可夫性质**是指系统的未来状态仅依赖于当前状态和当前动作，而与过去的状态和动作无关：

$$P(S_{t+1} \mid S_t, A_t) = P(S_{t+1} \mid S_0, A_0, S_1, A_1, \ldots, S_t, A_t)$$

上述公式展示了**马尔可夫状态转移性质**；将其中的 $S_{t+1}$ 换为 $R_{t+1}$，则得到**马尔可夫奖励性质**，同样成立。
