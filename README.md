# Zomato Bangalore Restaurant Analysis

> **Status:** Analysis complete — Tableau dashboard in progress

---

## Business Problem

Zomato's Business Development team needs a data-driven market overview of Bangalore's restaurant landscape to:

1. Identify underserved localities with high demand but low supply for new restaurant partnerships
2. Help BD team prioritise which locations, cuisines and restaurant types to target for onboarding

**Core analytical question:**
> Which locations have high demand but low supply of restaurants — indicating unmet market opportunity?

```
Demand proxy  → Mean votes per location (customer engagement signal)
Supply measure → Restaurant count per location
Opportunity   → High demand-supply ratio = underserved area
```

> **Data Limitation:** Votes is used as a proxy for demand. Higher votes indicate greater customer engagement — not exact order count. Findings should be validated with actual order volume data from Zomato's internal systems.

> **Rating Limitation:** 7,775 null ratings (15%) were filled with median value of 3.7 during cleaning. This creates low variation in location-wise rating analysis. Rating comparisons should be interpreted with caution.

---

## Dataset

- **Source:** Zomato Bangalore Restaurants — Kaggle
- **Shape:** 51,717 rows × 17 columns
- **Each row represents:** One restaurant listing on Zomato Bangalore

---

## Tools Used

- Python (Pandas, NumPy, Matplotlib, Seaborn, SciPy)
- Tableau Public
- Dataset: Zomato Bangalore Restaurants (Kaggle)

---

## Column Assessment

### Columns Used for Analysis

| Column | Description | Why Useful |
|---|---|---|
| `name` | Restaurant name | Identify top restaurants by area |
| `online_order` | Whether restaurant accepts online orders | Online vs offline split — impact on ratings |
| `book_table` | Whether table booking is available through Zomato | Indicates premium/fine dining segment |
| `rate` | Restaurant rating out of 5 | Quality signal per location |
| `votes` | Total number of votes received | Demand proxy — higher votes = greater engagement |
| `location` | Area/locality of restaurant | Primary supply analysis dimension |
| `rest_type` | Type of dining experience (Quick Bites, Casual Dining, Fine Dining etc) | Identifies restaurant type gaps per location |
| `cuisines` | Cuisine types served | Measures cuisine supply — identifies underserved cuisines |
| `approx_cost(for two people)` | Approximate cost for two people in INR | Price positioning by location |
| `listed_in(type)` | How restaurant is listed on Zomato platform | Listing type distribution analysis |

### Columns Excluded from Analysis

| Column | Reason for Exclusion |
|---|---|
| `url` | Web link — no analytical value |
| `address` | Too granular — location column is sufficient |
| `phone` | Contact information — no analytical value |
| `dish_liked` | 54% null values — unreliable for drawing conclusions |
| `reviews_list` | Raw text — requires NLP, out of scope |
| `menu_item` | No structured insights possible |
| `listed_in(city)` | Redundant — all restaurants are in Bangalore |

---

## Data Cleaning

**Raw data:** 51,717 rows × 17 columns

### Cleaning Steps Performed

| Column | Issue | Action |
|---|---|---|
| `rate` | Stored as "4.1/5" string, "NEW" and "-" values | Extracted numeric value, converted to float, filled 7,775 nulls with median (3.7) |
| `approx_cost` | Stored as "1,200" string with commas | Removed commas, converted to int, filled 346 nulls with median |
| `location` | "ITPL Main Road, Whitefield" and "Varthur Main Road, Whitefield" as separate entries | Merged both into "Whitefield" — same area, low counts |
| `rest_type` | 227 null values, comma-separated multi-values | Filled nulls with mode per location |
| `cuisines` | Comma-separated multi-values | Kept as-is in df1, exploded into df_cuisine for cuisine analysis |
| Duplicates | 19,824 duplicate rows identified | Removed, keeping first occurrence |

### Final Dataframes

| Dataframe | Shape | Purpose |
|---|---|---|
| `df1` (main) | 31,890 rows × 10 cols | Primary analysis — location, rating, cost, votes |
| `df_rest` | 37,183 rows × 10 cols | Restaurant type analysis (rest_type exploded) |
| `df_cuisine` | 81,229 rows × 10 cols | Cuisine analysis (cuisines exploded) |

> **Note:** df_rest and df_cuisine are exploded dataframes. Only COUNT and MEAN aggregations are valid — never SUM votes or rate on these dataframes as values are duplicated per row.

---

## Key Findings

### Finding 1 — Supply Analysis
- **BTM** (2,089) and **Whitefield** (2,013) have highest restaurant density — most saturated areas
- Top 5 saturated areas: BTM, Whitefield, Indiranagar, HSR, Marathahalli

### Finding 2 — Demand Analysis
- **Koramangala 5th Block** has highest avg votes (1,358) — most popular area
- Top demand areas differ significantly from top supply areas — indicating opportunity gaps

### Finding 3 — Demand-Supply Ratio (Core Finding)
After filtering locations with < 100 restaurants (unreliable sample), top opportunity locations:

| Location | Restaurants | Avg Votes | Avg Rating | Demand-Supply Ratio |
|---|---|---|---|---|
| Koramangala 3rd Block | 107 | 728 | 4.1 | 6.80 |
| St. Marks Road | 228 | 1,081 | 4.1 | 4.74 |
| Cunningham Road | 256 | 962 | 4.0 | 3.76 |
| Lavelle Road | 379 | 1,190 | 4.2 | 3.14 |
| Church Street | 434 | 1,275 | 4.1 | 2.94 |

### Finding 4 — Cuisine Gap
- **Pizza = 0 restaurants** in both Koramangala 3rd Block and St. Marks Road
- Pizza ranks 13th overall in Bangalore with 1,506 restaurants citywide — demand exists
- Both top opportunity locations have low supply across ALL cuisines — not just pizza

### Finding 5 — Restaurant Type Gap
- Fine Dining represents only **0.8%** of all Zomato Bangalore listings (306 restaurants)
- Significant onboarding opportunity in premium segment

### Finding 6 — Cost Analysis
| Location | Avg Cost for Two | Segment |
|---|---|---|
| Lavelle Road | ₹1,428 | Premium |
| MG Road | ₹1,274 | Premium |
| Koramangala 3rd Block | ₹890 | Mid-Premium |
| Koramangala 6th Block | ₹693 | Budget-Mid |

### Finding 7 — Online Ordering T-Test
- Restaurants with online ordering rate significantly higher (3.76 vs 3.72)
- T-statistic = 8.35, p-value ≈ 0.0000 — statistically significant
- However practical difference is marginal (0.04 on 5-point scale)

---

## Recommendations

### Recommendation 1 — Location Priority
**Finding:** Koramangala 3rd Block and St. Marks Road have highest demand-supply gap in Bangalore

**Action:** BD team should prioritise onboarding new restaurant partnerships in these two locations

**Evidence:**
- Koramangala 3rd Block: ratio = 6.80 | restaurants = 107 | avg votes = 728 | avg rating = 4.1
- St. Marks Road: ratio = 4.74 | restaurants = 228 | avg votes = 1,081 | avg rating = 4.1

---

### Recommendation 2 — Cuisine to Target
**Finding:** Zero pizza restaurants in both top opportunity locations despite pizza being top 10 cuisine citywide

**Action:** BD team should actively onboard pizza restaurants in Koramangala 3rd Block and St. Marks Road

**Evidence:**
- Pizza supply in Koramangala 3rd Block = 0
- Pizza supply in St. Marks Road = 0
- Pizza citywide = 1,506 restaurants (13th most common) — demand exists

---

### Recommendation 3 — Restaurant Type + Location Strategy
**Finding:** Fine Dining is severely underrepresented (0.8%) and Lavelle Road has highest avg cost (₹1,428)

**Action:**
- Lavelle Road → target for Fine Dining onboarding (premium customer base)
- Koramangala 3rd Block → target for Casual Dining and Cafes (mid-premium at ₹890 avg)

**Evidence:**
- Fine Dining = only 306 listings out of 37,183 (0.8%)
- Lavelle Road avg cost = ₹1,428 (highest among opportunity locations)
- Koramangala 3rd Block avg cost = ₹890 (mid-premium segment)

---

### Advisory Finding — Online Ordering
**Finding:** Statistically significant but practically marginal difference in ratings

**Advisory:** BD team may prefer onboarding restaurants that accept online ordering (slightly higher ratings) but this should not be a mandatory criterion — location and cuisine gap remain primary selection criteria

---

## Project Structure

```
zomato-bangalore-analysis/
├── zomato_analysis.ipynb     → Complete analysis notebook
├── data/
│   ├── zomato.csv            → Raw dataset (Kaggle)
│   ├── zomato_location_metrics.csv   → Location summary for Tableau
│   ├── zomato_cuisine_filtered.csv   → Cuisine data for Tableau
│   └── zomato_main.csv       → Clean main data for Tableau
├── README.md
└── dashboard/
    └── tableau_link.txt      → Tableau Public link (coming soon)
```

---

## Analysis Sections

- [x] Data Understanding
- [x] Data Cleaning
- [x] Univariate Analysis
- [x] Bivariate Analysis (Demand-Supply Ratio + Cuisine Heatmap + Cost Analysis)
- [x] Statistical Testing (T-test: Online Ordering vs Rating)
- [x] Business Recommendations
- [ ] Tableau Dashboard (in progress)