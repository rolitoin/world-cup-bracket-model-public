# World Cup Bracket Model

![World Cup bracket model banner](assets/world-cup-bracket-model-banner.png)

Small prediction model for a 2026 World Cup bracket.

## Codexito FC

![Codexito FC mascot](assets/codexito-fc-mascot.png)

Codex FC's bracket agent is **Codexito FC**: a small football analyst mascot with a headset, a clipboard, and just enough confidence to argue a 50/50 knockout pick.

The current workflow:

1. Download a local copy of a relevant bracket spreadsheet.
2. Read the bracket's group fixtures, team-strength field, and knockout formulas.
3. Generate group-stage scores from a lightweight rating model.
4. Apply the sheet's own 8-best-third-place logic and knockout bracket path.
5. Generate knockout scores, penalties, a filled workbook, and a JSON summary.
6. Run a lightweight Monte Carlo simulation to estimate advancement and champion probabilities.

The score engine now uses a small expected-goals / Poisson probability layer inspired by public World Cup forecasting projects. See `docs/github_research_notes.md`.
Exact scores are gently calibrated against 2014, 2018, and 2022 World Cup score patterns in `data/score_calibration.json`.
The rating layer also uses FIFA top-20 rankings, recent national-team results, squad age/caps profiles, coach/staff adjustments, and venue-specific climate/travel context where available.

## Model Inputs

The model starts with the bracket's built-in `N` column strength value for every team, then applies light adjustments:

- Host advantage for Mexico, USA, and Canada
- Recent form / expert consensus signal
- FIFA top-20 ranking nudge
- Recent results from 2024-01-01 through 2026-06-10
- Squad age and caps profile
- Coach / staff adjustment
- Heat, travel, and summer adaptation
- Venue-specific climate and group-stage travel fatigue
- Youth / energy versus experience
- Upset profile for teams that may overperform in tournament conditions

The upset variable is intentionally modest. It should create a few plausible tournament surprises without turning the whole bracket into chaos.

## Current Result

The current generated bracket picks:

- Champion: France
- Runner-up: England
- Third place: Spain
- Semifinalists: France, Spain, England, Argentina

Monte Carlo champion probabilities from 5,000 simulations:

- France: 18.3%
- Spain: 18.2%
- England: 12.4%
- Brazil: 9.2%
- Argentina: 7.9%

Notable upset/risk picks:

- Morocco over Netherlands
- USA over Belgium
- England over Brazil on penalties
- Colombia ahead of Portugal in the group

## Files

- `bracket_model.py` - model and workbook writer
- `world_cup_bracket_model_summary.json` - generated summary of picks and rationale
- `exports/codex_fc_picks.json` - shareable Codex FC picks export for comparison/scoring
- `exports/codex_fc_picks.csv` - CSV version of the same picks export
- `data/score_calibration.json` - historical exact-score patterns from recent World Cups
- `data/fifa_top20_rankings_2026_04_01.json` - FIFA top-20 ranking data
- `data/recent_form_adjustments.json` - recent national-team form adjustments
- `data/squad_profile_adjustments.json` - squad age/caps profile adjustments
- `data/coach_adjustments.json` - lightweight coach/staff adjustment layer
- `data/venue_context_2026.json` - venue climate bands and small travel adjustments
- `sheet_download.xlsx` - local source workbook, ignored by Git
- `world_cup_bracket_model_picks.xlsx` - generated filled workbook, ignored by Git
- `docs/github_research_notes.md` - notes from similar GitHub projects and ideas adopted

## Retrospective Plan

After the tournament, compare actual results against:

- winner correctness
- scoreline direction and margin
- group qualification accuracy
- third-place qualifiers
- knockout path accuracy
- missed upsets and false upsets

Then tune the weights instead of rewriting the model from scratch.
