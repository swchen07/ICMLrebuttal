## Reviewer jRvn

We sincerely thank the reviewer for the thoughtful evaluation and address each point below.

**W1 & Q2: Datastore analysis.** We agree a more systematic study would strengthen the paper and provide analysis from existing data.

*Domain mismatch.* Our ID vs. OOD setup is itself a controlled mismatch experiment—OOD datastores use ground-truth labels differing from the model's generation distribution. Table 1 shows SENSE(OOD) still outperforms all other RSD baselines on most models across four heterogeneous domains.

*Datastore size.* Table 8 shows sizes from 105 MB to 3,690 MB. SENSE achieves strong speedups even on the smallest datastores (GSM8K: 268 MB on Qwen2.5-7B → 2.94×; 406 MB on Qwen3-14B → 2.73×). C_d compensates limited C_s by accumulating in-context hidden states on-the-fly (Alg. 1, L28) at zero construction cost.

datastore scaling 实验补充

**W2 & Q1: Data separation.** We confirm strict separation. Table 11 (Appendix E, p.27) documents all 24 configurations: all datastores use training splits exclusively; all evaluations use test/validation splits. ID datastores contain the target model's own generated responses on training prompts—not ground-truth answers. C_d starts empty, accumulating only from the current generation (Alg. 1, L28). No evaluation data enters any datastore. We will make this prominent in the main text.

**W3: Novelty.** We appreciate this fair characterization. The integration is non-trivial because the components are not independently useful—they form a coupled design.

SENSE is not "DReSD retrieval + FLY verification"—these address incompatible settings. DReSD retrieves diverse candidates but rejects them via exact match (Table 1: speedup often ≈1×)—diversity is wasted. FLY relaxes verification for traditional drafters with sparse mismatches. Naively combining them fails: dense retrieval produces systematically divergent candidates, violating FLY's assumption of rare mismatches.

SENSE's co-design: (1) SEN's composite scoring (Eq. 8, α≫β) controls candidate risk exposure—prioritizing exact-match candidates, introducing semantic ones only as supplements. DReSD lacks this retrieval-side awareness. (2) SE's B_con computes local error density via convolution (continuous measure), unlike FLY's binary lookahead, designed for dense retrieval's unpredictable patterns. B_con and B_topk form a dual-pathway rescue (Eq. 17) absent in FLY or ARC. (3) Loose Trie (Alg. 3) compresses N unstructured results into a dynamic-topology tree in O(α·M)—absent in generative SD. Additionally, the Orchestration Framework (Section 3.4) provides standardized atomic primitives for future SD research.

Table 3 confirms synergy: on Qwen3-8B OOD (mean), ablating SEN costs 0.35× and SE costs 0.32×; their sum (0.67) nearly doubles the gain over the weakest ablation (0.35)—multiplicative interaction, not stacking.

**Theoretical grounding.** Relaxed SD has rigorous foundations: Yin et al. (NeurIPS 2024, Thm 5) proved a linear Pareto front between rejection probability and distributional bias. SENSE's θ_e traverses this front (θ_e=0.30 → exact match; θ_e=0.05 → bounded relaxation at high uncertainty). Judge Decoding (ICLR 2025) and 15+ concurrent works establish this as a mainstream direction. Table 5 empirically maps SENSE's operating points on this front.

**Q3: Runtime breakdown.** Figure 7 decomposes per-step latency into Draft Time (retrieval + Trie) and Verify Time (forward pass + KV update). SENSE's Draft Time is far lower than generative baselines since FAISS IVF-PQ runs in milliseconds. Verify Time benefits from Loose Trie's 73.4% compression (Figure 8). Table 9 confirms minimal memory overhead (Delta RSS: 2.76–19.89 MB). Speedup derives from both minimal drafting cost and higher acceptance rates.

**Revision commitments:** (1) Prominent data separation in main text; (2) datastore scaling experiments; (3) clearer system detail presentation.