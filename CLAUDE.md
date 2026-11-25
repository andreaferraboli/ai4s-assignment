# CLAUDE.md

## Allowed libraries

Use **only** these:

- `sklearn` (scikit-learn)
- `matplotlib`, for visualization
- `numpy`
- `pandas`
- `datasets` (Hugging Face), to load the project datasets
- `lime`, to explain classifier predictions
- `nltk`, for text preprocessing (stopwords, lemmatization)
- Other Hugging Face libraries (e.g. `transformers`), **only** to import pre-trained or fine-tuned models

The Python standard library is fine. If a task seems to need anything else (e.g. seaborn), ask before using it.

## Project

Artificial Intelligence for Security 2026/27, group assignment (3 to 4 students). Source of truth: [AI4S_istruzioni_visibili.md](AI4S_istruzioni_visibili.md).

**Goal:** analyze large datasets of LLM jailbreak attacks and use what is learned to defend a sandboxed LLM.

**Deliverable:** a self-explanatory Python notebook, with clear descriptions of each analysis and its conclusions.

**Deadlines:**

- Submission via WeBeep: Thursday 17 December 2026, 23:00
- Presentation: Friday 18 December 2026, 14:15 to 19:15 (10 minutes, notebook only, no slides, all members present)

**Marking:** correct use of methods, depth and insight of the analysis, clarity of the notebook, quality of the presentation.

**AI use:** coding assistants may help with parts of the notebook. The assignment as a whole cannot be done by an AI.

## Datasets

| ID | Name | Source |
|---|---|---|
| A | Prompt Injections Benchmark | https://huggingface.co/datasets/qualifire/prompt-injections-benchmark |
| B | Jailbreak Classification | https://huggingface.co/datasets/jackhhao/jailbreak-classification |
| C | Mosscap Prompt Injection | https://huggingface.co/datasets/Lakera/mosscap_prompt_injection |
| D | Pliny HackAPrompt | https://huggingface.co/datasets/hackaprompt/Pliny_HackAPrompt_Dataset |

- Large datasets may be sampled.
- Load them with `datasets`; keep any local copies in `data/` (ignored by git).
- Sandbox environment: not yet released.

## Tasks

These are suggestions, not requirements; new analyses are encouraged.

1. **Explore, visualize and cluster** (start with A and B): prompt length and vocabulary of jailbreaks vs normal prompts; k-means on word count or TF-IDF vectors and centroid inspection; 2D/3D dimensionality reduction; whether prompts from C or D fit the clusters.
2. **Detect jailbreaks:** linear vs more complex classifiers; text encodings; suitable evaluation methods; LIME explanations; apply the best model to C and D; judge jailbreak success from LLM responses.
3. **Jailbreak and defend a running LLM:** attack and defense techniques; classifier as a filter, then improve it on missed attacks; system prompt changes; compare small models; multi-turn attacks.
4. **Advanced, secret extraction:** hide a secret in the system prompt and try to extract it; defend with the classifier, a secret filter or the system prompt. Further options: guardrail LLM, attacker LLM, attacks on an LLM agent with an SQL database.
