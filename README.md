# Bias-aware resume matching

CMSC 678 project. We score resumes against jobs using sentence-bert and see if the score moves when we swap the applicant's name, pronouns, or university. It does. We also tried tf-idf, blind screening, a compound change (all 3 at once), and flan-t5-base.

## Folders

- `data/` — 10 base resumes, 5 jobs, plus the counterfactual variants
- `notebooks/` — 8 colab notebooks, one per step
- `results/` — the output CSVs from running the notebooks
- `report/` — final report PDF
- `requirements.txt` — what to install

## Order to run

1. `sbert_matching_final.ipynb` — sbert scores for every (resume, job) pair
2. `fairness_analysis_final.ipynb` — original vs counterfactual score, per signal
3. `blind_screening_final.ipynb` — same after removing name / pronouns / university
4. `tfidf_baseline_final.ipynb` — same thing with tf-idf instead
5. `compound_bias_final.ipynb` — change all 3 fields at once
6. `statistical_tests_final.ipynb` — wilcoxon + bootstrap CI on the diffs
7. `llm_demo_final.ipynb` — flan-t5-base labels each pair
8. `figures_final.ipynb` — makes the 3 plots used in the report

Open them in Colab. Each notebook starts by asking you to upload the files it needs (usually `jobs.csv` and `resume_variants.csv` from `data/`, plus a results CSV for some of them).

## What we got

- sbert score moves the most for name changes, then university, then pronouns
- tf-idf moves way less, so sbert is doing more than just word overlap
- blind screening drops everything to zero, but that's because the only thing different was the field we deleted, so the texts end up the same
- compound (all 3 at once) is less than the sum of the 3 single changes, so the signals partly cancel
- pronoun is the only signal where the direction of the shift is consistent (wilcoxon p around 5e-7)
- flan-t5-base just picked the same label for every input even after we sampled and shuffled the options, so it wasn't useful as an auditor

## Not done

The proposal said we'd also try data augmentation and adversarial debiasing and measure accuracy vs fairness. We didn't get to those. They're in future work.

## Setup

```
pip install -r requirements.txt
```

Or just run the notebooks in Colab — each one does `!pip install` at the top.
