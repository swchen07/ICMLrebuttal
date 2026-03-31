We thank the reviewer for the constructive feedback and address each point below.

**Q1:** All speedup results are based on end-to-end TPS measured on 4 NVIDIA RTX4090. The TPS is calculated by dividing the total number of generated tokens by the wall-clock time, which is strictly measured from the query input until the completion of decoding. All measurements are conducted under identical conditions (greedy, temperature = 0, FP32). Overhead including FAISS retrieval (CPU), Loose Trie construction, tree-attention verification, and KV cache updates (GPU) is fully accounted for in the total time.

**Q2:** $C_d$ starts empty and grows on-the-fly by accumulating context during generation, requiring no offline cost. Table below is the scale of $C_s$ (ID) for our benchmarks:
|GSM8K|CodeAlpaca|Ultrachat|TriviaQA|Longbenchv2|
|-|-|-|-|-|
|2.1M|16M|15M|5M|1.7M|

Like REST/DReSD, SENSE’s datastore construction is a one-time offline overhead, thus we strictly exclude this preprocessing cost from per-inference latency. As shown below for Longbench-v2 (1.7M entries), the persistent datastore is reused across all runs:
|Embedding Data for PCA|Training PCA|Generate Embedding|Faiss Indexing|Total|
|-|-|-|-|-|
|21m49s|8m46s|28m23s|3m24s|62m22s|

To evaluate SENSE's sensitivity to the $C_s$ scale, we subsampled the datastore from 25% to 100%. As shown below, SENSE maintains robust performance and consistently low retrieval overhead across all scales, confirming both its construction efficiency and strong scalability.
|datastore size|TPS|accept length|retrieve time ratio|verify time ratio|
|-|-|-|-|-|
|CodeAlpaca-25%|17.9|2.47|0.019|0.922|
|50%|19.3|2.76|0.025|0.915|
|75%|19.5|2.76|0.032|0.908|
|100%|21.7|2.79|0.041|0.900|
|GSM8K-25%|18.3|2.58|0.014|0.933|
|50%|18.2|2.35|0.016|0.923|
|75%|18.2|2.38|0.017|0.924|
|100%|20.3|2.45|0.017|0.930|
|LongBenchv2(8k)-25%|12.9|2.96|0.009|0.949|
|50%|11.1|2.73|0.008|0.935|
|75%|7.6|1.94|0.008|0.953|
|100%|11.6|3.10|0.007|0.960|

**Q3 & L2:** As highlighted in the concurrent work RAPID, processing long contexts bottlenecks the draft generation time of standard draft models.
Conceptually, SENSE leverages dense retrieval for drafting, keeping retrieval time robust against context length. This efficiency is twofold: the static $C_s$ ensures constant latency regardless of the prompt, and while the dynamic $C_d$ grows linearly, its search overhead remains orders of magnitude lower than standard autoregressive inference.

We evaluate SENSE on LongBench-v2 dataset. We randomly sampled 200 instances to build the datastore and used the remainder as the test set. Evaluated on Qwen3-8B using RAPID’s 8k/16k truncation strategy, we compared SENSE against SD baselines. As shown below, SENSE consistently maintains a clear advantage in long-context tasks.

||$a$(8k)|$\tau$(8k)|$s$(8k)|$a$(16k)|$\tau$(16k)|$s$(16k)|
|-|-|-|-|-|-|-|
|vanilla|0.25|-|-|0.36|-|-|
|PLD|0.25|1.73|1.31|0.34|1.86|1.85|
|DReSD|0.25|1.37|1.38|0.36|1.45|1.90|
|SENSE(OOD)|0.23|1.89|1.50|0.36|1.60|1.92|
|SENSE(ID)|0.25|3.10|2.36|0.37|2.29|2.34|

($\tau$:accept length, $s$:speed up rate, $a$:accuracy rate.)

**L1:** SENSE resolves the "Interface Mismatch" in Retrieval-based Speculative Decoding: we find that strict exact-match constraints completely reject 78% of retrieved candidates, wasting retrieval overhead and degenerating the process into standard autoregressive decoding.
Our co-design bridges this gap via three pillars: First, SENSE utilizes a hybrid datastore and a composite scoring mechanism that prioritizes exact-match while introducing semantic candidates as "rescuable" supplements; Second, SE’s Convolutional Confidence ($B_{con}$) provides a parallelizable, continuous measure of local error density. Combined with $B_{topk}$ (Eq. 17), it forms a dual-pathway rescue that captures both synonym equivalence and structural deviations;Third, To optimize the primary overhead in decoding and verification, we introduce the Loose Trie. It compresses $N$ unstructured retrieved sequences into a dynamic-topology tree, significantly reducing memory footprint and traversal costs.
Table 3 confirms our co-design's necessity via super-linear interaction: on Qwen3-8B OOD, the combined SEN+SE gain (0.67×) effectively surpasses their individual removal drops (0.35× and 0.32×).

**L3:** This is a well-documented SD phenomenon. Leviathan et al. (ICML 2023) derive Speedup=(1−α^{γ+1})/((1−α)(cγ+1)); low α or high c yields speedup<1. Judge Decoding (ICLR 2025) showed even GPT-4o drafts saturate under strict matching. In our setup: (1) the size and capacity gap of draft model in SpS makes draft overhead a net cost; (2) DReSD/REST exact-match yields near-zero hit rates in domains like math reasoning. All baselines use their reported optimal hyperparameters (Section E.3).

**L4:** Relaxed SD has rigorous foundations: Yin et al. (NeurIPS 2024, Thm 5) proved a linear Pareto front between rejection probability and distributional bias; DIVERSED (NeurIPS 2025 Workshop, Lemma 3.2) proved ensemble relaxation characterizes the Pareto-optimal tradeoff. SENSE's θ_e traverses this front: θ_e=0.30 degenerates to exact matching (Table 5: Acc=0.93, Spd=2.2×); θ_e=0.05 admits bounded relaxation only at high uncertainty. Concurrent MARS (2026) independently validates this via logit margin. We will incorporate this discussion in revision.