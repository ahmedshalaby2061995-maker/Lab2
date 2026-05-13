# MoHRSD 6 KPIs — Upgrade Instructions

This patch adds the 6 MoHRSD quarterly KPIs to your Labor Market Agent.

## What's new

### New file
- **`tools/kpi_calculator.py`** — Computes all 6 KPIs with exact GASTAT cell references, parses Register-based xlsx files, and exposes a historical-series function.

### Modified files
- **`agent.py`** — Registers `compute_kpis` as the 8th tool; adds KPI awareness to system prompt; adds keyword-based fallback for rule-based mode.
- **`app.py`** — Adds `/api/kpis` and `/api/kpi-history/{id}` endpoints; updates `/api/upload` to auto-detect Register-based files (by filename) and route them to the right ingestion pipeline.
- **`templates/app.html`** — Adds the MoHRSD KPI section to the dashboard (6 cards with values, formulas, and sources), updates the Customize panel with 6 new toggles, fetches KPIs in parallel with summary on dashboard load.

## How to apply

1. **Backup your current files** (`agent.py`, `app.py`, `templates/app.html`).
2. **Copy these 4 files** into your project:
   ```
   tools/kpi_calculator.py     →  labor_agent/tools/kpi_calculator.py    (new file)
   agent.py                    →  labor_agent/agent.py                   (replace)
   app.py                      →  labor_agent/app.py                     (replace)
   templates/app.html          →  labor_agent/templates/app.html         (replace)
   ```
3. **Restart uvicorn**:
   ```powershell
   .\.venv\Scripts\uvicorn app:app --host 0.0.0.0 --port 8000
   ```
4. **Upload the Register-based xlsx** to populate KPIs 5 and 6:
   - Open the app → Chat → drag in `Register-based_Labour_Market_Statistics-_Q4_2025_EN.xlsx`
   - The file is detected automatically by filename (must contain "register" or "register-based").
   - The KPIs section will populate immediately.

## What the agent knows now

You can ask any of these and the agent will use the `compute_kpis` tool:
- "What are the MoHRSD KPIs?"
- "Give me the 6 quarterly indicators."
- "What's the highly skilled Saudis percentage?"
- "Show me all the ministry indicators."

In LangChain mode, the LLM routes via the tool. In rule-based fallback mode, the same keywords trigger the same tool — so it works with or without an OpenAI key.

## The 6 KPIs (current Q4 2025 values)

| # | KPI | Value | Source |
|---|---|---|---|
| 1 | Total Unemployment Rate | 3.53% | GASTAT LFS Table 2-1, Col J |
| 2 | Saudi Unemployment Rate | 7.24% | GASTAT LFS Table 2-1, Col D |
| 3 | Total LFP Rate | 67.43% | GASTAT LFS Table 5-1, Col J |
| 4 | Saudi LFP Rate | 49.52% | GASTAT LFS Table 5-1, Col D |
| 5 | Highly Skilled Saudis | 42.37% | Register-based Table 3-5, Col D: (Managers 270,274 + Professionals 1,032,753) / Total 3,075,549 |
| 6 | Highly Skilled Expats | 9.57% | Register-based Table 3-5, Col G: (Managers 148,602 + Professionals 865,209) / Total 10,589,566 |

## New database table

The Register-based file is stored in a new `register_facts` table with columns:
`quarter, occupation, nationality, sex, count, source_file, ingested_at`

It's created automatically on first upload. No migration needed.

## New API endpoints

- `GET /api/kpis?quarter=2025-Q4` — Returns all 6 KPIs with values, formulas, sources.
- `GET /api/kpi-history/{id}` — Returns the historical time-series for KPI 1-6 (for sparklines).

Both auth-protected like the rest of the API.
