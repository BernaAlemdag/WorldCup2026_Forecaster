# ⚽ FIFA World Cup 2026 Forecaster

<p align="center">
  <a href="https://colab.research.google.com/github/your-username/wc2026-forecaster/blob/main/WorldCup2026_Forecaster.ipynb"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab"></a>
  <img src="https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Jupyter-notebook-F37626?logo=jupyter&logoColor=white" alt="Jupyter">
  <img src="https://img.shields.io/badge/status-reproducible-success" alt="Reproducible">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/NumPy-013243?logo=numpy&logoColor=white" alt="NumPy">
  <img src="https://img.shields.io/badge/pandas-150458?logo=pandas&logoColor=white" alt="pandas">
  <img src="https://img.shields.io/badge/SciPy-8CAAE6?logo=scipy&logoColor=white" alt="SciPy">
  <img src="https://img.shields.io/badge/Matplotlib-11557C?logo=plotly&logoColor=white" alt="Matplotlib">
</p>

A probabilistic, validated model that estimates every team's odds at the FIFA World Cup 2026. Instead of naming a single winner, it reports calibrated probabilities for reaching each round and winning the title, and it checks that those probabilities are honest by back-testing them on matches the model never saw.

Everything lives in one self-contained notebook. It fetches public match data by URL and needs no local files, no package install, and no Kaggle account. ▶️ Open it in Colab and choose **Runtime, Run all**.

## 📊 Results

50,000 simulated tournaments of the real 2026 bracket (12 groups, then the 8 best third-placed teams, then the Round of 32 through the Final), using Elo strength computed from the full history of internationals.

| # | Team | Group | Elo | Advance | Reach QF | Reach SF | Reach Final | 🏆 Champion |
|--:|------|:-----:|----:|--------:|---------:|---------:|------------:|------------:|
| 1 | Spain | H | 2204 | 97.6% | 58.9% | 47.2% | 34.2% | **23.4%** |
| 2 | Argentina | J | 2169 | 92.9% | 54.3% | 39.4% | 27.1% | **16.8%** |
| 3 | France | I | 2112 | 85.1% | 48.7% | 31.7% | 17.7% | **9.8%** |
| 4 | England | L | 2076 | 87.4% | 39.6% | 24.0% | 13.1% | **6.8%** |
| 5 | Brazil | C | 2047 | 82.0% | 35.3% | 19.9% | 9.5% | **4.6%** |
| 6 | Colombia | K | 2041 | 78.2% | 31.7% | 17.0% | 9.0% | **4.4%** |
| 7 | Portugal | K | 2028 | 76.5% | 30.1% | 15.6% | 8.1% | **3.7%** |
| 8 | Mexico (host) | A | 2017 | 85.3% | 34.0% | 17.8% | 8.3% | **3.4%** |

How far each team is expected to go:

## ✅ Out-of-sample calibration

A forecast is only useful if its probabilities hold up. The model is fit on internationals before 2022 and scored on every international since, which is 4,502 matches it never saw.

| Metric | Model | Naive baseline | Note |
|--------|------:|---------------:|------|
| Log-loss (lower is better) | **0.876** | 1.051 | proper scoring rule |
| Brier score (lower is better) | **0.515** | 0.634 | mean squared probability error |
| Accuracy (higher is better) | **59.9%** | 47.7% | most likely outcome |
| Skill vs baseline | **+16.7%** | n/a | log-loss reduction |

The reliability curve stays close to the diagonal, so when the model says 60% it happens about 60% of the time.

## 🧠 How it works

The forecast is built in three stages.

1. 💪 **Strength (Elo).** Ratings are computed from scratch over roughly 50,000 international matches (1872 to present), using the standard eloratings.net rules: match-importance weights, a goal-margin multiplier, and home advantage.
2. 🎯 **Match model (Dixon-Coles bivariate Poisson).** The Elo gap between two teams is mapped to a full scoreline distribution, not just win, draw, or loss, because the group stage is ranked on goal difference. Four parameters are fit by maximum likelihood on the modern era, using each match's pre-kickoff ratings so there is no look-ahead.
3. 🎲 **Tournament simulation (Monte Carlo).** The real bracket is simulated 50,000 times in seconds: group round-robins with tiebreakers, the eight best third-placed teams slotted into the Round of 32 by a constraint solver that respects FIFA's eligibility table, and the knockout tree with an Elo-weighted extra-time and penalty model.

Three analysis layers make the model interpretable:

* 🔍 **Sensitivity analysis.** Each team's title odds are attributed to causes by re-running the whole tournament with one input changed. Spain's 23% drops below 1% if they were an average side, so their favouritism is earned, not an easy draw.

* 🔄 **Scenario analysis.** Force a result and re-estimate the whole field.
* 🌱 **Living forecast.** Pin matches as they are played and re-estimate only what is left.

## 🚀 Run it

**Colab (recommended).** Click the Open in Colab badge above, then choose Runtime, then Run all. No setup.

**Locally.** Open the notebook in Jupyter and run all cells. It only needs the usual scientific stack:
```bash
pip install numpy pandas scipy matplotlib
jupyter notebook WorldCup2026_Forecaster.ipynb
```

## 🔮 Living forecast

As matches are played, pin them and re-estimate only what is left. Use real scorelines for group games (so goal difference is correct) and winners for decided knockout ties.

```python
live = forecast_from_state(
    elo, model,
    group_scores={("C", "Brazil", "Scotland"): (3, 1)},   # real result
    ko_winners={84: "Spain", 73: "Switzerland"},           # decided ties
    n_sims=50_000,
)
live.head(12)[["team", "group", "champion"]]
```

Knockout match numbers: 73 to 88 Round of 32, 89 to 96 Round of 16, 97 to 100 quarter-finals, 101 to 102 semi-finals, 104 final. As every match is pinned the probabilities collapse to the actual result.

## 📓 What's inside the notebook

1. Tournament structure: the real 2026 groups and full bracket
2. Strength engine: Elo from match history
3. Match model: Dixon-Coles bivariate Poisson
4. Tournament engine: vectorised Monte Carlo
5. Calibration, sensitivity analysis, scenarios, and plotting helpers
6. Load the data and fit the model
7. The forecast (50,000 simulations)
8. Out-of-sample calibration
9. Why this team (sensitivity analysis)
10. What-if scenarios
11. Living forecast

## ⚠️ Limitations

Football is hard to predict, and putting calibrated probabilities on it is the point, but the model is deliberately simple and makes a few approximations.

* Strength is Elo only. There is no squad, injury, or market-value data, and no manager or travel effects (the host advantage is folded in as a fixed Elo bonus).
* Group tiebreakers use points, then goal difference, then goals scored, then a random draw. The head-to-head step that FIFA applies first among level teams is omitted.
* Third-place allocation uses constraint-respecting bipartite matching rather than FIFA's exact lookup table. Both satisfy the same eligibility rules.
* Knockout draws are resolved by an Elo-weighted, heavily damped extra-time and penalty model.
* Forecasts are illustrative, not betting advice. A short knockout tournament carries large irreducible variance.

## 📚 Data and credits

* Historical results: [martj42/international_results](https://github.com/martj42/international_results) (CC-licensed), used to compute Elo and calibrate the goal model.
* Optional ratings source: the Kaggle dataset [pranishkessi/fifa-world-cup-2026-prediction-simulator](https://www.kaggle.com/datasets/pranishkessi/fifa-world-cup-2026-prediction-simulator).
* Bracket and draw: the official FIFA Final Draw (5 December 2025) and the resolved March 2026 play-offs.
* Method reference: Dixon and Coles (1997), Modelling Association Football Scores and Inefficiencies in the Football Betting Market.

