## Weaknesses

W1: The method relies heavily on the datastore, yet the paper provides limited analysis of how datastore size, composition, or domain mismatch affect performance. Since retrieval quality directly determines the effectiveness of speculative decoding, a more systematic study would strengthen the paper.

W2: It is not entirely clear whether datastore construction includes samples derived from the evaluation datasets (e.g., answers or reasoning traces). If datastore entries contain solutions from the evaluation distribution, this could inflate acceptance rates and speedups. Clarifying the separation between datastore and test data would improve the credibility of the evaluation.

W3: The core idea (moving retrieval from lexical matching to embedding space) is intuitive and closely related to existing retrieval and RAG approaches. The novelty lies more in integration than in a fundamentally new paradigm.

## Questions
Q1: How exactly is the datastore constructed for each experiment? Is it built solely from training data, or does it include samples derived from the evaluation datasets?

Q2: How does performance change when varying datastore size, domain mismatch, and retrieval recall.

Q3: Can the authors provide a detailed breakdown of runtime between retrieval, verification, and model forward passes?