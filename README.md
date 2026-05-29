# sp-criminal-incidence

**Criminal Incidence Rating — São Paulo, Capital**

A single-file interactive dashboard ranking all **93 police precincts (DPs)** of São Paulo by criminal incidence per capita. Covers Q1 2024, Q1 2025, and Q1 2026 with a logarithmic safety index, an interactive Leaflet map, regional breakdowns, and three analytical flags.

---

## Overview

Most public crime rankings sort precincts by raw incident volume — which systematically penalises dense, populous areas regardless of actual per-capita risk. This dashboard replaces volume rankings with a **0–100 Safety Index** that weights incidents by severity, normalises by territorial population, and applies a logarithmic scale to give meaningful resolution across the full distribution.

The result is a single comparable number per precinct, stable across years because it always uses Q1 (January–March) as the reference window.

---

## Features

### Three tabs

| Tab | What it shows |
|---|---|
| **Dashboard** | Full ranked table of all 93 DPs with scores, trends, and flag indicators. Sortable and filterable. |
| **Map** | Interactive choropleth map (Leaflet + GeoJSON polygons). Filter by rating tier, region, or analytical flag. Click any precinct to open its detail panel. |
| **City** | City-level summary: aggregate score trend (Q1 2024 → 2026), regional accordion breakdowns, coverage metrics for each flag. |

### Safety Index

- **Severity weights** — homicide (10×) down to minor theft (1×); 9 crime types weighted
- **Per-capita normalisation** — weighted score divided by territorial population (Pop_DP), scaled to per 100,000 inhabitants
- **Logarithmic scale** with fixed anchors: 150 pts/100k → score 100 (safest); 35,000 pts/100k → score 0 (most critical)
- **Fixed anchors** are calibrated above/below all observed values so no real DP reaches the extremes

### Rating levels

| Range | Level |
|---|---|
| 80–100 | Very Low |
| 60–79 | Low |
| 40–59 | Medium |
| 20–39 | High |
| 0–19 | Very High |

### Analytical flags

Three interpretive layers that do not affect the index score:

**↕ Volatile Trend** — flags DPs whose year-over-year score movement is unusually large relative to the city's own annual variation. Thresholds are asymmetric: deterioration is flagged at a lower bar than improvement.

```
Improving threshold   = min((2 × Municipal Variation) + 1, 5) pts
Deteriorating threshold = min(2 × Municipal Variation, 4) pts
```

For Q1 2026 (Municipal Variation = +1.5 pts): improving ≥ 4.0 pts · deteriorating ≤ −3.0 pts · **14 DPs flagged**.

**⚑ Composition Alert** — flags DPs where at least one sensitive crime category is significantly concentrated relative to the citywide share. Per-category calibrated thresholds (1.75× to 2.5× city share + minimum absolute delta). **27 DPs flagged (29%)**.

**🌙 Nocturnal Concentration** — flags DPs where night-time incidents (18h–06h) represent ≥ 72% of Q1 total, well above the citywide average of 66.6%. Based on Q1 2025 SSP-SP data. **9 DPs flagged**.

---

## Data sources

| Source | Period | Content |
|---|---|---|
| SSP-SP monthly worksheets | Q1 2026 | Occurrences by DP, crime type, and month |
| SSP-SP SPDadosCriminais CSV | Jan–Jun 2024, Jan–Jun 2025 | Individual BO records with DP, date, and crime type |
| SSP-SP SPDadosCriminais | Q1 2024, Q1 2025 | Occurrences by time-of-day period (Morning / Afternoon / Night) |
| SEADE | 2000–2050 | Population estimates and projections by municipal district |

**Coverage:** 93 DPs · São Paulo Capital · ~110k weighted incidents per Q1.

Population assignment: 47 DPs matched directly to SEADE municipal districts by name; 46 DPs use prototype estimates by geographic proximity.

---

## File structure

The entire dashboard is a **single self-contained HTML file** (~428 KB). No build step, no server, no dependencies to install.

```
criminal_incidence_rating_engine.html   # open in any modern browser
README.md
metodologia_safety_rating_sp_v5.docx    # full methodology document
auditoria_criminal_incidence_rating_engine_v2.docx  # data audit report
```

All data (RAW, BREAKDOWN, PERIOD_DATA, GEOJSON polygons, COORDS) is embedded as JavaScript constants. External dependencies loaded from CDN at runtime:

- [Leaflet](https://leafletjs.com/) — map rendering
- [Chart.js](https://www.chartjs.org/) — bar charts

---

## Usage

```bash
# No installation required — just open the file
open criminal_incidence_rating_engine.html
```

Or serve locally if your browser restricts local file access:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

---

## Methodology

See [`metodologia_safety_rating_sp_v5.docx`](./metodologia_safety_rating_sp_v5.docx) for the full methodology document, covering:

- Severity weight table (all 9 crime types)
- Per-capita normalisation and logarithmic scale derivation
- Fixed anchor calibration
- All three analytical flag algorithms with their formulas
- Population assignment and SEADE mapping
- Limitations and caveats

Data validation is documented in [`auditoria_criminal_incidence_rating_engine_v2.docx`](./auditoria_criminal_incidence_rating_engine_v2.docx):

- Count accuracy: −0.35% global deviation vs SSP-SP source (Q1 2026)
- Ranking validity: Spearman ρ = −0.711 to −0.855 across all three years (p < 10⁻¹⁵)
- Composite error margin: ±2.78% on the per-100k rate · ±1.5 pts on the 0–100 index

---

## Adapting to another city

The dashboard is built around a `CITY_CONFIG` object and a set of data constants. To adapt it:

1. Replace `RAW` with your city's precinct/sector data (same field schema)
2. Replace `GEOJSON_POLYGONS` and `COORDS` with your territory boundaries
3. Update `CITY_CONFIG` (city name, period label, unit labels, source string)
4. Recalibrate severity weights and population data as needed

---

## Coverage and confidence

| Dimension | Result |
|---|---|
| DP coverage | 100% — 93/93 matched across all three years |
| Count accuracy (Q1 2026) | −0.35% global · max −1.22% per DP |
| Ranking correlation (Q1 2024) | ρ = −0.711 (p < 10⁻¹⁵) |
| Ranking correlation (Q1 2025) | ρ = −0.758 (p < 10⁻¹⁸) |
| YoY direction consistency | 84% of DPs (78/93) |
| Composite error margin | ±2.78% rate · ±1.5 pts index |

Overall confidence level: **HIGH** — suitable for comparative analysis, trend studies, and resource prioritisation.

---

## License

Data sourced from SSP-SP (public) and SEADE (public). Index methodology and dashboard implementation are proprietary.
