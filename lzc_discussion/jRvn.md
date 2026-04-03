We thank the reviewer for the constructive follow-up. We address the remaining concerns about domain robustness and retrieval quality with new evidence.

**Cross-Domain Experiments.** To directly test robustness under domain shifts where "both input distributions and reasoning patterns differ significantly," we ran cross-domain experiments using completely mismatched datastores on Qwen3-8B:

... 

表格。



Three key findings: (1) **Speedup is stable at 1.93–1.97× across all configurations**, including extreme mismatch (math↔dialogue). Acceleration does not depend on domain-specific datastore coverage. (2) ~~GSM8K cross-domain accuracy (0.87) actually exceeds in-domain (0.83) and approaches Vanilla (0.91)—mismatched datastores avoid reinforcing model biases.~~ (3) UltraChat quality (rouge-l=0.08) is identical whether using in-domain or GSM8K datastores.



**Retrieval Quality Spectrum.** Combining cross-domain results with existing data and Round 1's scaling analysis:

(a) Matched ID (highest quality): 1.93–2.69×, accuracy close to Vanilla. (b) Matched OOD (moderate quality): competitive speedup across Table 1. (c) Cross-domain (low quality): 1.94–1.97×, accuracy preserved or improved. (d) 25% datastore (reduced scale): ~88% of full-scale TPS from Round 1 scaling experiments.

**Speedup degrades gracefully and never falls below 1×.** Two mechanisms explain this: (1) C_d's dynamic cache accumulates task-relevant context on-the-fly regardless of C_s domain; (2) the entropy gate ($B_{\text{ent}}$) automatically tightens verification when candidates are poor—this is why cross-domain accuracy can match or exceed in-domain.

**On embedding alignment and acceptance length.** When C_s embeddings are domain-mismatched, semantic retrieval returns lower-quality candidates. However, composite scoring (Eq. 8, α≫β) deprioritizes them, and SE's cascaded verification filters incompatible tokens. The net effect is moderate acceptance length decrease (τ ≈ 2.2 cross-domain vs ≈ 2.3 in-domain) while speedup is maintained because retrieval overhead is negligible (<4% of latency).

**Formal guarantee (for completeness).** We also established formal accuracy bounds: at any confident position ($\tilde{H}(p_t) \leq \gamma$), SENSE outputs exactly the greedy token (proof: $B_{\text{ent}}=0 \Rightarrow G=B_{\text{em}}$). At uncertain positions, accepted tokens reside in top-$k$ with bounded log-regret. Average sequence regret $\leq \rho(\gamma) \cdot \max\log(p^{(1)}/p^{(k)})$. As $\gamma \to 1$, SENSE recovers lossless SD. Table 5's hyperparameter sweep empirically validates this Pareto surface.

We will incorporate these analyses into the revised manuscript.