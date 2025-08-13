---
marp: true
theme: default
paginate: true
---

# 3. Explore-then-Commit (ETC) & Upper-Confidence-Bound (UCB)*  


---

## Review of Lectures  
*Explore-then-Commit (ETC) & Upper-Confidence-Bound (UCB)*  
*Bandit Algorithms for Sequential Decision Making*

---

#### 1. Explore–then–Commit (ETC) – High-Level Idea  
- **Phase 1 – Explore**: pull *every* arm exactly $m$ times.  
- **Phase 2 – Commit**: forever after pull the arm with the **largest empirical mean** from the first $m$ pulls.

---

#### 2. Notation & Core Quantities  
| Symbol | Meaning |
|--------|---------|
| $k$ | number of arms |
| $n$ | horizon (total rounds) |
| $m$ | exploration length per arm |
| $\Delta_i = \mu^* - \mu_i$ | sub-optimality gap of arm $i$ |
| $\hat{\mu}_{i,m}$ | empirical mean of arm $i$ after $m$ pulls |
| $\mu_i$ | true mean reward of arm $i$ |

---

#### 3. Regret Definition  
**Total regret after $n$ rounds:**

$$
R_n(m) = m \sum_{i=1}^{k}\Delta_i + (n-mk)\sum_{i=1}^{k}\Delta_i \mathbb{P}\left(\text{commit to arm }i\right)
$$

- First term: regret **during exploration**.  
- Second term: regret **due to possibly wrong commitment**.

---

#### 4. Bounding the Commitment Error  
When rewards are **1-sub-Gaussian**:

$$
\mathbb{P}\!\left(\hat{\mu}_{i,m} \ge \hat{\mu}_{1,m}\right) \le \exp\!\left(-\frac{m\Delta_i^2}{2}\right)
$$

Hence

$$
\mathbb{P}(\text{commit to sub-optimal }i) \le \exp\!\left(-\frac{m\Delta_i^2}{2}\right)
$$

---




## Understanding 1-sub-Gaussian Rewards
#### Key Concepts & Probability Bounds

---

###### Symbols & Setup
- $\hat{\mu}_{i,m}$: Sample mean of $m$ observations for option $i$
- Option 1 is **optimal**: $\mu_1$ (true mean) is largest
- For suboptimal $i$ ($i \neq 1$):  
  $\Delta_i = \mu_1 - \mu_i > 0$ (true mean gap)

---

###### What is 1-sub-Gaussian?
- A random variable with **strong concentration** around its mean
- Deviations from the mean have exponentially decaying tail probabilities
- Sample mean $\hat{\mu}_{i,m}$ stays close to true mean $\mu_i$

---

###### First Inequality Explained
$$\mathbb{P}\!\left(\hat{\mu}_{i,m} \ge \hat{\mu}_{1,m}\right) \le \exp\!\left(-\frac{m\Delta_i^2}{2}\right)$$

- Event: Suboptimal $i$'s sample mean ≥ optimal 1's sample mean
- This implies:  
  $(\hat{\mu}_{i,m} - \mu_i) - (\hat{\mu}_{1,m} - \mu_1) \ge \Delta_i$
- 1-sub-Gaussian property makes this event exponentially unlikely

---

###### Second Inequality Explained
$$\mathbb{P}(\text{commit to sub-optimal }i) \le \exp\!\left(-\frac{m\Delta_i^2}{2}\right)$$

- "Commit to sub-optimal $i$" = choosing $i$ over 1
- This happens **only if** $\hat{\mu}_{i,m} \ge \hat{\mu}_{1,m}$
- Thus, its probability inherits the same bound

---

###### Example
- Optimal arm 1: $\mu_1 = 10$ (1-sub-Gaussian, e.g., $N(10,1)$)
- Suboptimal arm 2: $\mu_2 = 8$ ($\Delta_2 = 2$)
- With $m=5$ samples:  
  $\mathbb{P}(\text{choose arm 2}) \le \exp(-\frac{5 \times 2^2}{2}) = e^{-10} \approx 4.5 \times 10^{-5}$

---

###### Takeaway
- Larger $m$ (samples) → smaller probability of choosing suboptimal
- Larger $\Delta_i$ (bigger gap) → smaller error probability
- Exponential decay ensures reliable selection with enough samples

---

#### 5. Two-Arms Case – Choosing $m$  
For $k=2$ and $\Delta = \Delta_2$:

$$
R_n(m) \approx m\Delta + (n-2m)\Delta \exp\!\left(-\frac{m\Delta^2}{2}\right)
$$

Take derivative w.r.t. $m$ and set to zero $\Rightarrow$ **optimal**

$$
m^* = \left\lceil\frac{4\log n}{\Delta^2}\right\rceil
$$

gives

$$
R_n = O\!\left(\frac{\log n}{\Delta}\right)
$$

---

#### 6. General $k$ Arms  
Optimal exploration length:

$$
m = \max_{i:\Delta_i>0}\frac{4\log n}{\Delta_i^2}
$$

Resulting regret:

$$
R_n \le \sum_{i:\Delta_i>0}\frac{4\log n}{\Delta_i} + \text{small terms}
$$

---

#### 7. Practical Caveat  
Choosing $m$ above **requires knowledge of $\Delta_i$ and $n$**  
– **often unknown in practice**.

---

#### 8. Data-Dependent Fix  
Instead of a fixed $m$, **keep exploring until**:

$$
t \ge 2^t \text{ threshold (function of observed data)}
$$

Achieves the *same* $\sum_i\frac{4\log n}{\Delta_i}$ regret without prior $\Delta_i$.

---

#### 9. Scenario Comparison  
| Scenario | $\Delta$ | Relative Regret |
|----------|----------|-----------------|
| 1        | 0.2      | smaller         |
| 2        | 0.1      | larger          |

Larger $\Delta$ $\Rightarrow$ fewer mistakes, **even though per-mistake regret is bigger**.

---

#### 10. Motivation for UCB  
ETC weaknesses:  
- Needs $\Delta_i$ to set $m$.  
- **Abrupt** switch from explore to exploit.  
- All arms **equally** explored.

**UCB** addresses all three issues via *optimism under uncertainty*.

---

#### 11. UCB – Main Idea  
In each round $t$ compute an **upper confidence bound (UCB)** index for every arm:

$$
\text{UCB}_{i,t-1} = \underbrace{\hat{\mu}_{i,T_i(t-1)}}_{\text{empirical mean}} + \underbrace{\sqrt{\frac{2\log(1/\delta)}{T_i(t-1)}}}_{\text{exploration bonus}}
$$

Pull arm

$$
A_t = \arg\max_i \text{UCB}_{i,t-1}
$$

---

#### 12. Confidence Bound Lemma  
For 1-sub-Gaussian rewards:

$$
\mathbb{P}\!\left(\forall s\ge 1,\ |\hat{\mu}_s - \mu| \le \sqrt{\frac{2\log(2/\delta)}{s}}\right) \ge 1-\delta
$$

Hence choose

$$
\delta_t = \frac{1}{t^4} \quad\Rightarrow\quad \text{bonus} = \sqrt{\frac{2\log t}{T_i(t-1)}}
$$

---



#### 13. UCB Algorithm (Pseudo-code)

The UCB algorithm operates as follows:

1. **Initialization**: Take the number of arms k as input
2. **For each round t from 1 to n**:
   - For each arm i:
     - If the arm has never been pulled before, set its UCB value to +∞
     - Otherwise, calculate its UCB value as: empirical mean $(μ̂_i)$ plus the exploration bonus  $\sqrt{\frac{2\log(1/\delta)}{T_i(t-1)}}$ , where $T_i(t-1)$ is the number of times arm i has been pulled before round $t$
   - Select the arm with the highest UCB value $(A_t = argmax_i UCB_{i,t-1})$
   - Pull the selected arm and observe the reward $X_t$
3. **Repeat** this process until all n rounds are completed
---

#### 14. Intuition – Why UCB Works  
- **Confidence intervals** shrink as $T_i$ grows.  
- Best arm’s UCB $\approx$ its true mean (many pulls).  
- Sub-optimal arm stops being pulled when

$$
\sqrt{\frac{2\log t}{T_i(t-1)}} \le \frac{\Delta_i}{2}
\quad\Rightarrow\quad
T_i(t-1) \approx \frac{8\log t}{\Delta_i^2}
$$

Hence pulls of arm $i$ are **inversely proportional to $\Delta_i^2$**.

---

#### 15. Visualization – 3 Arms  
Means: $\mu_1=0.8,\ \mu_2=0.7,\ \mu_3=0.6$

| Arm | $\Delta_i$ | Typical pulls |
|-----|------------|---------------|
| 1   | 0          | $\gg\log n$   |
| 2   | 0.1        | $\propto\frac{\log n}{0.1^2}$ |
| 3   | 0.2        | $\propto\frac{\log n}{0.2^2}$ |

Confidence intervals shrink until their width matches the respective $\Delta_i$.

---

#### 16. Take-Away Summary  
| Algorithm | Needs $\Delta_i$? | Regret Bound | Abrupt Switch? |
|-----------|-------------------|--------------|----------------|
| ETC       | Yes               | $O\!\left(\frac{k\log n}{\Delta}\right)$ | Yes |
| UCB       | No                | $O\!\left(\sum_i\frac{\log n}{\Delta_i}\right)$ | No (smooth) |

UCB achieves **near-optimal regret** while being **fully adaptive**.

---