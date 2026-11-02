# Metrics Definitions
 
Every metric in Product Pulse is defined here once and tested against a small hand-computed fixture dataset. If the dashboard and this document disagree, it is a bug.
 
## Conventions
 
- **Time zone:** all timestamps are stored and bucketed in UTC.
- **Active user:** a user who triggered at least one *qualifying event* in the period. By default every event qualifies; this can be restricted to a list of core actions via configuration (e.g. exclude `email_opened`).
- **Day** = UTC calendar day. **Week** = ISO week (Monday start). **Month** = calendar month.
- **Cohort:** the group of users sharing a signup week (`users.signup_date`).
---
 
## 1. DAU, WAU, MAU
 
| Metric | Definition |
|---|---|
| **DAU** | Distinct active users on a given day |
| **WAU** | Distinct active users in the 7 days ending on a given day (rolling) |
| **MAU** | Distinct active users in the 30 days ending on a given day (rolling) |
| **Stickiness** | DAU / MAU, averaged over the selected range |
 
```sql
-- DAU
SELECT date_trunc('day', timestamp) AS day,
       COUNT(DISTINCT user_id)      AS dau
FROM events
WHERE timestamp >= :start AND timestamp < :end
GROUP BY 1
ORDER BY 1;
```
 
**Edge cases:** users with no events are never counted. Rolling windows are inclusive of the end day.
 
---
 
## 2. Funnel conversion
 
**Input:** an ordered list of steps (e.g. `signup → onboarding_complete → first_project → purchase`) and a conversion window (default 7 days from the first step).
 
**Rules**
1. A user enters the funnel at their first occurrence of step 1 within the date range.
2. A user counts at step *N* only if they completed steps 1 to *N* **in order**, with step *N* occurring within the window of step 1.
3. Repeated events do not double-count; only the earliest valid sequence is used.

| Output | Formula |
|---|---|
| Users at step N | Count of users meeting the rules above |
| Step conversion | Users at step N / Users at step N-1 |
| Overall conversion | Users at last step / Users at step 1 |
| Drop-off | 1 - step conversion |
 
---
 
## 3. Retention (cohort)
 
**Definition:** For a signup-week cohort, *Week N retention* is the share of cohort users who were active during week N after their signup week.
 
```
retention(cohort, N) = active users from cohort in week (cohort_week + N)
                       -------------------------------------------------
                                  total users in cohort
```
 
- **Week 0** is the signup week itself (typically 100% if signup counts as activity).
- This is **N-week ("bracket") retention**: a user must be active *in* week N, not merely "on or after" it.
- Cells for weeks that have not finished yet are shown as incomplete, never as 0%.
```sql
WITH cohorts AS (
  SELECT id AS user_id, date_trunc('week', signup_date) AS cohort_week
  FROM users
),
activity AS (
  SELECT DISTINCT user_id, date_trunc('week', timestamp) AS active_week
  FROM events
)
SELECT c.cohort_week,
       (EXTRACT(EPOCH FROM (a.active_week - c.cohort_week)) / 604800)::int AS week_number,
       COUNT(DISTINCT a.user_id) AS active_users
FROM cohorts c
JOIN activity a USING (user_id)
GROUP BY 1, 2
ORDER BY 1, 2;
```
 
---
 
## 4. Churn
 
**Definition (inactivity-based):** A user is **churned** in a period if they were active in the previous period but have **no activity in the current period**.
 
```
churn_rate(period) = users active in previous period and inactive in this period
                     --------------------------------------------------------
                              users active in previous period
```

## 5. Segmentation
 
Every metric above accepts these filters, applied to the **users** table before aggregation:
 
| Dimension | Source column |
|---|---|
| Acquisition channel | `users.acquisition_channel` |
| Plan | `users.plan` |
| Country | `users.country` |
 
When comparing segments, segment sizes are shown next to every percentage so small-sample noise is visible.
 
---
 
## 6. Known limitations
 
- Rolling WAU/MAU double-handles users near window edges only by design (windows are inclusive).
- Funnel results depend on the chosen conversion window; changing it changes the numbers.
- Synthetic data has no bots, duplicate events, or clock skew. Production data would need a cleaning step.
 

- Default period: **month** (configurable to week).
- A user who returns later is counted as **reactivated**, not erased from past churn.
- Retention and churn are related but not complements: retention is cohort-based, churn is period-based.
---
