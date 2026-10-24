# AB-analysis

Hypothesis testing on two public A/B experiments, worked end to end in one notebook: an e-commerce landing-page test (binary conversion metric) and a mobile-game gate test (continuous and binary metrics). It also includes ANOVA examples for multi-group and multi-factor designs.

Notebook: [`a-b-testiNG.ipynb`](a-b-testiNG.ipynb)

## What's inside

| # | Experiment | Metric | Method | Result |
|---|---|---|---|---|
| 1 | E-commerce landing page, old vs new | `converted` (0/1) | Pearson chi-square, Fisher's exact | p = 0.192 / 0.190: no significant change (12.04% control vs 11.88% treatment) |
| 2 | Same test, split by country (US, UK, CA) | `converted` | Two-way ANOVA (group x country) | Group p = 0.191, country p = 0.202, interaction p = 0.299: nothing significant |
| 3 | Mobile game, gate at level 30 vs 40 | `sum_gamerounds` (continuous) | Shapiro-Wilk, then Levene, then t-test, Welch or Mann-Whitney U | Non-normal, so Mann-Whitney U: p = 0.035 (95th-percentile cutoff), 0.046 (99th), 0.050 (no outlier removal) |
| 4 | Same game test | `retention_1` | Chi-square | p = 0.072: not significant |
| 5 | Same game test | `retention_7` | Chi-square | p = 0.0004: significant |
| 6 | Illustrative 3-group example (30 synthetic scores) | `score` | One-way ANOVA, Tukey HSD | ANOVA p = 0.021; only the A-B pair differs (adjusted p = 0.017) |

## Data and cleaning

Both datasets are public Kaggle datasets and are **not included** in this repository (the notebook reads them from Kaggle's `../input/` paths).

**E-commerce landing page** (294,480 rows)
- 3,893 rows (1.32%) had a group and landing page that did not match (treatment on the old page, or control on the new page); these were dropped.
- Two repeated `user_id` rows remained after that and were dropped, leaving 290,585 rows.
- Countries were merged in from a second file for the two-way ANOVA.

**Mobile game, Cookie Cats** (90,189 players)
- `sum_gamerounds` is heavily right-skewed (maximum 49,854 rounds in `gate_30`).
- The IQR rule removed a different share of each group, which could bias the comparison, so outliers were instead cut at the same percentile within each group.
- A 95th-percentile cutoff leaves 85,702 players (4.98% removed). The retention tests use this cleaned set.
- Because the cutoff changes the outcome, the notebook reports the test at 95%, 99% and with no removal.

## Method-selection function

`AB_Test()` picks the test from the data instead of fixing it in advance:

1. Shapiro-Wilk on each group for normality.
2. If both groups are normal, Levene's test for equal variances, then a t-test (equal variances) or Welch's test (unequal).
3. If either group is non-normal, Mann-Whitney U.

For binary metrics, `AB_testing()` uses Fisher's exact test below 1,000 rows and Pearson chi-square above.

## Takeaways

- The new landing page showed no detectable change in conversion.
- The game-rounds result is borderline and depends on the outlier cutoff (p from 0.035 to 0.050), so it should not be reported without that context.
- The two retention metrics disagree: 1-day retention shows no significant difference, while 7-day retention does. As the notebook notes, the metric chosen drives the conclusion, so the metric that matters most to the product should be fixed in advance.

## Limitations

- Observational analysis of existing public datasets: no sample-size or power calculation, and no pre-registered metric.
- Three metrics are tested on the game data with no multiple-comparison correction.
- The 3-group ANOVA example uses a small synthetic dataset for illustration only.

## Running it

```bash
pip install pandas numpy scipy statsmodels seaborn matplotlib pingouin jupyter
```

Download the two Kaggle datasets (e-commerce A/B testing 2022 `dataset1` including `countries.csv`, and Mobile Games A/B Testing `cookie_cats.csv`), then update the `pd.read_csv(...)` paths in the notebook.

## Credits

The game-data description follows the one in Aurelia Sui's public Kaggle notebook. The Tukey/Bonferroni/Scheffe discussion links to a Statology article, and the two-way ANOVA section links to a tutorial on marsja.se (both referenced in the notebook).
