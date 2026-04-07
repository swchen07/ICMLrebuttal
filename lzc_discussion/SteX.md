We thank the reviewer for the specific follow-up questions.

**W1 (ARC/FLY accuracy).** We evaluated all three verification methods under identical SEN retrieval on Qwen2.5-7B across four benchmarks using LLM-as-Judge (Claude Opus 4.6, scoring Correctness/Completeness/Quality on a 1–5 scale).

| Method  | GSM8K SU | GSM8K AL | Code SU | Code AL | Chat SU | Chat AL | QA SU | QA AL |
| ------- | -------- | -------- | ------- | ------- | ------- | ------- | ----- | ----- |
| SEN_ARC | 2.51     | 3.09     | 2.87    | 3.71    | 2.14    | 2.71    | 1.97  | 2.42  |
| SEN_FLy | 2.43     | 3.02     | 2.83    | 3.71    | 2.02    | 2.57    | 2.32  | 2.89  |
| SENSE   | 2.94     | 3.44     | 3.30    | 3.95    | 2.59    | 3.04    | 2.72  | 3.63  |

*SU means SpeedUp. SpeedUp = TPS of SENSE / TPS of Vanilla

*AL means Accpeted Length.

*Code means CodeAlpaca. Chat means UltraChat. QA means TriviaQA.

| Task  | Method  | Correctness | Completeness | Quality   | Average   |
| ----- | ------- | ----------- | ------------ | --------- | --------- |
| Code  | SEN_ARC | 4.33±1.07   | 4.12±1.21    | 3.97±1.14 | 4.14±1.08 |
| Code  | SEN_FLy | 4.20±1.18   | 4.04±1.29    | 3.87±1.25 | 4.04±1.17 |
| Code  | SENSE   | 4.27±1.07   | 4.11±1.22    | 3.92±1.15 | 4.10±1.06 |
| GSM8K | SEN_ARC | 4.68±1.09   | 2.51±0.62    | 1.66±0.47 | 2.95±0.63 |
| GSM8K | SEN_FLy | 4.72±1.02   | 2.55±0.62    | 1.68±0.47 | 2.98±0.61 |
| GSM8K | SENSE   | 4.72±1.02   | 2.55±0.61    | 1.66±0.47 | 2.98±0.60 |
| QA    | SEN_ARC | 2.81±1.87   | 3.02±1.36    | 2.41±1.09 | 2.75±1.38 |
| QA    | SEN_FLy | 2.62±1.89   | 2.94±1.33    | 2.30±1.11 | 2.62±1.39 |
| QA    | SENSE   | 2.77±1.98   | 3.02±1.78    | 2.62±1.64 | 2.80±1.77 |
| Chat  | SEN_ARC | 3.79±0.99   | 4.08±0.98    | 3.87±0.84 | 3.91±0.86 |
| Chat  | SEN_FLy | 3.60±1.06   | 3.90±1.02    | 3.66±0.97 | 3.72±0.92 |
| Chat  | SENSE   | 3.58±1.08   | 3.88±1.09    | 3.56±1.09 | 3.67±1.00 |

Averaging across all four tasks, the overall quality scores are: ARC 3.44, FLY 3.34, SENSE 3.39, difference within noise. While the mean speedup is: ARC 2.37×, FLY 2.40×, SENSE 2.89×, a 20–22% advantage for SE.

Conclusion: **SE achieves statistically indistinguishable quality from ARC and FLY while delivering ~20% higher speedup.** This directly addresses the metric fairness concern, SENSE's speedup gains are real acceleration, not quality-for-speed trade.

We also provide formal justification. At any confident position ($\tilde{H}(p_t) \leq \gamma$), SE reduces to exact matching ($B_{\text{ent}}=0 \Rightarrow G = B_{\text{em}}$)—identical to strict verification. At uncertain positions, accepted tokens must lie in top-$k$ with log-regret $\leq \log(p^{(1)}/p^{(k)})$. Average sequence regret $\leq \rho(\gamma) \cdot \max\log(p^{(1)}/p^{(k)}) $. As $\gamma \to 1$ or $k \to 1$, SENSE recovers lossless SD—the guarantee applies equally to all three methods operating on the same $p_t$.

**On Q1/Q4 consistency.** We appreciate the observation and clarify. Both SENSE(ID) and SENSE(OOD) are strictly speculative decoding, neither constitutes RAG. In RAG, retrieved content is injected into the prompt as context, directly influencing the model's generation. In SENSE, retrieved candidates serve exclusively as draft proposals, every token must independently pass the target model's own logits-based verification to be accepted. The target model never receives datastore content as input; it only adjudicates proposed tokens against its own probability distribution.

Concretely, when a retrieved candidate token conflicts with the model's greedy prediction, it is not blindly accepted. SE evaluates it against the model's full output distribution at that position: the token must reside within the model's own top-$k$ probability nucleus ($B_{\text{topk}}$), at a position where the model itself is uncertain ($B_{\text{ent}}$). A candidate that the model assigns negligible probability to will be rejected regardless of its source. The datastore can only propose what the model already considers plausible, it cannot override the model's judgment or inject knowledge the model does not possess.

The OOD datastore plays the same functional role as a draft model in standard SD. EAGLE's draft head is trained on data containing ground-truth labels; SpS's draft model is pretrained on human corpora, neither is considered external knowledge injection, because the target model's verification remains the sole arbiter. SENSE(OOD) is analogous: the datastore proposes, the model disposes. The occasional outperformance of Vanilla is not a RAG signal, SpecReason (Pan et al., 2025), a purely generative SD without any datastore, reports similar accuracy improvements from accepting high-probability non-greedy alternatives.

That said, SENSE(ID), which uses the target model's own responses, eliminates even this distributional difference and is our recommended deployment setting. We will clarify this framing in revision.