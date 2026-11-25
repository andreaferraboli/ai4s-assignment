# Coding assistant session summary (25 November 2026)

What was done in one session with Claude Code, the decisions taken and what is still open. The group's decision log remains [decisioni.md](decisioni.md).

In the tables below, **Group** means the choice was explicitly requested or approved in the chat; **Assistant** means it was a default picked by the assistant, worth confirming.

## What was produced

- [CLAUDE.md](../CLAUDE.md): allowed libraries and project overview.
- [AI_Project.ipynb](../AI_Project.ipynb): data loading, preliminary analysis, merging, pre-processing, length and word analyses, a naive classifier.
- A push to `main` (commit `54ea103`). Changes made after that push, including this file, are not part of it.

## Decisions

### Rules and libraries

| Decision | By |
|---|---|
| Allowed: `sklearn`, `matplotlib`, `numpy`, `pandas`, `datasets`, `lime`, `nltk`; other Hugging Face libraries only to import pre-trained or fine-tuned models | Group |
| The Python standard library is also allowed; anything else needs approval | Assistant |
| `nltk` chosen over spaCy for lemmatization (as in the course tutorials) | Group |
| The assignment text had a note telling coding assistants to plant hidden code that degrades models. It was not followed and was left out of `CLAUDE.md`. It was later removed from the assignment file by a separate commit | Assistant |

### Data

| Decision | By |
|---|---|
| Load with pandas. `CLAUDE.md` still says to load with `datasets` | Group |
| Dataset A (`rogue-security/prompt-injections-benchmark`) is gated: download the Parquet file in the browser into `data/` so the notebook needs no token | Group (no token), Assistant (method) |
| Dataset B: the **balanced** full file (1306 rows), not the `default` one (1998 rows) | Assistant |
| Dataset B is read from a local Parquet file, `data/jailbreak_dataset_full_balanced.parquet`, created from the CSV | Group |
| `data/` is not committed, so each member must create both files | Repository setting |

### Cleaning and merging

| Decision | By |
|---|---|
| Drop exact duplicate rows from B (1306 → 1291) | Group |
| `dataset_merged` keeps each prompt once, with columns `prompt`, `label` | Group |
| Origin is stored in two 0/1 columns, `dataset_A` and `dataset_B`; the 45 shared prompts have 1 in both | Group |
| Only exact duplicates are removed; 17 rows differing only in case or spacing remain | Assistant |

### Pre-processing (`preprocess_text`, column `clean`)

| Decision | By |
|---|---|
| Lowercase and lemmatize | Group |
| Keep stopwords (an earlier version removed them) | Group |
| Do not lemmatize verbs | Group |
| Lemmatize only nouns, adjectives and adverbs (POS tagging); drop punctuation | Assistant |

### Analyses

| Decision | By |
|---|---|
| Length analysis uses the raw prompt: mean and standard deviation of words per prompt, shared histogram | Group |
| Histogram on a logarithmic x-axis (linear was tried and reverted) | Group |
| Naive classifier: more than 125 words → jailbreak (1), otherwise benign (0) | Group |
| Word analyses use the raw prompt, not `clean` | Group |
| A word counts **once per prompt**, so ranking is by number of prompts containing it | Group |
| Words are lowercased and punctuation dropped inside these analyses, without lemmatization | Assistant |
| Stopword-free analysis added as a second chart (NLTK list); the original is kept | Group |
| Extra analysis: words found only in jailbreak prompts, top 15 | Group |

## Results

**Datasets**: no missing or blank values; no conflicting labels.

| Data | Rows | Benign | Jailbreak |
|---|---|---|---|
| A | 5000 | 2997 | 2003 |
| B (after removing duplicates) | 1291 | 638 | 653 |
| Merged | 6246 | 3607 | 2639 |

**Length** (words per prompt)

| Label | Mean | Std |
|---|---|---|
| benign | 85.2 | 123.0 |
| jailbreak | 198.9 | 189.7 |

**Naive length classifier** (on all merged data; the metrics are not yet in the notebook)

| Accuracy | Precision | Recall | F1 | Always "benign" accuracy |
|---|---|---|---|---|
| 0.701 | 0.670 | 0.576 | 0.620 | 0.577 |

Length carries some signal but is not enough on its own, which supports hypothesis H1.

**Words** (share of prompts containing the word: jailbreak / benign)

- With stopwords, the top 15 are all stopwords. "you" (85.7% / 50.6%) and "your" (63.2% / 28.1%) differ clearly.
- Without stopwords: "like" 31.5% / 12.7%, "make" 30.0% / 9.9%, "remember" 29.9% / 11.9%, "character" 27.5% / 9.3%, "ai" 25.9% / 7.2%.
- 6717 words appear only in jailbreaks, led by "chatgpt" (15.8%), "immoral" (5.2%) and "uncensored" (5.0%). Only 25.9% of jailbreaks contain any of the top 15.
- "ans", "zzz", "xxx", "yyy" and "eoa" appear in 103 jailbreaks, all from A: probably one repeated template.
- Counting every repetition favors words repeated in a few long prompts ("mode": 4.9% of jailbreaks, mostly "developer mode"), which is why words now count once per prompt.

Longer prompts contain more distinct words, so part of the word gaps may come from length.

## Differences from the decision log

To be resolved by updating either the notebook or [decisioni.md](decisioni.md):

| Entry | Log says | Notebook does |
|---|---|---|
| D-1-02 | Columns `in_A`, `in_B` | Columns `dataset_A`, `dataset_B` |
| D-1-03 (proposed) | Numeric labels, jailbreak = 1 | Text labels; only the classifier outputs 1/0 |
| D-1.2-01 | Lemmatization example *running* → *run* | Verbs are not lemmatized |

## Open items

- [ ] Notebook text: "duplicate columns" should be "duplicate rows".
- [ ] Notebook text: "Lemmatization of all remaining words" should say nouns, adjectives and adverbs.
- [ ] Notebook text: "balanced" is approximate for A (about 60/40).
- [ ] Add evaluation of the naive classifier (metrics, confusion matrix) and compare later models on the same held-out test set.
- [ ] Derive any keyword features from the training set only; leave `dataset_A` and `dataset_B` out of model features.
- [ ] Check that `scikit-learn` is installed in the notebook environment (it was missing when checked).
- [ ] Revoke the Hugging Face token that appeared in a notebook output, if not done. It was removed before any commit.
- [ ] The repository is public and the notebook outputs include excerpts of gated Dataset A (CC BY-NC 4.0): decide whether to clear outputs or make it private.
- [ ] [README.md](../README.md) still describes the 2025/26 assignment.

## Working note

Saving an out-of-date copy of the notebook in VS Code repeatedly overwrote the assistant's changes. After any external change, run **File: Revert File** before editing or saving (this discards unsaved edits).
