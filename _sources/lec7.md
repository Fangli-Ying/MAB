---
marp: true
theme: default
paginate: true
header: "A Unified Approach to Translate Classic Bandit Algorithms to the Structured Bandit Setting"
footer: "Fangli Ying (ECUST) & Osman Yagan (CMU) | Multi-Armed Bandits"
---

# 7. A Unified Approach to Translate Classic Bandit Algorithms to the Structured Bandit Setting

---

## A Unified Approach to Translate Classic Bandit Algorithms to the Structured Bandit Setting
Author： Osman Yağan  
Carnegie Mellon University  
  
Lecturer: Dr. Fangli Ying


---

## Stochastic Multi-armed Bandits
- $k$ actions (arms) to choose from in each round $t=1,2,...,n$  
- Choosing arm $A_t$ at round $t$ returns a random reward (follows the reward distribution of $A_t$)  
- Reward distributions and mean rewards $\mu_1, \mu_2, ..., \mu_k$ are unknown  
- **Goal**: Maximize Expected Cumulative Reward
---

## Classic Multi-armed Bandits: Key Definitions
- Let $i^* = \arg \max_i \mu_i$ (best arm with largest mean reward)  
- Equivalent Goal: Minimize expected Regret  
  $R_n = \sum_{t=1}^n (\mu_{i^*} - \mu_{A_t}) = \sum_{i=1}^k \Delta_i \cdot E[T_i(n)]$  
  where $\Delta_i = \mu_{i^*} - \mu_i$ (sub-optimality gap), $T_i(n)$ = mean number of times arm $i$ is picked over $n$ rounds  

*Regret = Reward lost by taking suboptimal decisions*

---

## Classic Multi-armed Bandits: Algorithms & Limitation
- Algorithms: UCB [Auer et al.], Thompson Sampling [Thompson], KLUCB [Bubeck et al.], etc.  
- Expected Regret: $R_n = (k-1) \times O(\log n)$ (precisely: $\sum_{i: \Delta_i > 0} \frac{2 \log n}{\Delta_i}$)  

**Limitation**: Rewards assumed to be independent across arms  
- Information about an arm is only obtained if it is selected
---

## Variants: Contextual Bandits
- Each round $t$: receive side information (context vector $\theta$)  
  - Context carries info about rewards of different actions  
  - e.g., age, income, profession, location, app activity  
- Mean rewards $\mu_i(\theta)$ depend on context $\theta$ (known)  
- Goal: Learn $\mu_i(\theta)$ as functions of $\theta$ and choose optimal actions
---

## This Work: Structured Bandits
- Hidden parameter $\theta$ (unknown) - common across arms  
- Mean rewards $\mu_1(\theta), ..., \mu_k(\theta)$: known functions of $\theta$  
- Reward of an arm can be inferred *without selecting it* (via $\theta$)  

*Example*: $\theta$ = user attributes (age, occupation); $\mu_i(\theta)$ = rating of movie genre $i$ for user with $\theta$

---

## Related Work
- Linear Bandits: $\mu_i(\theta)$ is linear in $\theta$  
- Global & Regional Bandits [Atan et al., Wang et al.]: $\mu_i(\theta)$ is invertible  
- Known conditional reward distributions [Combes et al., 2017]  

**Closest work**: [Lattimore et al., 2014] - UCB-S algorithm  
- Does not assume linearity, invertibility, or known reward distributions  
- Limited to modified UCB scheme
---

## Main Contributions
- Proposed a class of algorithms for structured setting (no linearity/invertibility assumptions)  
- **ALGORITHM-C**: Adapt any classic MAB algorithm (UCB, TS, KL-UCB, etc.) to structured setting  
  - e.g., UCB-C, TS-C, KLUCB-C  
- Previous work [Lattimore et al., 2014] only works with modified UCB
---

## Competitive vs. Non-competitive Arms
- **Non-competitive arm $i$**: Can be identified as suboptimal using samples from the best arm $i^*$  
  Formally: For true $\theta^*$, $\exists \epsilon > 0$ s.t. $\mu_{i^*}(\theta) > \mu_i(\theta) \forall \theta: |\mu_{i^*}(\theta^*) - \mu_{i^*}(\theta)| < \epsilon$  

- **Competitive arm $i$**: Cannot be ruled out as suboptimal without direct sampling  

- $C(\theta^*)$: Number of competitive arms at true $\theta^*$
---

## Performance vs. Classical Bandits
- Classic algorithms: $R_n = (k-1) O(\log n)$  
  - Each suboptimal arm is pulled $O(\log n)$ times  

- Our algorithms (UCB-C, TS-C): $R_n = (C-1) O(\log n) + O(1)$  
  - Non-competitive arms: pulled $O(1)$ times  
  - If $C=1$, $R_n = O(1)$ (constant regret)
---

## Overview of Our Algorithm (Any Round $t$)
1. **Estimate confidence set $\widehat{\Theta}_t$ for $\theta^*$**  
   - $\widehat{\Theta}_t = \{\theta: |\hat{\mu}_k(t) - \mu_k(\theta)| \leq \sqrt{\frac{a \log t}{n_k(t)}} \forall k \in [K]\}$  
     where $\hat{\mu}_k(t)$ = empirical mean of arm $k$; $n_k(t)$ = number of times arm $k$ is picked by $t$  

2. **Remove $\widehat{\Theta}_t$-non-competitive arms**  
   - Focus on arms that could be optimal for some $\theta \in \widehat{\Theta}_t$  

3. **Play $\widehat{\Theta}_t$-competitive arms using any classic bandit algorithm**
---

## UCB-S Algorithm [Lattimore and Munos]
1. Estimate confidence set $\widetilde{\Theta}_t$ for $\theta^*$ (same as step 1 of our algorithm)  
2. In round $t$, play the arm with the largest *most optimistic mean reward* in $\widetilde{\Theta}_t$  
   - i.e., $\arg \max_i \sup_{\tilde{\theta} \in \widetilde{\Theta}_t} \mu_i(\tilde{\theta})$  

**Limitation**: Only uses a modified UCB scheme (not flexible to other algorithms)

---

## Regret Bound for UCB-C
- **Theorem 1 (Non-competitive arms)**:  
  $E[T_i(n)] \leq k t_0 + \sum_{t=1}^n 2k t^{1-\alpha} + k^3 \sum_{kt_0}^n 6\left(\frac{t}{k}\right)^{2-\alpha} = O(1)$ (for $\alpha > 3$)  

- **Theorem 2 (Competitive arms)**:  
  $E[T_i(n)] \leq \frac{8 \alpha \log n}{\Delta_i^2} + \frac{2 \alpha}{\alpha - 2} + \sum_{t=1}^n 2k t^{1-\alpha} = O(\log n)$  

- **Overall Regret**: $E[R_n] \leq (C-1) O(\log n) + O(1)$  
  - If $C=1$, $E[R_n] = O(1)$
---

## Simulations: Key Findings
- Parameters: $\theta^* = 0.5, 1.5, 2.6$ with $C(\theta^*) = 1, 2, 3$ respectively  
- Algorithms compared: UCB, UCB-S, UCB-C, TS-C  

**Results**:  
- TS-C performs best overall  
- UCB-S is too sensitive to $\theta^*$  
- Our algorithms (UCB-C, TS-C) outperform classic UCB and UCB-S in structured settings
---

## Experiments on MovieLens Dataset
- **Dataset**: 1M ratings for 3883 movies by 6040 users; 18 genres (arms)  
- $\theta$ = (age, occupation) - user attributes  
- Goal: Recommend optimal movie genre for unknown user types  

**Results**:  
- TS-C consistently achieves the lowest cumulative regret across user types (18-26 college students, 25-34 executives, 45-49 clerical/admin)
---

## Key Takeaways
1. A unified approach to adapt any classic bandit algorithm to structured settings  
2. Regret bound: $O((C-1) \log n)$ where $C$ = number of competitive arms  
3. Non-competitive arms are pulled only $O(1)$ times  
4. TS-C shows superior performance in empirical evaluations
---

## Q&A
Thank you!