## Reviewer awXN

We thank the reviewer for the constructive feedback and address each point below.

**Q1: Speedup definition.** All speedups are end-to-end wall-clock ratios (Vanilla time / SENSE time) on RTX 4090 under identical conditions (greedy, temperature=0, FP32). Every overhead is included: FAISS retrieval (CPU), Loose Trie construction, tree-attention verification, and KV cache updates (GPU). Figure 7 decomposes per-step latency into Draft Time and Verify Time, confirming no component is excluded. Table 9 shows runtime memory overhead (Delta RSS) is only 2.76–19.89 MB.

**Q2: Datastore scale and cost.** Table 8 reports sizes from 105 MB (Llama2-7B/GSM8K) to 3,690 MB (Qwen2.5-14B/CodeAlpaca). C_d starts empty and grows on-the-fly (Alg. 1, L28). OOD construction is near-zero cost (indexing training-set labels). ID construction requires one offline inference pass—O(N) forwards, no backpropagation—versus EAGLE's O(E×N) forward+backward, amortized over unlimited runs. Size sensitivity: GSM8K has the smallest datastores (105–406 MB) yet achieves 2.94× (Qwen2.5-7B) and 2.73× (Qwen3-14B). C_d compensates small C_s by accumulating context during generation.

补充我们在gsm8k上的实验，注入我们对于后续改进的承诺。

**Q3 & L2: Long-context.** We will add >8K evaluations (e.g., LongBench) with RAPID comparison in revision. Structurally, SENSE is advantaged for long contexts: C_d grows richer with longer generation; Loose Trie compression (73.4%, Figure 8) yields greater savings with larger candidate pools; retrieval-based drafting avoids generative drafters' quadratic attention cost on long sequences.

补充我们在长上下文中的实验，注入我们对于后续改进的承诺。

**L1: Novelty.** We will adopt more precise language. However, we want to clarify that SENSE is not "DReSD retrieval + FLY verification".

DReSD retrieves diverse semantic candidates but verifies via exact match. Table 1 shows this consistently fails (speedup often ≈1×): dense retrieval introduces lexically divergent candidates that exact match rejects at the first token, turning retrieval into pure overhead. FLY relaxes verification for traditional drafters where mismatches are sparse along single linear paths. Naively combining them fails because dense retrieval produces systematically divergent candidates, violating FLY's assumption that mismatches are rare.

从RSD与SD的差异入手。核心部分需要绍文哥把关了。🙌

SENSE's components are co-designed for this coupled problem: (1) SEN's composite scoring (Eq. 8, α≫β) prioritizes exact-match candidates and introduces semantic ones only as supplements—retrieval-side awareness of downstream SE that DReSD lacks. (2) SE's B_con computes local error density via convolution—a continuous measure for dense retrieval's unpredictable mismatch patterns—unlike FLY's binary sequential lookahead. B_con and B_topk form a dual-pathway rescue (Eq. 17: OR): distributional validity captures synonym equivalence; structural robustness captures isolated deviations. Neither FLY nor ARC offers this combination. (3) Loose Trie compresses N unstructured sequences into a dynamic-topology tree in O(α·M)—absent in generative SD where candidates inherently share prefixes.

Table 3 confirms multiplicative coupling: on Qwen3-8B OOD (mean), removing SEN costs 0.35× and removing SE costs 0.32×; their sum (0.67) nearly doubles the gain over the weakest ablation (0.35), evidencing super-linear interaction rather than independent stacking.

**L3: Baseline speedup < 1.** This is a well-documented SD phenomenon. Leviathan et al. (ICML 2023) derive Speedup=(1−α^{γ+1})/((1−α)(cγ+1)); low α or high c yields speedup<1. Judge Decoding (ICLR 2025) showed even GPT-4o drafts saturate under strict matching. SmartSpec (Liu et al., 2024) was built specifically to address this issue. In our setup: (1) SpS uses MicroLlama 300M against 7B+ targets—the extreme capacity gap makes draft overhead a net cost; (2) DReSD/REST exact-match yields near-zero hit rates in domains like math reasoning. All baselines use their reported optimal hyperparameters (Section E.3). These failures precisely motivate SENSE.

**L4: Theoretical guarantees.** Relaxed SD has rigorous foundations: Yin et al. (NeurIPS 2024, Thm 5) proved a linear Pareto front between rejection probability and distributional bias; DIVERSED (NeurIPS 2025 Workshop, Lemma 3.2) proved ensemble relaxation characterizes the Pareto-optimal tradeoff. SENSE's θ_e traverses this front: θ_e=0.30 degenerates to exact matching (Table 5: Acc=0.93, Spd=2.2×); θ_e=0.05 admits bounded relaxation only at high uncertainty. Concurrent MARS (2026) independently validates this via logit margin. We will incorporate this discussion in revision.

