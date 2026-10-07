# F1 Qualifying Dashboard

## Question

How much does starting position decide an F1 result?

## Preview

![Dashboard](f1_qualifying_dashboard.png)

[PDF](f1_qualifying_dashboard.pdf) · [Power BI file (.pbix)](f1_qualifying_dashboard.pbix) · [PowerPoint (.pptx)](f1_qualifying_dashboard.pptx)

## Data

209 races, 2017 to 2026 (2026 is the season so far). Sources: Jolpica-F1 (2017-2022) and
OpenF1 (2023 onwards). Tables prepared with Claude Code.

| File | One row is | Rows |
|------|-----------|------|
| `data/fact_backtest.csv` | one driver in one race: qualifying position, finishing position, podium flag, and the predicted podium chance, rank and pick | 4,211 |
| `data/fact_race_summary.csv` | one race: how many of the 3 podium picks were right for the model and for the qualifying baseline, plus the picks as text | 209 |
| `data/dim_driver.csv` | one driver: first and last season, races, latest team | 61 |
| `data/dim_team.csv` | one team (names cleaned across renames): first and last season, races | 15 |
| `data/dim_circuit.csv` | one circuit: first and last season, races | 34 |

## Data model

Star schema, with `fact_backtest` as the fact table. Relationships are many-to-one, single direction:

- `fact_backtest[driver]` to `dim_driver[driver]`
- `fact_backtest[team]` to `dim_team[team]`
- `fact_backtest[circuit]` to `dim_circuit[circuit]`

`fact_race_summary` is not connected to the model; its measures use only its own columns.

## Measures (DAX)

**Model hit rate**: the share of the model's 3 podium picks that finished on the podium, averaged over races.

```
Model hit rate = AVERAGE(fact_race_summary[model_hits]) / 3
```

**Baseline hit rate**: the same for the baseline, which picks the top 3 qualifiers.

```
Baseline hit rate = AVERAGE(fact_race_summary[baseline_hits]) / 3
```

**Races analysed**: the number of races in the current filter.

```
Races analysed = COUNTROWS(fact_race_summary)
```

**Pole win rate**: of the drivers who qualified first, the share who won the race.

```
Pole win rate =
DIVIDE(
    CALCULATE(COUNTROWS(fact_backtest), fact_backtest[quali_pos] = 1, fact_backtest[actual_finish] = 1),
    CALCULATE(COUNTROWS(fact_backtest), fact_backtest[quali_pos] = 1)
)
```

**Top 3 quali podium rate**: of the drivers who qualified in the top 3, the share who finished on the podium.

```
Top 3 quali podium rate =
VAR Top3Rows = CALCULATE(COUNTROWS(fact_backtest), fact_backtest[quali_pos] <= 3)
VAR Top3Podiums = CALCULATE(SUM(fact_backtest[actual_podium]), fact_backtest[quali_pos] <= 3)
RETURN DIVIDE(Top3Podiums, Top3Rows)
```

## Findings

- The top 3 qualifiers reach the podium 67.3% of the time, from about 50% at Interlagos to about 83% at Shanghai.
- Average finish rises from about P3 for the driver starting first to about P19 for the back of a
  22-car grid (a small sample; starting 20th averages about P16).
- Teams' average starting positions range from about P4.5 (Mercedes) to about P15.5 (Williams) among
  teams that raced most of these seasons, while they gain or lose only about 1 to 1.5 places on average.
- Poles turn into wins 55.0% of the time.

## Caveats

- 2026 is a partial season.
- The circuit chart shows only circuits with 5 or more races, because Madring (1 race) and Portimao
  (2 races) showed a misleading 100%.
- Teams are grouped across renames (Sauber includes Alfa Romeo and Kick Sauber, RB includes Toro Rosso
  and AlphaTauri). Audi and Cadillac (both new in 2026) are shown separately.
- A retirement counts as last place.
- `dim_driver`, `dim_team` and `dim_circuit` list everything since 2014, so a few rows have no races
  in the 2017+ fact tables.
- The model-vs-baseline chart uses out-of-sample backtest predictions (validation 2017-2024, test 2025
  onwards). The model matches the qualifying baseline, with no proven edge. The 2025+ races are not a
  fully clean test, because the model-selection rule was changed after seeing them.

## Credits

Power BI basics from a Coursera course. Layout, colors, chart titles and text with help from Claude.
Data tables prepared with Claude Code. Race data from [Jolpica-F1](https://github.com/jolpica/jolpica-f1)
and [OpenF1](https://openf1.org).

## How to open

Download [`f1_qualifying_dashboard.pbix`](f1_qualifying_dashboard.pbix) and open it in Power BI Desktop.
