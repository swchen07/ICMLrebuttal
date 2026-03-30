We thank the reviewer for the constructive feedback and address each point below.

**Q1: Speedup definition.** All speedup results are based on end-to-end throughput (TPS) measured on 4 NVIDIA RTX4090. The TPS is calculated by dividing the total number of generated tokens by the wall-clock time, which is strictly measured from the query input until the completion of decoding. Measurements through all methods are conducted under identical conditions (greedy search, temperature = 0, FP32). Every source of overhead—including FAISS retrieval (CPU), Loose Trie construction, tree-attention verification, and KV cache updates (GPU)—is fully accounted for in the total time.

**Q2: Datastore scale and cost.** 


The datastore in SENSE consists of two components: a static corpus $C_s$ and a dynamic cache $C_d$. As specified in Alg. 1 (L28), $C_d$ starts empty and grows on-the-fly by accumulating context during generation, requiring no offline cost. The scale of $C_s$ (ID method) for our primary datasets is summarized below:
|GSM8K|CodeAlpaca|Ultrachat|TriviaQA|Longbenchv2|
|-|-|-|-|-|
|2.1M|16M|15M|5M|1.7M|

Consistent with existing retrieval-based draft methods such as REST and DReSD, SENSE’s datastore construction is a one-time preprocessing overhead. The table below details the end-to-end construction time for Longbench-v2 (1.7M entries):

|Embedding Data for PCA|Training PCA|Generate Embedding|Faiss Indexing|Total|
|-|-|-|-|-|
|21m49s|8m46s|28m23s|3m24s|62m22s|

Once built, the datastore is stored on disk and reused across all subsequent inference runs. The amortized cost per request is thus negligible, making SENSE highly practical for real-world deployment.


To assess the sensitivity of SENSE to the scale of $C_s$, we conducted a scaling analysis by randomly subsampling the datastore from 25% to 100%:

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

Performance is robust across different scales and low retrieval overhead. In summary, the construction of SENSE is efficient, its overhead is fully amortized, and its performance remains high across a wide range of datastore scales.


**Q3 & L2: Long-context.** As highlighted in the excellent concurrent work $RAPID$, processing long contexts significantly bottlenecks the draft generation time of standard draft models. Guided by this insight, $RAPID$ utilizes RAG techniques to compress the long context into queries for the draft model, thereby substantially reducing its decoding time.

Conceptually, our $SENSE$ fundamentally relies on dense retrieval acting as the drafting mechanism and shares a similar structural advantage. In our framework, the drafting (retrieval) time does not scale drastically with context length. This efficiency stems from two factors:
1. The pre-constructed external datastore ($C_s$) remains constant in size, ensuring theoretically stable retrieval latency regardless of the input prompt;
2. While the dynamically constructed context datastore ($C_d$) grows linearly with the prompt length, its search overhead remains orders of magnitude lower than the standard auto-regressive inference cost of an LLM.

To provide positive empirical evidence as requested, we further evaluated $SENSE$ on the challenging LongBench-v2 dataset. We randomly sampled 200 instances to build the datastore and used the remainder as the test set.

Using Qwen3-8B as the base model, and adopting the same truncation strategy as $RAPID$ to test both 8k and 16k context lengths, we evaluated $SENSE$ under both In-Domain (ID) and Out-of-Domain (OOD) settings and other speculative decoding baseline. As shown in the table below, $SENSE$ consistently maintains a clear acceleration advantage over the vanilla baseline in long-context tasks.

||longbenchv2(8k)-$a$|longbenchv2(8k)-$\tau$|longbenchv2(8k)-$s$|longbenchv2(16k)-$a$|longbenchv2(16k)-$\tau$|longbenchv2(16k)-$s$|
|-|-|-|-|-|-|-|
|vanilla|0.25|-|-|0.36|-|-|
|PLD|0.25|1.73|1.31|0.34|1.86|1.85|
|DReSD|0.25|1.37|1.38|0.36|1.45|1.90|
|SENSE(OOD)|0.23|1.89|1.50|0.36|1.60|1.92|
|SENSE(ID)|0.25|3.10|2.36|0.37|2.29|2.34|

($\tau$ is accept length, $s$ is speed up rate, $a$ is accuracy rate.)

**L1: Novelty.** SENSE resolves the "Interface Mismatch" in existing Retrieval-based Speculative Decoding (RSD): strict exact-match constraints completely reject 78% of retrieved candidates, wasting retrieval overhead and degenerating the process into standard autoregressive decoding.
Our co-design bridges this gap via three pillars: First, unlike DReSD, SENSE utilizes a Hybrid Datastore and a composite scoring mechanism (Eq. 8, $\alpha \gg \beta$) that prioritizes exact-matches while introducing semantic candidates as "rescuable" supplements; Second, SE’s Convolutional Confidence ($B_{con}$) provides a parallelizable, continuous measure of local error density. Combined with $B_{topk}$ (Eq. 17), it forms a dual-pathway rescue that captures both synonym equivalence and structural deviations;
Third, To optimize the primary overhead in decoding and verification, we introduce the Loose Trie. It compresses $N$ unstructured retrieved sequences into a dynamic-topology tree, significantly reducing memory footprint and traversal costs.
Table 3 confirms that SEN and SE are not merely additive. On Qwen3-8B OOD, removing SEN or SE costs 0.35$\times$ and 0.32$\times$ respectively, whereas their combination yields a 0.67 gain. This super-linear interaction (nearly double the individual gains) proves the necessity of our "retrieval-verification" co-design.

**L3: Baseline speedup < 1.** This is a well-documented SD phenomenon. Leviathan et al. (ICML 2023) derive Speedup=(1−α^{γ+1})/((1−α)(cγ+1)); low α or high c yields speedup<1. Judge Decoding (ICLR 2025) showed even GPT-4o drafts saturate under strict matching. SmartSpec (Liu et al., 2024) was built specifically to address this issue. In our setup: (1) SpS uses MicroLlama 300M against 7B+ targets—the extreme capacity gap makes draft overhead a net cost; (2) DReSD/REST exact-match yields near-zero hit rates in domains like math reasoning. All baselines use their reported optimal hyperparameters (Section E.3). These failures precisely motivate SENSE.

**L4: Theoretical guarantees.** Relaxed SD has rigorous foundations: Yin et al. (NeurIPS 2024, Thm 5) proved a linear Pareto front between rejection probability and distributional bias; DIVERSED (NeurIPS 2025 Workshop, Lemma 3.2) proved ensemble relaxation characterizes the Pareto-optimal tradeoff. SENSE's θ_e traverses this front: θ_e=0.30 degenerates to exact matching (Table 5: Acc=0.93, Spd=2.2×); θ_e=0.05 admits bounded relaxation only at high uncertainty. Concurrent MARS (2026) independently validates this via logit margin. We will incorporate this discussion in revision.