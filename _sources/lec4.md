---
marp: true
theme: default
paginate: true
title: "Principles and Performance Comparison of ETC and UCB Algorithms in Multi-Armed Bandits"
---

# 4. Principles and Performance Comparison of ETC and UCB 

---

#### Table of Contents
1. Core Concepts and Notations
2. ETC (Explore-Then-Commit) Algorithm
3. UCB (Upper Confidence Bound) Algorithm
4. Regret Analysis
5. Improved UCB: Asymptotically Optimal UCB
6. Key Conclusions

---

#### I. Core Concepts and Notations

###### Key Symbols
- **Suboptimality Gap** ($\Delta_i$): $\Delta_i = \mu^* - \mu_i$, where $\mu^*$ is the optimal mean reward, and $\mu_i$ is the mean reward of arm $i$.
- **Horizon** ($n$): Total number of rounds the algorithm runs.
- **Cumulative Regret** ($R_n$): $R_n = \sum (\text{Losses from choosing suboptimal arms})$
- **Exploration vs. Exploitation**: Fundamental trade-off between gathering information (exploration) and maximizing immediate reward (exploitation).

---

#### II. ETC (Explore-Then-Commit) Algorithm

###### Two Phases
1. **Exploration Phase**: First $m$ rounds, each arm is sampled $m$ times.
2. **Exploitation Phase**: In remaining rounds, only the best-performing arm from exploration is chosen.

*Core challenge: Determining the optimal exploration count $m$.*

---

#### ETC: Optimal Parameter $m$ Selection

###### Single Suboptimality Gap ($\Delta$)
- When there's only one suboptimal arm (gap = $\Delta$):

  Optimal exploration count:
  $$ m = \frac{4\log n}{\Delta^2} $$

  Corresponding cumulative regret:
  $$ R_n = \frac{4}{\Delta}\log n $$

---



## ETC: Deriving Optimal m and Regret (Single Gap)

---

#### Scenario Setup
- 2-armed bandit problem:
  - 1 optimal arm with mean reward $\mu^*$
  - 1 suboptimal arm with mean reward $\mu$
  - Suboptimality gap: $\Delta = \mu^* - \mu$
  - Total rounds (horizon): $n$

---

#### ETC Structure
- **Exploration phase**: $2m$ rounds (each arm sampled $m$ times)
- **Exploitation phase**: Remaining $n - 2m$ rounds (choose best arm from exploration)

*Goal: Choose $m$ to minimize cumulative regret $R_n$*

---

#### Key Consideration
Probability of misidentifying the optimal arm during exploration must be small.

- Using concentration inequalities (e.g., Hoeffding's):
  - Need enough samples ($m$) to distinguish $\mu^*$ from $\mu$
  - Larger $\Delta$: Easier to distinguish (smaller $m$ needed)
  - Larger $n$: Need more exploration to keep misidentification probability low

---

#### Deriving Optimal m
To ensure misidentification probability is negligible (order $1/n$):

- Mathematical derivation shows $m$ must scale with:
  - $\log n$ (to control probability over horizon)
  - $1/\Delta^2$ (inverse square of gap, as larger gaps need fewer samples)

Result: Optimal exploration count
$$ m = \frac{4\log n}{\Delta^2} $$

---

#### Calculating Cumulative Regret
With optimal $m$, correct identification is highly likely:

- Regret comes primarily from exploration phase
- We sample the suboptimal arm $m$ times
- Each such sample incurs loss $\Delta$

Total regret: $R_n = m \cdot \Delta$

Substituting $m$:
$$ R_n = \frac{4\log n}{\Delta^2} \cdot \Delta = \frac{4}{\Delta}\log n $$

---

#### Interpretation
- Regret grows logarithmically with horizon $n$ (favorable scaling)
- Regret decreases as gap $\Delta$ increases (intuitive: easier to find optimal arm)
- The factor of 4 comes from concentration inequality constants
- 
---

#### ETC: Optimal Parameter $m$ Selection

###### Multiple Arms ($k$ arms)
- With suboptimality gaps $\Delta_1, \Delta_2, ..., \Delta_k$:

  Optimal exploration count considers the minimum squared gap:
  $$ m = \left\lceil \frac{4}{\min_{j \neq i} \Delta_j^2} \right\rceil $$

  Cumulative regret as sum of individual contributions:
  $$ R_n = \sum_{\substack{i=1 \\ i \neq *}}^k \left( \frac{4\log n}{\min_{j \neq i} \Delta_j^2} \right) \cdot \Delta_i $$

---


## ETC with Multiple Arms: Deriving Optimal m and Regret

---

#### Scenario Setup
- $k$ armed bandit problem
- One optimal arm with mean reward $\mu^*$
- $k-1$ suboptimal arms with mean rewards $\mu_1, \mu_2, ..., \mu_{k-1}$
- Suboptimality gaps: $\Delta_i = \mu^* - \mu_i$ for each suboptimal arm $i$
- Horizon (total rounds): $n$

---

#### Key Challenge with Multiple Arms
Need to ensure we correctly identify the optimal arm during exploration.

- For $k$ arms, exploration phase requires $k \cdot m$ rounds (each arm sampled $m$ times)
- Critical risk: Any suboptimal arm could be mistakenly identified as optimal if under-explored
- Smallest gap $\Delta_{\text{min}} = \min(\Delta_1, \Delta_2, ..., \Delta_{k-1})$ creates highest risk of misidentification

---

#### Why Minimum Squared Gap?
The smallest gap $\Delta_{\text{min}}$ determines required exploration:

- Arms with smaller $\Delta_i$ are harder to distinguish from the optimal arm
- Mathematically: Probability of misidentification depends on $m \cdot \Delta_i^2$
- To ensure all suboptimal arms are correctly identified, we must satisfy the most stringent condition (smallest $\Delta_i$)

Thus: $\Delta_{\text{min}}^2 = \min(\Delta_1^2, \Delta_2^2, ..., \Delta_{k-1}^2)$

---

#### Deriving Optimal m
Using concentration inequalities (e.g., Hoeffding's) to bound misidentification probability:

- For reliable identification of all suboptimal arms, $m$ must satisfy:
  $$ m \cdot \Delta_{\text{min}}^2 \geq 4\log n $$
- Rearranging gives the minimum required $m$:
  $$ m = \left\lceil \frac{4}{\Delta_{\text{min}}^2} \right\rceil $$
- The ceiling function ensures $m$ is an integer (cannot sample a fraction of rounds)

---

#### Calculating Cumulative Regret
Regret comes from two sources:

1. **Exploration phase**: Each suboptimal arm is sampled $m$ times, contributing $m \cdot \Delta_i$ per arm
2. **Exploitation phase**: Negligible if optimal arm is correctly identified (highly likely with proper $m$)

Total regret is sum of exploration losses across all suboptimal arms:
$$ R_n = \sum_{\substack{i=1 \\ i \neq *}}^{k-1} m \cdot \Delta_i $$

---

#### Substituting Optimal m
Substitute $m = \frac{4}{\Delta_{\text{min}}^2}$ into the regret formula:

$$ R_n = \sum_{\substack{i=1 \\ i \neq *}}^{k-1} \left( \frac{4}{\Delta_{\text{min}}^2} \right) \cdot \Delta_i $$

With logarithmic factor (to control probability over horizon $n$):

$$ R_n = \sum_{\substack{i=1 \\ i \neq *}}^{k-1} \left( \frac{4\log n}{\Delta_{\text{min}}^2} \right) \cdot \Delta_i $$

---

#### Interpretation
- The smallest gap $\Delta_{\text{min}}$ dominates both $m$ and total regret
- Arms with larger $\Delta_i$ contribute less to regret but still require exploration determined by $\Delta_{\text{min}}$
- Regret grows with $k$ (more suboptimal arms) and $\log n$ (horizon), but decreases with larger gaps

*This formulation ensures all suboptimal arms are properly identified while minimizing total regret.*

---

#### ETC: Core Principle
Performance heavily depends on $m$:
- Too small $m$: Insufficient exploration (misidentifying the optimal arm)
- Too large $m$: Wasted rounds (increased regret)

$m$ must be optimized based on $\Delta$ and $n$.

---

#### III. UCB (Upper Confidence Bound) Algorithm

###### Decision Rule
At round $t$, select arm $A_t$:
$$ A_t = \arg\max_i \left( \hat{\mu}_i(t-1) + \sqrt{\frac{4}{T_i(t-1)} \cdot \log t} \right) $$

Where:
- $\hat{\mu}_i(t-1)$: Sample mean reward of arm $i$ up to round $t-1$
- $T_i(t-1)$: Number of times arm $i$ has been selected up to round $t-1$

---

#### UCB: Core Principle - Dynamic Confidence Intervals

- Each arm's true mean $\mu_i$ is enclosed in a confidence interval
- Interval width shrinks as sample count $T_i$ increases (more samples → more accurate estimates)
- Optimal arm's UCB converges to its true mean $\mu^*$
- Suboptimal arms' UCBs eventually fall below the optimal arm's lower bound

*Balances exploration (uncertain arms get higher weights) and exploitation (high mean arms preferred) dynamically.*

---

#### UCB: Stopping Condition for Suboptimal Arms

A suboptimal arm $i$ stops being selected when:
$$ \text{UCB}_i \leq \text{LCB}_* \quad \Leftrightarrow \quad \text{UCB}_i - \text{LCB}_i \leq \Delta_i $$

When the interval width of arm $i$ is less than its suboptimality gap $\Delta_i$, the algorithm is "confident" it's suboptimal.

- Corresponding selection count: $T_i \approx 16\log n / \Delta_i^2$

---



## UCB: Stopping Condition for Suboptimal Arms
#### Principle and Derivation

---

#### Recall UCB Basics
- UCB (Upper Confidence Bound) algorithm selects arms based on:
  $$ \text{UCB}_i(t) = \hat{\mu}_i(t) + \sqrt{\frac{4 \log t}{T_i(t)}} $$
  - $\hat{\mu}_i(t)$: Sample mean of arm $i$ after $t$ rounds
  - $T_i(t)$: Number of times arm $i$ has been selected by round $t$
  - $\sqrt{\frac{4 \log t}{T_i(t)}}$: Confidence interval width (exploration bonus)

- LCB (Lower Confidence Bound) for the optimal arm $*$:
  $$ \text{LCB}_*(t) = \hat{\mu}_*(t) - \sqrt{\frac{4 \log t}{T_*(t)}} $$

---

#### Stopping Condition Logic
A suboptimal arm $i$ stops being selected when:
$$ \text{UCB}_i \leq \text{LCB}_* $$

**Intuition**: 
- The upper bound of arm $i$ (best-case scenario for $i$) is no better than the lower bound of the optimal arm $*$ (worst-case scenario for $*$).
- At this point, we are "confident" that arm $i$ is suboptimal and will never select it again.

---

#### Equivalence to Interval Width Condition
The stopping condition $\text{UCB}_i \leq \text{LCB}_*$ can be rewritten as:
$$ \text{UCB}_i - \text{LCB}_i \leq \Delta_i $$

**Derivation**:
1. Start with $\text{UCB}_i \leq \text{LCB}_*$
2. Substitute definitions:
   $$ \hat{\mu}_i + \sqrt{\frac{4 \log t}{T_i}} \leq \hat{\mu}_* - \sqrt{\frac{4 \log t}{T_*}} $$

---

3. Rearrange using $\Delta_i = \mu_* - \mu_i$ (true gap) and approximate $\hat{\mu}_* - \hat{\mu}_i \approx \Delta_i$ (for large $T_i, T_*$):
   $$ \sqrt{\frac{4 \log t}{T_i}} + \sqrt{\frac{4 \log t}{T_*}} \leq \Delta_i $$
4. For simplicity, assume $T_i \approx T_*$ (balanced sampling), so:
   $$ 2\sqrt{\frac{4 \log t}{T_i}} \leq \Delta_i \implies \text{UCB}_i - \text{LCB}_i \leq \Delta_i $$

---

#### Why Interval Width < Gap?
- The interval width for arm $i$ is $\text{UCB}_i - \text{LCB}_i = 2\sqrt{\frac{4 \log t}{T_i}} = \sqrt{\frac{16 \log t}{T_i}}$
- When this width is smaller than $\Delta_i$, the true mean of $i$ ($\mu_i$) must be less than the true mean of $*$ ($\mu_*$):
  $$ \mu_i + (\text{width}/2) < \mu_* - (\text{width}/2) \implies \mu_i < \mu_* - \text{width} < \mu_* - \Delta_i + \mu_i \implies \mu_* > \mu_i $$

---

#### Selection Count $T_i \approx 16\log n / \Delta_i^2$
To find when the interval width is less than $\Delta_i$:
1. Set $\sqrt{\frac{16 \log n}{T_i}} \leq \Delta_i$ (using horizon $n$ as $t \approx n$)
2. Square both sides: $\frac{16 \log n}{T_i} \leq \Delta_i^2$
3. Rearrange: $T_i \geq \frac{16 \log n}{\Delta_i^2}$

**Meaning**: Arm $i$ needs to be sampled at least $16\log n / \Delta_i^2$ times to ensure its interval width is smaller than $\Delta_i$, after which it stops being selected.

---

#### Key Takeaways
- Stopping condition ensures suboptimal arms are permanently discarded once we're confident they're worse than the best arm.
- The interval width shrinks as $T_i$ increases (more samples → more precision).
- The required number of samples $T_i$ depends on $\Delta_i$ (smaller gaps need more samples) and $n$ (larger horizons need more samples to maintain confidence).
- This balances exploration (sampling uncertain arms) and exploitation (focusing on proven good arms).
- 
---

#### IV. Regret Analysis

**Cumulative Regret ($R_n$)**: Measures total loss compared to "always choosing the optimal arm".

###### UCB Regret Bound
With $\delta = 1/n^2$ (confidence control):
$$ R_n \leq \sum_{\text{suboptimal } i} \frac{32\log n}{\Delta_i} $$

- Smaller $\Delta_i$ → larger regret (harder to identify suboptimality)

---

#### Recall UCB's Core Components
- UCB selects arms using the index:  
  $$ \text{UCB}_i(t) = \hat{\mu}_i(t) + \sqrt{\frac{4 \log t}{T_i(t)}} $$  
  where $T_i(t)$ is the number of times arm $i$ has been selected by round $t$.

- A suboptimal arm $i$ stops being selected when its interval width is smaller than its gap $\Delta_i$:  
  $$ \text{UCB}_i - \text{LCB}_i \leq \Delta_i $$  

- This occurs when $T_i \approx \frac{16 \log n}{\Delta_i^2}$ (from earlier stopping condition).

---

#### Regret Contribution of a Suboptimal Arm
For a suboptimal arm $i$ with gap $\Delta_i$, its total regret contribution comes from:  
**Number of times it is selected** × **Per-selection loss ($\Delta_i$)**.

- Let $T_i$ be the total selections of arm $i$ by horizon $n$.  
- Regret from arm $i$: $R_i = T_i \cdot \Delta_i$.

---

#### Bounding $T_i$ for UCB
To derive the regret bound, we first bound $T_i$ (selections of suboptimal arm $i$):

1. From the stopping condition, arm $i$ stops being selected when:  
   $$ \sqrt{\frac{16 \log n}{T_i}} \leq \Delta_i $$  

2. Rearranging gives the maximum $T_i$ needed to satisfy the condition:  
   $$ T_i \leq \frac{16 \log n}{\Delta_i^2} $$  

   *This is the upper bound on how many times arm $i$ is selected.*

---

#### Calculating Regret for One Suboptimal Arm
Substitute $T_i \leq \frac{16 \log n}{\Delta_i^2}$ into the regret formula for arm $i$:  
$$ R_i = T_i \cdot \Delta_i \leq \frac{16 \log n}{\Delta_i^2} \cdot \Delta_i = \frac{16 \log n}{\Delta_i} $$  

*This is the regret contribution from a single suboptimal arm.*

---

#### Why the Factor 32?
The 32 comes from **conservative confidence bounds** and **probability union bounds**:

1. UCB uses a confidence parameter $\delta = 1/n^2$ to control the probability of misestimation across all rounds and arms.  
2. To ensure the bound holds with high probability (e.g., $1 - 1/n$), we multiply by a safety factor of 2.  
3. This accounts for worst-case scenarios in concentration inequalities (e.g., Hoeffding's inequality constants) and union bounds over multiple arms/rounds.

Result: $16 \times 2 = 32$

---

#### Total Cumulative Regret
Summing over all suboptimal arms, the total regret bound becomes:  
$$ R_n = \sum_{\text{suboptimal } i} R_i \leq \sum_{\text{suboptimal } i} \frac{32 \log n}{\Delta_i} $$  

---

#### Key Takeaways
- The 32 is a conservative constant derived from:  
  - 16 (from the interval width condition for stopping selections)  
  - ×2 (to account for confidence bounds and union probabilities with $\delta = 1/n^2$).  

- Smaller $\Delta_i$ leads to larger regret because:  
  - Arms with smaller gaps require more samples ($T_i \propto 1/\Delta_i^2$) to be confidently identified as suboptimal.  
  - Each selection of such arms incurs a loss proportional to $\Delta_i$, leading to $R_i \propto 1/\Delta_i$.
- 
---

#### Regret Comparison: ETC vs. UCB

| Aspect | ETC | UCB |
|--------|-----|-----|
| **Advantages** | Slightly lower regret for $k=2$ with known $\Delta$ | No prior knowledge of $\Delta$ needed; Better performance for $k>2$ or small $\Delta_i$ |
| **Limitations** | Relies on knowing $\Delta$; Poor adaptation to multiple small gaps | Slightly higher regret than optimally tuned ETC in simple cases ($k=2$, known $\Delta$) |

---

#### V. Improved UCB: Asymptotically Optimal UCB

Original UCB requires prior knowledge of $n$. **Asymptotically optimal UCB** solves this, becoming an "anytime algorithm":

- Exploration bonus modified to:
  $$ \text{UCB Index} = \hat{\mu}_i(t-1) + \sqrt{\frac{2\log(f(t))}{T_i(t-1)}} \quad \text{where} \quad f(t) = 1 + t\log^2(t) $$

- Achieves near-optimal regret bound:
  $$ \limsup_{n \to \infty} \frac{R_n}{\sum \frac{2\log n}{\Delta_i}} \leq 1 $$

---

## Original UCB vs. Asymptotically Optimal UCB
#### Principle and Derivation Comparison

---

#### 1. Original UCB: Core Principles
- **Decision Rule**: Selects arm with highest upper confidence bound at each round:
  $$ \text{UCB}_i(t) = \hat{\mu}_i(t-1) + \sqrt{\frac{4\log t}{T_i(t-1)}} $$
- **Key Limitation**: Relies on knowing the total horizon $n$ in advance (needed to set confidence bounds for regret guarantees).
- **Exploration Bonus**: Scales with $\log t$, where $t$ is the current round, but tied to the fixed horizon $n$ for regret analysis.
- **Regret Bound**: $ R_n \leq \sum_{\text{suboptimal } i} \frac{32\log n}{\Delta_i} $ (constant factor 32, dependent on $n$).

---

#### 2. Asymptotically Optimal UCB: Motivation
- **Problem with Original UCB**: Requires prior knowledge of $n$ (total rounds), making it unsuitable for "anytime" scenarios (unknown horizon).
- **Goal**: Design an algorithm that works without knowing $n$ in advance and achieves near-optimal regret as $n \to \infty$.
- **Intuition**: Adjust the exploration bonus to grow appropriately with $t$ (current round) instead of relying on $n$, ensuring robustness to unknown horizons.

---

#### 3. Asymptotically Optimal UCB: Modified Exploration Bonus
- **New UCB Index**:
  $$ \text{UCB Index} = \hat{\mu}_i(t-1) + \sqrt{\frac{2\log(f(t))}{T_i(t-1)}} $$
  Where $$ f(t) = 1 + t\log^2(t) $$ (a function growing with $t$ to replace fixed $n$).

- **Why This Form?**:
  - $f(t)$ ensures the exploration bonus shrinks at the right rate: fast enough to avoid excessive exploration, slow enough to guarantee identification of optimal arms.
  - The coefficient 2 replaces 4 from original UCB, tightening the bound asymptotically.

---

#### 4. Derivation of the Exploration Bonus
- **Confidence Interval Adjustment**: Uses concentration inequalities (e.g., Hoeffding) with a time-dependent confidence level. For unknown $n$, the failure probability for each round $t$ is bounded by $1/t^2$ (instead of $1/n^2$ in original UCB).
- **Summing Failures**: By the union bound, the total failure probability over all rounds is bounded (since $\sum 1/t^2$ converges).
- **Resulting Bonus**: The term $\log(f(t))$ emerges from balancing the need to control cumulative failure probability while avoiding over-exploration.

---

#### 5. Near-Optimal Regret Bound
- **Asymptotic Regret**:
  $$ \limsup_{n \to \infty} \frac{R_n}{\sum \frac{2\log n}{\Delta_i}} \leq 1 $$

- **Interpretation**:
  - As $n$ becomes very large, the regret of the improved UCB is at most the "ideal" regret (scaling with $\sum \frac{2\log n}{\Delta_i}$).
  - The constant factor approaches 1, making it asymptotically optimal (original UCB has a larger constant like 32).

---

#### 6. Key Differences Summary

| Aspect                | Original UCB                          | Asymptotically Optimal UCB           |
|-----------------------|---------------------------------------|---------------------------------------|
| **Horizon Requirement** | Needs prior knowledge of $n$          | Works with unknown $n$ (anytime)      |
| **Exploration Bonus**  | $\sqrt{\frac{4\log t}{T_i}}$          | $\sqrt{\frac{2\log(f(t))}{T_i}}$ where $f(t)=1+t\log^2(t)$ |
| **Regret Constant**    | 32 (conservative)                     | Approaches 1 asymptotically           |
| **Use Case**           | Known horizon scenarios               | Unknown horizon, large $n$ scenarios  |

---

#### 7. Significance of Asymptotic Optimality
- Eliminates the need for tuning based on $n$, making it more practical for real-world problems (e.g., online recommendation systems, A/B testing with variable duration).
- Achieves the best possible regret scaling as $n$ grows, matching the theoretical lower bound for multi-armed bandits (up to a constant factor).

---

#### VI. Key Conclusions

1. **ETC**: Suitable for scenarios with known $\Delta$ and few arms ($k=2$); Requires precise tuning of $m$.

2. **UCB**: Better for unknown $\Delta$, multiple arms ($k>2$), or small suboptimality gaps; More robust and widely used in practice.

3. **Exploration-Exploitation Balance**: UCB achieves dynamic balance via confidence intervals, while ETC uses fixed exploration—making UCB more adaptable to complex scenarios.

---

## Q&A