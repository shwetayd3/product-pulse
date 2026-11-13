# Product Requirements Document: Product Pulse
 
| | |
|---|---|
| **Status** | Draft v0.1 |
| **Author** | Shweta |
| **Last updated** | October 2026 |
 
## 1. Problem statement
 
Early-stage teams need to understand how users behave (who signs up, where they drop off, who comes back) but the standard tools (Mixpanel, Amplitude) are costly, closed, and heavy to set up. Teams often fall back on ad-hoc SQL queries that are slow to write, inconsistent between people, and impossible to audit.
 
**Product Pulse** is a self-hosted analytics dashboard that turns raw product events into a small, trustworthy set of health metrics (activity, funnels, retention, and churn) in minutes.
 
## 2. Target users
 
| Persona | Needs | Pain today |
|---|---|---|
| **Maya, early-stage PM / founder** | A weekly view of product health to decide what to build next | Waits on engineers for every data question; metric definitions differ between people |
| **Arjun, junior data analyst** | Reliable, documented metric definitions he can query and extend | Rewrites the same cohort SQL for every request |
| **Dev, backend engineer** | A simple way to send events and verify they arrive | Does not want to adopt a heavy SaaS SDK for a small product |
 
## 3. Goals
 
1. A new user can run the project and see a populated dashboard in **under 5 minutes** (Docker Compose + seeded data).
2. Every metric has a **written, tested definition** (see `metrics-definitions.md`).
3. Answer the four core questions: *Are people active? Where do they drop off? Do they come back? Who leaves?*
4. Support **segmenting** every metric by acquisition channel, plan, and country.
5. Stay fast: dashboard queries return in **under 2 seconds on 1M events**.
## 4. Non-goals (v1)
 
- Real-time streaming analytics (batch/near-real-time is enough)
- Session replay, heatmaps, or user-level tracking UI
- A/B testing and experimentation
- Multi-tenant SaaS, billing, or user management
- Predictive modeling (churn prediction is a possible v2+)
