# CS405 Team 4: Prompt-Injection Detection

Team: Jordan, Tamea, Vasijile

This project builds and compares two detectors that label text sent to a AI customer-support assistant as **benign (0)** or **suspected prompt injection (1)**:

- **Baseline A:** rule-based phrase/regex matching
- **Baseline B:** TF-IDF vectorization + Logistic Regression (scikit-learn)

All examples are not real

## Setup

Requires Python 3.10+.


python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook prompt_injection_detection.ipynb


Then choose **Run → Run All Cells**.

## Project layout

```
data/                              one CSV per teammate (or one merged dataset.csv)
  jordan_attacks.csv               direct override + role spoofing attacks
  tamea_benign.csv                 benign examples (incl. misleading words)   [TODO]
  vasijile_attacks.csv             indirect + paraphrased attacks             [TODO]
prompt_injection_detection.ipynb   full pipeline: load → split → baselines → evaluation
split_assignments.csv              generated: which texts are train vs. test
confusion_matrices.png             generated: confusion matrices for both detectors
```

## Dataset format

Every CSV in `data/` must have these columns:

| Column     | Meaning |
|------------|---------|
| `id`       | Unique ID with an author prefix (J001, T001, V001) |
| `text`     | The message sent to the assistant |
| `label`    | 1 = suspected injection, 0 = benign |
| `category` | e.g. `direct_override`, `role_spoofing`, `indirect`, `paraphrased`, `benign`, `benign_misleading` |
| `seed_id`  | Group ID; every paraphrase of the same base example shares one (J-S01, T-S01, …) |
| `author`   | Who wrote it |

The notebook loads and merges every CSV in `data/` automatically. It also checks for missing columns, duplicate IDs, invalid labels and seed groups that mix labels.

## Split procedure

We use `GroupShuffleSplit` (70% train / 30% test) grouped by `seed_id`. All paraphrases of the same seed land on the same side, so near-duplicates cannot leak from training into testing. The notebook asserts that no `seed_id` appears in both splits. It saves the assignment to `split_assignments.csv` so both detectors are evaluated on the same held-out cases.

## Metrics

Precision, recall, F1, false-positive rate and a confusion matrix for each detector, followed by an inspection of misclassified examples.

## Limitations

Results come from a small, handcrafted dataset and are not a production security guarantee. A text filter alone does not make an LLM application secure (see the OWASP LLM Prompt Injection Prevention Cheat Sheet).

## Attribution

- scikit-learn: TfidfVectorizer, LogisticRegression, GroupShuffleSplit, metrics
- pandas, numpy, matplotlib
- OWASP LLM Prompt Injection Prevention Cheat Sheet; OWASP Top 10 for LLM Applications
