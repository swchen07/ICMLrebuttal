Q3: Behavior on long-context inputs. Have you evaluated SENSE on very long prompts or contexts (e.g., >8k tokens) where speculative drafts can be long and tree-based compression is especially beneficial? If so, how do speedup and accuracy behave versus REST/DReSD/PLD, and how might your method compare conceptually to long-context RSD like RAPID? Positive evidence here would increase my confidence in the method’s generality.

We sincerely thank you for this constructive feedback. We fully agree that the mechanisms of dense retrieval combined with soft matching are particularly advantageous in long-context tasks.

As highlighted in the excellent concurrent work $RAPID$, processing long contexts significantly bottlenecks the draft generation time of standard draft models. Guided by this insight, $RAPID$ utilizes RAG techniques to compress the long context into queries for the draft model, thereby substantially reducing its decoding time.

Conceptually, our $SENSE$ fundamentally relies on dense retrieval acting as the drafting mechanism and shares a similar structural advantage. In our framework, the drafting (retrieval) time does not scale drastically with context length. This efficiency stems from two factors:
1. The pre-constructed external datastore ($C_s$) remains constant in size, ensuring theoretically stable retrieval latency regardless of the input prompt;
2. While the dynamically constructed context datastore ($C_d$) grows linearly with the prompt length, its search overhead remains orders of magnitude lower than the standard auto-regressive inference cost of an LLM.

To provide positive empirical evidence as requested, we further evaluated $SENSE$ on the challenging LongBench-v2 dataset. We randomly sampled 200 instances to build the datastore and used the remainder as the test set.

Using Qwen3-8B as the base model, and adopting the same truncation strategy as $RAPID$ to test both 8k and 16k context lengths, we evaluated $SENSE$ under both In-Domain (ID) and Out-of-Domain (OOD) settings and other speculative decoding baseline. As shown in the table below, $SENSE$ consistently maintains a clear acceleration advantage over the vanilla baseline in long-context tasks.


||longbenchv2(8k)-acc|longbenchv2(8k)-accept length|longbenchv2(8k)-speed up rate|longbenchv2(16k)-acc|longbenchv2(16k)-accept length|longbenchv2(16k)-speed up rate|
|-|-|-|-|-|-|-|
|vanilla|||||||
|PLD|||||||
|DReSD|||||||
|SENSE(OOD)|||||||
|SENSE(ID)|||||||

($\tau$ is accept length, $s$ is speed up rate, $a$ is accuracy rate.)

We will include the comprehensive empirical comparison between $SENSE$ and $RAPID$ on LongBench-v2 in the revised manuscript, alongside a deeper analysis of $SENSE$'s robustness in long-context tasks.