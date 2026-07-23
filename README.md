# Hospital-Managemen
# Hospital Operations & Revenue Analysis

A SQL case study on a multi-branch hospital management dataset — tracing patient
visits from booking to payment to find where a hospital chain is losing revenue
and operational efficiency.

**Author:** Sayan Saha
**Skills demonstrated:** SQL (multi-table JOINs, CTEs, window functions, conditional
aggregation), data quality validation, business insight reporting

## The question

Where is this hospital chain losing revenue and operational efficiency — and which
doctors, specializations, and branches perform best?

## Dataset

Five linked tables, joined to trace a single patient visit from booking to payment:

| Table | Rows | Description |
|---|---|---|
| `patients` | 50 | Demographics, insurance provider |
| `doctors` | 10 | Specialization, branch, years of experience |
| `appointments` | 200 | Booking date/time, status (Scheduled / Completed / No-show / Cancelled) |
| `treatments` | 200 | Treatment type, cost, linked to an appointment |
| `billing` | 200 | Bill amount, payment method, payment status |

Source: Hospital Management Dataset (Kaggle — kanakbaghel).

A patient visit flows: **booked with a doctor → becomes a treatment → generates a bill**,
a 1:1 chain that lets revenue be traced back to the doctor and specialization that
generated it.

## Key findings

- **51.5%** of all 200 booked appointments end in a No-show or Cancellation.
- Dermatology (54.3%) and Pediatrics (51.0%) lose the highest share of appointment slots.
- Of **₹5,51,250** total billed revenue, only **31.5%** (₹1,73,425) has actually been collected;
  ₹3,77,825 remains at risk (Pending + Failed).
- **Oncology** has the weakest collection rate (22.3%) despite carrying the highest
  average treatment cost — Chemotherapy averages ₹2,630/case.
- **Central Hospital** handles the most appointments (84) of any branch but completes
  only 17.9% of them — well below Westside (27.8%) and Eastside (25.8%), pointing to
  overbooking or scheduling capacity issues rather than patient behavior.
- No-show rate varies 4.8%–35.7% across individual doctors with no clear link to
  years of experience.

## Recommendations

1. **Revenue leakage is systemic** — 96% of patients (48/50) carry at least one unpaid
   bill, so billing follow-up needs to be tightened chain-wide, not just for a few accounts.
2. **Oncology needs a billing review** — highest-cost specialization, lowest collection
   rate; investigate insurance approval delays on high-value claims.
3. **Central Hospital is overbooked** — highest volume, lowest completion rate; review
   scheduling capacity against Westside/Eastside.
4. **Experience ≠ reliability** — no-show rates vary widely across doctors independent
   of tenure; look at individual scheduling/reminder practices.

## SQL techniques used

- Multi-table JOINs across all five tables
- CTEs (Common Table Expressions)
- Window functions — `RANK()`, `ROW_NUMBER()`, running totals via `SUM() OVER (ORDER BY ...)`
- Conditional aggregation (`SUM(CASE WHEN ...)`) for rate calculations
- `GROUP BY` / `ORDER BY` for per-specialization and per-branch breakdowns

Example — monthly collected revenue with a running total:

```sql
WITH monthly_revenue AS (
    SELECT
        DATE_FORMAT(bill_date, '%Y-%m') AS month,
        SUM(amount) AS collected_revenue
    FROM billing
    WHERE payment_status = 'Paid'
    GROUP BY DATE_FORMAT(bill_date, '%Y-%m')
)
SELECT
    month,
    collected_revenue,
    SUM(collected_revenue) OVER (ORDER BY month) AS running_total
FROM monthly_revenue
ORDER BY month;
```

## Repository structure

```
├── data/                                  # Raw source CSVs
│   ├── patients.csv
│   ├── doctors.csv
│   ├── appointments.csv
│   ├── treatments.csv
│   └── billing.csv
├── sql/
│   └── hospital_management_sql_analysis.sql   # Full analysis script
├── presentation/
│   └── hospital_revenue_analysis__Sayan_Saha_.pptx   # Findings deck
└── README.md
```
