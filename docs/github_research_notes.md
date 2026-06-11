# GitHub Research Notes

Reviewed on 2026-06-10.

## Repositories Checked

- [Berliwu/world-cup-2026-match-prediction-engine](https://github.com/Berliwu/world-cup-2026-match-prediction-engine)
- [manuelpeba/world-cup-2026-forecast](https://github.com/manuelpeba/world-cup-2026-forecast)
- [javierruanohdez/world-cup-2026-prediction](https://github.com/javierruanohdez/world-cup-2026-prediction)
- [SupaTx/PredictaX-9](https://github.com/SupaTx/PredictaX-9)

## Useful Ideas

The common pattern across the stronger projects is:

- convert team strength into win/draw/loss probabilities
- estimate expected goals rather than hand-picking score buckets
- simulate tournament paths many times for advancement probabilities
- keep an audit trail of assumptions and model outputs

## What We Adopted

We adopted a lightweight version of the Berliwu-style statistical core:

- `expected_goals()` converts adjusted team ratings into home/away xG
- `score_grid()` uses a Poisson goal grid to estimate home/draw/away probabilities
- `analyze_match()` now records xG and probabilities in the JSON summary
- exact score picks are selected from plausible scores, then lightly adjusted for more realistic match expression

This keeps the project simple enough for a friends' bracket while fixing the earlier issue where rating buckets produced too many `3-0` and `0-3` scores.

## Added Next

After the expected-goals layer, we added a lightweight Monte Carlo simulation. It runs many tournament paths from the same xG probability grids and reports advancement/champion probabilities in `world_cup_bracket_model_summary.json`.

Monte Carlo answers a different question from the filled bracket:

- The filled bracket is one chosen path.
- Monte Carlo asks, "If we replayed this tournament thousands of times with these probabilities, how often would each team reach each stage?"

That helps separate a confident pick from a merely plausible pick.

## Score Calibration

We added `data/score_calibration.json` using match scores from the 2014, 2018, and 2022 World Cups.

The calibration excludes penalty shootout scores and includes listed extra-time scores. Across 192 matches, the most common winning score patterns were:

- `2-1`: 40
- `1-0`: 37
- `2-0`: 26
- `3-0`: 13
- `3-1`: 9

The most common draw patterns were:

- `1-1`: 16
- `0-0`: 15
- `2-2`: 7

The model uses these as a gentle prior on top of the Poisson xG grid. It should make exact scores look more like World Cup football without changing the underlying team-strength logic too aggressively.

## External Data Layers

We added five lightweight data layers after the score calibration:

- `data/fifa_top20_rankings_2026_04_01.json`: top-20 FIFA/Coca-Cola Men's World Ranking table as of 2026-04-01, with FIFA's official ranking page as the primary reference and Wikipedia's table as the extractable source.
- `data/recent_form_adjustments.json`: recent national-team form from public international results, using matches from 2024-01-01 through 2026-06-10. Friendlies are downweighted to 65%.
- `data/squad_profile_adjustments.json`: player-level squad profile from the 2026 World Cup squad tables, using average age and average caps.
- `data/coach_adjustments.json`: subjective coach/staff adjustment layer, capped to small rating nudges.
- `data/venue_context_2026.json`: match-specific venue climate bands and small group-stage travel adjustments.

The model treats these as small nudges on top of the sheet's existing strength value. This keeps the bracket stable while adding more current information.

The score selector was also adjusted so high-xG favorites can still produce occasional `3-0` and `3-1` style results. Historical score calibration should guide the model, not ban higher-margin wins.

## Coach Adjustment

The coach layer is intentionally small and reviewable. It does not try to forecast purely from manager reputation. Instead, it converts a few qualitative signals into a capped rating nudge:

- elite manager pedigree
- tournament continuity
- tactical clarity and squad fit
- game-state management, substitutions, set pieces, and penalty preparation
- evidence of overperforming or underperforming squad talent

Stress test after implementation:

- Deterministic group picks changed: `1`.
- Deterministic knockout picks changed: `0`.
- Champion stayed France.
- England's Monte Carlo champion probability moved from `11.5%` to `12.4%`.

## Venue Climate and Travel

The 2025 FIFA Club World Cup is the closest recent comparison for the climate layer: it was played in the United States from 2025-06-14 through 2025-07-13, used a large multi-city format, and produced real heat/humidity concerns. AP and Guardian reporting described high heat, humid conditions, shortened training sessions, cooling-break thresholds, and concerns about similar conditions in 2026.

We use that evidence only to set the direction and scale of the adjustment. Club results are not treated as national-team strength data.

The venue layer works like this:

- Assign each 2026 venue a summer stress band from `0.00` to `0.90`.
- Reduce stress for indoor or roof-mitigated venues such as Vancouver, Los Angeles, Houston, Dallas, and Atlanta.
- Apply a small positive or negative match-specific adjustment based on each team's climate adaptation profile.
- Add a small group-stage travel penalty when a team changes regions between matches.
- Omit knockout travel penalties because the exact path is uncertain, while still using knockout venue climate bands.

Stress test after implementation:

- Largest single group-stage venue adjustment: `0.61` rating points.
- Deterministic group picks changed: `0`.
- Champion stayed France.
- Monte Carlo champion probabilities moved only slightly, with France going from `18.6%` to `18.2%`.

## AI Analyst Overlay

An AI analyst overlay would be a bounded narrative adjustment after the statistical baseline. For example:

- baseline says Mexico beats South Africa 68% of the time
- an analyst note says Mexico's host opener may carry extra emotional pressure
- the overlay is allowed to nudge the probability slightly, not rewrite the model

This is useful for incorporating context that is hard to encode numerically, but it can also become vibes-in-a-trench-coat if unconstrained. For this project, it should stay deferred until the statistical pieces are more stable.

## Deferred Ideas

- Full FIFA ranking table ingestion, beyond the extractable top 20
- Player-level club minutes and club form features
- AI analyst overlay

The next best upgrade is probably player-level club minutes and club form features, but that requires a reliable player-level source.
