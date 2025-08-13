---
marp: true
theme: default
paginate: true
header: "Session 1: Introduction to Multi-Armed Bandits"
footer: "Fangli Ying (ECUST) & Osman Yagan (CMU) | Multi-Armed Bandits"
---

# 1. Introduction to Multi-Armed Bandits

---



## Session 1: Introduction to Multi-Armed Bandits
#### Overview
- Decision-Making with Uncertainty
- A/B Testing: Details and Limitations
- Formalizing A/B Testing in Multi-Armed Bandits Framework
- Key Definitions & Real-World Instances
- Algorithms & Performance Analysis
- Case Study: Social Media Advertising

---


## Decision-Making with Uncertainty
#### Core Challenge
- Making choices when outcomes are uncertain
- Example: Website design optimization
  - Version A: 50 signups
  - Version B: 75 signups
  - **Question**: Is Version B truly better? How to quantify uncertainty?
---


## Key Questions
1. How to compare options (e.g., Version A vs. B)?
2. How to formalize uncertainty in decisions?
3. How to balance learning (exploration) and earning (exploitation)?

---

## A/B Testing: Detailed Look
#### What is A/B Testing?
- A method to compare two versions (A/B) of a product/design
- Process:
  - Randomly assign users to Version A or B
  - Measure key metrics (e.g., signups, clicks)
  - Determine which version performs better
---

#### Limitations of A/B Testing
1. Fixed test phase: Wastes resources on inferior options
2. Sudden switch from exploration to exploitation
3. Ignores dynamic adaptation to new data
4. Fails to leverage context or user-specific information

---

## From A/B Testing to Multi-Armed Bandits
#### The Analogy
- A/B Testing ≈ 2-armed bandit problem
  - "Arms": Version A and Version B
  - "Reward": Signups, clicks, or other metrics
  - "Uncertainty": Unknown success probabilities of each arm
---

## Multi-Armed Bandits (MAB) Framework
- Generalizes to K arms (K options)
- Sequential decision-making: Choose an arm, observe reward, update strategy
- Goal: Maximize total reward over time by balancing exploration (learning) and exploitation (using known good arms)

---

## Formalizing the Bandit Problem
#### Key Components


1.  **Environment**: Defines reward distributions for each arm


*   e.g., $P_a$: Probability distribution of rewards for arm $a$

*   Mean reward: $\mu_a = \mathbb{E}[X_t | A_t = a]$

2.  **Policy**: Strategy to select arms over time


*   $\pi_t(\cdot | H_{t-1})$: Probability of choosing arm $a$ at time $t$ given history $H_{t-1}$

3.  **Reward**: Outcome of selecting an arm


*   $X_t \in [0,1]$ (e.g., 1 for success, 0 for failure)

---

## Formalizing the Bandit Problem

Where:




*   $P_a$ denotes the probability distribution of rewards for arm $a$

*   $\mu_a$ represents the mean reward of arm $a$, defined by the expectation $\mathbb{E}[X_t | A_t = a]$ (i.e., the expected value of reward $X_t$ when arm $a$ is chosen at time $t$)


*   $\pi_t(\cdot | H_{t-1})$ indicates the probability distribution strategy for choosing an arm at time $t$ given the history $H_{t-1}$

*   $X_t$ stands for the reward obtained at time $t$, with a value range of $[0,1]$ (e.g., 1 represents success, 0 represents failure)

---

## Key Definitions: Regret
#### Regret: Measure of Suboptimality
*   Total regret after $n$ rounds:

$R_n = n \max_{a \in \mathcal{A}} \mu_a - \mathbb{E}\left[\sum_{t=1}^n X_t\right]$



*   Difference between the best possible reward and the expected reward of the policy
---

## Regret Decomposition

*   Lemma: Regret can be decomposed using the number of times each arm is played:

$R_n = \sum_{a \in \mathcal{A}} \Delta_a \mathbb{E}[T_a(n)]$



*   $\Delta_a = \mu^* - \mu_a$ (gap between best arm $\mu^*$ and arm $a$)


*   $T_a(n)$: Number of times arm $a$ is played in $n$ rounds

---

## Regret: Definition and Core Idea
#### What is Regret?
- A fundamental metric to quantify the suboptimality of a decision-making policy over time.
- Measures the **opportunity cost** of not knowing the best action (arm) in advance.
---

## Formal Definition (Total Regret after n rounds)
$R_n = n \cdot \max_{a \in \mathcal{A}} \mu_a - \mathbb{E}\left[\sum_{t=1}^n X_t\right]$
- Where:
  - $\max_{a \in \mathcal{A}} \mu_a = \mu^*$: The highest mean reward among all arms (best possible per-round reward).
  - $n \cdot \mu^*$: Total reward if we always chose the best arm.
  - $\mathbb{E}\left[\sum_{t=1}^n X_t\right]$: Expected total reward of the policy over $n$ rounds.

---

## Regret: Definition and Core Idea
#### Intuition
- Regret captures the **difference** between:
  1. The best possible reward (if we knew the optimal arm from the start).
  2. The reward actually obtained by the algorithm’s strategy.
- Example: If $\mu^* = 0.8$ (best arm), and over 100 rounds the algorithm’s expected total reward is 60, then:

$R_{100} = 100 \cdot 0.8 - 60 = 20$

  The algorithm "regrets" missing out on 20 units of reward.

---

## Key Observation
- Regret is non-negative: $R_n \geq 0$, since $\mu^*$ is the maximum mean reward.

---

## Regret Decomposition
#### Breaking Down Total Regret
- Regret can be decomposed based on how often each arm is used:
$R_n = \sum_{a \in \mathcal{A}} \Delta_a \cdot \mathbb{E}[T_a(n)]$
- Where:
  - $\Delta_a = \mu^* - \mu_a$: The "gap" between the best arm and arm $a$ (measures how suboptimal $a$ is).
  - $T_a(n)$: The number of times arm $a$ is chosen in $n$ rounds (random variable).
  - $\mathbb{E}[T_a(n)]$: Expected number of times arm $a$ is used.
---

## Interpretation
- Each arm $a$ contributes to regret proportionally to:
  1. Its suboptimality gap $\Delta_a$.
  2. How often it is selected ($\mathbb{E}[T_a(n)]$).
- Poorly performing arms (large $\Delta_a$) should be used rarely to minimize regret.

---

## Regret Decomposition
#### Example: Two-Armed Bandit
- Suppose:
  - Arm 1: $\mu_1 = 0.9$ (optimal, $\mu^* = 0.9$)
  - Arm 2: $\mu_2 = 0.6$ (gap $\Delta_2 = 0.3$)
  - Over $n=100$ rounds, arm 2 is used 20 times on average.
- Regret decomposition:
$R_{100} = \Delta_2 \cdot \mathbb{E}[T_2(100)] = 0.3 \cdot 20 = 6$

- Intuition: Using the suboptimal arm 20 times costs $0.3 \times 20 = 6$ in total regret.
---

##  Regret Decomposition
- Helps analyze algorithm performance: Focus on limiting usage of arms with large $\Delta_a$.

---

## Properties and Bounds of Regret
#### Regret Over Time
- Regret typically grows with the number of rounds $n$, but the **rate** depends on the algorithm:
  - Poor algorithms: $R_n \approx O(n)$ (e.g., always choosing a suboptimal arm).
  - Good algorithms: $R_n \approx O(\sqrt{n})$ or $O(\log n)$ (balances exploration/exploitation).
---

## Instance-Dependent vs. Instance-Independent Bounds
- **Instance-dependent**: Bounds depend on gaps $\Delta_a$ (e.g., $R_n \leq O\left(\sum_{a} \frac{\log n}{\Delta_a}\right)$ for UCB).
- **Instance-independent**: Bounds hold for all problem instances (e.g., $R_n \leq O(\sqrt{Kn \log n})$ for UCB, where $K$ is the number of arms).

---

## Properties and Bounds of Regret
#### Lower Bounds
- No algorithm can avoid regret growing at least as fast as certain rates:
  - For $K$ arms: $\Omega(\sqrt{Kn})$ (worst-case).
  - For a fixed instance: $\Omega\left(\sum_{a} \frac{\log n}{\Delta_a}\right)$ (Lai-Robbins bound).
---

## Key Takeaway
- Regret quantifies how well an algorithm balances exploration (learning about arms) and exploitation (using known good arms).
- Minimizing regret is the core goal in multi-armed bandit problems.

---














## Exploration-Exploitation Tradeoff
#### Core Dilemma
- **Exploration**: Try new arms to learn their rewards (reduces uncertainty)
- **Exploitation**: Choose known good arms to maximize immediate reward

#### Example: Epsilon-Greedy Algorithm
*   With probability $\epsilon$: Explore (randomly select an arm)


*   With probability $1-\epsilon$: Exploit (select arm with highest observed mean)


*   Balances exploration (via $\epsilon$) and exploitation
---

## Probability Basics for Bandits
#### Concentration of Measure
###### Tools to bound uncertainty in reward estimates:



*   **Hoeffding Inequality**: Bounds deviation of sample mean from true mean

$\mathbb{P}(|\hat{\mu} - \mu| \geq \epsilon) \leq 2 \exp\left(-\frac{2n\epsilon^2}{\sigma^2}\right)$



*   **Subgaussian Random Variables**: Rewards with bounded tail probabilities, enabling tight confidence bounds
---

## Why This Matters
- Enables rigorous analysis of algorithms (e.g., proving regret bounds)
- Guides selection of exploration strategies (e.g., how many times to try each arm)

---

## Real-World Instances of Bandits
#### 1. Clinical Trials
- **Arms**: Treatment options (e.g., Drug A, Drug B, placebo)
- **Reward**: Patient recovery/effectiveness
- **Goal**: Minimize harm (regret) by quickly identifying the best treatment
---
## Real-World Instances of Bandits
#### 2. Recommendation Systems
- **Arms**: Items to recommend (e.g., movies, products)
- **Reward**: User clicks/purchases
- **Goal**: Maximize engagement by learning user preferences

---

## Real-World Instances of Bandits
#### 3. Pricing Optimization
- **Arms**: Price points (e.g., $9.99, $14.99)
- **Reward**: Revenue from sales
- **Goal**: Maximize total revenue by finding optimal price
---

## Real-World Instances of Bandits
#### 4. Social Media Advertising
- **Arms**: Ad versions (e.g., different visuals/copies)
- **Reward**: Clicks/conversions
- **Goal**: Maximize ad performance within budget

---

## Case Study: Social Media Advertising
#### Problem with A/B Testing
- In a campaign with 2M users:
  - A/B testing exposes 1M users to each version
  - Wastes ~100K impressions on the inferior version

#### MAB Algorithm Advantage (e.g., KLUCB)
- Adapts dynamically: Reduces exposure to poor versions
- Exposes only ~5K users to the inferior version
- Superior cumulative reward over time

---

## Performance Comparison: A/B Testing vs. Bandits
![Performance Chart Description: Bandit algorithms quickly shift to better options, outperforming A/B testing which wastes resources on inferior versions](https://conversion-uplift.co.uk/wp-content/uploads/2016/11/AB-Testing-vs-Bandit-Selection-Chart-1024x666.jpg)

---

## Performance Comparison: A/B Testing vs. Bandits
- **Bandits**: Lower regret by balancing exploration and exploitation
- **A/B Testing**: Higher regret due to fixed exploration phase

---

## Variants of Bandit Problems
#### 1. Contextual Bandits
- Incorporate context (e.g., user demographics) to personalize arm selection
- Example: Show different ads based on user location/age
---

## Variants of Bandit Problems
#### 2. Adversarial Bandits
- Rewards are chosen by an adversary (non-stochastic)
- Requires robust algorithms to handle worst-case scenarios
---
## Variants of Bandit Problems
#### 3. Combinatorial Bandits
- Select subsets of arms (e.g., a slate of ads)
- Extends to complex decision spaces

---

## Summary & Key Takeaways
1. Multi-armed bandits address uncertainty in sequential decision-making
2. Regret quantifies suboptimality; minimizing regret is core goal
3. Exploration-exploitation tradeoff is fundamental
4. Bandit algorithms outperform A/B testing in dynamic environments
5. Applications span advertising, healthcare, recommendations, and more

---

## Q&A
#### Discussion Topics
- How to choose between exploration and exploitation in practice?
- What are the challenges in scaling bandit algorithms to 1000+ arms?
- How to handle non-stationary environments (e.g., changing user preferences)?

