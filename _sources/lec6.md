---
marp: true
theme: default
paginate: true
header: "Class 5: Introduction to Multi-Armed Bandits（Thompson Sampling）"
footer: "Fangli Ying (ECUST) & Osman Yagan (CMU) | Multi-Armed Bandits"
---

# 6. Thompson Sampling

---

## Multi-armed Bandits: Core Concepts and Algorithms
- UCB Algorithm
- Bayesian Learning Framework
- Thompson Sampling

---

#### Fundamentals of Multi-armed Bandits
- **Problem Definition**: Sequential decision-making model with multiple "arms" (options/strategies)
- **Goal**: Maximize cumulative reward (or minimize "regret")
- **Key Challenge**: Exploration-Exploitation Tradeoff
  - Exploitation: Choose arms with highest observed reward
  - Exploration: Try other arms to gain more information

---

#### UCB (Upper Confidence Bound) Algorithm
- Balances exploration and exploitation using confidence intervals
- Core idea: Select arm with highest upper confidence bound of reward mean
  - Considers current average reward (exploitation)
  - Accounts for estimation uncertainty (exploration)

---

###### UCB Decision Rule
At round $t$, select arm:
$$A_t = \arg\max_i \left( \hat{\mu}_i(t-1) + \sqrt{\frac{4 \log n}{T_i(t-1)}} \right)$$

- $A_t$: Selected arm at round $t$
- $\hat{\mu}_i(t-1)$: Average reward of arm $i$ up to $t-1$ rounds
- $T_i(t-1)$: Number of times arm $i$ was selected up to $t-1$ rounds
- $n$: Total number of rounds
- Second term: Uncertainty penalty (larger for less explored arms)

---

###### Regret Analysis for UCB
**Regret** is defined as:
$$R(n) = \sum_{t=1}^n (\mu^* - \mu_{A_t})$$

- $\mu^*$: True mean of optimal arm ($\mu^* = \max_i \mu_i$)
- $\mu_{A_t}$: True mean of selected arm at round $t$

---

######## Suboptimal Arm Contribution
For suboptimal arm $i$ ($\mu_i < \mu^*$), define gap $\Delta_i = \mu^* - \mu_i$

UCB regret upper bound:
$$R(n) \leq \sum_{i: \Delta_i > 0} \frac{16 \log n}{\Delta_i^2}$$

---

######## Bound Justification
- Intuition: Limit selection frequency of suboptimal arms
- When $T_i$ (selection count) satisfies:
  $$2\sqrt{\frac{4 \log n}{T_i}} \leq \Delta_i$$
- Suboptimal arm $i$ will no longer be selected
- Solving gives $T_i \leq \frac{16 \log n}{\Delta_i^2}$

---

###### Asymptotically Optimal UCB
For large $n$, regret approximates:
$$R(n) \approx \sum_{i: \Delta_i > 0} \frac{2 \log n}{\Delta_i^2}$$

- Achieves theoretical lower bound for multi-armed bandits
- Near-ideal performance as $n \to \infty$

---

#### Bayesian Learning Framework
- Contrasts with frequentist UCB approach
- Models uncertainty through probability distributions
- Updates beliefs using observed data
- Selects strategies to minimize expected loss

---

###### Uncertainty Components
- **Environment**: Unknown scenario ($v$) - e.g., reward distributions of arms
- **Policy**: Decision rule ($\pi$) - e.g., arm selection method
- **Loss Function**: Measures policy performance in environment

---

###### Policy Properties
- **Dominance**: Policy $\pi_1$ is dominated by $\pi_2$ if:
  - $\pi_1$ has greater or equal loss in all environments
  - $\pi_1$ has strictly greater loss in at least one environment

- **Admissibility (Pareto Optimality)**:
  - Not dominated by any other policy
  - Minimizes loss in at least one environment

---

###### Bayesian Decision Core Idea
- **Prior distribution** $q(v)$: Initial belief about environment $v$
- Choose policy minimizing expected loss:
  $$\pi_{\text{Bayes}} = \arg\min_{\pi} \mathbb{E}_v [\text{loss}(\pi, v)]$$
- Expectation $\mathbb{E}_v$ computed using prior $q(v)$

---

###### Sequential Bayesian Learning
Update beliefs after each observation:
$$p(v | \text{data}) = \frac{p(\text{data} | v) \cdot q(v)}{p(\text{data})}$$

- $p(\text{data} | v)$: Likelihood of data under environment $v$
- $p(\text{data})$: Total probability (normalization constant)
- Updates from prior to posterior distribution with new data

---

#### Thompson Sampling Algorithm
- Bayesian approach to multi-armed bandits
- Achieves exploration-exploitation via stochastic sampling
- Selects arm with highest sample from posterior distributions

---

###### Core Idea
- Maintain **posterior distribution** $F_i(t)$ for each arm $i$'s mean $\mu_i$
- Sample $\hat{\mu}_i^{(s)}$ from each $F_i(t)$
- Select arm with maximum sample: $A_t = \arg\max_i \hat{\mu}_i^{(s)}$
- Update posterior of selected arm using observed reward

---

###### Algorithm Steps
1. **Input**: Prior cumulative distribution functions (CDFs) $F_i(0)$ for each arm
2. For each round $t=1,2,...,n$:
   - Sample $\hat{\mu}_i^{(s)}$ from $F_i(t-1)$ for each arm $i$
   - Select $A_t = \arg\max_i \hat{\mu}_i^{(s)}$
   - Observe reward $X_t$ and update:
     $$F_{A_t}(t) = \text{UPDATE}(F_{A_t}(t-1), X_t)$$
3. Repeat until completion

---

###### Update Rule (Gaussian Example)
For rewards ~ $\mathcal{N}(\mu_i, 1)$:
- After $T_i(t)$ selections with rewards $x_1,...,x_{T_i(t)}$
- Sample mean: $\hat{\mu}_i = \frac{1}{T_i(t)} \sum_{k=1}^{T_i(t)} x_k$
- Posterior distribution: $\mathcal{N}(\hat{\mu}_i, \frac{1}{T_i(t)})$
- Variance decreases with more selections (increased certainty)

---

###### Performance Analysis
- **Asymptotic Optimality** in Gaussian bandits:
  $$R(n) \approx \sum_{i: \Delta_i > 0} \frac{2 \log n}{\Delta_i^2}$$
  - Matches theoretical lower bound
- **Characteristics**:
  - Exploration through random sampling
  - Often outperforms UCB in experiments
  - Higher performance variance (more result fluctuation)

---

#### Supplementary Derivations
###### Gaussian Sample Mean Properties
For i.i.d. Gaussian variables $x_1,...,x_t$ ~ $\mathcal{N}(\mu, \sigma^2)$:
- Sample mean: $\hat{\mu}(t) = \frac{x_1+...+x_t}{t}$
- Expectation: $\mathbb{E}[\hat{\mu}(t)] = \mu$ (unbiased)
- Variance: $\text{Var}(\hat{\mu}(t)) = \frac{\sigma^2}{t}$ (decreases with $t$)

---

###### Bayesian Formula Application
**Example**: Bag containing balls (WW or WB)
- Prior: $P(WW) = P(WB) = 0.5$
- After observing "white ball":
  - Likelihoods: $P(\text{white}|WW)=1$, $P(\text{white}|WB)=0.5$
  - Total probability: $P(\text{white}) = 0.5 \cdot 1 + 0.5 \cdot 0.5 = 0.75$
  - Posterior: $P(WW | \text{white}) = \frac{1 \cdot 0.5}{0.75} = \frac{2}{3}$

---

#### Summary
| Aspect | UCB Algorithm | Thompson Sampling |
|--------|--------------|-------------------|
| Approach | Frequentist (confidence intervals) | Bayesian (posterior sampling) |
| Selection | Deterministic (max upper bound) | Stochastic (max sample) |
| Regret | Provable bounds, stable performance | Asymptotically optimal, better average performance |
| Variance | Lower | Higher |
| Use Case | Stability-critical scenarios | Average performance priority |

Both balance exploration-exploitation to minimize regret