1. 修改Tree Novelty这部分的动机。强调在retrieve base下的改进必要性。参考4分内容。

## Reviewer SteX

We thank the reviewer for the detailed evaluation and address each concern below.

**W1: Metric Fairness.** We understand this concern but note that it rests on an assumption that does not hold in SD's conditional framework. SD is Markovian: P^n conditions on all previously accepted tokens (Eq. 4). An incorrect acceptance at position i shifts h{i+1} away from the target distribution, causing cascading rejection of subsequent tokens. Table 5 confirms: at θ_e=0.01, k=10, accuracy collapses to 16%—naive relaxation destroys quality rather than accelerating.

Our design is grounded in the Pareto front by Yin et al. (NeurIPS 2024, Thm 5): lossy SD admits bounded output bias TV(P_A,q), yielding acceptance 1−TV(p,q)+TV(P_A,q). The front is linear—each unit of bias buys one unit of rejection reduction. SENSE's θ_e controls the operating point: θ_e=0.30 → exact match (Table 5: Acc=0.93); θ_e=0.05 → bounded bias only at high-entropy positions where the model assigns mass to multiple alternatives.

Furthermore, Table 4 benchmarks SE against ARC and FLY—both equally relaxed. Under identical retrieval, SE outperforms both on most models (16–22% gains). If relaxation trivially inflated metrics, ARC/FLY would match SE. Relaxed SD is a peer-reviewed direction: Judge Decoding (ICLR 2025) concluded strict matching is the bottleneck; 15+ concurrent works explore this paradigm.

**W2: Datastore Dependency.** Table 1 covers four heterogeneous domains (math, code, dialogue, QA) with positive speedup across all. Table 8 shows strong results even on the smallest datastores (GSM8K: 105–406 MB → 2.73–2.94× on Qwen). Two robustness mechanisms: (1) C_d accumulates hidden states on-the-fly (Alg. 1, L28), building a domain-adapted buffer even when C_s is mismatched; (2) Eq. 8's composite scoring (α≫β) falls back to standard RSD when semantic candidates are unavailable—SENSE never underperforms traditional RSD.

**W3: Generation Quality.** Quality preservation under relaxed SD is documented: Judge Decoding (ICLR 2025) reports near-perfect preservation; MARS (2026) maintains accuracy parity; FSD (2025) matches SD accuracy on GSM8K at 3–4 additional tok/s—consistent with Pareto front theory.

SENSE's cascaded safeguards (Eq. 17) are more conservative than single-criterion methods: B_ent gates relaxation to high-uncertainty positions; B_topk restricts to the high-probability nucleus; B_con verifies local consistency. Table 6: 98.02% average accuracy; Appendix D.5: 100% output fidelity on 2,056 tokens (byte-identical to Vanilla, 2.78× faster); Appendix D.6: failures originate from the base model, not SE. We will add Spec-Bench in revision.

**W4: Tree Novelty.** We appreciate the chance to clarify. EAGLE-2's candidates share prefixes inherently through autoregressive expansion—tree construction is trivial. SENSE retrieves N independent, unstructured sequences with no prefix relationship. Sorted-LCP Alignment (Alg. 3) discovers and compresses accidental prefix sharing on-the-fly, reducing complexity from O(N×M²) to O(α·M). Without this, verifying N dense-retrieved candidates requires N independent forward passes, negating speedup. This problem is unique to dense-retrieval SD.

**Q1: Hybrid Datastore.** C_s uses training-split data exclusively (Table 11, p.27); no evaluation data enters any datastore. C_d starts empty and grows as tokens are accepted (Alg. 1, L28). ID datastores contain the target model's own responses on training prompts—not ground-truth answers.

**Q2: τ Computation.** τ counts all accepted tokens identically regardless of pathway—shared by ARC/FLY which also use relaxed criteria. Once accepted, a token enters the KV cache (Alg. 1, L24); the model cannot distinguish its pathway. End-to-end wall-clock speedup confirms τ gains translate to real acceleration.

**Q3: Figure 5.** Relative Performance = Metric_SENSE / Metric_Vanilla (accuracy for GSM8K/TriviaQA; ROUGE-L for CodeAlpaca/UltraChat). Values correspond to Table 6. We will state this formula explicitly in revision.

**Q4: SENSE vs. RAG.** SENSE is strictly SD. SENSE(ID) uses the target model's own responses with zero external knowledge. Occasional Vanilla outperformance is not unique: SpecReason (Pan et al., 2025), a purely generative SD without external knowledge, also reports accuracy improvements—this arises because relaxed verification can accept high-probability alternatives outperforming the greedy choice, not from knowledge injection.
