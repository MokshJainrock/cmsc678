# Bias-aware resume matching

CMSC 678 project. We check if a resume scoring model treats two resumes the same when the only thing different is the person's name, pronouns, or school.

Short answer: no, the score moves a little.

## What is in here

- `data/` — the resumes and jobs we used
- `notebooks/` — the 8 notebooks we ran in Colab
- `results/` — the CSVs the notebooks made
- `report/` — final report (PDF)
- `requirements.txt` — packages to install

## What each notebook does

1. `sbert_matching_final` — gets a score for every resume + job pair using sentence-bert
2. `fairness_analysis_final` — checks how much the score changes when we swap a field
3. `blind_screening_final` — removes the name, pronouns and school, then scores again
4. `tfidf_baseline_final` — same idea but with tf-idf instead of sbert
5. `compound_bias_final` — changes all 3 fields at once
6. `statistical_tests_final` — runs a wilcoxon test and bootstrap CI to see if the small numbers are real
7. `llm_demo_final` — asks flan-t5-base to label each pair as strong / partial / weak match
8. `figures_final` — makes the 3 plots that go in the report

Run them in this order. Each one asks you to upload the files it needs at the top.

## What we found

- Name change moves the score the most. School next. Pronouns the least.
- Tf-idf moves a lot less, so sbert is reacting to more than just the words.
- Blind screening drops everything to zero, but that is because the texts end up the same after we delete the only thing that was different.
- Changing all 3 fields at once moves the score less than the sum of the 3 single changes. So they kind of cancel out.
- The pronoun change is the only one where the score moves in the same direction most of the time.
- Flan-t5-base just said the same thing for every input so it did not help much.

## What we did not do

We said in the proposal we would also try data augmentation, adversarial debiasing, and check accuracy vs fairness. We did not get to those, they are in the future work part of the report.

## Setup

```
pip install -r requirements.txt
```

Or just open the notebooks in Colab. Each one runs `!pip install` at the top.
