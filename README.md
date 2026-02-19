# Sentence Writing Scoring

Automated scoring system for a Sentence Writing Pilot Test that assesses how well students (grades 3–6) use target vocabulary words in their writing.

## Problem

Given a student sentence and a target word, determine whether the student used the word correctly — verifying (1) presence of a valid inflected form, (2) grammatical appropriateness, and (3) semantic appropriateness. The algorithm produces a binary score (correct/incorrect) and a confidence score for each response.

## Approach

- **Primary:** LLM-based scoring (Claude, prompt-engineered) to evaluate morphology, grammar, and semantics in one pass
- **Baseline:** Rule-based NLP pipeline (spaCy lemmatization + POS tagging) for interpretability and sanity-checking
- **Validation:** Inter-rater agreement between LLM, rule-based, and (when available) human raters via Cohen's Kappa

## Project Structure

```
sentence-writing-scoring/
├── data/
│   └── raw/
│       └── sentence_writing_test_data_unique_words.csv
├── notebooks/
│   └── scoring_analysis.ipynb   # All analysis — run this top to bottom
├── outputs/
│   ├── scored_responses.csv     # Final scored dataset (generated)
│   ├── llm_scores_cache.csv     # LLM scores cache (generated)
│   └── human_review_sample.csv  # Stratified review sample (generated)
├── .gitignore
└── README.md
```

## Quickstart for Reviewers

### 1. Prerequisites

- Python 3.10+ (Anaconda recommended)
- Jupyter Notebook or JupyterLab
- An [Anthropic API key](https://platform.claude.com/settings/keys) — new accounts receive free trial credits

### 2. Install dependencies

Run **Cell 2** of the notebook, or manually:

```bash
pip install anthropic spacy scikit-learn pandas numpy matplotlib seaborn
python -m spacy download en_core_web_sm
```

### 3. Add your API key

Copy `.env.example` to `.env` and add your key:

```bash
cp .env.example .env
```

Then edit `.env`:

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

The notebook loads this automatically. `.env` is gitignored and will never be committed.

### 4. Add the data

Place the raw dataset in:

```
data/raw/sentence_writing_test_data_unique_words.csv
```

Expected columns: `student_id`, `grade`, `age`, `gender`, `item_id`, `target_word`, `student_sentence`, `time_spent_seconds`.

### 5. Run the notebook

Open `notebooks/scoring_analysis.ipynb` and run all cells top to bottom.

The full LLM scoring run (~2,179 rows) takes roughly **45–60 minutes** on a free-tier Anthropic account due to the 50 requests/minute rate limit. The notebook uses `CONCURRENCY=3` and automatic retry with exponential backoff (`max_retries=6`) to stay within limits reliably. Progress is printed every 100 rows and results are cached to `outputs/llm_scores_cache.csv` — if the run is interrupted, re-running the cell will skip already-scored rows and only retry failures.

## Validation Strategy

With no ground-truth labels initially, agreement between the LLM and rule-based scorer flags uncertain cases for human review. Once human-rated labels are available, Cohen's Kappa is used to measure algorithm–human agreement, and failure modes are analyzed to refine the prompt or rules. Labeled examples also serve as future fine-tuning data for a lighter-weight classifier.
