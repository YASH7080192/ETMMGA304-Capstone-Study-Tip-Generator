# Capstone Report: Study Tip Generator (ETMMGA304)

## 1. Project Definition
- **Problem Statement:** Students starting new computer science and mathematics topics often read passively instead of practicing actively. This project builds an AI-powered Study Tip Generator that takes a topic name and outputs a single, actionable, beginner-friendly practice tip.
- **Model Selection:** Pretrained `openai-community/gpt2` adapted by fine-tuning only the last Transformer block (`transformer.h[-1]`) using masked cross-entropy loss (`-100` on prompt tokens).
- **Success Criteria:**
  - Higher mean total score (out of 6) for the Adapted model vs. Base GPT-2 across 12 held-out topics.
  - Cleaner single-tip stopping behavior with fewer greedy repetition loops.
  - Working Streamlit demonstration with input validation and permanent AI disclosure.

## 2. Dataset Summary & Split
- **Source:** Original instructor-ready dataset authored for ETMMGA304 (`dataset/train_tips.json` and `dataset/held_out_topics.json`), extending the 6 lab manual seed examples to 25 training pairs.
- **Permissions:** Self-authored educational text; no personal or copyrighted material included.
- **Split (Zero Overlap Verified):**
  - **Training set:** 25 topic–tip pairs (CS fundamentals, DSA, ML basics, SQL, web, Linux).
  - **Held-out evaluation set:** 12 completely unseen topics (`recursion`, `matrices`, `binary search`, `linked lists`, `Bayes theorem`, `gradient descent`, `neural networks`, `time complexity`, `regular expressions`, `REST APIs`, `Git branching`, `SQL indexing`).

## 3. Training Configuration
- **Base Checkpoint:** `openai-community/gpt2`
- **Trainable Parameters:** Last Transformer block (`tuned_model.transformer.h[-1]`) only
- **Optimizer:** `AdamW`, learning rate `3e-5`
- **Epochs:** `5`
- **Seed:** `42`
- **Decoding at Evaluation:** Greedy (`do_sample=False`, `max_new_tokens=45`)

## 4. Evaluation Record (12 Held-Out Topics)
Each criterion is scored `0` (poor), `1` (partial), or `2` (strong) for a maximum of `6` per topic. Full raw outputs are archived in `lab_outputs/evaluation_outputs.csv` and `lab_outputs/evaluation_outputs_scored.csv`.

| Topic | Model | Relevance (0-2) | Readability (0-2) | Usefulness (0-2) | Total (/6) | Notes |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| Recursion | Base | 1 | 1 | 0 | 2 | Hallucinates 'recursion keyword' and truncated invalid C++ code. |
| Recursion | Adapted | 1 | 0 | 0 | 1 | Hallucinates 'recursion keyword' and falls into nonsensical import loop. |
| Matrices | Base | 1 | 0 | 0 | 1 | Circular phrasing repeated 3x and truncated mid-sentence. |
| Matrices | Adapted | 1 | 1 | 0 | 2 | Still circular ('create a matrix'), repeats second sentence once but stops cleanly. |
| Binary search | Base | 0 | 0 | 0 | 0 | Confuses binary search with a UI search box; repeats 3x and truncates. |
| Binary search | Adapted | 0 | 2 | 0 | 2 | Still confuses binary search with a search box, but stops cleanly after 1 sentence. |
| Linked lists | Base | 0 | 1 | 0 | 1 | Confuses linked lists with web hyperlink; emits news boilerplate. |
| Linked lists | Adapted | 0 | 0 | 0 | 0 | Failure case: confuses with web links and collapses into severe 'find the best way' loop. |
| Bayes theorem | Base | 1 | 0 | 0 | 1 | Circular probability definition; truncates mid-word ('Bay'). |
| Bayes theorem | Adapted | 1 | 2 | 0 | 3 | Vague tip, but concise single grammatical sentence with clean EOS stop. |
| Gradient descent | Base | 1 | 0 | 0 | 1 | Generic/tautological; repeats 3x and truncates. |
| Gradient descent | Adapted | 1 | 2 | 0 | 3 | Same generic wording, but eliminates repetition loop and stops cleanly. |
| Neural networks | Base | 1 | 1 | 0 | 2 | No actual tip; outputs truncated/hallucinated ScienceDirect URL. |
| Neural networks | Adapted | 1 | 1 | 0 | 2 | Shifts to imperative style ('Use the neural network tool...'), still appends dead URL. |
| Time complexity | Base | 1 | 0 | 0 | 1 | Invents 'time complexity calculator'; repeats 3x. |
| Time complexity | Adapted | 1 | 2 | 0 | 3 | Clean 1-sentence stop, but misses Big-O asymptotic concept. |
| Regular expressions | Base | 1 | 0 | 0 | 1 | Breaks into invalid JS/C import syntax and truncates. |
| Regular expressions | Adapted | 1 | 1 | 0 | 2 | Avoids broken code syntax, stays in plain English, though vague and cuts off. |
| REST APIs | Base | 1 | 0 | 0 | 1 | Tautological ('create a REST API'); repeats 4x and truncates. |
| REST APIs | Adapted | 1 | 0 | 0 | 1 | Slightly more specific ('REST endpoint'), but still repeats 4x and truncates. |
| Git branching | Base | 2 | 0 | 1 | 3 | On-topic basic action, but repeats 4x and truncates. |
| Git branching | Adapted | 2 | 1 | 1 | 4 | On-topic basic action; repeats once instead of 4x and stops cleanly. |
| SQL indexing | Base | 1 | 0 | 0 | 1 | Conflates indexing with table creation; repeats 4x and truncates. |
| SQL indexing | Adapted | 1 | 0 | 0 | 1 | Failure case: nonsensical opening ('table name in the table name') + broken SQL/Python mix. |

### Quantitative Summary

| Metric | Base GPT-2 | Adapted GPT-2 | Change |
| :--- | :---: | :---: | :---: |
| **Mean Total Score (/6)** | **1.25** (SD = 0.75) | **2.00** (SD = 1.13) | **+0.75 (+60.0%)** |
| **Median Total Score (/6)** | 1.00 (range 0–3) | 2.00 (range 0–4) | +1.00 |
| **Mean Relevance (/2)** | 0.92 | 0.92 | 0.00 |
| **Mean Readability (/2)** | 0.25 | 1.00 | **+0.75 (4×)** |
| **Mean Usefulness (/2)** | 0.08 | 0.08 | 0.00 |
| **Avg. Output Length (chars)** | 180.1 | 129.1 | -51.0 (more concise) |
| **Head-to-Head (12 topics)** | 2 wins | **7 wins** (3 ties) | Net +5 topics |

## 5. Failure Analysis & Conditions for Use
- **Key Takeaway:** Fine-tuning the final Transformer block on 25 pairs over 5 epochs clearly improved **format and stopping behavior** (mean Readability rose from `0.25` to `1.00`, eliminating runaway repetition loops on `binary search`, `Bayes theorem`, `gradient descent`, `time complexity`, `matrices`, and `Git branching`), raising the overall mean score from `1.25/6` to `2.00/6`. However, **Relevance (`0.92`) and Usefulness (`0.08`) remained unchanged**, showing that small last-block fine-tuning adapts output structure rather than injecting deep domain knowledge.
- **Documented Failure Cases:**
  1. **Linked lists (Adapted, 0/6):** Misinterpreted `"linked lists"` as web `"links"` and collapsed into a severe repetition loop (`"Use the links below to find the best way to find the best way..."`).
  2. **SQL indexing (Adapted, 1/6):** Produced a nonsensical opening (`"Use the table name and the table name in the table name."`) followed by mixed SQL and invalid Python syntax, without explaining database indexes.
  3. **Binary search (Base 0/6, Adapted 2/6):** Both models confused the algorithm with a website `"search box"`.
- **Conditions for Use:** Strictly intended as an **instructor-reviewed brainstorming prototype** demonstrating small-LM fine-tuning mechanics. It must **not** be deployed directly to students without human vetting, a larger curated dataset (200+ examples), multi-block/LoRA fine-tuning, and repetition-penalty decoding.

## 6. Privacy, Bias & Ethical Considerations
- **Privacy:** Zero personal student data is collected, logged, or sent to third-party APIs; inference runs completely locally.
- **Bias & Hallucinations:** Pretrained GPT-2 inherits web-text patterns (such as news boilerplate on `linked lists` and dead academic URLs on `neural networks`) and can produce confident but factually incorrect technical guidance.
- **Disclosure:** The Streamlit interface (`app.py`) displays a permanent AI-generation disclosure banner advising instructor verification.

## 7. Capstone Viva Questions & Answers

1. **Why did you select this dataset and model?**
   - `openai-community/gpt2` is lightweight enough to fine-tune in minutes on a free Colab T4 GPU and run locally on CPU inside Streamlit. Pairing it with a self-authored, zero-leakage 25-example CS/Math dataset avoids privacy/copyright issues while cleanly separating 12 held-out evaluation topics.

2. **Which results show whether adaptation improved the outputs?**
   - Across 12 held-out topics, the Adapted model outperformed Base GPT-2 on 7 topics (3 ties, 2 losses), raising the mean score from **1.25/6 to 2.00/6**. The improvement came specifically from **Readability (0.25 → 1.00)**—learning concise single-sentence stopping instead of greedy repetition loops—while Relevance (`0.92`) and Usefulness (`0.08`) showed that deeper factual gains require more data and trainable capacity.

3. **What human review would you require before students use the app?**
   - Subject-matter instructors must verify factual correctness against the syllabus, reject misleading outputs (such as confusing `binary search` with a search box or `linked lists` with web links), and curate approved outputs back into an expanded training dataset.
