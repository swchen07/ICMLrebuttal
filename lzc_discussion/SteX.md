We thank the reviewer for the specific follow-up questions.

**On W1 (ARC/FLY accuracy).** All three verification methods (ARC, FLY, SE) operate on the same target model's probability distribution through the identical SEN retrieval module. By construction, all three constrain accepted tokens to high-probability regions of $p_t$: ARC via JSD risk bounds, FLY via entropy-gated lookahead, SE via Eq. 17's cascaded masks. Their accuracy profiles are therefore structurally similar—the key differentiator is speedup, not quality.

LLM as Judge 表格

We further provide formal justification. At any position where the model is confident ($\tilde{H}(p_t) \leq \gamma$), SE reduces to exact matching (proof: $B_{\text{ent}}=0 \Rightarrow G = B_{\text{em}}$ in Eq. 17)—identical behavior to strict verification. When the model is uncertain ($\tilde{H} > \gamma$), accepted non-greedy tokens must lie in top-$k$, with log-regret $\leq \log(p^{(1)}/p^{(k)})$. Average sequence regret is bounded by $\rho(\gamma) \cdot \max\log(p^{(1)}/p^{(k)})$, where $\rho$ is the fraction of high-entropy positions. As $\gamma \to 1$ or $k \to 1$, SENSE recovers lossless SD. This guarantee applies equally to SE, ARC, and FLY since all operate on the same $p_t$—our claim is not that SE has better accuracy, but that it achieves comparable accuracy at higher speedup.

To provide additional empirical evidence, we conducted cross-domain experiments on Qwen3-8B (Vanilla GSM8K acc=0.91, 13.30 TPS):

表格数据

Speedup is stable at 1.93–1.97× even under complete domain mismatch, while GSM8K cross-domain accuracy (0.87) actually exceeds in-domain (0.83). The entropy gate adapts automatically, tightening verification when candidates are poor. We will add the full ARC/FLY accuracy comparison in revision.



**On Q1/Q4 consistency.** We apologize for the imprecise phrasing. To clarify:

SENSE(ID) uses the target model's own generated responses as the datastore. No ground-truth labels or external knowledge are involved—this is strictly speculative decoding. The statement "zero external knowledge" in Q4 applies specifically to this setting, which is our recommended deployment configuration.

SENSE(OOD) uses training-set ground-truth labels, which does introduce external information into the candidate pool. It is not purely SD in the strictest sense—the occasional outperformance of Vanilla (Table 6) reflects this external signal. We presented OOD as an analytical setting to study datastore-model alignment (Appendix D.1), not as the primary configuration.

We acknowledge this distinction should have been stated more clearly and will correct it in revision.