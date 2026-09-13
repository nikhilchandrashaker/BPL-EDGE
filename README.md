# BPL EDGE v1 dataset

Built from the uploaded Cricsheet JSON archive.

## Method
- Unit: team-season.
- No-result matches are excluded entirely from EDGE calculations.
- Completed sample size n = count of team-match rows with a non-null winner/outcome.
- Ordinary metrics: shrink only when n < 6 using X* = n/(n+6) X + 6/(n+6) season mean.
- Fielding: always shrunk with k=6 because named-fielder attribution is noisy.
- Pressure metrics: event-count shrinkage with k=6; teams with no eligible events revert to the season mean.
- All metrics are z-scored within season.
- Lower-is-better metrics are sign-inverted.
- Six components are equal-weighted (1/6 each).
- Batting excludes redundant run_rate because run_rate = 6 * runs_per_ball.
- Run prevention uses runs_allowed / legal_balls_bowled.
- EDGE 0-100 is an empirical-percentile transform fit only on pre-2026 team-seasons in this league package; 2026 is not used for calibration.

## Components
1. Batting: runs_per_ball, dot_ball_rate_bat
2. Bowling: wickets_taken per match, dot_ball_rate_bowl
3. Run prevention: powerplay economy, death economy, runs allowed per legal ball
4. Explosiveness: boundary rate per ball, death run rate, powerplay run rate
5. Fielding: fielding dismissals per match
6. Pressure: close-game win pct, chase win pct, close-game run differential

## Files
- team_match_edge.csv: audit-level team-match inputs and derived rates.
- team_season_edge_v1.csv: final component scores and EDGE v1.
- excluded_matches.csv: matches excluded because no winner/outcome was recorded.
