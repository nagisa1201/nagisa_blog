<!--
 * @Author: Nagisa 2964793117@qq.com
 * @Date: 2026-06-28 09:54:45
 * @LastEditors: Nagisa 2964793117@qq.com
 * @LastEditTime: 2026-06-28 11:37:20
 * @FilePath: \nagisa_blog\source\_posts\RL1.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
---
title: 强化学习1——基本概念
categories: 
  - RL
  - 技术
tags: [强化学习,数学原理]
date: 2026-6-28
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  强化学习1——基本概念
</div>

**强化学习**（Reinforcement Learning, RL）是一种机器学习方法，它通过与环境的交互来学习最优策略，以最大化累积奖励。强化学习的核心思想是智能体（Agent）在环境（Environment）中采取行动（Action），根据环境的反馈（Reward）调整其行为，从而逐步优化其策略（Policy）。

# 强化学习基本概念
- State（状态）：agent有关的环境的status。
  - state space（状态空间）：所有可能的状态的集合。
- Action（动作）：agent可以采取的行为。
  - action space（动作空间）：所有可能的动作的集合，action依赖于state，对于不同的state，action space可能不同。
- Policy（策略）：agent在一个State下采用何种Action的规则。
  - deterministic policy（确定性策略）：在给定状态下，策略总是选择相同的动作。即某项策略选择概率为1，其他策略概率为0。
$$\pi(a|s) = P(A_t = a \mid S_t = s) = \begin{cases} 1, & \text{if } a = \mu(s) \\ 0, & \text{if } a \neq \mu(s) \end{cases}$$
  - stochastic policy（随机策略）：在给定状态下，策略以一定概率选择不同的动作，没有选择概率为1的策略。
$$\pi(a|s) = P(A_t = a \mid S_t = s)$$
  - 满足公理如下：
    - 非负性： $\pi(a|s) \geq 0, \quad \forall a \in \mathcal{A}, s \in \mathcal{S}$
    - 归一性： $\sum_{a \in \mathcal{A}} \pi(a|s) = 1, \quad \forall s \in \mathcal{S}$ （对于离散动作空间）
    - 无绝对确定动作： $\pi(a|s) < 1, \quad \forall a \in \mathcal{A}, s \in \mathcal{S}$ （对应你提到的“没有选择概率为1的策略”）
- Reward（奖励）：agent在采取某个动作后，环境给予的反馈信号，用于评估该动作的好坏。
  - 同样存在确定性和随机性奖励的概念：
    - deterministic reward（确定性奖励）：在给定状态和动作下，奖励总是相同的。
    - stochastic reward（随机性奖励）：在给定状态和动作下，奖励可能不同，具有一定的概率分布。
- trajectory（轨迹）：agent在环境中经历的一系列状态、动作和奖励的序列（a state-action-reward chain）。
- Return（回报）：对于一个trajectory，回报是从当前时间步开始，未来所有奖励的累积和。通常使用折扣因子 $\gamma$ 来计算未来奖励的现值。
  - discount rate（折扣因子） $\gamma$，当 $\gamma$ 趋近于0时，后步骤reward很快衰减，return主要取决于起始步的reward。反之$\gamma$ 趋近于1时，return更注重于最近得到的reward。
- episode（回合）：伴随一个teminal state的trajectory称为一个episode，也叫一个trial。**episode通常是有限步的**。
  - 有些任务没有终止状态，称为continuing tasks。
  - 事实上，我们通常会把episodic tasks转化成continuing tasks
    - Option1：将**absorbing state的reward设置为0**，继续进行下去。
    - Option2：将**target state视作一个普通策略**，并且策略较好，可继续收集reward。
# MDP（Markov Decision Process）
强化学习通常使用马尔可夫决策过程（Markov Decision Process, MDP）来建模。MDP是一个数学框架，用于描述具有随机性和决策性的环境。
## MDP要素（sets）
- 状态集（State Space, $\mathcal{S}$）：所有可能状态的集合。
- 动作集（Action Space, $\mathcal{A}(s)$）：所有可能动作的集合。
- 奖励集（Reward Space, $\mathcal{R}(s,a)$）：所有可能奖励的集合。
- 状态转移概率集（State transition Probability, $\mathcal{P}(s^,|s,a)$），即在状态 $s$ 下采取动作 $a$ 后转移到状态 $s'$ 的概率。
- 奖励概率集（Reward Probability, $\mathcal{P}(r|s,a)$），即在状态 $s$ 下采取动作 $a$ 后获得奖励 $r$ 的概率。
- 策略（Policy, $\pi(a|s)$）：在状态 $s$ 下采取动作 $a$ 的概率分布。
- **当Policy确定时，Markov Decision Process变成了一个Markov Process**（Decision对应Policy）
## Markov Property（马尔可夫性质）
***马尔可夫性质是指系统的未来状态仅依赖于当前状态和当前动作，而与过去的状态和动作无关。*** 下公式展示了马尔可夫状态改变性质，将其中的 $S_{t+1}$ 换为 $r_{t+1}$，则为马尔可夫奖励性质，同样成立
$$P(S_{t+1} | S_t, A_t) = P(S_{t+1} | S_0, A_0, S_1, A_1, \ldots, S_t, A_t)$$
