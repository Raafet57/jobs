# AI Exposure of the Job Market — US & France

  Analyzing how susceptible every occupation in the economy is to AI and automation. The project started with the US using Bureau of Labor Statistics data, and has since been extended with a full **France edition** built on real government census data.

  **Live demos:**
  - 🇺🇸 US: [joshkale.github.io/jobs](https://joshkale.github.io/jobs/)
  - 🇫🇷 France: [joshkale.github.io/jobs/france](https://joshkale.github.io/jobs/france/)
  - 🔗 Cascade Risk: [raafet57.github.io/jobs/cascade](https://raafet57.github.io/jobs/cascade/)

  ![AI Exposure Treemap](jobs.png)

  ---

  ## 🇺🇸 US Edition

  ### What's here

  The BLS OOH covers **342 occupations** spanning every sector of the US economy, with detailed data on job duties, work environment, education requirements, pay, and employment projections. We scraped all of it, scored each occupation's AI exposure using an LLM, and built an interactive treemap visualization.

  ### Data pipeline

  1. **Scrape** (`scrape.py`) — Playwright (non-headless, BLS blocks bots) downloads raw HTML for all 342 occupation pages into `html/`.
  2. **Parse** (`parse_detail.py`, `process.py`) — BeautifulSoup converts raw HTML into clean Markdown files in `pages/`.
  3. **Tabulate** (`make_csv.py`) — Extracts structured fields (pay, education, job count, growth outlook, SOC code) into `occupations.csv`.
  4. **Score** (`score.py`) — Sends each occupation's Markdown description to an LLM (Gemini Flash via OpenRouter) with a scoring rubric. Each occupation gets an AI Exposure score from 0-10 with a rationale. Results saved to `scores.json`.
  5. **Build site data** (`build_site_data.py`) — Merges CSV stats and AI exposure scores into a compact `site/data.json` for the frontend.
  6. **Website** (`site/index.html`) — Interactive treemap visualization where area = employment and color = AI exposure (green to red).

  ### Key files

  | File | Description |
  |------|-------------|
  | `occupations.json` | Master list of 342 occupations with title, URL, category, slug |
  | `occupations.csv` | Summary stats: pay, education, job count, growth projections |
  | `scores.json` | AI exposure scores (0-10) with rationales for all 342 occupations |
  | `html/` | Raw HTML pages from BLS (source of truth, ~40MB) |
  | `pages/` | Clean Markdown versions of each occupation page |
  | `site/` | Static website (treemap visualization) |

  ### Setup

  ```
  uv sync
  uv run playwright install chromium
  ```

  Requires an OpenRouter API key in `.env`:
  ```
  OPENROUTER_API_KEY=your_key_here
  ```

  ### Usage

  ```bash
  uv run python scrape.py        # Scrape BLS pages (cached in html/)
  uv run python process.py       # Generate Markdown from HTML
  uv run python make_csv.py      # Generate CSV summary
  uv run python score.py         # Score AI exposure (uses OpenRouter API)
  uv run python build_site_data.py  # Build website data
  cd site && python -m http.server 8000
  ```

  ---

  ## 🇫🇷 France Edition

  ### What's different

  The France edition uses **real government census data** throughout — not LLM-estimated employment figures. The goal was to make every number traceable to an official source.

  | Data point | Source |
  |------------|--------|
  | Occupation taxonomy | DARES FAP-225 (226 official French occupational families) |
  | Employment (2023) | DARES "Dynamique de l'emploi" 2018 × Eurostat LFS 2023 scale factor |
  | Salaries | DARES ACEMO/DADS median monthly wages (2017–2019) × 1.12 CPI adjustment |
  | AI exposure scores | Gemini 2.5 Flash, French labels + 0–10 score per FAP occupation |

  The total employment anchors to **28.42M workers** (Eurostat Labour Force Survey 2023, France), with 0% deviation after normalization.

  ### France data pipeline

  1. **Employment**: DARES publishes the "Dynamique de l'emploi" dataset with employment counts for all 226 FAP-225 occupational families. The 2018 figures are scaled by `28,417,700 / 27,312,000 ≈ 1.0405` to match the Eurostat 2023 LFS total.
  2. **Salaries**: DARES ACEMO survey provides median monthly wages by FAP family. Multiplied by 12 and adjusted by 1.12 for cumulative CPI inflation (2018–2023) to get approximate 2023 annual values.
  3. **AI scoring**: Each FAP-225 code is sent to Gemini 2.5 Flash in batches with its official French label. The model returns a clean occupation title, a 0–10 AI exposure score, a rationale, and a category assignment.
  4. **Deduplication**: Three FAP-87 aggregate codes (T2Z60, T2A, T2B) that overlap with their FAP-225 sub-codes (T2A60, T2B60) are removed to avoid double-counting ~1M workers.
  5. **Normalization**: After deduplication, all job counts are rescaled so the total exactly matches the 28.42M Eurostat anchor.

  The result: **225 occupations**, 28.42M workers, all employment and salary figures sourced from official French government statistics.

  ### France vs US — key findings

  | | 🇫🇷 France | 🇺🇸 USA |
  |---|---|---|
  | Occupations | 225 | 342 |
  | Total employment | 28.4M | 143M |
  | Weighted avg AI exposure | **3.6 / 10** | **4.9 / 10** |
  | Workers at high exposure (≥7) | **15%** | **34%** |

  France scores significantly lower than the US. This is not a data artifact — it reflects a genuine structural difference in labour market composition:

  **The 5 largest occupations in France by employment** are almost entirely low-exposure physical or care roles:

  | Occupation | Workers | AI Exposure |
  |------------|---------|-------------|
  | Agents d'entretien de locaux (cleaning) | 976K | 2/10 |
  | Aides-soignants (nursing assistants) | 785K | 1/10 |
  | Infirmiers (nurses) | 668K | 2/10 |
  | Aides à domicile (home carers) | 602K | 1/10 |
  | Assistantes maternelles (childminders) | 455K | 0/10 |

  These five occupations alone — 3.5M workers at exposure 0–2 — pull the national average down substantially. By contrast, the US has nearly 10M workers in "Customer service reps" and "General office clerks" at exposure 9/10.

  The difference comes down to France's proportionally larger public sector, healthcare system, and personal services economy, all of which are inherently resistant to AI substitution. When the previous version estimated employment figures with an LLM, it unconsciously overweighted visible white-collar work and missed this mass of low-exposure workers. Real census data tells a different story.

  ### France key files

  | File | Description |
  |------|-------------|
  | `france/data.json` | 225 FAP-225 occupations with employment, salary, AI score |
  | `france/index.html` | Standalone treemap visualization (no build step required) |

  ---


    ## 🔗 Cascade & Dependency Risk (NEW)

    ### The insight

    Direct AI exposure only tells half the story. When AI eliminates white-collar office work, the economic shockwave ripples through the entire economy:

    - **Office buildings empty** → janitors, security guards, maintenance workers lose jobs
    - **Workers lose income** → restaurants, retail, personal services lose customers
    - **Commercial construction halts** → construction workers lose projects
    - **Families lose health insurance** → healthcare demand shifts
    - **Dual-income families can't afford daycare** → childcare workers displaced

    ### The numbers

    | Metric | Direct AI Risk | Cascade Risk |
    |--------|---------------|--------------|
    | Weighted avg score | **4.9 / 10** | **7.4 / 10** |
    | Workers at high risk (≥7) | **49M (34%)** | **99M (69%)** |
    | Exposed wage bill | ~$2.5T | ~$4.7T |

    The cascade model nearly **doubles** the number of workers at high risk — from 49M to 99M.

    ### Biggest cascade jumps

    | Occupation | Direct | Cascade | Jump | Workers |
    |-----------|--------|---------|------|---------|
    | Janitors & building cleaners | 1 | 6 | +5 | 2.4M |
    | Home health & personal care aides | 2 | 7 | +5 | 4.3M |
    | Waiters & waitresses | 3 | 8 | +5 | 2.3M |
    | Childcare workers | 2 | 7 | +5 | 992K |
    | Food & beverage serving workers | 3 | 7 | +4 | 5.0M |
    | Cooks | 3 | 7 | +4 | 2.8M |
    | Construction laborers | 1 | 6 | +5 | 1.6M |
    | Registered nurses | 4 | 8 | +4 | 3.4M |

    ### Methodology

    For each of the 342 BLS occupations, Gemini 2.5 Flash estimates the **derived demand percentage** (0–100): what fraction of this occupation's total employment depends on the continued existence of high-exposure white-collar/office work.

    Two dependency channels are analyzed:
    1. **Employer channel**: % of workers employed by organizations serving high-exposure sectors (e.g., janitors in office buildings)
    2. **Consumer channel**: % of revenue from spending by high-exposure workers (e.g., restaurants near business districts)

    The cascade risk score is then:

    \`\`\`
    cascade_risk = max(direct, round(direct + (10 - direct) × derived_pct/100 × 0.8))
    \`\`\`

    This ensures cascade risk is always ≥ direct risk, and the derived demand amplifies the remaining "room to grow" above the direct score.

    ### Cascade key files

    | File | Description |
    |------|-------------|
    | `cascade/data.json` | 342 occupations with both direct + cascade scores |
    | `cascade/index.html` | Interactive treemap with Direct/Cascade toggle |

    **Live demo:** [raafet57.github.io/jobs/cascade](https://raafet57.github.io/jobs/cascade/)

    ---
    ## AI exposure scoring rubric

  Each occupation is scored on a single **AI Exposure** axis from 0 to 10, measuring how much AI will reshape that occupation. The score considers both direct automation (AI doing the work) and indirect effects (AI making workers so productive that fewer are needed).

  A key signal is whether the job's work product is fundamentally digital — if the job can be done entirely from a home office on a computer, AI exposure is inherently high. Conversely, jobs requiring physical presence, manual skill, or real-time human interaction have a natural barrier.

  **Calibration examples:**

  | Score | Meaning | US examples | French examples |
  |-------|---------|-------------|-----------------|
  | 0–1 | Minimal | Roofers, janitors | Assistantes maternelles, aides-soignants |
  | 2–3 | Low | Electricians, nurses aides | Infirmiers, professeurs des écoles |
  | 4–5 | Moderate | Registered nurses, retail | Attachés commerciaux, ingénieurs IT |
  | 6–7 | High | Teachers, accountants | Cadres fonction publique, ouvriers manutention |
  | 8–9 | Very high | Software developers, paralegals | Secrétaires bureautiques, développeurs |
  | 10 | Maximum | Medical transcriptionists | — |
  