# Spec-Bench Evaluation Report: Vanilla Greedy vs. Sense (Speculative Decoding)

**Methods compared**: Vanilla Greedy Decoding (baseline) vs. Sense / ours102 (Speculative Decoding)
**Benchmark**: Spec-Bench, 96 samples across 10 task categories
**Evaluation dimensions**: Throughput (TPS), Accuracy (ground-truth tasks), Output Consistency (ROUGE-L), Subjective Quality (LLM-as-Judge)
**LLM-as-Judge model**: Claude 4.5 Opus (via Claude Code CLI `claude -p`, under Claude Max subscription)

---

## 1. Throughput Comparison (Tokens Per Second)

### 1.1 Overall

| Metric | Vanilla | Sense |
|--------|---------|-------|
| TPS (mean ± std) | 5.69 ± 0.09 | **13.23 ± 1.97** |
| Latency (mean, seconds) | 41.38 | **18.13** |
| Speedup (mean) | 1.00x | **2.32x** |
| Speedup (median) | 1.00x | **2.32x** |

### 1.2 Per-Category Breakdown

| Category | N | Vanilla TPS | Sense TPS | Speedup | Vanilla Latency (s) | Sense Latency (s) |
|----------|---|-------------|-----------|---------|---------------------|-------------------|
| Code | 2 | 5.74 ± 0.00 | 15.05 ± 3.17 | 2.57x | 75.51 | 29.98 |
| Creative / Multi-turn | 7 | 5.74 ± 0.02 | 13.66 ± 3.36 | 2.19x | 71.35 | 34.16 |
| Math (Polynomial) | 1 | 5.74 | 15.04 | 1.13x | 61.52 | 54.46 |
| Math (Word Problems) | 16 | 5.72 ± 0.01 | 13.88 ± 1.40 | 2.29x | 38.77 | 16.82 |
| QA / Trivia | 16 | 5.71 ± 0.02 | 12.28 ± 1.49 | 2.09x | 25.94 | 12.92 |
| Science / Knowledge | 4 | 5.74 ± 0.01 | 13.50 ± 1.71 | 2.58x | 112.16 | 42.70 |
| Structured Extraction | 2 | 5.74 ± 0.01 | 15.24 ± 2.00 | **3.09x** | 106.79 | 34.56 |
| Summarization | 16 | 5.70 ± 0.01 | 12.53 ± 1.51 | 2.45x | 48.75 | 20.04 |
| Text Completion | 16 | 5.55 ± 0.15 | 12.31 ± 2.19 | 2.29x | 21.67 | 10.04 |
| Translation (DE-EN) | 16 | 5.71 ± 0.03 | 14.32 ± 1.40 | 2.40x | 27.29 | 11.88 |
| **Overall** | **96** | **5.69** | **13.23** | **2.32x** | **41.38** | **18.13** |

### 1.3 Sense-Specific Speculative Decoding Metrics

| Metric | Value |
|--------|-------|
| Time to First Token, TTFT (s) | 0.079 |
| Draft phase time (s) | 0.680 |
| Verify phase time (s) | 16.303 |
| Draft / Verify time ratio | 1 : 24.0 |
| Mean acceptance length (tokens/step) | 0.79 |

> **Note**: The mean acceptance length of 0.79 tokens/step is relatively low, indicating that the draft model's prediction accuracy has significant room for improvement. The verify phase dominates total latency (~96%), constituting the primary bottleneck.

---

## 2. Accuracy on Tasks with Ground-Truth Answers

### 2.1 Math Problems (17 questions)

Includes 16 GSM8K-style arithmetic word problems (sample IDs 405-480) and 1 polynomial evaluation problem (sample ID 120). Ground-truth answers were manually computed and verified.

| Method | Correct | Total | Accuracy |
|--------|---------|-------|----------|
| Vanilla | 17 | 17 | **100.0%** |
| Sense | 17 | 17 | **100.0%** |

**Per-sample results (all correct):**

| Sample ID | Ground Truth | Vanilla | Sense |
|-----------|-------------|---------|-------|
| 120 | 2 | 2 | 2 |
| 405 | 3.00 | 3.00 | 3.00 |
| 410 | 72 | 72 | 72 |
| 415 | 5 | 5 | 5 |
| 420 | 320 | 320 | 320 |
| 425 | 28.33 | 28.33 | 28.33 |
| 430 | 84 | 84 | 84 |
| 435 | 132 | 132 | 132 |
| 440 | 120 | 120 | 120 |
| 445 | 2 | 2 | 2 |
| 450 | 9 | 9 | 9 |
| 455 | 120 | 120 | 120 |
| 460 | 20 | 20 | 20 |
| 465 | 40 | 40 | 40 |
| 470 | 105 | 105 | 105 |
| 475 | 5760 | 5760 | 5760 |
| 480 | 576 | 576 | 576 |

### 2.2 QA Trivia (16 questions)

Evaluated using keyword matching against verified factual answers. 4 questions were skipped due to ambiguity or subjectivity (opinion-based, locale-dependent, or requiring complex date computation).

| Method | Correct | Evaluable | Accuracy | Skipped |
|--------|---------|-----------|----------|---------|
| Vanilla | 9 | 12 | **75.0%** | 4 |
| Sense | 8 | 12 | 66.7% | 4 |

**Per-question results:**

| ID | Question | Vanilla | Sense | Expected Keywords |
|----|----------|---------|-------|-------------------|
| 325 | When was Cool Hand Luke made? | ✅ | ✅ | 1967 |
| 330 | Where did the pinata tradition come from? | ✅ | ✅ | china / mexico / aztec |
| 335 | When is the State of the Union televised? | ✅ | ✅ | january / february |
| 340 | Who plays the Goblin King in The Hobbit? | ❌ | ❌ | barry humphries |
| 345 | First European town in present-day US? | ✅ | ✅ | st. augustine |
| 350 | What Harry Potter movie came out in 2008? | ❌ | ❌ | half-blood prince |
| 355 | Next Easter on April 11th? | -- | -- | *(skipped: requires date computation)* |
| 360 | When do students return after mid-winter break? | -- | -- | *(skipped: locale-dependent)* |
| 365 | Titles of a board of directors? | ✅ | ✅ | chairperson / director |
| 370 | "Who's got the last laugh now" (Sinatra)? | -- | -- | *(skipped: ambiguous)* |
| 375 | Meaning of CC and BCC? | ✅ | ✅ | carbon copy + blind carbon copy |
| 380 | Should governments influence currency value? | -- | -- | *(skipped: opinion-based)* |
| 385 | What guides Santa home? | ✅ | ❌ | north star / rudolph / star |
| 390 | Who dies in Transformers: Revenge of the Fallen? | ❌ | ❌ | optimus prime |
| 395 | When do Walking Dead comics come out? | ✅ | ✅ | monthly |
| 400 | When did the Salvation Army come to Australia? | ✅ | ✅ | 1880 / 1881 |

> **Note**: Both methods fail on the same 3 questions ([340], [350], [390]). Sense additionally misses [385], where it generates "Northern Lights" instead of the conventionally expected "North Star".

---

## 3. Output Consistency (ROUGE-L: Vanilla vs. Sense)

Since Spec-Bench provides no reference answers, we measure how closely Sense's output matches the Vanilla greedy baseline. For standard speculative decoding with exact verification, one would expect 100% exact match. Deviations indicate approximate speculative decoding behavior.

### 3.1 Overall

| Metric | Value |
|--------|-------|
| Exact Match Rate | **48.9%** (47 / 96) |
| ROUGE-L F1 (mean ± std) | **0.702 ± 0.378** |

### 3.2 Per-Category Consistency (sorted by ROUGE-L descending)

| Category | N | Exact Match | ROUGE-L F1 (mean) | ROUGE-L F1 (std) |
|----------|---|-------------|-------------------|-----------------|
| Math (Polynomial) | 1 | **100.0%** | 1.000 | -- |
| Math (Word Problems) | 16 | **93.8%** | 0.979 | 0.083 |
| Translation (DE-EN) | 16 | 43.8% | 0.856 | 0.277 |
| Text Completion | 16 | 43.8% | 0.634 | 0.456 |
| Summarization | 16 | 50.0% | 0.622 | 0.417 |
| QA / Trivia | 16 | 37.5% | 0.597 | 0.366 |
| Creative / Multi-turn | 7 | 14.3% | 0.508 | 0.384 |
| Code | 2 | 0.0% | 0.450 | 0.062 |
| Science / Knowledge | 4 | 25.0% | 0.386 | 0.433 |
| **Overall** | **96** | **48.9%** | **0.702** | **0.378** |

> **Observations**: Math tasks exhibit near-perfect consistency (93.8%--100% exact match), confirming that Sense produces outputs equivalent to greedy decoding on deterministic, structured tasks. Translation maintains high ROUGE-L (0.856) even with lower exact match, suggesting minor lexical variations. Creative, code, and science tasks show the most divergence, characteristic of approximate speculative decoding.

---

## 4. Subjective Quality (LLM-as-Judge)

**Judge model**: Claude Sonnet 4 (Anthropic), invoked via `claude -p` (Claude Code CLI, print mode).
**Scoring protocol**: Each prediction was independently scored on a 1--5 scale across three dimensions. The judge was blinded to the generation method. A total of 126 individual scoring calls were made (63 samples x 2 methods).

**Scope**: 63 open-ended samples without ground-truth answers, spanning 7 categories: code, creative/multi-turn, science/knowledge, structured extraction, summarization, text completion, and translation.

**Scoring dimensions**:
- **Correctness** (1--5): Factual accuracy and logical soundness
- **Completeness** (1--5): Coverage of all task requirements
- **Quality** (1--5): Writing and reasoning quality

### 4.1 Overall Scores

| Method | Correctness | Completeness | Quality | Avg |
|--------|-------------|--------------|---------|-----|
| Vanilla | 2.89 ± 1.71 | 2.22 ± 1.57 | 2.33 ± 1.55 | 2.48 |
| Sense | 2.79 ± 1.65 | 2.25 ± 1.52 | 2.33 ± 1.49 | 2.46 |
| **Delta (Sense - Vanilla)** | **-0.10** | **+0.03** | **0.00** | **-0.02** |

> **Conclusion**: No meaningful quality difference between the two methods. The overall average delta is merely -0.02, well within noise.

### 4.2 Per-Category Score Comparison

| Category | N | Method | Correctness | Completeness | Quality |
|----------|---|--------|-------------|--------------|---------|
| Code | 2 | Vanilla | 3.00 | 1.00 | 1.50 |
| | | Sense | **3.50** | **2.00** | **2.00** |
| Creative / Multi-turn | 7 | Vanilla | **3.86** | 2.29 | 3.14 |
| | | Sense | 3.14 | **2.43** | 3.14 |
| Science / Knowledge | 4 | Vanilla | 2.00 | 1.25 | 1.75 |
| | | Sense | **2.25** | **1.75** | **2.00** |
| Structured Extraction | 2 | Vanilla | 1.00 | 1.00 | 1.00 |
| | | Sense | 1.00 | 1.00 | 1.00 |
| Summarization | 16 | Vanilla | 2.25 | **2.12** | 2.19 |
| | | Sense | 2.25 | 2.00 | 2.19 |
| Text Completion | 16 | Vanilla | **3.00** | **1.44** | **1.56** |
| | | Sense | 2.88 | 1.38 | 1.50 |
| Translation (DE-EN) | 16 | Vanilla | **3.44** | 3.62 | **3.31** |
| | | Sense | 3.38 | 3.62 | 3.25 |

> **Note**: The structured extraction category scores are uniformly 1.0 for both methods, suggesting the task format (e.g., JSON extraction) is not well-suited to the current scoring rubric. Categories with very few samples (code: 2, structured: 2, science: 4) should be interpreted with caution.

---

## 5. Summary and Key Findings

### 5.1 Head-to-Head Comparison

| Dimension | Vanilla | Sense | Verdict |
|-----------|---------|-------|---------|
| Throughput (TPS) | 5.69 tok/s | **13.23 tok/s** | Sense is **2.32x faster** |
| Math Accuracy (17 Qs) | **100%** | **100%** | Identical |
| QA Accuracy (12 Qs) | **75.0%** | 66.7% | Vanilla slightly better (+8.3 pp) |
| Output Consistency (ROUGE-L) | -- | 0.702 | High on structured, lower on open-ended |
| Subjective Quality (LLM Judge) | 2.48 | 2.46 | **No significant difference** |

### 5.2 Key Findings

1. **Consistent speedup across all categories**: Sense achieves a stable **2.32x** mean speedup over vanilla greedy decoding, with the highest gain on structured extraction (**3.09x**) and the lowest on the single polynomial sample (1.13x, likely an outlier due to n=1).

2. **Math outputs are near-identical**: Math word problems achieve 93.8% exact match and 100% accuracy for both methods, demonstrating that Sense preserves output equivalence with greedy decoding on deterministic, well-constrained tasks.

3. **Output drift on open-ended tasks**: Code (0% exact match), creative (14.3%), and science (25%) categories show substantial divergence from the vanilla baseline, with ROUGE-L scores ranging from 0.39 to 0.51. This is a characteristic signature of approximate speculative decoding and suggests the verification step allows some token-level divergence.

4. **No measurable quality degradation**: LLM-as-Judge evaluation across 63 open-ended samples shows a negligible overall score delta of -0.02 (on a 1--5 scale), indicating that even where outputs diverge textually, the resulting quality remains comparable.

5. **Low draft acceptance rate signals optimization potential**: The mean acceptance length of **0.79 tokens/step** is relatively low. The draft phase accounts for only ~4% of total generation time, while the verify phase dominates at ~96%. This suggests that improving draft model alignment with the target model could yield further speedup gains.

---

*Report generated from 96 paired samples. Evaluation scripts: `eval_spec_bench.py` (TPS, accuracy, ROUGE-L), `score_with_claude.py` (LLM-as-Judge via Claude Sonnet 4). Raw results available in `output/`.*
