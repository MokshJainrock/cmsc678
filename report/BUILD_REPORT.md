# How to build the report PDF

The report is written in the ICML 2022 LaTeX template. The easiest way to compile it is on Overleaf.

## On Overleaf (recommended)

1. Open https://www.overleaf.com/latex/templates/icml2022-template/ and click **Open as Template**. This gives you a project that already has `icml2022.sty`, the example bibliography style, and the rest of the template.
2. In that new Overleaf project, open the example `.tex` file and replace its full contents with the contents of `report/report.tex` from this repo.
3. Make a folder called `figures/` in the Overleaf project and upload the three PNGs that `notebooks/figures_final.ipynb` produces:
   - `fig_before_after_blind.png`
   - `fig_by_domain.png`
   - `fig_name_distribution.png`
4. Click **Recompile**. You should get a 4–6 page PDF.

## On your own machine

If you have a local LaTeX install (TeX Live / MacTeX):

1. Download `icml2022.sty` from the ICML 2022 author kit and place it next to `report.tex`.
2. Place the three figure PNGs in a `figures/` folder next to `report.tex`.
3. Run:
   ```
   pdflatex report.tex
   pdflatex report.tex
   ```
   (Two passes are needed for the references.)

## If a figure file is missing

If you haven't run the figures notebook yet, you can comment out the `\includegraphics` lines and the report will still compile. Then run the figures notebook and put the PNGs in `figures/` to bring them back.
