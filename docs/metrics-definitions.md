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
 

