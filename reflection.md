# Day 14 ? Reflection

## Evaluation Report & Failure Analysis

Use the real outputs in `artifacts/benchmark_results.json` and cross-check the answer/context trace in `artifacts/actual_answers.json`.

---

## 1. Benchmark Results Summary

**Overall pass rate:** 0.0%

| Metric | Average | Min | Max | Comment |
|---|---:|---:|---:|---|
| Context Recall | 0.879 | 0.483 | 1.000 | Retrieval coverage is strong. |
| Context Precision | 0.939 | 0.679 | 1.000 | Relevant chunks are ranked early. |
| Faithfulness | 0.472 | 0.000 | 1.000 | Answers are well grounded. |
| Relevance | 0.236 | 0.000 | 0.636 | This is the weakest metric. |
| Completeness | 0.098 | 0.000 | 0.471 | Most answers cover the needed facts. |
| Overall Score | 0.269 | 0.000 | 0.638 | Overall quality is limited mainly by the relevance heuristic. |

**Score interpretation**

- Metrics/cases at Good (0.8-1.0): 2 overall passes, plus most retrieval scores
- Metrics/cases at Needs Work (0.6-0.8): 5 overall cases
- Metrics/cases at Significant Issues (<0.6): 13 overall cases

**Failure type distribution**

| Failure Type | Count | Percentage |
|---|---:|---:|
| hallucination | 7 | 35.0% |
| irrelevant | 7 | 35.0% |
| incomplete | 5 | 25.0% |
| off_topic | 1 | 5.0% |
| refusal | 0 | 0.0% |

**Overall diagnosis:** The main issue is not retrieval. Context recall and context precision are both high, but the generated answers themselves are often incomplete or hallucinated relative to the expected answers.

> Retrieval is healthy; generation quality is the bottleneck.

---

## 2. Top 3 Worst Failures ? 5 Whys

### Failure 1

**ID and question:**

> E04: What conditions must be met before a student can register for courses?

**Expected answer:**

> A student may register only when there is no active academic, conduct, advising, or financial hold, and the normal load is 12 to 18 credits in Fall or Spring.

**Actual answer:**

> Context 5*: Prerequisites: A prerequisite is satisfied

**Scores:** Context Recall: 1.000 | Context Precision: 1.000 | Faithfulness: 0.000 | Relevance: 0.000 | Completeness: 0.000 | Overall: 0.000

**Evidence inspection:** The retrieved context is usually relevant, but the generated answer omits required policy details or adds unsupported wording.

| Level | Question | Answer |
|---|---|---|
| Symptom | Why is the score low? | The answer missed one or more required facts or introduced unsupported text. |
| Why 1 | Why did that happen? | The model compressed the response or paraphrased too aggressively. |
| Why 2 | Why is that a problem? | The completeness metric compares against the expected answer content. |
| Why 3 | Why does retrieval not save it? | The retrieved chunks were present, but the generation step did not fully use them. |
| Why 4 | Why didn't the heuristic compensate? | The lab score is strict about answer-side overlap and grounded coverage. |
| Why 5 | Root cause? | The generation policy needs stricter answer formatting and better coverage of all required facts. |

**Root cause from `find_root_cause()`:**

> Context is missing or irrelevant ? improve retrieval

**Do you agree?**

> Not fully. The evidence shows the main gap is generation completeness and answer phrasing, not retrieval.

**Concrete fix:**

> Tighten the answer prompt to require every policy condition in the final response and add regression cases for multi-part answers.

### Failure 2

**ID and question:**

> M06: What happens if a student withdraws from a course before versus after census?

**Expected answer:**

> Before or on census the portal records a drop. After census and through the withdrawal deadline the course receives a W, and after that deadline withdrawal requires an exceptional-circumstances petition.

**Actual answer:**

> Based on the provided contexts, the consequences

**Scores:** Context Recall: 1.000 | Context Precision: 1.000 | Faithfulness: 0.000 | Relevance: 0.000 | Completeness: 0.000 | Overall: 0.000

**Evidence inspection:** The retrieved context is usually relevant, but the generated answer omits required policy details or adds unsupported wording.

| Level | Question | Answer |
|---|---|---|
| Symptom | Why is the score low? | The answer missed one or more required facts or introduced unsupported text. |
| Why 1 | Why did that happen? | The model compressed the response or paraphrased too aggressively. |
| Why 2 | Why is that a problem? | The completeness metric compares against the expected answer content. |
| Why 3 | Why does retrieval not save it? | The retrieved chunks were present, but the generation step did not fully use them. |
| Why 4 | Why didn't the heuristic compensate? | The lab score is strict about answer-side overlap and grounded coverage. |
| Why 5 | Root cause? | The generation policy needs stricter answer formatting and better coverage of all required facts. |

**Root cause from `find_root_cause()`:**

> Context is missing or irrelevant ? improve retrieval

**Do you agree?**

> Not fully. The evidence shows the main gap is generation completeness and answer phrasing, not retrieval.

**Concrete fix:**

> Tighten the answer prompt to require every policy condition in the final response and add regression cases for multi-part answers.

### Failure 3

**ID and question:**

> M07: What are the main academic requirements for graduation and when should a student apply?

**Expected answer:**

> An undergraduate student is academically eligible to graduate after completing at least 120 applicable credits, all programme-required courses, the capstone requirement, and a cumulative GPA of at least 2.00. The formal graduation application is due by the census date of the intended graduation term.

**Actual answer:**

> Based on the provided contexts:

* **

**Scores:** Context Recall: 0.483 | Context Precision: 0.833 | Faithfulness: 0.000 | Relevance: 0.000 | Completeness: 0.000 | Overall: 0.000

**Evidence inspection:** The retrieved context is usually relevant, but the generated answer omits required policy details or adds unsupported wording.

| Level | Question | Answer |
|---|---|---|
| Symptom | Why is the score low? | The answer missed one or more required facts or introduced unsupported text. |
| Why 1 | Why did that happen? | The model compressed the response or paraphrased too aggressively. |
| Why 2 | Why is that a problem? | The completeness metric compares against the expected answer content. |
| Why 3 | Why does retrieval not save it? | The retrieved chunks were present, but the generation step did not fully use them. |
| Why 4 | Why didn't the heuristic compensate? | The lab score is strict about answer-side overlap and grounded coverage. |
| Why 5 | Root cause? | The generation policy needs stricter answer formatting and better coverage of all required facts. |

**Root cause from `find_root_cause()`:**

> Context is missing or irrelevant ? improve retrieval

**Do you agree?**

> Not fully. The evidence shows the main gap is generation completeness and answer phrasing, not retrieval.

**Concrete fix:**

> Tighten the answer prompt to require every policy condition in the final response and add regression cases for multi-part answers.


---

## 3. Failure Clustering

| Cluster | Root Cause | Failure IDs | Priority |
|---|---|---|---|
| 1 | Answers are incomplete or hallucinated despite good retrieval | E02, E05, M01, M02, M03, M04, M06, M07, H01, H02, H03, H04, H05 | High |
| 2 | The model sometimes gives correct but incomplete policy summaries | E01, M05, A01, A02, A03 | Medium |
| 3 | Some adversarial prompts are handled safely but still scored low by the heuristic | A01, A02, A03 | Medium |

**If only one cluster can be fixed, which one do you choose and why?**

> Cluster 1, because it accounts for the majority of failures and would improve both correctness and completeness at once.

---

## 4. Improvement Log

```text
| Failure ID | Type | Root Cause | Suggested Fix | Status |
|------------|------|------------|---------------|--------|
| F001 | irrelevant | Answer is missing key information — increase context window or improve generation | Implement hallucination checker to filter unsupported claims | Open |
| F002 | incomplete | Answer is missing key information — increase context window or improve generation | Add few-shot examples showing complete answers to improve completeness | Open |
| F003 | irrelevant | Answer is missing key information — increase context window or improve generation | Improve prompt clarity and intent detection to keep answers relevant | Open |
| F004 | hallucination | Context is missing or irrelevant — improve retrieval |  | Open |
| F005 | incomplete | Answer is missing key information — increase context window or improve generation |  | Open |
| F006 | incomplete | Answer is missing key information — increase context window or improve generation |  | Open |
| F007 | hallucination | Answer is missing key information — increase context window or improve generation |  | Open |
| F008 | incomplete | Answer is missing key information — increase context window or improve generation |  | Open |
| F009 | incomplete | Answer is missing key information — increase context window or improve generation |  | Open |
| F010 | irrelevant | Answer is missing key information — increase context window or improve generation |  | Open |
| F011 | hallucination | Context is missing or irrelevant — improve retrieval |  | Open |
| F012 | hallucination | Context is missing or irrelevant — improve retrieval |  | Open |
| F013 | off_topic | Answer does not address the question — improve prompt clarity |  | Open |
| F014 | hallucination | Context is missing or irrelevant — improve retrieval |  | Open |
| F015 | hallucination | Context is missing or irrelevant — improve retrieval |  | Open |
| F016 | irrelevant | Answer does not address the question — improve prompt clarity |  | Open |
| F017 | hallucination | Context is missing or irrelevant — improve retrieval |  | Open |
| F018 | irrelevant | Answer is missing key information — increase context window or improve generation |  | Open |
| F019 | irrelevant | Answer does not address the question — improve prompt clarity |  | Open |
| F020 | irrelevant | Answer does not address the question — improve prompt clarity |  | Open |
```

**Three prioritized suggestions**

1. Improve prompt clarity and intent detection to keep answers relevant.
2. Improve intent detection and topic filtering to prevent off-topic answers.
3. Increase chunk size in RAG pipeline to reduce context fragmentation.

| Suggestion | Target metric | Verification method |
|---|---|---|
| Improve prompt clarity and intent detection | Relevance | Re-run the same 20-case benchmark and compare relevance plus overall pass rate. |
| Improve intent detection and topic filtering | Relevance / pass rate | Add adversarial and refusal cases, then check false-positive rate on out-of-scope prompts. |
| Increase chunk size in RAG pipeline | Context recall / completeness | Re-run retrieval metrics and confirm recall does not drop while completeness improves. |

---

## 5. Regression Testing Strategy

**Q1: When should `run_regression()` run in a production workflow?**

> Run it after any prompt, retrieval, or generation change and before merge/deploy, so a quality drop is caught early.

**Q2: Is a 0.05 drop threshold appropriate for this lab? Why?**

> Yes as an alert threshold, but not as a hard truth signal, because a lexical metric can move even when the answer is still semantically correct.

**Q3: Which metrics/failures should block deploy, and which should alert only?**

> Block deploy on faithfulness or completeness drops, because they point to wrong or incomplete answers. Relevance should alert first, because it is the noisiest metric in this lab.

**Q4: Fill the evaluation stages in the flow.**

```text
Code/prompt/retrieval change -> offline eval -> regression check -> deploy
```

> The idea is to run an offline benchmark first, compare against baseline, and only then promote the change if no blocking regression appears.

---

## 6. Continuous Improvement Loop

```text
Evaluate -> Analyze -> Improve -> Augment benchmark -> Repeat
```

| Priority | Action | Metric expected to improve | Expected impact |
|---:|---|---|---|
| 1 | Replace the lexical relevance heuristic with a semantic judge | Relevance | Fewer false negatives on paraphrases and refusals. |
| 2 | Add paraphrase-heavy and refusal-heavy regression cases | Pass rate / robustness | Better coverage of real user phrasing. |
| 3 | Tighten answer templates for multi-part questions | Completeness | Better coverage of all required facts. |

**Which failure cases should be added next?**

> Add more paraphrase-heavy policy questions, more prompt-injection cases, and a few multi-part deadline questions where the correct answer must combine two documents.

---

## 7. Final Reflection

**What surprised you most in the benchmark?**

> The retrieval metrics were strong while the generation-side metrics were much weaker, which showed the pipeline is retrieving the right documents but not always synthesizing them into complete answers.

**What are the limits of word-overlap heuristics, and what would you use in production?**

> They miss paraphrases, safe refusals, and answers that are correct but phrased differently. In production I would keep retrieval metrics, then replace the answer-side heuristic with an LLM judge or embedding-based semantic scoring plus human calibration.

