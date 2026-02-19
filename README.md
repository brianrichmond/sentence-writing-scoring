# Sentence Writing Scoring

Automated scoring system for a Sentence Writing Pilot Test that assesses how well students (grades 3–6) use target vocabulary words in their writing.

## Problem

Given a student sentence and a target word, determine whether the student used the word correctly — verifying (1) presence of a valid inflected form, (2) grammatical appropriateness, and (3) semantic appropriateness. The algorithm produces a binary score (correct/incorrect) and a confidence score for each response.

## Approach

- **Primary:** LLM-based scoring (prompt-engineered, few-shot) to evaluate morphology, grammar, and semantics in one pass
- **Baseline:** Rule-based NLP pipeline (spaCy lemmatization + POS tagging) for interpretability and sanity-checking
- **Validation:** Inter-rater agreement between LLM, rule-based, and (when available) human raters via Cohen's Kappa

See `notebooks/` for full methodology and results write-up.

## Project Structure

```
sentence-writing-scoring/
├── data/
│   ├── raw/                  # Original dataset (not committed)
│   └── processed/            # Cleaned/transformed data
├── notebooks/
│   └── scoring_analysis.ipynb
├── src/
│   ├── llm_scorer.py         # LLM-based scoring logic
│   ├── rule_based_scorer.py  # spaCy pipeline
│   └── evaluate.py           # Validation and agreement metrics
├── outputs/
│   └── scored_responses.csv  # Final scored dataset
├── .env.example              # Template for environment variables
├── .gitignore
├── requirements.txt
└── README.md
```

## Setup

1. Clone the repo and create a virtual environment:
   ```bash
   git clone https://github.com/your-username/sentence-writing-scoring.git
   cd sentence-writing-scoring
   python -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. Copy `.env.example` to `.env` and add your API key:
   ```bash
   cp .env.example .env
   ```

3. Add the raw dataset to `data/raw/` (not committed — see Data section below).

## Data

The dataset is not committed to this repo. Place the raw CSV in `data/raw/` before running any notebooks or scripts. The dataset contains student responses with the following columns: `student_id`, `grade`, `age`, `gender`, `item_id`, `target_word`, `student_sentence`, `time_spent_seconds`.

## Environment Variables

See `.env.example`. Required:
- `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` depending on the LLM provider used

## Requirements

See `requirements.txt`. Key dependencies:
- `spacy` + `en_core_web_sm` model
- `anthropic` or `openai`
- `pandas`, `numpy`
- `scikit-learn` (for evaluation metrics)
- `jupyter`

## Validation Strategy

With no ground-truth labels initially, agreement between the LLM and rule-based scorer flags uncertain cases for human review. Once human-rated labels are available, Cohen's Kappa is used to measure algorithm-human agreement, and failure modes are analyzed to refine the prompt or rules. Labeled examples also serve as future fine-tuning data for a lighter-weight classifier.