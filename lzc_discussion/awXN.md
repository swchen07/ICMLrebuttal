We thank the reviewer for the continued engagement. We take the concern about accuracy guarantees seriously and provide formal analysis here.

**Formal Accuracy Guarantee.** We establish three results for SENSE's Soft-gated Evaluation under greedy decoding. Let $p_t$ denote the target model's distribution at position $t$, $\hat{x}_t = \arg\max p_t$ the greedy token, $\tilde{H}(p_t) = H(p_t)/\log|\mathcal{V}|$ the normalized entropy, and $\mathcal{S}_k(p_t)$ the top-$k$ token set.

*Theorem 1 (Exact Recovery).* At any position where $\tilde{H}(p_t) \leq \gamma$, SENSE outputs exactly the greedy token. And $\tilde{H} \leq \gamma \Rightarrow B_{\text{ent}}=0$, so Eq. 17 reduces to $G = B_{\text{em}}$—only exact matches pass. If no draft matches, the system falls back to $\hat{x}_t$ (Eq. 5, $\ell^n=0$). 

*Theorem 2 (Probability Bound).* When $\tilde{H}(p_t)>\gamma$ and SE accepts $\tilde{x}_t \neq \hat{x}*t$ via $B*{\text{topk}}$, then $\tilde{x}_t \in \mathcal{S}_k(p_t)$ with per-token log-regret $\Delta_t \leq \log(p_t^{(1)}/p_t^{(k)})$. At high-entropy positions, top-$k$ tokens share similar mass, making this ratio inherently small (e.g., for (0.35, 0.33, 0.30), $\Delta_t \leq 0.16$ nats).

*Theorem 3 (Sequence Bound).* Average regret $\bar{\Delta} \leq \rho(\gamma) \cdot \max_t \log(p_t^{(1)}/p_t^{(k)})$, where $\rho(\gamma)$ is the fraction of high-entropy positions. As $\gamma \to 1$ or $k \to 1$, $\rho \to 0$ or top-$k$ collapses to ${\hat{x}_t}$, and SENSE continuously recovers lossless greedy decoding. 

**Configurable guarantee.** The pair $(\gamma, k)$ defines a continuous Pareto surface over (speed, quality), with lossless SD as a strict special case—not a separate system. As table 5 validates: $\gamma$=0.30 yields exact Vanilla accuracy; default $\gamma$=0.05, $k$=3 achieves 98% at 2.8×. This gives practitioners a principled knob to set their own accuracy-speed tradeoff.

**Error propagation.** We acknowledge that Theorems 1–3 provide per-position guarantees, while autoregressive generation propagates context. Two properties of SENSE mitigate this: (1) The entropy gate creates a negative feedback loop—if an accepted token shifts the next distribution, this typically increases entropy, triggering stricter verification at subsequent positions, dampening error propagation. (2) $B_{\text{con}}$'s neighborhood check (Eq. 16) explicitly ensures mismatches are isolated rather than systematic, preventing cascade failures.

We will incorporate the formal analysis, configure guarantee and error propagation discussion in the revised manuscript. Appreciate!