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

