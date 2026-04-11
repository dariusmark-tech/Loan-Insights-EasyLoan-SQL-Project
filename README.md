# Practical Exam: Loan Insights — EasyLoan SQL Project

## Overview

This project is a SQL-based data cleaning and analysis exam for **EasyLoan**, a company offering personal loans, car loans, and mortgages to clients in the **USA**, **UK**, and **Canada (CA)**.

The analytics team needs clean, reliable data before they can start reporting on loan performance across geographic regions. This project covers 4 SQL tasks across 4 tables in the `lending` database.

---

## Database Schema

The `lending` database contains 4 tables:

```
client         → client_id (PK), date_of_birth, employment_status, country
loan           → loan_id (PK), client_id (FK), contract_id (FK), principal_amount, interest_rate, loan_type
contract       → contract_id (PK), contract_date
repayment      → repayment_id (PK), loan_id (FK), repayment_date, repayment_amount, repayment_channel
```

---

## Tasks & Solutions

### Task 1 — Clean the `client` Table
**DataFrame name:** `client`

Clean invalid data types and string values from the `client` table without modifying the original.

```sql
SELECT
    client_id,
    CASE
        WHEN date_of_birth ~ '^\d{4}-\d{2}-\d{2}$'
            THEN date_of_birth::DATE
        WHEN date_of_birth ~ '^\d{4}-\d{2}-\d{2}[- ]'
            THEN SUBSTRING(date_of_birth, 1, 10)::DATE
        ELSE NULL
    END AS date_of_birth,
    CASE
        WHEN LOWER(TRIM(employment_status)) = 'employed' THEN 'employed'
        WHEN LOWER(TRIM(employment_status)) = 'unemployed' THEN 'unemployed'
        ELSE NULL
    END AS employment_status,
    CASE
        WHEN UPPER(TRIM(country)) = 'USA' THEN 'USA'
        WHEN UPPER(TRIM(country)) = 'UK' THEN 'UK'
        WHEN UPPER(TRIM(country)) = 'CA' THEN 'CA'
        ELSE NULL
    END AS country
FROM client;
```

**Cleaning rules applied:**
| Column | Rule |
|--------|------|
| `client_id` | Already an integer set by DB — no modification |
| `date_of_birth` | Cast string to DATE; handle `YYYY-MM-DD-hh-ss-SSS` format by extracting first 10 chars; invalid → NULL |
| `employment_status` | Lowercase + trim; only keep `employed` / `unemployed`; else → NULL |
| `country` | Uppercase + trim; only keep `USA`, `UK`, `CA`; else → NULL |

---

### Task 2 — Fill Missing `repayment_channel` Values
**DataFrame name:** `repayment`

Fill missing repayment channel values based on repayment amount rules.

```sql
SELECT
    repayment_id,
    loan_id,
    repayment_date,
    repayment_amount,
    CASE
        WHEN (
            repayment_channel IS NULL
            OR TRIM(repayment_channel) = ''
            OR LOWER(TRIM(repayment_channel)) = 'missing'
            OR TRIM(repayment_channel) = '-'
        ) AND repayment_amount > 4000 THEN 'bank account'
        WHEN (
            repayment_channel IS NULL
            OR TRIM(repayment_channel) = ''
            OR LOWER(TRIM(repayment_channel)) = 'missing'
            OR TRIM(repayment_channel) = '-'
        ) AND repayment_amount < 1000 THEN 'mail'
        ELSE repayment_channel
    END AS repayment_channel
FROM repayment;
```

**Fill rules:**
| Condition | Channel Assigned |
|-----------|-----------------|
| Missing + amount > $4,000 | `bank account` |
| Missing + amount < $1,000 | `mail` |
| All other rows | Keep existing value |

> ⚠️ "Missing" values include: `NULL`, empty string `''`, dash `'-'`, and the word `'missing'`

---

### ✅ Task 3 — US Clients on the New Online System
**DataFrame name:** `us_loans`

Return loans for US clients who signed contracts on or after January 1st, 2022.

```sql
SELECT
    c.client_id,
    ct.contract_date,
    l.principal_amount,
    l.loan_type
FROM loan l
JOIN client c ON l.client_id = c.client_id
JOIN contract ct ON l.contract_id = ct.contract_id
WHERE c.country = 'USA'
  AND ct.contract_date >= '2022-01-01';
```

---

### ✅ Task 4 — Average Interest Rate by Loan Type & Country
**DataFrame name:** `average_data`

Compare average interest rates for each loan type across countries.

```sql
SELECT
    l.loan_type,
    c.country,
    AVG(l.interest_rate) AS avg_rate
FROM loan l
JOIN client c ON l.client_id = c.client_id
GROUP BY l.loan_type, c.country
ORDER BY l.loan_type, c.country;
```

---

## Key Lessons & Gotchas

- **DataLab stores dates as strings** in `YYYY-MM-DD-hh-ss-SSS` format — always extract the first 10 characters before casting to `DATE`
- **Missing values aren't always NULL** — check for empty strings `''`, `'-'`, and `'missing'` too
- **Never modify source tables** — always use pure `SELECT` queries
- **Column name matching is strict** — the grader checks exact DataFrame column names
- **Regex anchor matters** — using `$` at end of date regex caused all dates with time suffixes to return NULL

---

## Project Structure

```
lending (database)
├── client
├── loan
├── contract
└── repayment
```

---

## Requirements

- SQL (PostgreSQL dialect)
- DataLab / Jupyter notebook environment
- `lending` database access
