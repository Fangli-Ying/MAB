---
marp: true
theme: default
paginate: true
header: "Session 2: Stochastic Multi-Armed Bandits"
footer: "Fangli Ying (ECUST) & Osman Yagan (CMU) | Multi-Armed Bandits"
---

# 2.Stochastic Multi-Armed Bandits


---
## Session 2: Stochastic Multi-Armed Bandits
#### Exploration and Exploitation

Dr. Fangli Ying  
ECUST Shanghai China
Email: yfangli@ecust.edu.cn

---



#### Table of Contents
1. Basic Concepts of Multi-armed Bandits
2. Probability Basics
3. Different Bandit Problem Formulations
4. Stochastic Stationary Bandits
5. Regret: Definition and Decomposition
6. Explore-Then-Commit (ETC) Algorithm
7. Exploration-Exploitation Tradeoff
8. UCB Motivations
---


#### 1. Basic Concepts of Multi-armed Bandits
- **Problem Definition**: A sequential game between a learner and an environment with uncertainty in decision outcomes.
- **Horizon**: The game is played over $n$ rounds ($t = 1, 2, \dots, n$).
- **Actions & Rewards**: In each round $t$, the learner chooses an action $A_t$ from $k$ possible actions, and receives a random reward $X_t$.
- **Objective**: Maximize the cumulative reward:  
  $$ \sum_{t=1}^n X_t = X_1 + X_2 + \dots + X_n $$
---


#### 1. Basic Concepts (Cont.)
- **Regret**: The reward lost by taking suboptimal decisions. Defined as:  
  $$ \text{Regret} = \sum_{t=1}^n \mu^* - \sum_{t=1}^n \mu(A_t) $$  
  where $\mu^*$ is the largest mean reward among all arms, and $\mu(a)$ is the mean reward of arm $a$.
- **Exploration vs. Exploitation**:
  - Exploration: Gain information by selecting all actions to learn their rewards.
  - Exploitation: Choose the action with the highest observed reward to maximize immediate reward.
---


#### 1. Basic Concepts (Cont.)
- **Key Note**: Multi-armed bandits do not change the environment or reward distributions.
- **Relationships**:
  - A special case of Reinforcement Learning (RL).
  - Falls under Online Machine Learning (data is obtained on the go).
  - RL differs: Actions may change the environment and reward distributions.
---

###### Bandits vs. Reinforcement Learning
| **Multi-armed Bandits** | **Reinforcement Learning** |
|-------------------------|----------------------------|
| Static reward distributions | Actions change environment |
| No state transitions | Stateful decision processes |
| *Special case of RL* | *General framework* |

**Bandits fall under Online ML:** Decisions made sequentially with streaming feedback

---

#### 2. Probability Basics
- **Probability Space**: Defined by 3 components:
  - Sample space $\Omega$: Set of all possible outcomes.
  - $\sigma$-algebra $\mathcal{F}$: Collection of subsets of $\Omega$ (events).
  - Probability measure $P: \mathcal{F} \to [0,1]$ assigning probabilities to events.
- **Example (Coin Toss)**:  
  $\Omega = \{H, T\}$, $\mathcal{F} = \{\emptyset, \{H\}, \{T\}, \{H,T\}\}$  
  $P(H) = P(T) = \frac{1}{2}$, $P(\emptyset) = 0$, $P(\{H,T\}) = 1$
---

## Formal Definition of Random Experiments

**Sample Space ($\Omega$):**  
Set of all possible outcomes  
*Example: Coin toss → $\Omega = \{H, T\}$*

**Event Space ($\mathcal{F}$):**  
Subsets of $\Omega$ representing measurable events

**Probability Measure ($P$):**  
$P: \mathcal{F} \rightarrow [0,1]$ satisfying:
1. $P(\Omega) = 1$
2. Countable additivity

---



#### 2. Probability Basics (Cont.)
- **Independence of Events**: Two events $E_1, E_2$ are independent if:  
  $$ P(E_1 \cap E_2) = P(E_1) \cdot P(E_2) $$
- **Conditional Probability**: Probability of $E_1$ given $E_2$:  
  $$ P(E_1 | E_2) = \frac{P(E_1 \cap E_2)}{P(E_2)} $$  
  Note: If $E_1, E_2$ are independent, $P(E_1 | E_2) = P(E_1)$.
---

## Random Variables & Distributions
###### Formal Definition
**Random Variable:** Function $X: \Omega \rightarrow \mathbb{R}$  
*Assigns numerical values to outcomes*

**Example (Coin Toss):**  
$X = \begin{cases} 0 & \text{if Tails} \\ 1 & \text{if Heads} \end{cases}, \quad 
Y = \begin{cases} 5 & \text{if Tails} \\ 10 & \text{if Heads} \end{cases}$

---

#### 2. Probability Basics (Cont.)
- **Random Variable**: A mapping $X: \Omega \to \mathbb{R}$ assigning a real number to each outcome.
  - **Discrete RV**: Takes countable values. Probability mass function (PMF) $P(X = x_k) = p_k$, with $\sum p_k = 1$.
  - **Continuous RV**: Takes values in an uncountable set (e.g., $[0,1]$). Probability density function (PDF) $f_X(x)$, where $P(X \in B) = \int_B f_X(x) dx$.
---

#### 2. Probability Basics (Cont.)
- **Expected Value**: For a random variable $X$:
  - Discrete: $\mathbb{E}[X] = \sum_k x_k \cdot p_k$
  - Continuous: $\mathbb{E}[X] = \int_{-\infty}^{\infty} x \cdot f_X(x) dx$
- **Linearity of Expectation**: For random variables $X_1, X_2, \dots$:  
  $$ \mathbb{E}\left[\sum_{i=1}^{\infty} X_i\right] = \sum_{i=1}^{\infty} \mathbb{E}[X_i] $$
---


#### 3. Different Bandit Problem Formulations
- **Stationary Bandits**: Reward distributions of actions are fixed over time.
- **Non-stationary (Restless) Bandits**: Reward distributions may change over time (but not based on actions).
- **Structured Bandits**: Known structure in reward distributions:
  - **Linear Bandits**: Reward of action $a$ is a linear function: $x_t = f_t(\theta, a)$ with unknown parameter $\theta$.
  - **Correlated Bandits**: Rewards of different actions are correlated.
---

#### 3. Different Bandit Problem Formulations (Cont.)
- **Contextual Bandits**: 
  - Before each round, the learner observes context (e.g., user demographics).
  - Goal: Learn the best action for each context.
  - Application: Personalized recommendations.
---

#### 4. Stochastic Stationary Bandits
- **Model**: 
  - Set of actions $A$ with $|A| = k$.
  - Each action $a$ has a reward distribution $P_a$.
  - In round $t$, learner chooses $A_t \in A$, receives $X_t \sim P_{A_t}$.
- **Goal**: Maximize the expected cumulative reward $\mathbb{E}[S_n]$, where $S_n = \sum_{t=1}^n X_t$.
---

#### 4. Stochastic Stationary Bandits (Cont.)
- **Environment Class**: Set of all possible reward distributions $\{P_a: a \in A\}$ (e.g., Bernoulli with unknown $\mu_a$).
- **Mean Reward**: For arm $a$, $\mu_a = \mathbb{E}[X_t | A_t = a]$.
  - Discrete case: $\mu_a = \sum_j j \cdot P(\text{reward from } a = j)$.
- **Best Arm**: Arm with the largest mean reward: $\arg\max_{a \in A} \mu_a$, with $\mu^* = \max_{a \in A} \mu_a$.
---

#### 5. Regret: Definition and Decomposition
- **Regret for Policy $\pi$**:  
  $$ R_n(\pi) = \mathbb{E}\left[\sum_{t=1}^n \mu^* - \sum_{t=1}^n \mu(A_t)\right] $$  
  (Expected difference between optimal cumulative reward and policy's reward.)
- **Suboptimality Gap**: $\Delta_a = \mu^* - \mu_a$ (gap between best arm and arm $a$; $\Delta_a = 0$ for best arm).
---

#### 5. Regret Decomposition
- Let $T_a(n)$ = number of times arm $a$ is chosen in first $n$ rounds.
- Expected regret decomposition:  
  $$ R_n = \sum_{a \in A} \Delta_a \cdot \mathbb{E}[T_a(n)] $$  
  (Weighted sum of expected counts of each arm, weighted by their suboptimality gaps.)
---

#### 5. Regret Performance
- **Sublinear Regret**: $R_n = o(n)$ (algorithm chooses best action almost always as $n \to \infty$).
- **Logarithmic Regret**: $R_n = O(\log n)$ (optimal in most cases).  
  Achieved when $\mathbb{P}(\text{choosing suboptimal arm in round } t) \propto \frac{1}{t}$.
---

#### 6. Explore-Then-Commit (ETC) Algorithm
- **Idea**: Separate exploration and exploitation phases.
- **Steps**:
  1. **Exploration Phase**: Play each arm a fixed number of times $m$.
  2. **Exploitation Phase**: Commit to the arm with the largest average reward from exploration.
- **Example**: With $k$ arms, each arm is explored $m$ times. From round $mk + 1$, the best observed arm is played.
---

#### 6. ETC Regret Analysis
- **Regret During Exploration**:  
  $$ \sum_{a \in A} m \cdot \Delta_a $$  
  (Each arm is chosen $m$ times; regret per selection is $\Delta_a$.)
- **Regret During Exploitation**:  
  If the wrong arm is chosen, regret is $(n - mk) \cdot \Delta_{\hat{a}}$, where $\hat{a}$ is the suboptimal arm selected.
- **Total Expected Regret**: Sum of exploration and exploitation regrets.
---

#### 7. Exploration-Exploitation Tradeoff
- **Exploration**: Necessary to learn reward distributions but incurs regret from suboptimal actions.
- **Exploitation**: Maximizes immediate reward but may miss better arms.
- **ETC Tradeoff**: Choose $m$ to balance exploration (more $m$ → better estimates) and exploitation (less $m$ → fewer suboptimal rounds in exploitation).
---

#### Regret Bound of ETC Process
###### High-Probability Bounds
For 1-subgaussian rewards, the empirical mean $\hat{\mu}_k(m)$ satisfies:
$$
P\left( |\hat{\mu}_k(m) - \mu_k| \geq \epsilon \right) \leq 2 \exp\left( -\frac{m \epsilon^2}{2} \right)
$$

With $m = \Omega\left( \frac{\log n}{\Delta_k^2} \right)$, the probability of misidentifying the optimal arm is negligible.

---

#### Regret Bound of ETC Process
###### Theorem 6.1
For ETC with $1 \leq m \leq n/K$ in 1-subgaussian bandits:
$$
R_n \leq C \cdot \sum_{k=1}^K \frac{\log n}{\Delta_k} + O(K m)
$$
where $C$ is a constant. Optimal $m$ balances exploration cost and misidentification risk.

---

#### Variants of ETC Policies: $m$ and $n$
###### Exploration Parameter $m$
- $m$ controls exploration effort: Larger $m$ reduces misidentification risk but increases exploration regret.
- Optimal $m$ depends on gaps $\Delta_k$: $m = \max\left\{ 1, \frac{\log n}{\Delta_k^2} \right\}$.
---

#### Expected regret of ETC for different $m$ values
![Expected Regret ](https://dl.acm.org/cms/attachment/html/10.1145/3672758.3672802/assets/html/images/image55.png)  


---

#### Variants of ETC Policies: $m$ and $n$
###### Anytime Policy and $n$
For unknown horizon $n$, use doubling trick: Phase $i$ has $n_i = 2^i$ trials, with $m_i$ adapted to $n_i$. This ensures regret bounds hold for all $n$.

---

## ETC for Unknown Horizon: The Doubling Trick
#### Adapting Explore-Then-Commit When $n$ is Unknown

---

###### Problem: Standard ETC Needs Known $n$
- ETC relies on knowing total rounds $n$ to set exploration trials $m$ (e.g., $m \propto \log n$)
- **If $n$ is unknown**:
  - Too small $m$ → misidentify optimal arm (high exploitation regret)
  - Too large $m$ → waste rounds on exploration (high exploration regret)

---

###### Solution: The Doubling Trick
Convert ETC into an **anytime policy** using phases with doubling lengths:

1. **Phases with Doubling Lengths**  
   - Phase $i = 1, 2, 3, ...$  
   - Trials per phase: $n_i = 2^i$  
     (Phase 1: 2 trials, Phase 2: 4 trials, Phase 3: 8 trials, ...)

---

2. **Adapt Exploration per Phase**  
   - For phase $i$, treat $n_i$ as current guess for $n$  
   - Set $m_i$ (exploration trials per arm) based on $n_i$:  
     $m_i \propto \frac{\log n_i}{\Delta^2}$  
     ($\Delta$ = smallest gap between best and suboptimal arms)

---

3. **Run Sequential Phases**  
   In each phase $i$:  
   - **Explore**: Try each arm $m_i$ times  
   - **Exploit**: Play best observed arm for remaining $n_i - K \cdot m_i$ trials  
   - Continue until experiment stops (unknown $n$)

---

###### Example: 2-Arm Bandit ($K=2$)
| Phase $i$ | $n_i$ (trials) | $m_i$ (exploration/arm) | Explore Trials | Exploit Trials |
|-----------|----------------|-------------------------|----------------|----------------|
| 1         | 2              | 1                       | 2 (1/arm)      | 0              |
| 2         | 4              | 1                       | 2 (1/arm)      | 2              |
| 3         | 8              | 2                       | 4 (2/arm)      | 4              |

*If experiment stops after 5 trials: uses Phase 1 (2 trials) + Phase 2 (3 trials)*

---

###### Why It Works
- **Guaranteed Calibration**: For any unknown $n$, $n$ falls in phase $i$ where $n_i \geq n$  
- **Balanced Regret**: $m_i$ scales with phase length to avoid over/under-exploration  
- **Anytime Property**: Works for *any stopping time* without prior knowledge of $n$

---

###### Key Takeaway
The doubling trick transforms ETC into a robust algorithm for unknown horizons by:  
- Using adaptive exploration per phase  
- Ensuring regret remains bounded across all possible $n$

#### Motivation of UCB Policy
###### Limitation of ETC
ETC’s fixed exploration phase leads to suboptimal regret for non-stationary or large $n$.

###### UCB Idea: Optimism Under Uncertainty
Select the arm with the highest upper confidence bound:
$$
UCB_t(k) = \hat{\mu}_k(t) + \sqrt{\frac{2 \log t}{T_k(t)}}
$$
- Balances exploration (uncertain arms with large bounds) and exploitation (high empirical means).

---


#### Motivation of UCB Policy
###### Algorithm 3: UCB($\delta$)
1. For $t = 1, ..., n$:
   - Choose $A_t = argmax_k UCB_k(t-1, \delta)$
   - Observe $X_t$ and update confidence bounds
2. End for

UCB adapts to data dynamically, achieving better regret bounds than ETC in many scenarios.

---

## UCB vs ETC
| Aspect               | ETC                          | UCB                          |
|----------------------|------------------------------|------------------------------|
| Exploration          | Fixed phase                  | Adaptive (ongoing)           |
| Horizon Requirement  | Known $n$                    | Works with unknown $n$       |
| Regret Bound         | $O(\sqrt{Kn \log n})$        | $O(\sqrt{Kn \log n})$        |
| Flexibility          | Low (fixed $m$)              | High (adapts to data)        |

---







#### Key Takeaways
- Multi-armed bandits balance exploration (learning) and exploitation (maximizing reward).
- Regret measures performance against the optimal arm.
- Different problem formulations (stationary, non-stationary, structured) require tailored algorithms.
- ETC is a simple algorithm with clear exploration-exploitation separation, achieving sublinear regret.
---
