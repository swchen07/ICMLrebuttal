We thank the reviewer for the continued engagement. We take the concern about accuracy guarantees seriously and provide both formal analysis and empirical validation.

**Formal Accuracy Guarantee.** We establish three results for SENSE's Soft-gated Evaluation under greedy decoding. Let $p_t$ denote the target model's distribution at position $t$, $\hat{x}_t = \arg\max p_t$ the greedy token, $\tilde{H}(p_t) = H(p_t)/\log|\mathcal{V}|$ the normalized entropy, and $\mathcal{S}_k(p_t)$ the top-$k$ token set.

*Theorem 1 (Exact Recovery).* At any position where $\tilde{H}(p_t) \leq \gamma$, SENSE outputs exactly the greedy token. Proof: $\tilde{H} \leq \gamma \Rightarrow B_{\text{ent}}=0$, so Eq. 17 reduces to $G = B_{\text{em}}$—only exact matches pass. If no draft matches, the system falls back to $\hat{x}_t$ (Eq. 5, $\ell^n=0$).

*Theorem 2 (Probability Bound).* When $\tilde{H}(p_t)>\gamma$ and SE accepts $\tilde{x}_t \neq \hat{x}_t$ via $B_{\text{topk}}$, then $\tilde{x}_t \in \mathcal{S}_k(p_t)$ with per-token log-regret $\Delta_t \leq \log(p_t^{(1)}/p_t^{(k)})$. At high-entropy positions, top-$k$ tokens share similar mass, making this ratio inherently small (e.g., for (0.35, 0.33, 0.30), $\Delta_t \leq 0.16$ nats).

*Theorem 3 (Sequence Bound).* Average regret $\bar{\Delta} \leq \rho(\gamma) \cdot \max_t \log(p_t^{(1)}/p_t^{(k)})$, where $\rho(\gamma)$ is the fraction of high-entropy positions. As $\gamma \to 1$ or $k \to 1$, $\rho \to 0$ or top-$k$ collapses to $\{\hat{x}_t\}$, and SENSE continuously recovers lossless greedy decoding.

**Configurable guarantee.** The pair $(\gamma, k)$ defines a continuous Pareto surface over (speed, quality), with lossless SD as a strict special case. Table 5 validates: $\gamma$=0.30 yields exact Vanilla accuracy; default $\gamma$=0.05, $k$=3 achieves 98% at 2.8×. Practitioners can set their accuracy-speed tradeoff via a principled, continuous knob.

**Error propagation.** We acknowledge Theorems 1–3 are per-position guarantees, while autoregressive generation propagates context. Two properties mitigate this: (1) The entropy gate creates a negative feedback loop—an accepted divergent token typically increases subsequent entropy, triggering stricter verification and dampening error propagation. (2) $B_{\text{con}}$'s neighborhood check (Eq. 16) ensures mismatches are isolated rather than systematic, preventing cascade failures.

**Empirical validation on Spec-Bench.** To complement the formal analysis, we evaluated SENSE on Spec-Bench (Qwen3-8B) with LLM-as-Judge (Claude Opus 4.6) across three 1–5 scale dimensions. SENSE achieves 1.63× throughput gain with 100% math accuracy (identical to Vanilla on all 17 problems). The overall quality delta across Correctness, Completeness, and Quality dimensions is merely −0.02—within noise, confirming that SENSE's acceleration comes at no measurable cost to generation quality.

| Dimension    | Vanilla    | SENSE      |
| :----------- | :--------- | :--------- |
| TPS          | 5.69 tok/s | 9.28 tok/s |
| Quality      | 2.33±1.55  | 2.33±1.49  |
| Correctness  | 2.89±1.71  | 2.79±1.65  |
| Completeness | 2.22±1.57  | 2.25±1.52  |

We will incorporate the formal analysis and error propagation discussion in the revised manuscript. Appreciate!



We thank the reviewer for the continued engagement. We take the concern about accuracy guarantees seriously and provide both formal analysis and empirical validation.

**Formal Accuracy Guarantee.** We establish three results for SENSE's Soft-gated Evaluation under greedy decoding. Let $p_t$ denote the target model's distribution at position $t$, $\hat{x}_t = \arg\max p_t$ the greedy token, $\tilde{H}(p_t) = H(p_t)/\log|\mathcal{V}|$ the normalized entropy, and $\mathcal{S}_k(p_t)$ the top-$k$ token set.

*Theorem 1 (Exact Recovery).* At any position where $\tilde{H}(p_t) \leq \gamma$, SENSE outputs exactly the greedy token. Proof: $\tilde{H} \leq \gamma \Rightarrow B_{\text{ent}}=0$, so Eq. 17 reduces to $G = B_{\text{em}}$—only exact matches pass. If no draft matches, the system falls back to $\hat{x}_t$ (Eq. 5, $\ell^n=0$).

*Theorem 2 (Probability Bound).* When $\tilde{H}(p_t)>\gamma$ and SE accepts $\tilde{x}_t \neq \hat{x}_t$ via $B_{\text{topk}}$, then $\tilde{x}_t \in \mathcal{S}_k(p_t)$ with per-token log-regret $\Delta_t \leq \log(p_t^{(1)}/p_t^{(k)})$. At high-entropy positions, top-$k$ tokens share similar mass, making this ratio inherently small (e.g., for (0.35, 0.33, 0.30), $\Delta_t \leq 0.16$ nats).

*Theorem 3 (Sequence Bound).* Average regret $\bar{\Delta} \leq \rho(\gamma) \cdot \max_t \log(p_t^{(1)}/p_t^{(k)})$, where $\rho(\gamma)$ is the fraction of high-entropy positions. As $\gamma \to 1$ or $k \to 1$, $\rho \to 0$ or top-$k$ collapses to $\{\hat{x}_t\}$, and SENSE continuously recovers lossless greedy decoding.

**Configurable guarantee.** The pair $(\gamma, k)$ defines a continuous Pareto surface over (speed, quality), with lossless SD as a strict special case. Table 5 validates: $\gamma$=0.30 yields exact Vanilla accuracy; default $\gamma$=0.05, $k$=3 achieves 98% at 2.8×. Practitioners can set their accuracy-speed tradeoff via a principled, continuous knob.

**Error propagation.** We acknowledge Theorems 1–3 are per-position guarantees, while autoregressive generation propagates context. Two properties mitigate this: (1) The entropy gate creates a negative feedback loop—an accepted divergent token typically increases subsequent entropy, triggering stricter verification and dampening error propagation. (2) $B_{\text{con}}$'s neighborhood check (Eq. 16) ensures mismatches are isolated rather than systematic, preventing cascade failures.

**Empirical validation on Spec-Bench.** To complement the formal analysis, we evaluated SENSE on Spec-Bench (Qwen3-8B) with LLM-as-Judge (Claude Opus 4.6) across three 1–5 scale dimensions. SENSE achieves 1.63× throughput gain with 100% math accuracy (identical to Vanilla on all 17 problems). The overall quality delta across Correctness, Completeness, and Quality dimensions is merely −0.02—within noise, confirming that SENSE's acceleration comes at no measurable cost to generation quality.

| Dimension    | Vanilla    | SENSE      |
| ------------ | ---------- | ---------- |
| TPS          | 5.69 tok/s | 9.28 tok/s |
| Quality      | 2.33±1.55  | 2.33±1.49  |
| Correctness  | 2.89±1.71  | 2.79±1.65  |
| Completeness | 2.22±1.57  | 2.25±1.52  |

We will incorporate the formal analysis and error propagation discussion in the revised manuscript. Appreciate!