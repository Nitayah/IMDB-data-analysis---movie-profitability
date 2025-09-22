# IMDB Producer Profitability Dashboard

## 1) Purpose & Audience
An interactive Power BI report for film producers to explore the characteristics related to films’ profitability (budget tier, genre, rating, source, etc.), and to identify directors/distributors with relevant experience and strong results.  
Defaults emphasize reliable, coverage-safe data; users can opt into broader views.

---

## 2) Data & Model
**Source**
- `imdb_raw` (main facts; 3,195 rows originally)

**Core relationships**
- `Date[Date] (1)` → `imdb_raw[Release Date Clean] (*)`

**Keys**
- Movie Key (`Title (Year)`, e.g., *Crash (1985)*) for disambiguation

---

## 3) Data Preparation (Power Query)
- **Headers / types:** Promoted headers; set data types; cleaned text (`Trim`/`Clean`).
- **Dates:** `Release Date Clean` parsed *en-GB* (dd/mm/yyyy).  
  Month-only → 1st of month; year-only → Jan 1st. QA columns used during validation then hidden.
- **Revenue hygiene:**
  - Replaced `"Unknown"` with blanks before numeric conversion.
  - Derived `RevenueChannel`:  
    - No revenue (WW blank/0 and DVD ≤0)  
    - DVD only (WW 0/blank and DVD >0)  
    - Box Only (DVD 0/blank and WW >0)  
    - Box & DVD (rest)
- **Flags (columns):**
  - `IsReleased` (parsed date exists)  
  - `Released6MonthsAgo` (recency gate)  
  - `HasBudget` (>0)  
  - `HasAnyRevenue` (WW or DVD >0)  
  - `US_gt_WW` (data inconsistency)  
  - `USOnlyRelease`, `WWOnlyRelease`  
  - `Pre-/Post-Streaming` (before/after Netflix launch in 2007)  
  - **Master include flag:**  
    `IncludeInProfitAnalysis = IsReleased & Released6MonthsAgo & HasBudget & HasAnyRevenue & NOT US_gt_WW & RevenueChannel≠No revenue`
- **Standardizations:**
  - `Based On`: strips `"based on …"` prefix (e.g., `"based on a book"` → `"book"`).  
  - `Release Year`, `Movie Key (TitleYear)` added for keys/labels.

---

## 4) Measures & Columns (Analysis Layer)
**Row-level columns (for distributions/filters)**
- Pre-streaming (year < 2007) vs. post-streaming (year ≥ 2007).  
- Budget Band: `"Low (<$10M)"`, `"Mid ($10–50M)"`, `"High (≥$50M)"`.  
  Verified distribution: Mid ~40%, Low/High ~30% each.  
- ROI per Movie = `(WW + US DVD − Budget) / Budget`

**Key measures (examples)**
- Totals:  
  `[Total Revenue] = [WW Gross Sum] + [US DVD Sum]`  
  `[Profit] = [Total Revenue] − [Budget Sum]`
- Portfolio ROI:  
  `[ROI] = DIVIDE([Profit], [Budget Sum])`
- Per-movie medians:  
  `[Median ROI per Movie]`, `[Median Budget]`, `[Median Profit per Movie]`, `[Median Revenue per Movie]`
- Counts: `[Movie Count]`
- “Included” variants wrap the above with `IncludeInProfitAnalysis = TRUE()`

**Selectors / parameters**
- Metric Selector (field parameter) → swaps Y-axis metric (Median ROI, Total Revenue, Profit, etc.).  
- Attribute Selector (field parameter) → swaps category (Genre, Source Clean, MPAA, Creative Type, Distributor, Director, Year).  
- MinN → threshold for minimum sample size in visuals.  
- Data quality mode: default shows Years 2000–2010 (coverage-safe, data relevance); user can switch to “All years”.

---

## 5) Report Pages & How to Read
**A. KPIs (Exec)**  
Cards summarizing the current slice:
- Movie Count, Total Revenue, Median Profit, Portfolio ROI, Median ROI per Movie, Median Budget, US vs Non-US vs DVD revenue

**B. Playground**  
Flexible bar chart where the producer selects metric (Y) and category (X).
- Reset bookmark to default state
- Bars turn red when N < MinN

**C. What Movie to Make — Budget & Style**
- Trend strip: Titles per Year (context)
- Line by Year with Budget Tier split and metric selector
- Box & Whisker with a metric selector by a category selector (median + 1.5 IQR)
- Year filter; ROI outlier trim slicer; Reset button

**D. Partners — Directors & Distributors**
- Leaderboard bars with Director/Distributor and a success metric selector
- Min. Experience sliders per chart; bars color red when N below threshold
- Year filter; ROI outlier trim slicer; Reset button

**E. Data Cleaning / QA**
- Cards: total rows, included rows, excluded rows, % excluded
- Titles per Year bar
- One-paragraph note on exclusion logic (invalid/young dates, budget missing/0, no revenue, US>WW)

---

## 6) Usage Notes
- Defaults show a coverage-safe and comprehensible window (Year ≤ 2010, ROI ≤ 100); change the slicers to include more data.  
- Prefer Median ROI per Movie to rank categories; check N and IQR (predictability).  
- Use Total Revenue for scale context.  
- Use Reset at any time to return to recommended defaults.

---

## 7) QA & Validation
- Verified no duplicate Movie Key (`TitleYear`) for primary labeling  
- Sanity checks: US Gross ≤ Worldwide Gross, no negative Non-US Gross, no revenue rows excluded  
- Post-2010 coverage flagged as faulty/sparse in this file; model allows opt-in if data updates

---

## 8) Assumptions & Limitations
- No inflation or currency normalization; amounts assumed USD  
- ROI = `(WW + US DVD − Budget) / Budget`; other costs not modeled  
- Some dimensions (Director/Distributor) can be sparse; visuals enforce MinN to avoid misleading ranks  
- Extreme ROI handled via trim toggle and IQR display

