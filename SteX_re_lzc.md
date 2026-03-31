We thank the reviewer for the detailed evaluation and address each concern below.

**W1: Metric Fairness.** We understand this concern but note that it rests on an assumption that does not hold in SD's conditional framework. SD is Markovian: P^n conditions on all previously accepted tokens (Eq. 4). An incorrect acceptance at position i shifts h_{i+1} away from the target distribution, causing cascading rejection of subsequent tokens. Table 5 confirms: at θ_e=0.01, k=10, accuracy collapses to 16%—naive relaxation destroys quality rather than accelerating.

Our design is grounded in the Pareto front by Yin et al. (NeurIPS 2024, Thm 5): lossy SD admits bounded output bias TV(P_A,q), yielding acceptance 1−TV(p,q)+TV(P_A,q). The front is linear—each unit of bias buys one unit of rejection reduction. SENSE's θ_e controls the operating point: θ_e=0.30 → exact match (Table 5: Acc=0.93); θ_e=0.05 → bounded bias only at high-entropy positions where the model assigns mass to multiple alternatives.

Furthermore, Table 4 benchmarks SE against ARC and FLY—both equally relaxed. Under identical retrieval, SE outperforms both on most models (16–22% gains). If relaxation trivially inflated metrics, ARC/FLY would match SE. Relaxed SD is a peer-reviewed direction: Judge Decoding (ICLR 2025) concluded strict matching is the bottleneck; 15+ concurrent works explore this paradigm.

**W2: Datastore Dependency.** Table 1 covers four heterogeneous domains (math, code, dialogue, QA) with positive speedup across all. Two robustness mechanisms mitigate domain mismatch: (1) C_d accumulates hidden states on-the-fly (Alg. 1, L28), capturing session-specific patterns even when C_s is mismatched; (2) Eq. 8's composite scoring (α≫β) falls back to standard RSD when semantic candidates are unavailable—SENSE never underperforms traditional RSD. Table 8 further shows strong results even on the smallest datastores (GSM8K: 105–406 MB → 2.73–2.94× on Qwen), confirming robustness to datastore scale.

**W3: Generation Quality.** Quality preservation under relaxed SD is well-documented across the literature (Judge Decoding/ICLR'25; MARS/2026; FSD/2025), consistent with Pareto front theory. SENSE's cascaded safeguards (Eq. 17: B_ent ∧ (B_topk ∨ B_con)) enforce multi-layered filtering strictly more conservative than single-criterion methods. Our existing evidence already demonstrates this: Table 6 reports 98.12% average accuracy; Appendix D.5 shows 100% output fidelity on 2,056 tokens at 2.78× speedup.

As requested, we further evaluated on Spec-Bench (Qwen3-8B, ID datastore from UltraChat). SENSE achieves 2.32× throughput gain with identical math accuracy. Crucially, LLM-as-Judge (Claude Opus 4.6) yields a quality delta of merely −0.02 on a 1–5 scale (within noise), confirming that SENSE's speedup comes at no measurable cost to generation quality.

| Dimension                                 | Vanilla     | Sense       | Verdict                   |
| ----------------------------------------- | ----------- | ----------- | ------------------------- |
| Throughput (TPS)                          | 5.69 tok/s  | 13.23 tok/s | Sense is 2.32x faster     |
| Math Accuracy (17 Qs)                     | 100%        | 100%        | Identical                 |
| QA Accuracy (12 Qs)                       | 75.0%       | 66.7%       | Vanilla slightly better   |
| Subjective Quality (LLM Judge) -- Quality | 2.33 ± 1.55 | 2.33 ± 1.49 | No significant difference |
| Correctness                               | 2.89 ± 1.71 | 2.79 ± 1.65 | A bit loss                |
| Completeness                              | 2.22 ± 1.57 | 2.25 ± 1.52 | Slightly better           |

QA质量不好这个问题我感觉不太想提出来。掉了一个题目，2分这个人会纠着。表格确实太占字符了。下面这一版本了作参考吧。

> **W3: Generation Quality.** Quality preservation under relaxed SD is well-documented (Judge Decoding/ICLR'25; MARS/2026; FSD/2025), consistent with Pareto front theory. SENSE's cascaded safeguards (Eq. 17: B_ent ∧ (B_topk ∨ B_con)) enforce multi-layered filtering strictly more conservative than single-criterion methods. Table 6: 98.12% average accuracy; Appendix D.5: 100% output fidelity on 2,056 tokens at 2.78× speedup.
>
> As requested, we evaluated on Spec-Bench (96 samples, Qwen3-8B, ID datastore). SENSE achieves 2.32× throughput (13.23 vs 5.69 TPS) with 100% math accuracy (identical to Vanilla on all 17 problems). LLM-as-Judge (Claude Opus 4.6) scores across 63 open-ended samples yield an overall quality delta of −0.02 on a 1–5 scale—within noise, confirming no measurable degradation.

**W4: Tree Novelty.** We appreciate the opportunity to clarify. Prior works (SpecInfer, REST, EAGLE) employ tree attention because their candidates are inherently tree-structured. SENSE's dense retriever fetches N independent, unstructured sequences lacking prefix relationships. Sorted-LCP Alignment (Alg. 3) compresses these into a unified tree on-the-fly, reducing complexity from O(N×M²) to O(α·M). Without this, verifying N candidates requires N independent forward passes, negating speedup. This problem is unique to dense-retrieval SD and does not arise in generative approaches.

**Q1: Hybrid Datastore.** C_s uses training-split data exclusively (Table 11, p.27); no evaluation data enters any datastore. C_d starts empty and grows as tokens are accepted (Alg. 1, L28). ID datastores contain the target model's own responses on training prompts—not ground-truth answers. OOD datastores use training-set labels.

**Q2: τ Computation.** τ counts all accepted tokens identically regardless of pathway—shared by ARC/FLY which also use relaxed criteria. Once accepted, a token enters the KV cache (Alg. 1, L24); the model cannot distinguish its pathway. End-to-end wall-clock speedup confirms τ gains translate to real acceleration.

**Q3: Figure 5.** Relative Performance = Metric_SENSE / Metric_Vanilla (accuracy for GSM8K/TriviaQA; ROUGE-L for CodeAlpaca/UltraChat). Values correspond to Table 6. We will state this explicitly in revision.

**Q4: SENSE vs. RAG.** SENSE is strictly SD. SENSE(ID) uses the target model's own responses with zero external knowledge. Occasional Vanilla outperformance is not unique: SpecReason (Pan et al., 2025), a purely generative SD without external knowledge, also reports accuracy improvements—this arises because relaxed verification can accept high-probability alternatives outperforming the greedy choice, not from knowledge injection.
