# BPL EDGE v1 dataset

A 0–100 composite measure of franchise strength in the Bangladesh Premier
League, same methodological family as MLC EDGE / PSL EDGE — six
independently interpretable components, season-adjusted, frozen
calibration for out-of-sample scoring.

Built from the uploaded Cricsheet JSON archive.

![BPL EDGE heatmap](visuals/bpl_edge_heatmap.png)

## ⚠️ Calibration bug found and corrected

The uploaded `team_season_edge_v1.csv` computed its 0–100 percentile
transform across **all 80 team-seasons, including `2025/26`** — the season
that was supposed to be held out as frozen out-of-sample, per this
package's own stated method. Likely cause: BPL's final season is labeled
`2025/26` (a season spanning two calendar years) rather than a clean
`2026`, so whatever logic was meant to detect "the current season to
exclude" didn't match it.

Verified by reverse-engineering the exact transform from PSL's file (which
*was* done correctly — historical rows follow
`EDGE = (rank(EDGE_z) − 0.5)/N_hist × 100` fit only on pre-current-season
rows, with the held-out season mapped in afterward via
`percentileofscore` against that frozen distribution) and confirming
BPL's stored values instead matched the same formula computed over all 80
rows at once.

**Effect:** small numerically in most rows, but real — under the leaked
version, Rajshahi Warriors (91.875) and Chattogram Royals (90.625) show as
separate ranks in 2025/26. Their underlying composite z-scores are
effectively identical; under a correctly frozen transform they tie exactly
at 91.89, both rank 1.

`team_season_edge_v1.csv` in this folder has been **recomputed** with
2025/26 properly held out of calibration. `EDGE_z` (the pre-transform
composite) is untouched — only the final `EDGE` column and
`rank_within_season` were corrected.

## Coverage — and why this league needed a different visual

12 seasons (2011/12–2025/26, with real gaps: no BPL 2013/14–2014/15 or
2020/21), but **33 distinct team names**, not the ~6–8 stable franchises
seen in MLC or PSL. Only 7 names appear in 4+ seasons; 14 names appear
exactly once. This reflects BPL's actual history of teams folding,
rebranding, and re-entering under new ownership between seasons — a
trajectory line chart (one line per team across seasons) would either be
an unreadable 33-color tangle or misleadingly imply continuity between,
say, "Dhaka Dynamites" and "Dhaka Capitals" that the data has no basis for
claiming.

Instead: a **heatmap**, one row per team name (ordered by how many seasons
they appear, most persistent first), one column per season, colored by
EDGE. Gaps are genuinely blank — a team not playing a season, not a zero
score. This makes the churn itself legible rather than hiding it.

3 matches were excluded entirely from EDGE calculations — all ties, no
rain-outs this time:

| Season | Match | Result |
|---|---|---|
| 2018/19 | Khulna Titans vs Chittagong Vikings | tie |
| 2019/20 | Cumilla Warriors vs Sylhet Thunder | tie |
| 2025/26 | Rajshahi Warriors vs Rangpur Riders | tie |

Full list with match IDs: `excluded_matches.csv`.

## Method

Identical to PSL EDGE v1 (same six components, same shrinkage constants,
same z-scoring and sign-inversion rules) — see `PSL EDGE v1`'s README for
the full formula writeup. Only the calibration boundary differs by
necessity: **`2025/26`** is BPL's held-out season, in place of a literal
`2026`.

## Files

- `team_match_edge.csv` — audit-level team-match inputs and derived rates (as uploaded, unchanged).
- `team_season_edge_v1.csv` — final component scores and EDGE v1, **corrected** calibration (see above).
- `excluded_matches.csv` — matches excluded because no winner/outcome was recorded (as uploaded, unchanged).
- `visuals/bpl_edge_heatmap.png` — franchise-season EDGE heatmap.

## What this package does not do

Same non-goals as MLC EDGE v1 and PSL EDGE v1: no recency weighting, no
opponent adjustment or in-season rolling updates, no weights fit to
observed wins, no roster/player-availability effects — and, specific to
BPL, **no attempt to link rebranded franchises into a single lineage**.
Each team name is scored as its own entity, which is the only defensible
choice given the data, but means "franchise strength over time" isn't a
meaningful question for most rows here the way it is for MLC's or PSL's
stable rosters.
