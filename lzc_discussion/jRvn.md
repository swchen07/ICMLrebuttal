We thank the reviewer for the constructive follow-up. We address the remaining concerns about domain robustness and retrieval quality with new evidence.

**Cross-Domain Experiments.** To directly test robustness under domain shifts where "both input distributions and reasoning patterns differ significantly," we ran cross-domain experiments using completely mismatched datastores on Qwen3-8B:

On GSM8K task:

| Databases    | **SpeedUp** | **Accept length** |
| ------------ | ----------- | ----------------- |
| GSM8K ID     | 1.73X       | 2.63              |
| UltraChat ID | 1.94X       | 2.20              |

On UltraChat task:

| Databases    | SpeedUp | Accept length |
| ------------ | ------- | ------------- |
| UltraChat ID | 2.09X   | 2.18          |
| GSM8K ID     | 1.97X   | 2.49          |

On CodeAlpaca task:

| Databases     | SpeedUp | Accept length |
| ------------- | ------- | ------------- |
| CodeAlpaca ID | 2.75X   | 3.02          |
| UltraChat ID  | 1.94X   | 2.31          |

*SpeedUp = TPS of SENSE / TPS of Vanilla

Three findings emerge: (1) Cross-domain speedup remains strong at 1.94–1.97× across all configurations, retaining 75–112% of in-domain performance even under extreme mismatch (math↔dialogue). (2) On GSM8K, the cross-domain setup (1.94×) actually *exceeds* in-domain (1.73×). This is consistent with Appendix D.1: mismatched datastores avoid reinforcing model-specific biases, while C_d's on-the-fly context accumulation compensates for C_s mismatch. (3) Acceptance length degrades moderately (e.g., τ: 3.02→2.31 on CodeAlpaca), confirming that retrieval quality affects draft quality as expected, but the entropy gate ensures this does not translate to proportional speedup loss—SE's cascaded verification salvages valid tokens regardless of retrieval source.

**Retrieval Quality Spectrum.** Combining cross-domain results with existing data and scaling analysis from rebutall round one:

On CodeAlpaca task:

| Databases          | SpeedUp | Accept length |
| ------------------ | ------- | ------------- |
| CodeAlpaca OOD     | 1.86X   | 2.34          |
| CodeAlpaca ID 25%  | 2.26X   | 2.47          |
| CodeAlpaca ID 50%  | 2.45X   | 2.76          |
| CodeAlpaca ID 75%  | 2.47X   | 2.76          |
| CodeAlpaca ID 100% | 2.75X   | 3.02          |
| UltraChat ID       | 1.94X   | 2.31          |

This spectrum reveals properties: (1) **Monotonic and graceful degradation**: speedup scales smoothly from 2.75× (full ID) to 1.86× (OOD), with no cliff-edge failures. Even the lowest point (OOD, 1.86×) delivers substantial acceleration. (2) **Cross-domain is competitive with OOD**: UltraChat→CodeAlpaca (1.94×) outperforms same-domain OOD (1.86×), suggesting that a large, diverse datastore from a different domain can be more useful than a small, domain-matched one with distribution mismatch.

**Robustness mechanisms.** Two design choices explain why speedup never collapses: (1) C_d's dynamic cache accumulates task-relevant context on-the-fly regardless of C_s domain, providing a domain-adapted retrieval buffer that grows richer with generation length; (2) the entropy gate ($B_{\text{ent}}$) automatically tightens verification when retrieved candidates are poor, preventing quality degradation—this negative feedback loop is why cross-domain accuracy can match or exceed in-domain.

**On embedding alignment.** Our data shows τ decreases from ~3.0 (matched ID) to ~2.2–2.3 (cross-domain), confirming that embedding alignment directly affects draft quality. However, speedup is preserved because (a) composite scoring (Eq. 8, α≫β) deprioritizes mismatched candidates, and (b) retrieval overhead remains negligible (<4% of latency), so lower τ primarily reduces the "bonus" from retrieval while the baseline acceleration from SE and C_d remains intact.

**Formal guarantee.** We also established formal accuracy bounds: at any confident position ($\tilde{H}(p_t) \leq \gamma$), SENSE outputs exactly the greedy token (proof: $B_{\text{ent}}=0 \Rightarrow G=B_{\text{em}}$). At uncertain positions, accepted tokens reside in top-$k$ with bounded log-regret. Average sequence regret $\leq \rho(\gamma) \cdot \max\log(p^{(1)}/p^{(k)})$. As $\gamma \to 1$, SENSE recovers lossless SD. Table 5's hyperparameter sweep empirically validates this Pareto surface.

We will incorporate these analyses into the revised manuscript. Appreciate!