We sincerely thank the reviewer for the thoughtful evaluation and address each point below.

**W2 & Q1: Data separation.** We confirm strict separation. Table 11 (Appendix E, p.27) documents all 24 configurations: all datastores use training splits exclusively; all evaluations use test/validation splits. ID datastores contain the target model's own generated responses on training prompts—not ground-truth answers. C_d starts empty, accumulating only from the current generation (Alg. 1, L28). No evaluation data enters any datastore. We will make this prominent in the main text.


**W1 & Q2: Datastore analysis.** We agree a more systematic study would strengthen the paper and provide analysis from existing data.

*Domain mismatch.* Our ID vs. OOD setup is itself a controlled mismatch experiment—OOD datastores use ground-truth labels differing from the model's generation distribution. Table 1 shows SENSE(OOD) still outperforms all other RSD baselines on most models across four heterogeneous domains.

*Datastore size.* As shown in the table below, we conducted a scaling analysis by subsampling the datastore from 25% to 100% to evaluate the influence of datastore size. Benefiting from our Hybrid Datastore Construction, SENSE demonstrates high performance resilience across various scales.

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

**Q3: Runtime breakdown.** All throughput results (TPS) are measured on four NVIDIA RTX 4090 GPUs. TPS is calculated by dividing total generated tokens by the wall-clock time, strictly measured from query input to decoding completion. All methods are evaluated under identical conditions (greedy search, temperature = 0, FP32). Every overhead—including FAISS retrieval (CPU), Loose Trie construction, tree-attention verification, and KV cache updates (GPU)—is fully accounted for.

Figure 7 decomposes per-step latency into Draft Time (retrieval + Trie) and Verify Time (forward pass + KV update). SENSE's Draft Time is significantly lower than generative baselines as FAISS IVF-PQ retrieval completes in milliseconds. Verify Time further benefits from the Loose Trie’s 73.4% compression (Figure 8). Table 9 confirms minimal memory overhead (Delta RSS: 2.76–19.89 MB).

The table above illustrates the latency distribution across various datastore sizes. While retrieval time increases slightly with scale, the primary bottleneck remains in decoding, highlighting a core advantage of retrieval-based speculative decoding: the retrieval overhead is orders of magnitude lower than the generation cost of a draft model.

**W3: Novelty.** We appreciate this fair characterization. The integration is non-trivial because the components are not independently useful—they form a coupled design.
SENSE resolves the "Interface Mismatch" in existing Retrieval-based Speculative Decoding (RSD): strict exact-match constraints completely reject 78% of retrieved candidates, wasting retrieval overhead and degenerating the process into standard autoregressive decoding.

Our co-design bridges this gap via three pillars: First, unlike DReSD, SENSE utilizes a Hybrid Datastore and a composite scoring mechanism (Eq. 8, $\alpha \gg \beta$) that prioritizes exact-matches while introducing semantic candidates as "rescuable" supplements; Second, SE’s Convolutional Confidence ($B_{con}$) provides a parallelizable, continuous measure of local error density. Combined with $B_{topk}$ (Eq. 17), it forms a dual-pathway rescue that captures both synonym equivalence and structural deviations;
Third, To optimize the primary overhead in decoding and verification, we introduce the Loose Trie. It compresses $N$ unstructured retrieved sequences into a dynamic-topology tree, significantly reducing memory footprint and traversal costs.
Table 3 confirms that SEN and SE are not merely additive. On Qwen3-8B OOD, removing SEN or SE costs 0.35$\times$ and 0.32$\times$ respectively, whereas their combination yields a 0.67 gain. This super-linear interaction (nearly double the individual gains) proves the necessity of our "retrieval-verification" co-design.

**Revision commitments:** (1) Prominent data separation in main text; (2) datastore scaling experiments; (3) clearer system detail presentation.