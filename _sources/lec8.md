---
marp: true
theme: default
paginate: true
header: "Multi-Armed Bandits: Full Feedback & Adversarial Bandits"
footer: "Fangli Ying (ECUST) & Osman Yagan (CMU) | Multi-Armed Bandits"
---

# 8. Full Feedback and Adversarial Costs & Adversarial Bandits

---

#### Overview
- **Full feedback setting with adversarial costs**
  - Problem formulation
  - Adversary models
  - Key algorithms: Weighted Majority, Hedge
  - Regret analysis
- **Adversarial bandits (limited feedback)**  
  - Problem formulation
  - Key algorithm: Exp3
  - Regret analysis and extensions

---

## Recap: Key Algorithms for Stochastic Bandits

---

#### UCB (Upper Confidence Bound)
###### Core Idea: "Optimism in the Face of Uncertainty"
- **How it works**: 
  - For each arm, calculate an upper confidence bound (UCB) of its mean reward.
  - The UCB combines the arm's observed average reward and a "confidence term" (increases with uncertainty, i.e., fewer samples).
  - Choose the arm with the highest UCB in each round.

- **Formula**:  
  $$UCB_t(a) = \bar{\mu}_t(a) + \sqrt{\frac{2 \log t}{n_t(a)}}$$  
  ( $\bar{\mu}_t(a)$: average reward of arm $a$; $n_t(a)$: number of times $a$ is played by round $t$)

---

#### UCB (Continued)
###### Strengths
- Works well in **stochastic environments** (rewards from fixed distributions).
- Balances exploration (uncertain arms get more trials) and exploitation (good arms get more plays) automatically.
- Provable regret bounds: $O(\sqrt{KT \log T})$ for $K$ arms and $T$ rounds.

###### Limitations
- Relies on **stable reward distributions** (fails if rewards are adversarial/arbitrary).
- Confidence terms can be overly conservative in non-stationary settings.

---

#### TS (Thompson Sampling)
###### Core Idea: "Bayesian Sampling"
- **How it works**:
  - Start with a prior distribution for each arm's mean reward.
  - After each round, update the posterior distribution using observed rewards (Bayesian update).
  - Sample a mean reward from each arm's posterior and choose the arm with the highest sampled value.

---

- **Key Intuition**: 
  - Arms with higher posterior probability of being optimal are more likely to be chosen.
  - Naturally balances exploration (uncertain arms have wider posteriors, so more varied samples) and exploitation.

---

#### TS (Continued)
###### Strengths
- Excellent performance in **stochastic environments**, often better than UCB in practice.
- Adapts well to prior knowledge (if available) via the initial prior.
- Provable regret bounds: $O(\log T)$ for many stochastic settings.

###### Limitations
- Still assumes **rewards follow some underlying distribution**.
- Fails in adversarial settings where rewards are manipulated to mislead the posterior.

---

## Why Chapter 5 & 6? (Motivation)
#### The Problem with UCB/TS

---

#### Limitation of Stochastic Algorithms
UCB and TS are designed for **IID/stochastic rewards** (e.g., coin flips with fixed probabilities).  

But many real-world scenarios are **adversarial**:
- Rewards can be chosen arbitrarily (e.g., a competitor intentionally lowering your rewards).
- Rewards can depend on your past actions (e.g., dynamic pricing where competitors react to your choices).
- No underlying "true" distribution to learn.

In such cases, UCB/TS perform poorly—their assumptions about reward stability are violated.

---

#### Need for New Frameworks
Chapters 5 and 6 address **adversarial rewards** with different feedback settings:

| Setting               | Feedback Available       | Key Challenge                                  |
|-----------------------|--------------------------|------------------------------------------------|
| Chapter 5: Full Feedback | Observe all arms' rewards | Adapt to arbitrary rewards with full information. |
| Chapter 6: Adversarial Bandits | Observe only chosen arm's reward | Balance exploration/exploitation with limited, potentially misleading feedback. |

---

#### Why These Chapters Matter
- They extend bandit theory to **worst-case scenarios** (no assumptions on reward distributions).
- Provide robust algorithms (e.g., Hedge for full feedback, Exp3 for bandit feedback) that work even when rewards are adversarial.
- Lay the foundation for applications like:
  - Adversarial recommendation systems (competitors manipulate clicks).
  - Dynamic pricing under competitor interference.
  - Online learning with malicious noise.
  
---

## 5. Full Feedback and Adversarial Costs
#### 5.1 Problem Definition: Full Feedback

---

###### What is Full Feedback?
- After each round, the algorithm observes **costs of all arms**, not just the chosen one.
- Focus: Adversarial costs (costs can be arbitrary, chosen by an adversary).

---

###### Problem Protocol: Full Feedback
Parameters: $K$ arms, $T$ rounds.  
Each round $t \in [T]$:
1. Adversary chooses costs $c_t(a) \geq 0$ for all arms $a \in [K]$.
2. Algorithm picks arm $a_t \in [K]$.
3. Algorithm incurs cost $c_t(a_t)$.
4. **All costs $c_t(a)$ are revealed to the algorithm**.

---

###### Example: Sequential Prediction with Experts
- Setting: Predict labels with advice from $K$ experts.
- Each round $t$:
  1. Adversary chooses observation $x_t$ and true label $z_t^*$.
  2. Experts predict labels $z_{1,t}, ..., z_{K,t}$.
  3. Algorithm selects expert $e_t$.
  4. True label $z_t^*$ is revealed; cost $c_t = \mathbb{1}_{\{z_{e_t,t} \neq z_t^*\}}$.

---

#### 5.2 Adversaries and Regret

---

###### Types of Adversaries
1. **Oblivious**: Costs $c_t(a)$ are fixed before round 1 (no dependence on algorithm's choices).
2. **Adaptive**: Costs $c_t(a)$ depend on the algorithm's past choices $a_1, ..., a_{t-1}$.

---

###### Regret Definitions
- Total cost of algorithm: $\text{cost}(ALG) = \sum_{t=1}^T c_t(a_t)$
- Total cost of arm $a$: $\text{cost}(a) = \sum_{t=1}^T c_t(a)$

1. **Regret**: $R(T) = \text{cost}(ALG) - \min_{a \in [K]} \text{cost}(a)$  
   (Worst vs. best-in-hindsight arm)

2. **Pseudo-regret**: $R(T) = \text{cost}(ALG) - \min_{a \in [K]} \mathbb{E}[\text{cost}(a)]$  
   (Worst vs. best-in-foresight arm, for randomized adversaries)

---

#### 5.3 Algorithms for Full Feedback

---

###### Algorithm 1: Weighted Majority
**Idea**: Assign weights to arms; choose arm with weight proportional to its past performance.

**Steps**:
1. Initialize weights $w_a(1) = 1$ for all $a \in [K]$.
2. For each round $t$:
   - Choose arm $a_t$ with probability $\frac{w_a(t)}{\sum_{a'} w_{a'}(t)}$.
   - Observe all costs $c_t(a) \in \{0,1\}$ (binary costs).
   - Update weights: $w_a(t+1) = w_a(t) \cdot (1 - \epsilon)^{c_t(a)}$ ( $\epsilon \in (0,1)$ ).

---

###### Analysis of Weighted Majority
- For binary costs ($c_t(a) \in \{0,1\}$) and oblivious adversary:  
  $\mathbb{E}[R(T)] \leq 2\sqrt{T K \log K}$ (with appropriate $\epsilon$).

- Key insight: Weights decrease for arms with high cumulative cost, so the algorithm focuses on low-cost arms.

---

###### Algorithm 2: Hedge (Multiplicative Weights Update)
**Generalization**: Works for arbitrary bounded costs ($c_t(a) \in [0,1]$).

**Steps**:
1. Initialize weights $w_a(1) = 1$ for all $a \in [K]$.
2. For each round $t$:
   - Choose arm $a_t$ with probability $p_t(a) = \frac{w_a(t)}{\sum_{a'} w_{a'}(t)}$.
   - Observe all costs $c_t(a) \in [0,1]$.
   - Update weights: $w_a(t+1) = w_a(t) \cdot \exp(-\eta c_t(a))$ ( $\eta > 0$ is a learning rate).

---

###### Analysis of Hedge
- For $c_t(a) \in [0,1]$ and oblivious adversary:  
  With $\eta = \sqrt{\frac{\log K}{T}}$,  
  $\mathbb{E}[R(T)] \leq \sqrt{T K \log K}$.

- Tighter bound: $\mathbb{E}[R(T)] \leq 2\sqrt{T \log K} + \frac{\log K}{2\sqrt{T \log K}}$ (approximates $O(\sqrt{T \log K})$).

---

#### 5.4 Key Results for Full Feedback
- **Oblivious adversary**: Hedge achieves $O(\sqrt{T \log K})$ expected regret.
- **Adaptive adversary**: Same bounds hold (algorithms are robust to adaptivity).
- **IID costs**: Even easier – no exploration needed; simple averaging achieves $O(\sqrt{T \log K})$ regret.

---

## 6. Adversarial Bandits
#### 6.1 Problem Definition: Adversarial Bandits

---

###### What is Adversarial Bandits?
- **Limited feedback**: After each round, the algorithm observes only the cost of the **chosen arm** (not others).
- Costs are chosen by an adversary (oblivious or adaptive).

---

###### Problem Protocol: Adversarial Bandits
Parameters: $K$ arms, $T$ rounds.  
Each round $t \in [T]$:
1. Adversary chooses costs $c_t(a) \geq 0$ for all arms $a \in [K]$.
2. Algorithm picks arm $a_t \in [K]$.
3. Algorithm incurs cost $c_t(a_t)$.
4. **Only $c_t(a_t)$ is revealed to the algorithm**.

---

###### Challenge: Exploration-Exploitation Tradeoff
- Without full feedback, the algorithm cannot directly learn costs of unchosen arms.
- Must balance:
  - **Exploration**: Try new arms to learn their costs.
  - **Exploitation**: Choose arms believed to have low costs.

---

#### 6.2 Algorithm: Exp3 (Exponential Weights for Exploration and Exploitation)

---

###### Idea of Exp3
- Combine Hedge's multiplicative weights with explicit exploration.
- Choose each arm with probability that includes a small "exploration" term.

---

###### Exp3 Steps
1. Initialize weights $w_a(1) = 1$ for all $a \in [K]$.
2. For each round $t$:
   - Compute probabilities: $p_t(a) = \frac{(1 - \gamma) w_a(t)}{\sum_{a'} w_{a'}(t)} + \frac{\gamma}{K}$, where $\gamma \in (0,1)$ (exploration rate).
   - Choose arm $a_t$ according to $p_t$.
   - Observe $c_t(a_t) \in [0,1]$.
   - Estimate cost for all arms: $\hat{c}_t(a) = \begin{cases} \frac{c_t(a_t)}{p_t(a_t)} & \text{if } a = a_t, \\ 0 & \text{otherwise}. \end{cases}$
   - Update weights: $w_a(t+1) = w_a(t) \cdot \exp(-\eta \hat{c}_t(a))$ ( $\eta > 0$ is learning rate).

---

###### Why the Estimator $\hat{c}_t(a)$?
- Unbiased estimate: $\mathbb{E}[\hat{c}_t(a)] = c_t(a)$ for all $a$.
- Compensates for low-probability choices (large $1/p_t(a_t)$ when $p_t(a_t)$ is small).

---

###### Analysis of Exp3
- For $c_t(a) \in [0,1]$ and oblivious adversary:  
  With $\gamma = \sqrt{\frac{\log K}{T K}}$ and $\eta = \gamma$,  
  $\mathbb{E}[R(T)] \leq O(\sqrt{T K \log K})$.

- Key: The exploration term $\gamma/K$ ensures all arms are tried, preventing large regret from untested arms.

---

#### 6.3 Extensions of Exp3
- **Exp3-IX**: Improves exploration by using importance weighting, achieving $O(\sqrt{T K \log K})$ with better constants.
- **Adaptive adversaries**: Exp3 bounds extend to adaptive adversaries.
- **Unbounded costs**: With modifications (e.g., clipping), Exp3 works for costs bounded above by $C$, with regret scaled by $C$.

---

#### 6.4 Lower Bounds for Adversarial Bandits
- For any algorithm, there exists an oblivious adversary such that:  
  $\mathbb{E}[R(T)] \geq \Omega(\sqrt{T K})$.

- Matches the upper bound of Exp3, showing optimality.

---

#### Summary
| Setting               | Feedback           | Algorithm       | Regret Bound               |
|-----------------------|--------------------|-----------------|----------------------------|
| Full Feedback         | All costs          | Hedge           | $O(\sqrt{T \log K})$    |
| Adversarial Bandits   | Only chosen cost   | Exp3            | $O(\sqrt{T K \log K})$ |

---

#### Exercises (Key Takeaways)
- Full feedback simplifies learning (no exploration needed for IID costs).
- Adversarial bandits require balancing exploration and exploitation.
- Exp3 is optimal for adversarial bandits, matching lower bounds.