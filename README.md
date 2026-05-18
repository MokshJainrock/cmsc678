# Bias-Aware Resume Matching

This project checks whether an automated resume-job matching system gives different scores when only demographic signals in a resume are changed. The main scoring model is Sentence-BERT, and we audit it by making counterfactual versions of each resume where one demographic field is changed at a time (the applicant's name, pronouns, or university).

The project also includes a TF-IDF baseline, a blind-screening mitigation step, a compound-signal version where all three signals change at once, statistical tests on the per-signal differences, and a comparison with the FLAN-T5-base language model. All of it runs in Google Colab on CPU.

## How to run

Each notebook in `notebooks/` is self-contained. The expected order is:

1. `sbert_matching_final.ipynb` — produces `results/sbert_scores.csv`
2. `fairness_analysis_final.ipynb` — reads `sbert_scores.csv`, produces `results/fairness_comparison.csv` and `results/fairness_summary.csv`
3. `blind_screening_final.ipynb` — produces blind-screening result files
4. `tfidf_baseline_final.ipynb` — produces TF-IDF baseline result files
5. `compound_bias_final.ipynb` — produces the compound-signal result files
6. `statistical_tests_final.ipynb` — reads `fairness_comparison.csv`, produces Wilcoxon + bootstrap results
7. `llm_demo_final.ipynb` — produces the FLAN-T5-base label files
8. `figures_final.ipynb` — reads result CSVs and saves PNG figures for the report

Each notebook starts with an upload step in Colab where you upload the files it needs from `data/` (and, for fairness/statistical/figures, the CSVs from `results/`).

## Repository layout

```text
data/
  base_resumes.csv
  jobs.csv
  resume_variants.csv

notebooks/
  sbert_matching_final.ipynb
  fairness_analysis_final.ipynb
  blind_screening_final.ipynb
  tfidf_baseline_final.ipynb
  compound_bias_final.ipynb
  statistical_tests_final.ipynb
  llm_demo_final.ipynb
  figures_final.ipynb

results/
  sbert_scores.csv
  fairness_comparison.csv
  fairness_summary.csv
  blind_screening_scores.csv
  blind_screening_comparison.csv
  blind_screening_summary.csv
  tfidf_scores.csv
  tfidf_comparison.csv
  tfidf_summary.csv
  compound_resume_variants.csv
  compound_comparison.csv
  compound_summary.csv
  fairness_statistical_tests.csv
  llm_match_decisions.csv
  llm_counterfactual_comparison.csv
  llm_flip_summary.csv

report/
  report.tex
  BUILD_REPORT.md

requirements.txt
README.md
```

## Dataset

We have five domains: Software Engineering, Finance, Marketing, Healthcare, and Education. One job per domain (5 total) and two base resumes per domain (10 total). Each base resume has three single-signal counterfactual versions (name, pronouns, university), so 40 resume variants in total. The compound-signal notebook also makes one extra variant per base resume where all three fields change at once.

All other resume content (skills, experience, projects, certifications) is held fixed across counterfactuals, so any score difference comes from the single field we changed.

## Methods

- **SBERT matching.** `all-MiniLM-L6-v2` encodes each resume and each job description. The score is cosine similarity between the two embeddings.
- **Counterfactual audit.** For each resume-job pair we compute the signed and absolute score difference between the original resume and each counterfactual.
- **Blind screening.** We remove the name, the university, and pronoun words from the resume text and re-score. Important note: on this dataset, removing the only field that differs between two counterfactuals forces the difference to zero by construction. See the discussion in the report.
- **TF-IDF baseline.** Same counterfactual audit, but with TF-IDF cosine similarity instead of SBERT. If TF-IDF shows the same pattern, the signal is in the resume text itself, not just in the neural embedding.
- **Compound-signal audit.** For each base resume we score a version where the name, the pronouns, and the university all change at the same time. We compare to the original.
- **Statistical tests.** Paired Wilcoxon signed-rank test and 95% bootstrap confidence interval on the per-signal absolute differences.
- **LLM comparison.** FLAN-T5-base is asked to label each resume-job pair as Strong / Partial / Weak match. We use sampling with temperature 0.7 and shuffle the order of the options on every call, because beam search with a fixed option order makes the model return the same label every time. We also score the counterfactual versions and report how often the label flips.

## What we found

- Changing only the applicant's name produced the largest average SBERT score shift, followed by university, then pronouns.
- The TF-IDF baseline showed the same ordering, so the effect is partly a property of the resume text and not only the neural embedding.
- Blind screening dropped the name and university differences to (almost) zero, but on our dataset that result is forced by construction. The pronoun residual (~0.0036) is the more meaningful number because pronouns appear inside sentences that our simple removal step does not fully strip.
- The compound-signal change moves the score by an amount similar to (but not exactly equal to) the sum of the three single-signal changes.
- FLAN-T5-base also flips its label on some demographic-only changes once the prompt and decoding are set up so the model is not just defaulting to the first option.

## Limitations

Small dataset (10 base resumes, 5 jobs). The counterfactual edits are stylized. The blind-screening result is an upper bound, not a real mitigation result on a realistic ATS. The LLM is small and the prompt is short. We discuss all of this in the report.

## Scope changes from the proposal

We promised three mitigation strategies in the proposal (blind screening, data augmentation, adversarial debiasing) plus an accuracy-vs-fairness tradeoff. We ended up running blind screening only, and we did not measure a held-out accuracy metric. We added the TF-IDF baseline and the compound-signal audit to partly cover those gaps. We added the FLAN-T5-base comparison in response to the proposal feedback that asked us to try a current LLM. Data augmentation and adversarial debiasing are listed as future work.

## Technologies

- Python 3.10 in Google Colab (CPU)
- `sentence-transformers` (`all-MiniLM-L6-v2`)
- `transformers` (`google/flan-t5-base`)
- `scikit-learn` (TF-IDF, cosine similarity)
- `scipy` (Wilcoxon test)
- `pandas`, `numpy`, `matplotlib`

See `requirements.txt` for pinned versions.
