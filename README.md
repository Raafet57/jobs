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


    ## 🔗 Cascade & Dependency Risk (US + France)

  ### Methodology: Leontief Input-Output Model

  Both cascade visualizations use a **Leontief input-output (I-O) model** to compute how AI-driven contraction in high-exposure sectors propagates demand shocks through the economy:

  1. **Technical coefficients matrix (A)**: Loaded from checked-in data extracts of official government I-O tables:
     - **US**: BEA 2022 Input-Output Accounts, Use of Commodities by Industries, Sector Level (16 sectors)
       - Source: [bea.gov/industry/input-output-accounts-data](https://www.bea.gov/industry/input-output-accounts-data)
     - **France**: INSEE 2020 Comptes nationaux, Tableau des entrées intermédiaires (TEI), nomenclature A17 (17 branches)
       - Source: [insee.fr/fr/statistiques/2832834](https://www.insee.fr/fr/statistiques/2832834)

  2. **Leontief inverse L = (I − A)⁻¹**: Total requirements matrix computed via Gaussian elimination. Column sums give Type I output multipliers (US range: 1.24–1.80x; France range: 1.21–2.25x).

  3. **Sector direct exposure**: Computed as employment-weighted average of occupation AI exposures within each sector, using occupation-to-sector mapping weights derived from:
     - **US**: BLS Occupational Employment and Wage Statistics (OEWS) industry matrix
     - **France**: DARES Familles Professionnelles (FAP 2009) to NAF/NACE correspondence

  4. **Cascade scoring**: When AI contracts high-exposure sectors (sector avg ≥ 5.0), the Leontief inverse propagates demand shocks. Each occupation's cascade risk:
     `cascade_risk = max(direct, round(direct + (sector_cascade − direct) × 0.6))`
     where sector_cascade is the employment-weighted cascade exposure across the occupation's mapped sectors.

  ### Key Results

  | Metric | US (BEA 2022) | France (INSEE 2020) |
  |--------|---------------|---------------------|
  | I-O sectors | 16 | 17 |
  | Occupations | 342 | 225 |
  | Direct avg | 4.91/10 | 3.64/10 |
  | Cascade avg | 5.35/10 | 4.04/10 |
  | Cascade uplift | +0.44 | +0.40 |
  | High-risk (≥7) direct | 49.0M (34%) | 4.2M (15%) |
  | High-risk (≥7) cascade | 49.3M (34%) | 4.5M (16%) |

  ### Data Files

  Source data extracts and mappings are checked into `scripts/data/`:
  - `bea-2022-use-sector.json` — BEA Use table technical coefficients (16×16)
  - `insee-2020-tes-a17.json` — INSEE TEI coefficients (17×17)
  - `bls-to-bea-sector-map.json` — BLS occupation category → BEA sector weights
  - `fap-to-insee-sector-map.json` — FAP category → INSEE branch weights
  - `PROVENANCE.md` — Full source documentation and access URLs

  ### Scripts

  - `scripts/src/generate-cascade-io.ts` — Loads I-O tables from data files, computes Leontief inverse, derives sector cascade multipliers, and scores all occupations for both countries.

  ---

  