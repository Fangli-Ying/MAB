---
marp: true
theme: default
paginate: true
math: mathjax
header: "Multi-Armed Bandits with Probing (Explained by Dr. Fangli Ying)"
footer: "Fangli Ying (ECUST) & Osman Yağan (CMU) | Multi-Armed Bandits"
---

# 5. Multi-Armed Bandits with Probing


---

## Multi-Armed Bandits with Probing: Paper Explained by Dr. Fangli Ying
#### UCBP and Its Theoretical Guarantees
**Authors**: Eray Can Elumar, Cem Tekin, Osman Yağan  
**Institution**: Carnegie Mellon University & Bilkent University

---

## Introduction to Multi-Armed Bandits (MAB)

- **Classic MAB Problem**: A decision-maker (agent) selects from $K$ arms, each with an unknown reward distribution.
- **Goal**: Maximize cumulative reward over $T$ rounds by balancing *exploration* (learning arm rewards) and *exploitation* (choosing known high-reward arms).
- **Regret**: Traditional regret = (Optimal arm's total reward) - (Agent's total reward).
- **Novel Extension: MAB with Probing**
  - Agent can *probe* an arm (observe its reward) at cost $c \geq 0$ before deciding to pull it or a backup arm.
  - Probing adds flexibility but complicates action space (now $O(K^2)$ actions).

---

## Key Applications

1. **Hyperparameter Optimization**:
   - "Pull" = Run a model with a hyperparameter setting (no oversight).
   - "Probe" = Run with expert supervision (terminate poor runs early, paying $c$ for expert time).

2. **Online Learning with ML Advice**:
   - Probing = Using ML predictions to estimate rewards (cost $c$ = computation expense).

---
## Key Applications

3. **Wireless Communications**:
   - Probing = Sending small packets to check channel quality before transmission.

4. **Healthcare (ER Queuing)**:
   - Probing = Preliminary tests to assess patient urgency; "pull" = full treatment.

---

## Problem Formulation

- **Arms**: $K$ arms, each with reward distribution $\Gamma_i$, mean $\mu_i$.
- **Actions**:
  - *Pull*: Directly pull arm $i$, get reward $r_i \sim \Gamma_i$. Denoted as $(i, \emptyset)$.
  - *Probe*: Choose probe arm $i$ and backup arm $j \neq i$:
    - Observe $r_i \sim \Gamma_i$.
    - Pull $i$ (reward $r_i - c$) if $r_i$ is good; else pull $j$ (reward $r_j - c$). Denoted as $(i, j)$.
- **Action Set**: $A = A_s \cup A_p$, where $A_s$ (pulls) has size $K$, $A_p$ (probes) has size $K(K-1)$. Total: $|A| = K^2$.

---

## Optimal Action and Regret Definition

- **Optimal Action**: Maximizes expected reward $\nu^*$, where:
  - For pull $(i, \emptyset)$: $\nu_{(i, \emptyset)} = \mu_i$.
  - For probe $(i, j)$: $\nu_{(i, j)} = \mathbb{E}[\max(r_i, \mu_j)] - c$.

- **Regret Definitions**:
  - Empirical cumulative regret: $\hat{R}_T = T\nu^* - \sum_{t=1}^T r(t)$.
  - Expected cumulative regret: $R_T = \mathbb{E}[\hat{R}_T]$.

- **Key Innovation**: Regret is relative to $\nu^*$ (optimal action's reward), not just the best arm's mean $\mu_1$.

---

## Baseline Algorithm: UCB-naive-probe

- **Idea**: Treats each probe action as a "super arm" with a fixed reference point (threshold for pulling the probe arm).
- **Action Space**: Includes triples $(i, j, d_l)$, where $d_l \in D$ (discrete reward support) is the reference point.
- **Steps**: 1. At round $t$, select the super arm with the highest UCB index.
  2. If probing: observe $r_i$; pull $i$ if $r_i \geq d_l$, else pull $j$ (reward $-c$).

- **Regret Bounds**:
  - Gap-independent: $O(K\sqrt{T \log T})$.
  - Gap-dependent: $O(K^2 \log T)$.

---

## Proposed Algorithm: UCBP (UCB with Probes)

- **Goal**: Reduce regret by leveraging the structure of probe actions and optimal reference points.
- **Core Insight**: Use UCB indices to balance exploration/exploitation and dynamically set reference points.

- **UCB Indices Calculation**:
  1. **Pull action**: $U_i(t) = \hat{\mu}_i(t) + \sqrt{\frac{3 \log t}{N_i(t)}}$,
     where $\hat{\mu}_i(t)$ = empirical mean of $i$, $N_i(t)$ = times $i$ was observed.
  2. **Probe action**: $P_{(i,j)}(t) = \hat{\nu}_{(i,j)}(t) + \sqrt{\frac{3 \log t}{N_i(t)}} + \sqrt{\frac{3 \log t}{N_j(t)}}$,
     where $\hat{\nu}_{(i,j)}(t)$ = empirical mean of $\max(r_i, \hat{\mu}_j) - c$.

---

## UCBP Decision Process (Algorithm 2)

1. Initialize: Sample each arm once to estimate initial means.
2. For each round $t$:
   a. Compute $U_i(t)$ (pull indices) and $P_{(i,j)}(t)$ (probe indices).
   b. Choose action with highest index:
      - If pull: Pull arm $i^* = \arg\max_i U_i(t)$, get $r(t) = r_{i^*}$.
      - If probe: Probe arm $j_t$, observe $r_{j_t}$.
        - Pull $j_t$ (reward $r_{j_t} - c$) if $r_{j_t} > U_{k_t}(t)$ (backup arm's UCB).
        - Else pull backup arm $k_t$ (reward $r_{k_t} - c$).
3. Update $\hat{\mu}_i$, $N_i$, and indices.

---

## Regret Decomposition

Total regret $R_T$ splits into two parts:

1. **Action Selection Regret**: Loss from choosing suboptimal actions (e.g., pulling a bad arm instead of probing).
   - For action $a$, gap $\Delta_a = \nu^* - \nu_a$; total loss from suboptimal actions is $\sum_{a \neq a^*} \Delta_a \cdot (\text{times } a \text{ is chosen})$.

2. **Reference Point Regret**: Loss from using estimated thresholds (instead of true $\mu_j$) to decide post-probe.
   - Defined as $R_{ref}(T) = \sum_{t=1}^T |\tilde{\mu}_j(t) - \mu_j| \cdot \mathbb{P}(r_i \in [\min(\mu_j, \tilde{\mu}_j(t)), \max(\mu_j, \tilde{\mu}_j(t))])$,
     where $\tilde{\mu}_j(t)$ = estimated mean of $j$.

---

## Reference Point Regret Analysis

- **Lemma IV.3**:
  a. For arbitrary bounded rewards: $R_{ref}(T) = O(\sqrt{K T \log T})$.
  b. For discrete rewards (support $D$): $R_{ref}(T) = O(K \log T)$,
     where $\gamma_i = \min |d_l - \mu_i|$ (gap between $\mu_i$ and nearest support point).

- **Rationale**: Discrete rewards limit the range of estimation errors, making $R_{ref}(T)$ smaller.

---

## Gap-Independent Regret Bound for UCBP

- **Theorem IV.1**: Under Assumption 1 ($\mathbb{P}(r_i \leq \mu_j) \geq \epsilon > 0$):

$$ R_T \leq \frac{4\sqrt{6 K T \log T}}{\epsilon} + R_{ref}(T) + \frac{2\pi^2 K^2}{3} + K $$

- **Result**: Combining with $R_{ref}(T) = O(\sqrt{K T \log T})$, total gap-independent regret is $O(\sqrt{K T \log T})$.
- **Significance**: Scales better than UCB-naive-probe ($O(K\sqrt{T \log T})$) for large $K$.

---

## Gap-Dependent Regret Bound for UCBP

- **Theorem IV.2**: For discrete rewards, gap-dependent regret satisfies:

$$ R_T \leq \sum_{i=1}^K \frac{12 \log T}{\delta_i} + R_{ref}(T) + \frac{2\pi^2 K^2}{3} + K $$

where $\delta_i$ is a gap parameter related to suboptimal actions.

- **Result**: With $R_{ref}(T) = O(K \log T)$, total gap-dependent regret is $O(K \log T)$.

---

## Order Optimality of UCBP

- **Lower Bound (Theorem IV.4)**: For any algorithm, regret is at least $\Omega(K \log T)$.
- **Conclusion**: UCBP's gap-dependent regret $O(K \log T)$ matches the lower bound, making it *order-optimal*.

---

## Empirical Results (MovieLens Dataset)

- **Setup**: 18 arms (movie genres), rewards = user ratings (1-5). Compare UCBP vs. UCB-naive-probe over 500,000 rounds.
- **Key Finding**: UCBP consistently outperforms UCB-naive-probe across probing costs $c = 0, 0.3, 1$.
  - Both show logarithmic regret growth, but UCBP has smaller cumulative regret.

---

## Contributions and Future Work

- **Contributions**:
  1. First MAB model with costly probing for bounded rewards.
  2. UCBP algorithm with gap-independent regret $O(\sqrt{K T \log T})$ and order-optimal gap-dependent regret $O(K \log T)$.
  3. Novel regret definition accounting for optimal probe/pull actions.

- **Future Work**:
  - Extend to noisy probes (instead of exact reward observations).
  - Handle correlated arm rewards.

---

## Q&A

**Thank You!**

For details, see full paper: [Multi-armed bandits with costly probes](https://users.ece.cmu.edu/~oyagan/Journals/ProbingBandits.pdf)