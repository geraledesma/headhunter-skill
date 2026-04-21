# headhunter

A strategic job search assistant that works from plain markdown files. It scans roles, tailors
CVs, manages outreach, tracks your pipeline, and runs structured career sessions — keeping your
search sharp and your positioning tight.

Optionally integrates with Indeed (role scanning), Notion (pipeline sync), and Gmail (outreach
drafts). For ATS review, LinkedIn optimization, interview prep, and company research, headhunter
delegates to the **career-helper** skill.

---

## Quick start

Install the skill in Claude Code, then say:

> "headhunter"

If this is your first time, headhunter will run Onboarding to scaffold your vault. If your vault
already exists, it will orient and surface any pending items before asking what you'd like to do.

---

## Vault structure

headhunter expects a folder layout inside your vault:

```
<vault-root>/
└── agents/headhunter/
    ├── raw/
    │   ├── profile.md            ← who you are, what you're targeting, hard filters
    │   ├── experience-bank.md    ← verified work history — the single source of truth for CV content
    │   ├── firms-contacts.md     ← target firms by tier, key contacts, outreach history
    │   └── linkedin.md           ← LinkedIn positioning (headline, about, skills)
    ├── wiki/
    │   ├── cv-strategy.md        ← CV version registry, tailoring rules, approval log
    │   ├── pipeline.md           ← active + historical application log
    │   ├── sessions.md           ← session cadence tracking + log
    │   └── jd-log/               ← full job descriptions, one file per role
    ├── cv-versions/              ← named .docx/.pdf CV files
    └── outputs/
        ├── cover-letters/
        ├── firm-profiles/
        └── 90day-plans/
```

Run onboarding to create this structure from templates:
> "Set up headhunter" or "onboard me"

---

## What headhunter can do

### Everyday operations
| What you say | What happens |
|---|---|
| "Scan for jobs" | Searches Indeed + career pages, scores matches against your filters, offers to save JDs |
| "Check my pipeline" | Shows all active applications, flags stale entries, collects status updates |
| "Tailor my CV for [firm/role]" | Reads JD + experience-bank, produces tailored bullets, can generate .docx |
| "Draft a message to [contact]" | Writes warm LinkedIn (≤300 chars) or email — shows before sending |
| "I applied to [firm – role]" | Logs application with CV version, date, channel, saves JD |
| "CV status" | Shows all CV versions, pending approvals, missing files |

### Career sessions
Sessions are structured, focused work blocks. headhunter tracks when each was last run and
nudges you when you're overdue.

| Session | Cadence | What it does |
|---|---|---|
| `weekly-review` | 7 days | Summarizes week's activity, flags bottlenecks, sets next-week goals |
| `strategy-session` | 30 days | Diagnoses where the funnel is leaking, proposes fixes |
| `cv-strategy-session` | 30 days | Reviews CV versions vs. pipeline, proposes what to build/retire |
| `networking-strategy` | 30 days | Maps contacts to target firms, builds outreach sequence |
| `linkedin-session` | 60 days | Full LinkedIn audit (delegates to career-helper) |
| `experience-bank-session` | 90 days | Deep-dive Q&A to extract metrics and granular details |
| `ats-review` | Per application | Scores CV against JD for keyword coverage (delegates to career-helper) |
| `cover-letter` | Per application | Drafts a 3-paragraph, evidence-based cover letter |
| `rejection-debrief` | Per event | Diagnoses root cause, defines next steps |
| `offer-evaluation` | Per event | Evaluates offer vs. floor + market (delegates to career-helper) |
| `outreach-campaign` | Per campaign | Batch-drafts sequenced messages with approval for each |
| `firm-deep-dive` | Per firm | Company research before applying (delegates to career-helper) |
| `ninety-day-plan` | Per interview | Builds a 30/60/90 day plan for final-round interviews |

---

## Key behaviours

**Nothing is sent without your approval.** All CV writes, pipeline entries, outreach messages,
and Notion entries show a preview/diff and require an explicit "yes" before anything changes.

**experience-bank.md is the single source of truth.** headhunter will never add unverified facts
to your CV or cover letters. If something can't be sourced from your experience bank, it's
marked `[MISSING]`.

**JDs are saved for future use.** Every role you're interested in gets its full JD logged in
`wiki/jd-log/` — so ATS reviews, cover letters, and 90-day plans always have the original source.

**Sessions are tracked.** `wiki/sessions.md` records when each session type was last run.
headhunter checks cadences at the start of every conversation and surfaces what's overdue.

---

## Integrations

| Integration | Required | Used for |
|---|---|---|
| Indeed MCP | Optional | Job scanning |
| Notion MCP | Optional | Pipeline sync (secondary to pipeline.md) |
| Gmail MCP | Optional | Saving outreach email drafts |
| career-helper skill | Recommended | LinkedIn optimization, ATS review, offer evaluation, company research |

Configure Notion integration in `raw/profile.md § Config`:
```
notion_enabled: true
notion_pipeline_db: [database name or ID]
```

---

## profile.md — key fields

```markdown
## § Config

timezone: America/Mexico_City
notion_enabled: false
seniority_floor: Senior
salary_floor_local: 120000     # monthly local currency
salary_floor_remote_usd: 100000  # annual USD for remote roles
geography: [City] or remote
exclusions:
  - [firm or role type to never recommend]
```

---

## Reference templates

The `references/` directory contains blank templates for all vault files:
- `profile-template.md`
- `experience-bank-template.md`
- `firms-contacts-template.md`
- `cv-strategy-template.md`

These are used by Onboarding to scaffold a new vault. You can also copy them manually.

---

## career-helper dependency

headhunter delegates four capabilities to career-helper:
- **LinkedIn optimization** (`linkedin-session`)
- **ATS keyword review** (`ats-review`)
- **Offer evaluation** (`offer-evaluation`)
- **Company research** (`firm-deep-dive`)

headhunter provides the context (your profile, the JD, the CV); career-helper provides the
analysis. Both skills need to be installed for the full feature set.

---

## Example workflow

```
Day 1: Run onboarding → fill in profile + experience bank
Week 1: experience-bank-session (deep-dive one past role)
Week 1: cv-strategy-session (plan which CV variants to build)
Week 1: linkedin-session → update linkedin.md
Ongoing: scan jobs → tailor CV → log application → track pipeline
Weekly: weekly-review
Monthly: strategy-session, networking-strategy
Per application: ats-review, cover-letter
Per interview: firm-deep-dive, ninety-day-plan
Per rejection: rejection-debrief
Per offer: offer-evaluation
```
