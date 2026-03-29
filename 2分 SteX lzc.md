Weaknesses:

1. Inequitable Baseline Comparisons (Metric Fairness): The proposed method employs a relaxed verification standard (Soft-gated Evaluation) compared to the strict, exact-match criteria used by standard baselines. Because the acceptance threshold is inherently looser, it is an expected mathematical outcome that SENSE will achieve higher speedups and longer mean acceptance lengths. Comparing these specific metrics directly against baselines with stricter verification rules may artificially inflate the perceived effectiveness of the method.
2. Datastore Dependency and Domain Generalization: While a foundational target LLM possesses broad, cross-domain generation capabilities, the acceleration performance of this retrieval-based approach is bottlenecked by the external datastore. If the datastore lacks comprehensive coverage across diverse fields, the framework's effectiveness could be severely limited, thereby restricting the model's generalizability in open-domain scenarios.
3. Insufficient Proof of Generation Quality: Because the framework intentionally relaxes the verification standard to accept semantically similar tokens, there is a significant risk of degrading the final generation quality. While the authors present accuracy preservation data in Figure 5, this limited evaluation is not comprehensive enough to definitively prove the robustness of the approach. The paper requires evaluation across a wider, more rigorous set of downstream benchmarks to guarantee generation quality of the draft model (e.g. Spec-Bench)
4. Limited Novelty: The concept of constructing a tree structure to verify multiple draft branches in a single forward pass has already been established in prior literature, such as the EAGLE-2 framework.

**Key Questions For Authors:**

1. Clarity on Hybrid Datastore Construction:Could the authors provide more details regarding the construction of the hybrid datastore (C_db)? Specifically, what exact source texts were utilized to populate the static corpus (C_s), and how is the dynamic context buffer (C_d) updated?
2. Metric Equivalency for Mean Accepted Tokens: Given that the Soft-gated Evaluation module deliberately employs a relaxed acceptance criterion, how exactly is the Mean Accepted Tokens metric computed? Is a token accepted via the relaxed criteria counted identically to a strict exact-match token? If so, how do the authors ensure this is a fair comparison against baselines that strictly enforce exact matching?
3. Specification of Quality Metrics in Figure 5: How is the relative performance (generation quality) mathematically computed in Figure 5?
4. In standard speculative decoding, the native target model's performance serves as a strict mathematical upper bound. However, Table 6 shows the proposed method occasionally outperforming the Vanilla target model. Does this indicate that SENSE is functionally acting as a Retrieval-Augmented Generation (RAG) system? If so, does this approach highly relies on the datastore's quality?



# Rebuttal to Reviewer SteX

We thank the reviewer and clarify several points below.

**W1: Metric Fairness.** The claim that relaxed acceptance necessarily yields higher speedup holds for i.i.d. sequences but breaks in SD's conditional framework. SD is Markovian: P^n conditions on all previously accepted tokens (Eq. 4). An incorrect acceptance at position i shifts h{i+1} away from the target distribution, causing cascading rejection of all subsequent tokens. Table 5 confirms: θe=0.01, k=10 collapses accuracy to 16%—naive relaxation destroys output, not accelerates it.

Our design is grounded in the Pareto front by Yin et al. (NeurIPS 2024, Thm 5). For draft distribution p and target q, standard SD achieves acceptance probability 1−TV(p,q). Lossy SD admits bounded output bias TV(P_A,q), yielding acceptance 1−TV(p,q)+TV(P_A,q). The front is linear: each unit of bias buys exactly one unit of rejection reduction. SENSE's θ_e controls the operating point: θ_e=0.30 → TV(P_A,q)≈0 (exact match, Table 5: Acc=0.93, Spd=2.2×); θ_e=0.05 → bounded bias at high-entropy positions only, where the model itself assigns mass to multiple alternatives. This is a theoretically grounded tradeoff, not unprincipled relaxation.

Table 4 benchmarks SE against ARC and FLY—both equally relaxed. Under identical retrieval, SE outperforms both on most models (16–22% gains). Relaxed SD is mainstream: Judge Decoding (ICLR 2025) concluded strict matching is the bottleneck; Yin et al. (NeurIPS 2024) established the Pareto theory; 15+ concurrent works explore this direction.

**W2: Datastore Dependency.** Table 1 covers four diverse domains with positive speedup across all. Two robustness mechanisms: (1) C_d accumulates hidden states on-the-fly (Alg. 1, L28), building a domain-specific buffer even when C_s is mismatched; (2) Eq. 8's composite scoring (α≫β) falls back to standard RSD when semantic candidates are unavailable—SENSE never underperforms traditional RSD.

**W3: Generation Quality.** Quality preservation under relaxed SD is documented: Judge Decoding (ICLR 2025) reports near-perfect preservation; MARS (2026) maintains accuracy parity; FSD (2025) matches SD accuracy on GSM8K at 3–4 additional tok/s—all consistent with Pareto front theory.

SENSE's cascaded safeguards (Eq. 17) are more conservative than single-criterion methods: B_ent gates relaxation to high-uncertainty positions; B_topk restricts to the high-probability nucleus; B_con verifies local consistency. Table 6: 98.12% average accuracy; Appendix D.5: 100% output fidelity on 2,056 tokens (byte-identical to Vanilla, 2.78× faster); Appendix D.6: failures originate from the base model, not SE. We will add Spec-Bench in revision.

**W4: Novelty (Tree).** This conflates two different problems. EAGLE-2's candidates share prefixes inherently through autoregressive expansion—tree construction is trivial. SENSE retrieves N independent, unstructured sequences with no prefix relationship. Sorted-LCP Alignment (Alg. 3) compresses accidental prefix sharing on-the-fly, reducing complexity from O(N×M²) to O(α·M). This problem does not exist in generative SD; SENSE's topology is dynamically recomputed per step, unlike EAGLE's static structure.

**Q1: Hybrid Datastore.** C_s uses training-split data exclusively (Table 11, p27); no evaluation data enters any datastore. C_d starts empty and grows as tokens are accepted (Alg. 1, L28). ID datastores contain the target model's own responses on training prompts—not ground-truth answers.

**Q2: τ Computation.** τ counts all accepted tokens identically regardless of pathway—shared by ARC/FLY which also use relaxed criteria. Once accepted, a token enters the KV cache (Alg. 1, L24); the model cannot distinguish its pathway. End-to-end wall-clock speedup validates that τ gains translate to real acceleration.

**Q3: Figure 5.** Relative Performance = Metric_SENSE / Metric_Vanilla. For GSM8K and TriviaQA, Metric is exact-match accuracy; for CodeAlpaca and UltraChat, Metric is ROUGE-L. A value of 1.0 means identical performance to vanilla. The underlying values correspond to Table 6 (e.g., Qwen2.5-14B GSM8K: 0.89/0.95=0.937 for ID, 0.95/0.95=1.0 for OOD). We will state this formula explicitly in revision.

**Q4: SENSE vs. RAG.** SENSE is strictly SD. SENSE(ID) uses the target model's own responses—zero external knowledge. Occasional Vanilla outperformance is not unique: SpecReason (Pan et al., 2025), a purely generative SD without external knowledge, also reports accuracy improvements. This arises because relaxed verification can accept high-probability alternatives outperforming the greedy choice, not from knowledge injection.