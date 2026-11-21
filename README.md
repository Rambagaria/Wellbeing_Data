# Wellbeing App Analytics: From Messy Events to Trustworthy Insights

An end-to-end analytics project on a meditation and wellbeing app. It covers instrumentation auditing, data cleaning, feature engineering, retention and engagement analysis, wellbeing outcome measurement, and A/B testing.

> **About the data:** The dataset is **synthetic**. I designed it to reproduce the problems real product-analytics teams run into: tracking gaps, schema drift, duplicate events, a messy backend migration, survey data with psychometric pitfalls, and an experiment that needs careful analysis. No real users are represented.

---

## Project status

| Step | Topic | Status |
|---|---|---|
| 1 | Data profiling and quality audit | ⏳ In progress |
| 2 | Cleaning and standardization | ⬜ Not started |
| 3 | Feature engineering | ⬜ Not started |
| 4 | Retention and churn analysis | ⬜ Not started |
| 5 | Engagement → wellbeing outcomes | ⬜ Not started |
| 6 | A/B test: `onb_personal_plan_v1` | ⬜ Not started |
| 7 | Business metrics and impact summary | ⬜ Not started |

Each step's findings are summarized in [Results](#results) as the project progresses.

---

## The questions

1. **Can we trust the data?** Audit the instrumentation and data quality, and document every issue found and how it was resolved.
2. **What does "effective engagement" mean?** The current North Star metric is *Weekly Active Practitioners (WAP)*: users with at least one completed practice in a calendar week. Which usage patterns are associated with real improvement in wellbeing? Is WAP the right North Star?
3. **Retention.** Who churns, when, and what predicts staying?
4. **A/B test.** In September 2026, new iOS and Android signups were randomized 50/50 between the standard onboarding and a new "Personal Plan" onboarding (pick a goal, get a 14-day starter plan, and get prompted to set a daily reminder). Should it ship?
   - **Primary metric:** D7 retention, meaning the user completed at least one practice on days 7–13 after signup.
   - **Secondary metrics:** week-4 practice days and week-4 wellbeing change.
   - **Guardrails:** onboarding completion rate and notification opt-out rate.
5. **Business and impact.** Customer acquisition cost (CAC) by paid channel, and an outcome summary suitable for an external impact report.

---

## Repository structure

```
├── data/raw/            # original CSVs (never modified)
├── data/processed/      # cleaned outputs from the pipeline
├── notebooks/           # step-by-step analysis notebooks
├── src/                 # reusable cleaning / feature / stats code
├── reports/             # figures and write-ups
└── README.md
```

---

## Data

The data covers signups from 2026-06-01 to 2026-10-31. Events run through the data cutoff on 2026-11-15.

| File | Grain | Rows (approx) |
|---|---|---|
| `users.csv` | one row per user (in theory) | 15.1k |
| `events.csv.gz` | one row per client event | 604k |
| `assessments.csv` | one row per survey submission | 15.6k |
| `experiment_assignments.csv` | one row per assignment log | 4.3k |
| `content.csv` | one row per practice or lesson | 120 |
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
| age_band | Self-reported (optional) |
| gender | Self-reported (optional) |
| acquisition_channel | organic, paid_social, app_store_search, partner_org (employer or partner programs), referral |
| meditation_experience | From onboarding: none, some or regular |
| primary_goal | From onboarding |
| source_system | `legacy` = account created on the old backend; `core` = new backend |

### events.csv.gz
| Column | Description |
|---|---|
| event_id | Unique event ID |
| user_id | User ID (may be null before login) |
| event_name | See the tracking plan below |
| event_ts | Event timestamp |
| session_id | Client session ID |
| platform | Client platform |
| app_version | Client app version |
| schema_version | Tracking-plan version |
| properties | JSON string of event properties |

**Tracking plan (current, v2):** `app_install`, `app_open`, `onboarding_start`, `goal_selected`, `plan_created`, `onboarding_complete`, `reminder_set`, `notification_open`, `notification_opt_out`, `practice_start`, `practice_complete`, `assessment_complete`.

Key properties:
- `practice_start`: `contentId`
- `practice_complete`: `contentId`, `durationMs`
- `reminder_set`: `reminder_time` (local time)
- `assessment_complete`: `wave`

### assessments.csv
The **Flourishing Index** is a 16-item wellbeing measure with 4 pillars (awareness, connection, insight and purpose). It is paired with a 4-item distress screener in the PHQ-4 format.

| Column | Description |
|---|---|
| response_id | Submission ID |
| user_id | User ID |
| wave | `baseline` (taken during onboarding), `week4` (prompted at day 28), `week8` (prompted at day 56) |
| submitted_ts | Submission time (UTC) |
| scoring_version | Survey backend version |
| completion_seconds | Time spent on the survey |
| q01–q04 | Awareness pillar (1–5 Likert scale) |
| q05–q08 | Connection pillar |
| q09–q12 | Insight pillar |
| q13–q16 | Purpose pillar |
| phq4_1–phq4_4 | Distress items scored 0–3; higher means more distress. Items 1–2 measure anxiety, items 3–4 depression |

Scoring rules from the measurement spec:
- q04, q08, q12 and q16 are reverse-worded. Reverse them (6 − x) before scoring.
- Pillar score = mean of its 4 items. Flourishing Index = mean of the 4 pillar scores.
- `99` means the item was skipped.

### experiment_assignments.csv
`assignment_id`, `user_id`, `experiment_id`, `variant` (control or treatment), `assigned_ts` (UTC).

Eligibility: iOS and Android users who signed up between 2026-09-01 and 2026-09-30.

### content.csv
`content_id`, `content_type` (seated_meditation, active_meditation, lesson), `pillar`, `duration_min`, `release_date`.

### marketing_spend.csv
`date`, `channel`, `spend_usd`, `impressions`, `clicks`, `attributed_installs` (attributed by the ad platforms, not first-party).

### Stakeholder context
- **Engineering:** "We migrated from the legacy backend on July 1. Should be clean, but a few people have asked about it."
- **Engineering:** "App 5.0 (mid-August) shipped the new tracking plan."
- **Product:** "Engagement on iOS looked soft in early October. Probably seasonal?"
- **Product:** "Android 5.1.0 got a hotfix (5.1.1) on Sept 22. Nothing to do with the experiment, I think."
- **Science:** "Please don't report wellbeing change without saying how missing follow-ups were handled."

---

## Results

### Step 1: Data profiling and quality audit
_To be added._

### Step 2: Cleaning and standardization
_To be added._

### Step 3: Feature engineering
_To be added._

### Step 4: Retention and churn analysis
_To be added._

### Step 5: Engagement → wellbeing outcomes
_To be added._

### Step 6: A/B test
_To be added._

### Step 7: Business metrics and impact summary
_To be added._

---

## Tech stack
Python (pandas, numpy, scipy, statsmodels, lifelines, scikit-learn), SQL (DuckDB) and matplotlib.
