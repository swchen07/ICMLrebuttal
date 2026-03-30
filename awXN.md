## Questions

Q1: Speedup definition and overheads. How exactly is the “Speedup” in Tables 1–4 defined? Does it include FAISS retrieval time, datastore lookups, and KV cache updates, and is it measured as end‑to‑end wall‑clock time per generated token on GPU or CPU? Clarifying this could change how I interpret the magnitude of gains; if key overheads are currently excluded, I would view the results more cautiously.

Q2: Datastore scale and construction cost. For each dataset and model, **approximately how many entries (and tokens) are in the static corpus C_s and dynamic cache C_d**, and **how long does it take to construct the ID and OOD datastores?** Is this cost amortized over many inference runs, and **how sensitive are speedups to datastore size**? If datastore construction is very heavy, that would reduce the perceived practicality of SENSE.

需要解释两个层面，一个是entries 数量，一个是加速比对于datastore的敏感程度

Q3: Behavior on long-context inputs. Have you evaluated SENSE on very long prompts or contexts (e.g., >8k tokens) where speculative drafts can be long and tree-based compression is especially beneficial? If so, how do speedup and accuracy behave versus REST/DReSD/PLD, and how might your method compare conceptually to long-context RSD like RAPID? Positive evidence here would increase my confidence in the method’s generality.


## Limitations
L1: Moderate conceptual novelty relative to existing components. While SENSE is a well-engineered combination, most underlying ideas have prior analogs: hidden-state based dense retrieval is standard in RAG and recent RSD (e.g., DReSD), approximate nearest neighbor search with PCA compression is standard, and entropy/top‑k based relaxed acceptance has appeared in “loosely speculative decoding” and Judge Decoding. The unique aspect is arguably the specific soft-gated mask combination and its integration with tree-based verification, but the paper sometimes overstates how fundamentally different this is from, say, ARC or FLY. A more explicit and honest comparison of mechanisms would help.

L2: Lack of long-context evaluation and comparison to long-context RSD. The benchmarks used are not obviously long-context heavy. Recent work like RAPID specifically targets long-context inference with retrieval-augmented speculative decoding. Without experiments on such settings or direct comparison to RAPID, it is hard to judge how SENSE performs when the advantages of dense retrieval and deeper drafts should be most apparent. This is a missed opportunity and a gap in positioning.

L3: Baseline performance and configuration questions. In Table 1, several baselines yield speedups less than 1 (e.g., SpS on LLaMA2‑13B, Qwen2.5‑7B, Qwen3‑14B; DReSD often close to 1×). This may accurately reflect their performance in a uniform framework, but the paper does not discuss why these methods underperform or whether their hyperparameters were tuned for this setup. Given that SENSE’s gains are reported relative to these baselines, more discussion of baseline behavior would strengthen claims of superiority and avoid a perception that they may be disadvantaged by suboptimal configuration.

L4: No theoretical guarantees or risk bounds. While not strictly required for a systems paper, the method would benefit from at least a qualitative discussion of potential worst-case effects of aggressive soft acceptance and how to bound them, perhaps leveraging insights from ARC or risk‑bounded decoding. Currently, SE is entirely heuristic, making it harder for risk-averse users to reason about failure modes.

微调Q1

完善Q2

Q3和L2一起回答

润色L1

L3L4直接使用
