We sincerely thank the reviewer for the thoughtful evaluation and address each point below.

**W1 & Q2: Datastore analysis.** We agree a more systematic study would strengthen the paper and provide analysis from existing data.

*Domain mismatch.* Our ID vs. OOD setup is itself a controlled mismatch experiment—OOD datastores use ground-truth labels differing from the model's generation distribution. Table 1 shows SENSE(OOD) still outperforms all other RSD baselines on most models across four heterogeneous domains.

*Datastore size.* Table 8 shows sizes from 105 MB to 3,690 MB. SENSE achieves strong speedups even on the smallest datastores (GSM8K: 268 MB on Qwen2.5-7B → 2.94×; 406 MB on Qwen3-14B → 2.73×). C_d compensates limited C_s by accumulating in-context hidden states on-the-fly (Alg. 1, L28) at zero construction cost.

datastore scaling 实验补充
添加上测试不同规模datastore的结果

CodeAlpaca
|datastore size|TPS|accept length|retrieve time ratio|verify time ratio|
|-|-|-|-|-|
|25%|17.9|2.47|0.019|0.922|
|50%|19.3|2.76|0.025|0.915|
|75%|19.5|2.76|0.032|0.908|
|100%|21.7|2.79|0.041|0.900|

GSM8K
|datastore size|TPS|accept length|retrieve time ratio|verify time ratio|
|-|-|-|-|-|
|25%|18.3|2.58|0.014|0.933|
|50%|18.2|2.35|0.016|0.923|
|75%|18.2|2.38|0.017|0.924|
|100%|20.3|2.45|0.017|0.930|


longbench_8k
|datastore size|TPS|accept length|retrieve time ratio|verify time ratio|
|-|-|-|-|-|
|25%|12.9|2.96|0.009|0.949|
|50%|11.1|2.73|0.008|0.935|
|75%|7.6|1.94|0.008|0.953|
|100%|11.6|3.10|0.007|0.960|

如果字符还剩的话把详细的时间也写进去，不是光比例


**W2 & Q1: Data separation.** We confirm strict separation. Table 11 (Appendix E, p.27) documents all 24 configurations: all datastores use training splits exclusively; all evaluations use test/validation splits. ID datastores contain the target model's own generated responses on training prompts—not ground-truth answers. C_d starts empty, accumulating only from the current generation (Alg. 1, L28). No evaluation data enters any datastore. We will make this prominent in the main text.

**W3: Novelty.** We appreciate this fair characterization. The integration is non-trivial because the components are not independently useful—they form a coupled design.

SENSE is not "DReSD retrieval + FLY verification"—these address incompatible settings. DReSD retrieves diverse candidates but rejects them via exact match (Table 1: speedup often ≈1×)—diversity is wasted. FLY relaxes verification for traditional drafters with sparse mismatches. Naively combining them fails: dense retrieval produces systematically divergent candidates, violating FLY's assumption of rare mismatches.

SENSE's co-design: (1) SEN's composite scoring (Eq. 8, α≫β) controls candidate risk exposure—prioritizing exact-match candidates, introducing semantic ones only as supplements. DReSD lacks this retrieval-side awareness. (2) SE's B_con computes local error density via convolution (continuous measure), unlike FLY's binary lookahead, designed for dense retrieval's unpredictable patterns. B_con and B_topk form a dual-pathway rescue (Eq. 17) absent in FLY or ARC. (3) Loose Trie (Alg. 3) compresses N unstructured results into a dynamic-topology tree in O(α·M)—absent in generative SD. Additionally, the Orchestration Framework (Section 3.4) provides standardized atomic primitives for future SD research.

Table 3 confirms synergy: on Qwen3-8B OOD (mean), ablating SEN costs 0.35× and SE costs 0.32×; their sum (0.67) nearly doubles the gain over the weakest ablation (0.35)—multiplicative interaction, not stacking.

**Q3: Runtime breakdown.** Figure 7 decomposes per-step latency into Draft Time (retrieval + Trie) and Verify Time (forward pass + KV update). SENSE's Draft Time is far lower than generative baselines since FAISS IVF-PQ runs in milliseconds. Verify Time benefits from Loose Trie's 73.4% compression (Figure 8). Table 9 confirms minimal memory overhead (Delta RSS: 2.76–19.89 MB). Speedup derives from both minimal drafting cost and higher acceptance rates.

要不塞在上面一起

**Revision commitments:** (1) Prominent data separation in main text; (2) datastore scaling experiments; (3) clearer system detail presentation.