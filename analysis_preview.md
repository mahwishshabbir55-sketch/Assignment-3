# Statistical Analysis Preview

## Research Question
**Do natural disaster event counts differ on rainy days versus non-rainy days in the Los Angeles region?**

## Outcome Variable
`event_count` — the number of NASA EONET natural events (wildfires + severe storms) recorded on a given day.

## Grouping Variable
`rainy_day` — a binary indicator (1 = precipitation > 0 mm, 0 = no precipitation) derived from Open-Meteo weather data for Los Angeles.

## Binary Variable Created
`event_day` — equals 1 if at least one EONET event occurred on that date, 0 otherwise. This was created to support a proportion-based z-test: we can compare the proportion of "event days" across rainy vs non-rainy days, or across seasons.

## Hypotheses

### Two-sample t-test
- **H₀:** The mean daily event count is the same on rainy days and non-rainy days (μ_rainy = μ_dry).
- **H₁:** The mean daily event count differs between rainy and non-rainy days (μ_rainy ≠ μ_dry).

### Proportion z-test
- **H₀:** The proportion of event days is the same on rainy days and non-rainy days (p_rainy = p_dry).
- **H₁:** The proportion of event days differs between rainy and non-rainy days (p_rainy ≠ p_dry).

### One-sample t-test
- **H₀:** The mean daily event count equals a benchmark value of 5 (μ = 5).
- **H₁:** The mean daily event count is different from 5 (μ ≠ 5).

## Recommended Tests
- **Two-sample t-test** on `event_count` grouped by `rainy_day` — tests whether weather conditions are associated with different levels of natural event activity.
- **Proportion z-test** on `event_day` grouped by `season` — tests whether the likelihood of any event occurring varies by season.
- **One-sample t-test** on `event_count` — tests whether the observed mean differs from a hypothesized benchmark.

These tests are appropriate because the Gold dataset contains continuous numeric columns (`event_count`, `temp_max`), binary columns (`rainy_day`, `event_day`), and categorical grouping columns (`season`, `rainy_day`).
