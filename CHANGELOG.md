# Changelog

- Record the new frontend journey test coverage and update the remaining validation priorities.

- Document shared Redis caches, distributed Open-Meteo quotas, client rate limits, and degraded operation.

## 0.3.0 - 2026-09-09

- Align the project synthesis and implementation status with the completed scoring, forecast, resilience, and interface work, including current verification gaps.

## 0.2.0 - 2026-09-08

- Align the scope, journey, scoring rules, and project synthesis with the validated three-score, pollen, data timestamp, and incomplete-data behavior.

- Complete the validated interface states for loading, refresh failures, stale or incomplete data, pollen availability, and unsuccessful city searches.
- Add a repository version file and an interactive release script that requires a clean repository and pending changelog entries, dates the release, and creates the gitmoji release commit.
- Add shared agent instructions for French documentation and the validation, changelog, and gitmoji commit workflow.
- Initialize the changelog from the existing product and technical documentation.

## 0.1.0 - 2026-09-07

- Document the product vision, target users, first-version scope, and city-to-activity forecast journey validated on September 6.
- Define the planned weather, air quality, and overall scores, including activity-specific thresholds, weights, progressive caps, alerts, and explanations.
- Specify European AQI categories and conversion rules, with pollen concentrations displayed separately from the scores.
- Define 24 consecutive one-hour slots, local timezone and daylight-saving handling, and weather, air, and pollen aggregation over each outing.
- Specify refresh behavior, a 15-minute cache, stale-data handling, and rules for missing or invalid data.
- Record the validated screen summary, detailed condition blocks, chronological slot list, and best-slot behavior; keep loading and refresh presentation marked as pending.
- Add a documentation index and project synthesis covering repository organization, architecture, implementation status, and priorities.
- Distinguish validated product rules from the existing prototype, which still requires implementation updates.
