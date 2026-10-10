# Social Media Trust Data Analysis

This project estimates a structural equation model (SEM) of social media trust from Likert-scale survey items. It tests whether **perceived personalisation value (PPV)** and **perceived privacy risk (PPR)** predict **trust (TR)** and **intention to use (ITU)**.

Run the analysis in Jupyter with `processingscript2.ipynb` and one of the cleaned age-group CSV files.

## Files

| File | Role |
| --- | --- |
| `processingscript2.ipynb` | Analysis notebook: reliability, SEM paths, global fit |
| `clean_data18-29.csv` | Cleaned responses, ages 18–29 |
| `clean_data30-45.csv` | Cleaned responses, ages 30–45 |
| `clean_data45-61.csv` | Cleaned responses, ages 45–61 |
| `clean_data61-80.csv` | Cleaned responses, ages 61–80 (file currently loaded in the notebook) |

Each CSV has the same columns (no header extras besides the item names):

`PPV1, PPV2, PPV3, PPR1, PPR2, TR1, TR2, ITU1, ITU2`

Values are numeric Likert scores. Keep the CSVs in the **same folder** as the notebook so `pd.read_csv("...")` finds them.

## What the model tests

**Measurement model (latent constructs from items):**

- PPV =~ PPV1 + PPV2 + PPV3
- PPR =~ PPR1 + PPR2
- TR =~ TR1 + TR2
- ITU =~ ITU1 + ITU2

**Structural paths (hypotheses):**

- Trust is predicted by personalisation value and privacy risk: `TR ~ PPV + PPR`
- Intention to use is predicted by personalisation value, privacy risk, and trust: `ITU ~ PPV + PPR + TR`

## Setup

Use a Python 3 environment with Jupyter (the notebook was last run with Conda `base` and Python 3.13). Install:

```bash
pip install pandas numpy pingouin semopy jupyter
```

Or with Conda:

```bash
conda install pandas numpy jupyter
pip install pingouin semopy
```

Start Jupyter from this project folder:

```bash
cd "path/to/Social Media Trust Data Analysis"
jupyter notebook
```

Open `processingscript2.ipynb` and select a Python kernel that has the packages above.

## How to run

1. Confirm the four CSV files sit next to `processingscript2.ipynb`.
2. In the load cell, set the CSV you want to analyse. The notebook currently uses:

   ```python
   df_mapped = pd.read_csv("clean_data61-80.csv")
   ```

   Change the filename to another age band, for example:

   ```python
   df_mapped = pd.read_csv("clean_data18-29.csv")
   ```

3. Run **all cells in order** (Run → Run All Cells). Later cells depend on `df_mapped` and the fitted `model`.
4. Check the load message: `Dataset successfully synchronized. Validated records: N`. If you see a `FileNotFoundError` message, the CSV name or working directory is wrong. Jupyter’s working directory must be this project folder.

To compare age groups, change the CSV, re-run all cells, and record Phase 1–3 outputs for each file.

## How to read the output (testing the model)

### Phase 1 — Measurement reliability (Cronbach’s alpha)

The notebook prints alpha for PPV, PPR, Trust, and Intention to Use.

| Alpha | Typical reading |
| --- | --- |
| ≥ 0.70 | Acceptable internal consistency |
| 0.60–0.69 | Marginal (common with 2-item scales) |
| < 0.60 | Weak; interpret that construct with caution |

A successful run looks like:

```text
PHASE 1: MEASUREMENT MODEL RELIABILITY (ALPHA)
Alpha - Perceived Personalisation Value (PPV): ...
Alpha - Perceived Privacy Risk (PPR):         ...
Alpha - Trust (TR):                           ...
Alpha - Intention to Use (ITU):               ...
```

### Phase 2 — Structural path coefficients

`model.fit(df_mapped)` should finish with `success=True`. Then the table of paths (`op` is `~`) shows:

- **Beta**: estimated path (or loading) strength and direction
- **z-value** and **p-value**: whether that path differs from zero

Focus on the five structural rows: `TR ~ PPV`, `TR ~ PPR`, `ITU ~ PPV`, `ITU ~ PPR`, `ITU ~ TR`. A conventional cutoff is **p < 0.05**. Positive Beta means a positive association; negative Beta means an inverse association.

The remaining `~` rows are item loadings (PPV1, PPR1, TR1, ITU1 are fixed at 1.0 as scale anchors).

### Phase 3 — Global model fit

`calc_stats(model)` prints χ²/df, CFI, TLI, and RMSEA against the notebook’s targets:

| Index | Target in the notebook |
| --- | --- |
| χ²/df | < 3.0 |
| CFI | > 0.90 |
| TLI | > 0.90 |
| RMSEA | < 0.08 |

If indices miss those targets, the model does not fit that sample well. That is still a valid test result, especially on small age-band files.

## Notes and expected warnings

- Age-band files are small. SEM needs more observations than parameters for stable estimates. Poor fit, non-significant paths, or numerical warnings are likely on a single age file.
- You may see `Sample covariance matrix is not PD` or `Fisher Information Matrix is not PD`. `semopy` then uses near-PD / Moore-Penrose fallbacks. Treat estimates as exploratory on small N.
- The `FileNotFoundError` text still mentions `cleaner_data.csv`; the files you actually use are the `clean_data*.csv` names above.

## Quick checklist

- [ ] Packages installed (`pandas`, `numpy`, `pingouin`, `semopy`)
- [ ] Notebook opened from this folder
- [ ] Load cell points at the CSV you want
- [ ] All cells run top to bottom
- [ ] Phase 1 alphas printed
- [ ] Phase 2 `success=True` and path table printed
- [ ] Phase 3 fit indices printed and compared to targets
