# Football Season Modelling: Poisson and Dixon-Coles

Modelling a 20-team football league from partial-season data: pricing match outcomes,
simulating the rest of the season for title, top-four and relegation probabilities, and
testing whether a more complex model is justified.

## The task

Given 160 played matches (final scores plus pre-match bookmaker odds) from a 20-team
double round-robin season, the goals were to:

1. Build the current league table.
2. Produce 1 / X / 2 probabilities for the upcoming round of fixtures.
3. Estimate each team's probability of winning the league, finishing in the top 4, and being relegated (bottom 3).

## Approach

The core is a single **Poisson goals model**. Each team gets an attack rating (how much it
tends to score) and a defence rating (how little it tends to concede), plus one shared home
advantage, fitted as a Poisson GLM. For any fixture the model pairs the attacking side's
attack with the opponent's defence to produce each side's expected goal rate, then a Poisson
scoreline matrix turns those rates into a full distribution over results, summed into the
1 / X / 2 prices.

The same fitted model drives a 50,000-season Monte Carlo simulation for the outright
probabilities, so pricing and simulation are fully consistent. Modelling goals rather than
results is deliberate: it prices every market from one engine and naturally provides the
goal difference the league tiebreakers require.

As a sanity check, the model's prices are compared against the bookmaker odds (after
removing the margin) on the played games.

## Is Dixon-Coles worth it?

Dixon-Coles extends the Poisson with a correction for low-scoring outcomes, adding one
parameter (`rho`). The notebook `dixon_coles_comparison.ipynb` fits it properly by maximum
likelihood and asks whether the added complexity is justified, three ways:

| Test | Result | Verdict |
|------|--------|---------|
| Likelihood ratio test | p = 0.79 | Not significant |
| AIC | Poisson 982.15 vs Dixon-Coles 984.08 | Poisson preferred |
| Out-of-sample log-loss | Poisson 1.0070 vs Dixon-Coles 1.0072 | Effectively tied |

All three agree: the correction earns nothing on this dataset, so the simpler Poisson is
the right choice. The fitted `rho` is small and, unlike real-world football where
low-scoring draws are inflated, slightly positive here, which suggests this data does not
carry that low-score dependence.

The point is methodological: decide whether to add complexity by testing it, in-sample with
AIC and a likelihood ratio test and out-of-sample with a proper scoring rule, then keep the
simpler model when the evidence does not justify the complex one.

## Repository contents

- `dixon_coles_comparison.ipynb` — fits Dixon-Coles by maximum likelihood and compares it to the Poisson baseline (in-sample and out-of-sample).
- the main solution notebook — builds the table, fits the Poisson model, prices the fixtures and simulates the season.
- `sample_data.csv` — the played matches (scores and pre-match odds).

## Running it

```bash
pip install pandas numpy statsmodels scipy
jupyter notebook dixon_coles_comparison.ipynb
```

The notebook expects `sample_data.csv` in the same directory.

## Notes and possible extensions

- The out-of-sample check uses a single temporal split; k-fold cross-validation would be more robust, though the conclusion is consistent across all three tests.
- Natural extensions: time-weighting recent matches, blending in market information, using expected goals (xG) rather than raw goals, and a hierarchical or Bayesian fit that carries parameter uncertainty through to the simulation.
