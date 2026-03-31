We sincerely thank the reviewer for the thoughtful evaluation and address each point below.

**W2 & Q1: Data separation.** We confirm strict separation. Table 11 (Appendix E, p.27) documents all 24 configurations: all datastores use training splits exclusively; all evaluations use test/validation splits. ID datastores contain the target model's own generated responses on training prompts—not ground-truth answers. C_d starts empty, accumulating only from the current generation (Alg. 1, L28). No evaluation data enters any datastore. We will make this prominent in the main text.

**W1 & Q2: Datastore analysis.** We agree and provide analysis from existing data plus new experiments.

*Domain mismatch.* Our ID vs. OOD setup is itself a controlled mismatch experiment—OOD datastores use ground-truth labels differing from the model's generation distribution. Table 1 shows SENSE(OOD) still outperforms all other RSD baselines on most models across four heterogeneous domains.

*Datastore size.* We conducted a scaling analysis by subsampling C_s from 25% to 100% on Qwen3-8B (ID). On CodeAlpaca, TPS scales from 17.9 (25%) to 21.7 (100%) with acceptance length from 2.47 to 2.79. On GSM8K, TPS remains stable at 18.2–20.3 across all scales. Retrieval consistently accounts for <4% of total latency. These results confirm robust performance across datastore sizes.

*Retrieval recall.* Table 7 reports average acceptance rate of 0.07 (range 0.02–0.35). Despite this sparsity, SENSE achieves significant speedups—high retrieval recall is not required.

**W3: Novelty.** We appreciate this fair characterization. The integration is non-trivial because the components are not independently useful—they form a coupled design.

DReSD retrieves diverse candidates but rejects them via exact match (Table 1: speedup often ≈1×)—diversity is wasted. FLY relaxes verification for traditional drafters with sparse mismatches. Naively combining them fails: dense retrieval produces systematically divergent candidates, violating FLY's rare-mismatch assumption.

SENSE's co-design: (1) SEN's composite scoring (Eq. 8, α≫β) prioritizes exact-match candidates, introducing semantic ones only as supplements—retrieval-side awareness absent in DReSD. (2) SE's B_con computes local error density via convolution (continuous measure), unlike FLY's binary lookahead. B_con and B_topk form a dual-pathway rescue (Eq. 17) absent in FLY/ARC. (3) Loose Trie (Alg. 3) compresses N unstructured results into a dynamic-topology tree in O(α·M)—absent in generative SD. The Orchestration Framework (Section 3.4) further provides standardized atomic primitives for future SD research.

Table 3 confirms synergy: on Qwen3-8B OOD (mean), ablating SEN costs 0.35× and SE costs 0.32×; their sum (0.67) nearly doubles the gain over the weakest ablation (0.35)—multiplicative interaction, not stacking.

**Theoretical grounding.** Relaxed SD has rigorous foundations: Yin et al. (NeurIPS 2024, Thm 5) proved a linear Pareto front between rejection probability and distributional bias. SENSE's θ_e traverses this front (θ_e=0.30 → exact match; θ_e=0.05 → bounded relaxation at high uncertainty). Judge Decoding (ICLR 2025) and 15+ concurrent works establish this as a mainstream direction. Table 5 empirically maps SENSE's operating points on this front.

**Q3: Runtime breakdown.** All TPS is measured on 4×RTX 4090, calculated as total tokens / wall-clock time from query input to completion (greedy, temperature=0, FP32). All overheads included. Figure 7 decomposes per-step latency into Draft Time (retrieval + Trie) and Verify Time (forward + KV update). SENSE's Draft Time is far lower than generative baselines; Verify Time benefits from Loose Trie's 73.4% compression. Retrieval accounts for <4% of total latency—the bottleneck is verification, not retrieval.

**Revision commitments:** (1) Prominent data separation in main text; (2) datastore scaling analysis; (3) clearer system detail presentation.
