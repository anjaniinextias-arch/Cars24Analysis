# Cars24 Supply (A2I) — Seller Funnel Conversion Audit

> Builder-Analyst screening case: why did lead-to-inspection conversion fall from 37.5% (June) to 27.4% (August), and what should the business do about it?

**Tools:** Excel (master workbook, pivots, dashboard) · MySQL · CSV exports
**Data:** synthetic, ~50,000 leads across 6 cities, 1 Jun – 31 Aug 2026 (raw data is **not** included in this repo)

---

## 1. Business problem

Cars24 buys used cars directly from sellers:

```
Lead → Call & qualify → Slot booked → Inspector visits → Inspection done
     → Price quoted → Offer revised → Purchased → Resold
```

Each completed vendor inspection costs about ₹450, so lead-to-inspection conversion is the core metric. The business head reported a drop in conversion, flat spend, rising leads, disputed lead quality and an inspection count that Finance says is wrong. The task was to find out what is actually happening.

## 2. Approach

1. Loaded all raw exports into one master Excel workbook and into MySQL.
2. Audited data quality: duplicates, mixed date formats, two source systems with different timezones, orphan keys.
3. Answered the 12 graded questions (Part A).
4. Wrote SQL for rolling conversion, inspector anomaly detection, Finance reconciliation and lead cohorts (Part B).
5. Built analysis sheets, a dashboard and an action plan in the workbook.

## 3. Key findings (Part A answer sheet)

| # | Metric | Result | Note |
|---|---|---|---|
| A1 | August lead-to-inspection conversion (raw) | 27.88% (4,872 / 17,478) | See "Points to verify" |
| A2 | Unique sellers | 44,701 | Confirm the uniqueness key (phone vs customer ID) |
| A3 | Duplicate rows in inspections | 286 | |
| A4 | Inspections >20 km from seller pin and <5 min | _[406, 2.3%]_ |in August Fake cases increases to 206, and in July it was 190 in June it was 0 |
| A5 | % of A4 by vendor-partner inspectors | 23.38% | Confirm the base is the A4 set |
| A6 | Lucknow call connect rate after 21 Jul | 25.86% (by leads) / 25.97% (unique phones) | State which is the headline |
| A7 | Show-up rate, lead-to-slot wait > 48h | 71.34% (1,327 / 1,860, all appts) / 84.52% (1,327 / 1,570, excl. cancelled) | State which is the headline |
| A8 | Contribution (₹), 2012-or-older cars, Jaipur + Lucknow | _[₹4,67,41,381, 16.78% Purchase Contribution ]_ |Jaipur + Lucknow are the main region for dales down|
| A9 | Offers whose `inspection_id` is not in inspections | 675 | |
| A10 | Purchase rate, offer revised down >10% | 16.04% (346 / 2,157) | |
| A11 | Purchase rate, no revision | 69.54% (1,931 / 2,777) | |
| A12 | Points of the 10.1-pt drop that are not real | -10.65|
<img width="284" height="73" alt="image" src="https://github.com/user-attachments/assets/2512f796-ee88-4d82-83d3-30e056c8f07c" />
]_ |in June 0 fake cases and in August it was 206 fake case|

**Headline insight:** purchase rate falls from about 70% with no revision to about 16% when the offer is cut by more than 10%. This is a correlation, and the direction of causality still has to be tested (see Part C).

## 4. Repository structure

```
cars24-supply-a2i-analysis/
├── README.md
├── excel/
│   └── Master_Sheet_Cars24.xlsx      # all data, summaries, dashboard, action plan
├── sql/
│   └── cars24_analysis.sql           # Part B queries (MySQL)
├── outputs/
│   ├── SQL_Cars24_Q1.csv             # results of SQL query 1
│   ├── SQL_Cars24_Q2.csv
│   ├── SQL_Cars24_Q3.csv
│   └── SQL_Cars24_Q4.csv
├── docs/
│   └── Cars24.docx                   # written summary
└── .gitignore
```

Check that Q1–Q4 map to B1–B4 in the same order before publishing.

## 5. Workbook guide

| Sheet | Purpose |
|---|---|
| Raw data sheets | Imported exports, untouched |
| Summary / Calculations | Part A metrics with formulas |
| Conversion Audit | Raw vs corrected conversion, and what is real vs artefact |
| Analysis sheets | Call connect, show-up, revisions, inspector anomalies |
| Dashboard | One-screen view for the business head |
| Action Plan | 7 / 30 / 90-day plan with owner, impact, cost, two-week read |

_Rename the rows above to match your actual sheet names._

## 6. SQL overview (`sql/cars24_analysis.sql`, MySQL)

| Part | What it does |
|---|---|
| B1 | Daily lead-to-inspection conversion by city with a 7-row rolling average |
| B2 | Per-inspector count, median duration, median distance (Haversine), offer rate and anomaly flag |
| B3 | Reconciliation of inspection count to Finance, one row per gap component |
| B4 | Weekly lead cohorts with share inspected within 24h, 48h and 7 days |

### How to reproduce
1. Create a schema `cars24` and import the raw CSVs as tables `leads`, `inspections`, `offers_purchases`, etc.
2. Run `sql/cars24_analysis.sql` top to bottom (it sets `sql_mode = ''` for lenient date parsing).

## 7. Limitations and known issues

Listing these openly is deliberate.

- **B1:** the cleaned `inspection_date` (with the +330 min timezone shift) is built but never used in the join, so the timezone correction does not affect the result. The rolling average is an average of daily percentages over 7 rows, not a ratio of 7-day sums and not calendar days.
- **B2:** the inspections table is not de-duplicated before the offer join, which can inflate counts. Any inspector with even one suspicious record is flagged, which is a strict rule.
- **B3:** the query does not read Finance's monthly file. It reconciles internal counts only, and duplicates and ghost inspections may overlap and be subtracted twice. Adding orphan offers to an inspection count needs a written justification.
- **B4:** the `%m/%d/%Y` fallback is ambiguous with `%d/%m/%Y`, and negative time-to-inspection values are not handled.
- **Timezone:** the +5:30 shift for `FIELD_APP_V2` is an assumption; confirm the source timezone.
- **Metrics with two versions** (A6, A7): the chosen headline definition should be stated in the workbook.

## 8. Author

**Anjani Tripathi** · Delhi, India

_Disclaimer: the dataset is synthetic and provided for a screening exercise. No real seller, employee or transaction is included._
