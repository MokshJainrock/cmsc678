# Bias-aware resume matching

Small project for CMSC 678. We score resumes against jobs with sentence-bert and check if the score changes when we swap the applicant's name, pronouns, or university.

## How to run it

All notebooks run on Google Colab on CPU. Run them in this order:

1. `sbert_matching_final.ipynb` – gets the sbert scores
2. `fairness_analysis_final.ipynb` – compares original vs counterfactual scores
3. `blind_screening_final.ipynb` – removes the demographic fields, scores again
4. `tfidf_baseline_final.ipynb` – same check using tf-idf
5. `compound_bias_final.ipynb` – changes name + pronouns + university all at once
6. `statistical_tests_final.ipynb` – wilcoxon + bootstrap on the score diffs
7. `llm_demo_final.ipynb` – asks flan-t5-base to label each pair
8. `figures_final.ipynb` – makes 3 plots for the report

Each notebook starts with a Colab upload step. Upload the files from `data/` (and for some notebooks, the CSVs from `results/`).

## Folders

```
data/        the resumes and jobs
notebooks/   the 8 colab notebooks
results/     output CSVs from the runs
report/      the report (PDF + LaTeX source + ICML template files)
figures/     PNG plots (also copied inside report/ so the .tex finds them)
```

## Data

5 jobs (1 per domain) and 10 resumes (2 per domain). Five domains: software engineering, finance, marketing, healthcare, education. Each resume has 3 single-signal counterfactual versions (one with the name changed, one with pronouns, one with university). The other content stays the same so any score change has to be from that one field.

## What we found

- Name change moves the sbert score the most on average. Pronoun the least. University in between.
- The tf-idf baseline moves much less. So sbert is doing more than just word overlap with these fields.
- The blind screening drops everything to zero. That's expected: once you delete the one thing that was different, the two texts are the same so the score has to be the same.
- The compound change (all three fields at once) is smaller than the sum of the three single-field changes. So the signals partly cancel each other out.
- The pronoun change is the only one where the direction of the shift is consistent (Wilcoxon p = 5e-7). Name and university change the score by more but go both ways.
- Flan-t5-base just returned the same label for every input even after sampling and shuffling the options. So we couldn't use it as a real auditor.

## What's missing

The proposal said we'd also do data augmentation and adversarial debiasing, and measure the accuracy vs fairness tradeoff. We didn't get to those, they are in the future work section of the report.

## Setup

`pip install -r requirements.txt` (or just install in Colab — each notebook does `!pip install` at the top).
