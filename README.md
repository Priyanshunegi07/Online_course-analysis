# 📊 Online Courses Insights — Power BI EDA Project

An exploratory data analysis and Power BI dashboard project built on a raw, multi-platform online-course dataset scraped from **Coursera, Future Learn, Udacity, and Simplilearn**.

---

## 1. Project Overview

| | |
|---|---|
| **Goal** | Clean, explore, and visualize a messy real-world dataset of online courses to surface platform, category, rating, and engagement trends |
| **Raw data** | `Online_Courses.csv` — 8,092 rows × 45 columns |
| **Deliverable** | `online_course_insights.pbix` — 1-page interactive Power BI dashboard |
| **Type** | Exploratory Data Analysis (EDA) + BI Reporting |
| **Tools** | Power BI Desktop, Power Query (M), DAX, Excel/Python (pre-analysis) |

This is a **union dataset**: four course-listing sites were scraped separately and stacked into one CSV, so each row only fills in the columns relevant to its own source platform. That single fact drives almost every cleaning decision in this project — see [Section 4](#4-data-quality-findings).

---

## 2. Repository / File Structure

```
online-courses-insights/
│
├── data/
│   └── Online_Courses.csv          # Raw scraped dataset (8,092 rows, 45 cols)
│
├── report/
│   └── online_course_insights.pbix # Power BI report file
│
├── docs/
│   ├── README.md                   # This file — GitHub documentation
│   ├── EDA_Report.docx             # Full structured EDA report
│   └── Project_Presentation.pptx   # Stakeholder / viva presentation
│
└── charts/                          # Exported EDA visuals (PNG)
```

---

## 3. Dataset at a Glance

| Metric | Value |
|---|---|
| Total records | 8,092 |
| Total columns | 45 |
| Source platforms | 4 (Future Learn, Coursera, Udacity, Simplilearn) |
| Unique course titles | 4,908 |
| Exact duplicate rows | 0 |
| Fully-populated columns | `Title`, `URL`, `Site` |

### Records per platform

| Site | Records | % of dataset |
|---|---|---|
| Future Learn | 4,843 | 59.9% |
| Coursera | 2,819 | 34.8% |
| Udacity | 282 | 3.5% |
| Simplilearn | 148 | 1.8% |

### Common columns vs. platform-specific columns

Only **`Title`, `URL`, `Short Intro`, `Duration`, `Site`** are shared across all four sources. Everything else belongs to one platform's schema:

| Platform | Columns unique to it (examples) |
|---|---|
| **Coursera** | `Category`, `Sub-Category`, `Course Type`, `Language`, `Skills`, `Instructors`, `Rating`, `Number of viewers` |
| **Future Learn** | `Courses`, `Weekly study`, `ExpertTracks`, `Topics related to CRM`, `Course Title/URL` |
| **Udacity** | `Program Type`, `Level`, `Prequisites`, `What you learn`, `School` |
| **Simplilearn** | `Program`, `Number of ratings`, `Price`, `COURSE CATEGORIES` |

This is why 30+ columns show **>90% missing values** — it isn't bad scraping, it's four schemas stacked into one table.

---

## 4. Data Quality Findings

- **Structural nulls, not random missingness.** ~38 of 45 columns are only populated for one source platform (see table above).
- **Mixed data types inside single columns.** `Rating` mixes `"4.9stars"` strings with occasional bare numbers; `Number of viewers` mixes plain integers with strings like `"108,885 reviews"`.
- **Duration is inconsistent across platforms.** Coursera uses `"Approx. 9 hours to complete"` / `"Approximately 3 months to complete"`; Future Learn and Udacity use different phrasing entirely — this column needs platform-aware parsing to normalize into hours.
- **`Instructors` is a flat comma-separated string** that also swallows credentials (e.g., `PhD`, `MBA`, `MD` appear as if they were instructor names after a naive split) — a good example of why exploding delimited text fields needs care.
- **High title duplication.** 3,184 duplicate titles / 2,819 duplicate URLs exist, mostly because Coursera lists the same course under multiple specializations.
- **`Price` is populated for only 65 rows (Simplilearn only)** — not usable as a general pricing feature without heavy imputation or scope reduction.

Full details, plots, and platform-by-platform breakdowns are in **`docs/EDA_Report.docx`**.

---

## 5. Power BI Report Structure

The `.pbix` contains a single dashboard page, **"Online Course Analysis"**, built from two model tables (`Online_Courses`, `Teachers`) with several calculated columns layered onto the raw data:

| Calculated field (DAX/Power Query) | Purpose |
|---|---|
| `course id` | Surrogate key for counting/aggregating rows |
| `Duration in hours` | Normalizes text durations into a numeric hour value |
| `Subtitle_lang_count` | Count of subtitle languages offered per course |
| `Count of skills Provided` | Count of skills listed per course |
| `Rank_category_by_avg_views` | Category ranked by average viewer count |
| `Instructor_rating` (Teachers table) | Rating rolled up per instructor |

### Visuals on the page

| Visual | Fields | Insight it answers |
|---|---|---|
| Slicer | `Category` | Filter the whole page by subject area |
| Bar chart | Course Type × Count of courses | How many Courses vs. Specializations vs. Certificates |
| Column chart | Sub-Category, Language × Sum of viewers | Which sub-categories/languages drive engagement |
| Word cloud | `Skills` | Most in-demand skills across all courses |
| Pie chart | Language × Course count | Language mix of the catalog |
| Pivot table | Category, Language × Rank by avg. views | Best-performing category/language combinations |
| Line chart | Subtitle language count × Viewers | Does more subtitle coverage correlate with more views |
| Clustered column | Instructor × Instructor rating | Top / bottom-rated instructors |
| Slicer | Sub-Category (Teachers table) | Filter instructor view by subject |
| Line chart | Duration (hours) × Viewer count | Does course length affect viewership |
| Pivot table | Category, Sub-Category × Skills count | Skill density by subject area |

---

## 6. Key Insights

- **Future Learn dominates by record volume** (60%) but **Coursera is the richest data source** — it's the only platform carrying category, rating, skill, and viewership fields.
- **Average Coursera rating is ~4.66/5** (median 4.7), and ratings cluster very tightly — most courses sit between 4.5 and 4.9, so rating alone barely differentiates course quality.
- **Rating and viewer count are only weakly correlated (~0.11)** — a course being highly rated does not mean it is highly viewed; popularity and quality move fairly independently here.
- **Business and Data Science dominate Coursera's catalog** (895 and 448 courses respectively), followed by Computer Science and Health.
- **`Data Analysis`, `Python Programming`, and `Machine Learning`** are the three most frequently taught skills across Coursera's catalog.
- Recurring instructor "names" like `PhD`, `Google Cloud Training`, and `Google Career Certificates` reveal that the `Instructors` field mixes individuals, organizations, and credentials — a cleanup target for any downstream analysis.

---

## 7. How to Reproduce / Explore

1. Clone this repo and open `report/online_course_insights.pbix` in **Power BI Desktop**.
2. To refresh from source: point Power Query at `data/Online_Courses.csv`, keep the platform-specific transformation steps, then hit **Refresh**.
3. To rebuild the EDA independently: load `data/Online_Courses.csv` in Python (`pandas`) or Excel and follow the cleaning steps documented in `docs/EDA_Report.docx`.

## 8. Tech Stack

`Power BI Desktop` · `Power Query (M)` · `DAX` · `Python (pandas, matplotlib)` for pre-modeling EDA

## 9. Author / Contact

Add your name, LinkedIn, and portfolio link here before publishing.

## 10. License

Dataset is publicly scraped course-listing data; verify original source terms before redistribution. Project code/report © you — add a license (e.g. MIT) if open-sourcing.
