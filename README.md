# Wellbeing App Analytics: Practice Dataset

**Synthetic data.** This dataset was built to mirror the problems in Humin's Data Scientist role: a free, science-based meditation and wellbeing app, its event stream, a validated-style wellbeing survey, and an onboarding A/B test. No real users. In the interview, describe it as "a synthetic dataset I designed to reproduce the instrumentation, measurement and experimentation challenges in the role."

Data window: signups from 2026-06-01 to 2026-10-31. Events run through 2026-11-15 (the data cutoff).

---

## The business questions 

1. **Can we trust the data?** Audit the instrumentation and data quality, then write down every issue you find and how you fixed it.
2. **What does "effective engagement" mean?** The current North Star is *Weekly Active Practitioners (WAP)*: users with ≥1 completed practice in a calendar week. Which usage patterns go with real improvement on the wellbeing measure? Is WAP the right North Star?
3. **Retention.** Who churns, when, and what predicts staying?
4. **A/B test: `onb_personal_plan_v1`.** In September 2026, new iOS and Android signups were randomized 50/50 between the standard onboarding and a new "Personal Plan" onboarding (pick a goal, get a 14-day starter plan, and get prompted to set a daily reminder). Should we ship it?
   - Primary metric: **D7 retention**, meaning the user completed ≥1 practice on days 7–13 after signup.
   - Secondary metrics: week-4 practice days and week-4 wellbeing change.
   - Guardrails: onboarding completion rate and notification opt-out rate.
5. **Business and impact (stretch).** CAC by paid channel, and an outcome summary fit for a funder report.

---

## Files

| File | Grain | Rows (approx) |
|---|---|---|
| `users.csv` | one row per user (in theory) | 15.1k |
| `events.csv` | one row per client event | 604k |
| `assessments.csv` | one row per survey submission | 15.6k |
| `experiment_assignments.csv` | one row per assignment log | 4.3k |
| `content.csv` | one row per practice/lesson | 120 |
| `marketing_spend.csv` | one row per day × paid channel | 306 |

### users.csv
| Column | Description |
|---|---|
| user_id | Unique user ID |
| signup_ts | Account creation time (UTC) |
| platform | Device platform at signup |
| signup_app_version | App version at signup |
| country | ISO-2 country code |
| timezone | IANA timezone of the user's device |
| age_band | Self-reported, optional |
| gender | Self-reported, optional |
| acquisition_channel | organic, paid_social, app_store_search, partner_org (employer/partner programs), referral |
| meditation_experience | From onboarding: none / some / regular |
| primary_goal | From onboarding |
| source_system | `legacy` = account created on the old backend; `core` = new backend |

### events.csv
| Column | Description |
|---|---|
| event_id | Unique event ID |
| user_id | User ID (may be null before login) |
| event_name | See the event list below |
| event_ts | Event timestamp |
| session_id | Client session ID |
| platform | Client platform |
| app_version | Client app version |
| schema_version | Tracking-plan version |
| properties | JSON string of event properties |

**Tracking plan (current, v2):** `app_install`, `app_open`, `onboarding_start`, `goal_selected`, `plan_created`, `onboarding_complete`, `reminder_set`, `notification_open`, `notification_opt_out`, `practice_start`, `practice_complete`, `assessment_complete`.

Key properties:
- `practice_start` has `contentId`.
- `practice_complete` has `contentId` and `durationMs`.
- `reminder_set` has `reminder_time` (local time).
- `assessment_complete` has `wave`.

### assessments.csv
The **Flourishing Index** is a 16-item, 4-pillar wellbeing measure modeled on the awareness / connection / insight / purpose framework. It is paired with a 4-item distress screener in the PHQ-4 format.

| Column | Description |
|---|---|
| response_id | Submission ID |
| user_id | User ID |
| wave | Survey timepoint: `baseline` (in onboarding), `week4` (prompted at day 28), `week8` (prompted at day 56) |
| submitted_ts | UTC |
| scoring_version | Survey backend version |
| completion_seconds | Time spent on the survey |
| q01–q04 | Awareness pillar (1–5 Likert) |
| q05–q08 | Connection pillar |
| q09–q12 | Insight pillar |
| q13–q16 | Purpose pillar |
| phq4_1–phq4_4 | Distress items (0–3); higher = more distress. Items 1–2 = anxiety, 3–4 = depression |

From the measurement spec:
- **q04, q08, q12 and q16 are reverse-worded.** Reverse them (6 − x) before scoring.
- Pillar score = mean of its 4 items. Flourishing Index = mean of the 4 pillars.
- `99` = item skipped.

### experiment_assignments.csv
`assignment_id`, `user_id`, `experiment_id`, `variant` (control/treatment), `assigned_ts` (UTC). Eligibility: iOS or Android users who signed up between 2026-09-01 and 2026-09-30.

### content.csv
`content_id`, `content_type` (seated_meditation, active_meditation, lesson), `pillar`, `duration_min`, `release_date`.

### marketing_spend.csv
`date`, `channel`, `spend_usd`, `impressions`, `clicks`, `attributed_installs` (attributed by the ad platforms, not by us).

---

## Notes from the team (as you'd hear them in Slack)

- Engineering: "We migrated from the legacy backend on July 1. Should be clean, but a few people have asked about it."
- Engineering: "App 5.0 (mid-August) shipped the new tracking plan."
- Product: "Engagement on iOS looked soft in early October. Probably seasonal?"
- Product: "Android 5.1.0 got a hotfix (5.1.1) on Sept 22. Nothing to do with the experiment, I think."
- Science: "Please don't report wellbeing change without telling us how you handled missing follow-ups."
