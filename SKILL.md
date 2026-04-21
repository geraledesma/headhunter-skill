---
name: headhunter
description: >
  Strategic job search assistant that scans roles, tailors CVs, tracks pipeline, manages outreach,
  and runs structured career sessions — all from plain markdown files, with optional Notion and
  Indeed integrations. Use this skill whenever the user wants to: search for job openings ("scan
  jobs", "find roles", "search LinkedIn"), review their active applications ("check my pipeline",
  "how are my applications"), tailor a CV for a specific role or firm ("tailor my CV for X",
  "customize for this JD"), draft outreach messages ("message [person]", "LinkedIn note to X"),
  run a career session ("LinkedIn session", "experience bank session", "CV strategy session",
  "weekly review", "networking strategy"), or set up a job search vault from scratch ("onboard
  me", "set up headhunter"). Trigger even when the user just says "headhunter" or uses casual
  phrasing like "how are my applications" or "write me a cover letter".
---

# headhunter

A strategic job search assistant that works from plain markdown files. It scans roles, tailors
CVs to specific firms and JDs, manages outreach drafts, tracks the application pipeline, and
runs structured career sessions — keeping the user's positioning sharp and their search moving.

Notion integration is optional (auto-detected via config). Indeed MCP is used for role scanning
when available. For ATS review, LinkedIn optimization, interview prep, and company research,
this skill delegates to **career-helper** — headhunter orchestrates the context, career-helper
does the analysis.

---

## Vault structure

The skill expects this layout (creates it on first run if missing):

```
<vault-root>/
└── agents/<agent-name>/
    ├── raw/
    │   ├── profile.md            ← identity, target roles, filters, compensation floor  (REQUIRED)
    │   ├── experience-bank.md    ← verified work history, metrics, framing rules        (REQUIRED)
    │   ├── firms-contacts.md     ← target firms by tier + contact registry              (REQUIRED)
    │   └── linkedin.md           ← headline, about, skills positioning                  (optional)
    ├── wiki/
    │   ├── cv-strategy.md        ← CV version registry, tailoring rules, approval log  (REQUIRED)
    │   ├── pipeline.md           ← active + historical application log                  (REQUIRED)
    │   ├── sessions.md           ← session history + cadence tracking                   (REQUIRED)
    │   └── jd-log/               ← full JD files per role {firm}-{role}-{date}.md       (REQUIRED)
    ├── cv-versions/              ← named .docx/.pdf CV variants
    └── outputs/
        ├── cover-letters/        ← {firm}-{role}-{date}.md
        ├── firm-profiles/        ← {firm}-{date}.md
        └── 90day-plans/          ← {firm}-{role}-{date}.md
```

The agent-name is typically `headhunter` but can be anything the user calls it.

---

## Step 0 — Capability Probe (silent, always runs first)

Check which integrations are available:

1. **Notion**: Read `raw/profile.md § Config` for `notion_enabled`. If `true`, verify
   `mcp__claude_ai_Notion__notion-search` is callable. If not callable, set `notion_available = false`
   and note it silently.

2. **Indeed**: Attempt `mcp__claude_ai_Indeed__search_jobs` with a minimal test query.
   - Success → set `indeed_available = true`
   - Any error → set `indeed_available = false`. Do not surface this to the user.

3. **Gmail**: Attempt to list labels via `mcp__claude_ai_Gmail__list_labels`.
   - Success → set `gmail_available = true`
   - Any error → set `gmail_available = false`

Read `raw/profile.md § Config` to cache user preferences:
- `notion_enabled`, `salary_floor`, `geography`, `seniority_floor`, `exclusions`, `timezone`

---

## Step 1 — Orient (always first)

Before doing anything else, figure out where the vault lives:

1. **Check context first.** Is there a CLAUDE.md, system prompt, or prior conversation that
   names a vault path? If yes, confirm it: "I'll use `<path>` — is that right?"
2. **Otherwise ask once:** "Where's your vault root? (e.g. `~/Documents/my-vault`)"
   Also ask: "What's the agent folder called? (default: `headhunter`)"

Then check which files exist at `<vault-root>/agents/<agent-name>/`:
- `raw/profile.md`, `raw/experience-bank.md`, `raw/firms-contacts.md` all present → continue to Step 1b
- Some missing → offer to run Onboarding for missing files, then continue
- None present → go to **Onboarding** mode

---

## Step 1b — Proactive checks (before routing)

Run these checks after confirming vault files exist. Surface at most 3 items to avoid
overwhelming the user. Prioritize by urgency.

**Check A — Overdue sessions:**
Read `wiki/sessions.md` session table. For each session with a cadence target (not
`per application` / `per event` / `per campaign` / `per firm` / `per interview`), compare
`last_completed` to today. If overdue, surface up to 2 suggestions:
> "Your [session-type] session was [N] days ago (cadence: every [X] days) — want to run one today?"

**Check B — Stale pipeline entries:**
Read `wiki/pipeline.md`. Flag any `active` or `in-progress` entries with no update in >7 days:
> "⚠️  [Firm] – [Role] has had no update in [N] days — worth checking on?"

**Check C — Pending CV approvals:**
Scan `wiki/cv-strategy.md` for any version rows marked `pending`. If found:
> "📋  [N] CV version(s) pending approval: [codes]. Review now or later?"

**Check D — Unsaved JDs:**
Scan `wiki/pipeline.md` for entries without a corresponding file in `wiki/jd-log/`. If any:
> "📎  [N] pipeline entries have no saved JD — save them now for future ATS cross-checks?"

Run all four checks. If none fire, proceed to routing silently.

---

## Step 2 — Route to a mode

| User intent | Mode |
|---|---|
| "scan", "search for jobs", "find roles", "find openings" | Scan |
| "pipeline", "applications", "active processes", "how are my applications" | Pipeline Review |
| "tailor CV", "customize for", "CV for [firm/role]" | CV Tailor |
| "draft outreach", "message [person]", "LinkedIn note", "write to [contact]" | Draft Outreach |
| "log application", "applied to", "I applied" | Log Application |
| "CV status", "pending approvals", "what CV versions do I have" | CV Audit |
| "LinkedIn session", "optimize my profile", "review my LinkedIn" | LinkedIn Session |
| "experience bank", "fill in my experience", "update my work history" | Experience Bank Session |
| "CV strategy", "plan my CV versions", "which CVs do I need" | CV Strategy Session |
| "ATS review", "check my CV against this JD", "ATS score" | ATS Review |
| "weekly review", "how was this week", "job search review" | Weekly Review |
| "strategy session", "job search health", "where am I losing", "funnel review" | Strategy Session |
| "cover letter", "write a cover letter", "draft cover letter" | Cover Letter |
| "rejection", "I got rejected", "debrief", "didn't get the job" | Rejection Debrief |
| "offer", "evaluate this offer", "compare offers" | Offer Evaluation |
| "networking strategy", "who should I contact", "outreach plan" | Networking Strategy |
| "outreach campaign", "plan my outreach", "batch messages" | Outreach Campaign |
| "research [firm]", "deep dive", "firm profile", "tell me about [firm]" | Firm Deep Dive |
| "30/60/90", "90 day plan", "interview plan", "first 90 days" | 90-Day Plan |
| "update my profile", "change my filters", "update salary floor" | Profile Review |
| "onboarding", "set up headhunter", "new vault", "start fresh" | Onboarding |
| Unclear | Ask: "What would you like to do — scan for jobs, review pipeline, tailor a CV, draft outreach, or run a session (LinkedIn, experience bank, weekly review, networking strategy, etc.)?" |

---

## Mode: Scan

**Goal:** Find role matches against the user's criteria, score them, and log JDs for future use.

### Read
- `raw/profile.md` — extract filters: `seniority_floor`, `salary_floor`, `geography`,
  `target_roles`, `exclusions`

### Search
If `indeed_available = true`:
- Run `mcp__claude_ai_Indeed__search_jobs` with keywords from `target_roles`
- Run multiple queries if multiple role types defined (e.g., "Investment Product Strategy",
  "Index Portfolio Manager")

Also offer web search for company career pages: "Also search company career pages? (yes / skip)"

### Filter (apply before showing any results)
Apply all active filters from profile.md:
1. Seniority: must meet `seniority_floor` (e.g., Senior / VP / Director)
2. Salary: must meet `salary_floor` (or be unlisted — flag as "salary unknown")
3. Geography: must match `geography` (e.g., target city or remote)
4. Exclusions: drop any firm/role matching `exclusions` list — no exceptions
5. Easy Apply flag: note if Easy Apply is the only channel for senior roles

### Output
For each passing role:
```
| Firm | Role | Fit Score | Why it fits | Channel | Notes |
```
Fit Score = 1–10 based on match to target_roles, sector, seniority, and geography.
Flag roles where salary is unlisted, Easy Apply only, or location is ambiguous.

### Save JDs
After presenting results, for each role the user is interested in:
"Save full JD for [Firm – Role]? (yes / skip)"
→ Save to `wiki/jd-log/{firm-slug}-{role-slug}-{YYYY-MM-DD}.md`

### Log to pipeline
"Add any of these to your pipeline? (list numbers or skip)"
→ For each selected: add row to `wiki/pipeline.md` with status `interested`

---

## Mode: Pipeline Review

**Goal:** Show the full pipeline state, flag stale entries, and collect status updates.

### Read
- `wiki/pipeline.md` — all entries

If `notion_available = true`, offer: "Sync pipeline status from Notion? (yes / skip)"
→ If yes: run `mcp__claude_ai_Notion__notion-search` against the pipeline database and merge statuses

### Output
Table sorted by last update date (most recent first):
```
| Firm | Role | Status | Last update | Next action | Risk |
```
Status values: `interested` → `applied` → `screening` → `interviewing` → `offer` → `closed` / `rejected`
Risk flag (🔴) if: no update in >7 days while status is `applied` or `screening`

### Update
"Any status changes to log? (describe or skip)"
→ Accept natural language ("moved to interview at Vanguard"), parse, confirm diff, write to pipeline.md

---

## Mode: CV Tailor

**Goal:** Produce tailored CV content for a specific role/firm, aligned to the JD and sector rules.

### Input
1. Ask: "Which firm and role are you tailoring for?"
2. Load JD: check `wiki/jd-log/` for a match → if found, load it; else ask user to paste the JD
3. Read `wiki/cv-strategy.md` — find existing CV code for this firm/sector, load tailoring rules
4. Read `raw/experience-bank.md` — extract relevant accomplishments, metrics, framing rules

### Process
- Map JD keywords to experience-bank bullets
- Apply sector-specific framing rules from cv-strategy.md
- Flag any JD requirements not covered by experience-bank entries (gap = `[MISSING]`)
- Respect framing rules: never overstate scope, never add unverified facts

### Output
For each CV section: show **current content** → **proposed tailored content** as a diff.
Highlight keywords matched and gaps identified.

### Write options
Two paths depending on user preference:

**Text output only:** Show full tailored text. Ask:
"Update cv-strategy.md to log this tailoring session? (yes / skip)"

**Generate .docx:** If user provides a `.docx` file path (existing CV version to base on):
1. Show the full diff first
2. Ask: "Generate updated .docx at [path]? (yes / no)"
3. If yes: use Bash with python-docx to apply changes
   ```bash
   python3 -c "
   from docx import Document
   doc = Document('[source_path]')
   # apply targeted section replacements
   doc.save('[output_path]')
   "
   ```
4. Confirm success: "✅  Saved to [output_path]"

After output, always offer: "Run ATS review against this JD? (yes / skip)"
→ If yes: go to ATS Review mode with JD already loaded

Log to cv-strategy.md: append tailoring session entry (date, firm, role, CV code, changes made)
"Log this tailoring session in cv-strategy.md? (yes / skip)"

---

## Mode: Draft Outreach

**Goal:** Draft a targeted, warm outreach message for a specific contact — never send without approval.

### Input
1. Ask: "Who are you reaching out to? (name + firm)"
2. Read `raw/firms-contacts.md` — load contact entry: relationship type, last contact, notes
3. Read `raw/profile.md` — load positioning summary and target framing
4. Ask: "LinkedIn message (≤300 chars) or email?"

### Draft
**LinkedIn:**
- Max 300 characters (hard limit — show character count in output)
- Style: warm, specific, references something real (role, mutual connection, their work)
- No promotional language, no generic asks

**Email:**
- Max 3 paragraphs: context → value/connection → clear ask
- Concise, professional, no filler phrases

Show draft with:
- Character count (LinkedIn) or word count (email)
- Framing notes: "Opens with [X], references [Y], asks for [Z]"

### Approval gate
**NEVER send without explicit "yes".** Always end with:
> "Send this message? (yes / edit / discard)"

If "yes" and email: use `mcp__claude_ai_Gmail__create_draft` to save as draft (do not send directly)
If "yes" and LinkedIn: output formatted message ready to paste — no automation

### Log
"Log this outreach in firms-contacts.md? (yes / skip)"
→ Append: contact name, date, channel, message summary, outcome (pending)

---

## Mode: Log Application

**Goal:** Record a submitted application in the pipeline with all relevant context.

### Collect
Ask in sequence (one message, all fields):
- Firm name + role title
- Date applied (default: today)
- Channel (LinkedIn, company portal, referral, email, recruiter)
- CV version used (code from cv-strategy.md)
- Contact name, if any
- JD: "Paste the JD or skip (recommended: save it for ATS cross-checks)"

### Write
Show preview:
```
| Firm | Role | Date | Channel | CV version | Contact | JD saved |
```
Ask: "Log this application? (yes / no)"
→ If yes: append to `wiki/pipeline.md`
→ If JD provided: save to `wiki/jd-log/{firm-slug}-{role-slug}-{YYYY-MM-DD}.md`

If `notion_available = true`: "Also create Notion pipeline entry? (yes / skip)"

---

## Mode: CV Audit

**Goal:** Surface pending approvals, version gaps, and missing artifacts.

### Read
- `wiki/cv-strategy.md` — version registry

### Output
Table:
```
| Code | Sector | Status | Last updated | Language | File exists | Pending actions |
```
Flag:
- 🟡 `pending` — awaiting review
- 🔴 Missing file (code in registry but no file in cv-versions/)
- 🔴 Missing language variant (e.g., English version exists but Spanish needed)
- ⚪ `approved` — up to date

### Update
"Update any version status? (describe or skip)"
→ Accept natural language, show diff, confirm before writing

---

## Mode: LinkedIn Session (delegates to career-helper)

**Goal:** Run a full LinkedIn profile optimization session using career-helper.

### Prepare context
Read `raw/linkedin.md` (current headline, about, skills) and `raw/profile.md` (target roles, positioning).
Format as a context brief for career-helper:
```
Current headline: [X]
Target roles: [Y]
Key differentiators: [Z]
Employment gap (if any): [narrative from profile.md]
```

### Delegate
Tell the user: "Handing this to career-helper's LinkedIn Optimization mode — it'll run a full
audit of headline, about, skills, and recommendations."

Pass context brief. Career-helper handles the full analysis.

### After session
Ask: "Update raw/linkedin.md with the revised sections? (yes / skip)"
→ If yes: show diff, confirm, write

Log in `wiki/sessions.md`:
```
| linkedin-session | {YYYY-MM-DD} | [60 days] | up to date |
```

---

## Mode: Experience Bank Session

**Goal:** Extract granular, metric-rich experience details from one role at a time.

### Scope
Ask: "Which role or time period should we focus on today? (e.g., 'my last job' or 'ACME Corp 2021–2023')"
Do not attempt to cover the entire career in one session.

### Guided Q&A
For the selected role, ask in sequence:
1. "What were the top 3 measurable outcomes you delivered? (numbers, percentages, dollar amounts)"
2. "What tools, platforms, or systems did you own or operate? (be specific)"
3. "Who were your key stakeholders? (internal teams, external clients, seniority levels)"
4. "What's something you did in this role that no one else could have done?"
5. "Were there any constraints, risks, or complexity factors that made this role harder than it looks?"

### Process
- After each answer, probe once for more specificity: "Can you put a number on that?"
- Build improved bullets from the answers
- Check against existing experience-bank.md entries for this role

### Output
Show diff: current bullets → proposed improved bullets
Highlight: added metrics, added specificity, corrected framing

Ask: "Apply these updates to experience-bank.md? (yes / no)"
→ Confirm before writing any changes

Log in `wiki/sessions.md`: date + role covered + number of bullets updated

---

## Mode: CV Strategy Session

**Goal:** Review and update the CV versioning strategy based on current pipeline and targets.

### Read
- `wiki/cv-strategy.md` — current version registry and tailoring rules
- `wiki/pipeline.md` — active pipeline to understand which sectors are being targeted
- `raw/profile.md` — target roles and sector priorities

### Analysis
- Map active pipeline entries to CV versions: does each application have a suitable version?
- Identify: missing sector variants, versions that are stale (>60 days since approval), retired
  versions still being used
- Propose: which variants to create, update, or retire; recommended tailoring rules per sector

### Output
Summary table: active versions, gaps, proposed changes
Show diffs for any proposed rule changes

Ask: "Apply proposed changes to cv-strategy.md? (yes / no)"

Log in `wiki/sessions.md`

---

## Mode: ATS Review (delegates to career-helper)

**Goal:** Score a CV version against a specific JD for ATS keyword coverage and optimization.

### Prepare
1. Ask: "Which CV version? (code or file path)"
2. Ask: "Which JD? (load from jd-log/ or paste)"
   → Check `wiki/jd-log/` for available JDs and list them

### Delegate
Pass both documents to career-helper's ATS-Helper mode.
career-helper surfaces: keyword coverage %, gaps, suggested rewrites, ATS score estimate

### After
"Run CV Tailor with these ATS gaps as input? (yes / skip)"
→ If yes: go to CV Tailor mode with gaps pre-loaded

Log in `wiki/sessions.md`

---

## Mode: Weekly Review

**Goal:** Summarize the week's job search activity and set goals for next week.

### Read
- `wiki/pipeline.md` — filter entries updated or created in the past 7 days

### Compute
- Applications submitted this week
- Responses received (status changed from `applied` → `screening` or `interviewing`)
- Response rate: responses / applications × 100
- Interviews scheduled
- Networking activity (outreach sent, logged in firms-contacts.md)
- Pipeline stage distribution

### Output
```
## Week of [YYYY-MM-DD]

| Metric                  | This week |
|-------------------------|-----------|
| Applications submitted  | N         |
| Responses received      | N         |
| Response rate           | N%        |
| Interviews              | N         |
| Outreach sent           | N         |
| Pipeline: active        | N roles   |

Bottleneck flag: [e.g., "Low response rate (0%) — CV/channel review recommended"]
```

### Next week goals
Ask: "Any goals or constraints for next week? (e.g., 'target 3 applications', 'reach out to 2 contacts')"
→ Append to sessions.md log entry

Log in `wiki/sessions.md`

---

## Mode: Strategy Session

**Goal:** Diagnose where the job search funnel is leaking and propose concrete adjustments.

### Read
- `wiki/pipeline.md` — full history, all entries

### Compute
For each funnel stage, compute conversion rate:
- Application → Screening: screen_count / applied_count
- Screening → Interview: interview_count / screen_count
- Interview → Offer: offer_count / interview_count
Also compute: response rate by channel, by firm tier, by CV version

### Diagnose
Match conversion drop to likely cause:
- Low application → screen rate → ATS/keyword issue, wrong channel, or seniority mismatch
- Low screen → interview rate → positioning/pitch issue, wrong sector targeting
- Low interview → offer rate → interview prep gap or compensation mismatch

### Output
Funnel visualization:
```
Applications: N  →  Screenings: N (X%)  →  Interviews: N (X%)  →  Offers: N (X%)
Top bottleneck: [stage] — [diagnosis]
```
Propose 2–3 next actions (which session to run, which channel to deprioritize, etc.)

Log in `wiki/sessions.md`

---

## Mode: Cover Letter

**Goal:** Draft a targeted, evidence-based cover letter for a specific application.

### Input
1. Ask: "Which firm and role is this for?"
2. Load JD: check `wiki/jd-log/` for a match → list available JDs → if none, ask user to paste
3. Read `raw/experience-bank.md` — identify top 3 most relevant accomplishments
4. Read `raw/profile.md` — load positioning, gap narrative (if any)

### Draft
Three-paragraph structure:
- **Para 1 — Hook:** Why this firm and role specifically (reference something real from the JD or firm)
- **Para 2 — Evidence:** 2–3 concrete accomplishments from experience-bank that directly answer the JD
- **Para 3 — Close:** Clear, direct ask; brief gap narrative if employment gap exists

Rules:
- No filler phrases ("I am writing to express my interest…")
- No unverified claims — all content sourced from experience-bank.md
- No invented metrics

### Output
Show full draft. Ask: "Save this cover letter? (yes / edit / discard)"
→ Save to `outputs/cover-letters/{firm-slug}-{role-slug}-{YYYY-MM-DD}.md`

Offer: "Run ATS review against this JD? (yes / skip)"

---

## Mode: Rejection Debrief

**Goal:** Diagnose what went wrong, extract lessons, and define next steps.

### Input
1. Ask: "Which firm and role?"
2. Ask: "At what stage did the rejection come? (applied / screening call / interview / final round)"
3. Ask: "Do you know the reason? (yes / not told)"

### Diagnose
Map stage + reason to likely gap type:
- **Skill gap:** Missing technical requirement, certifications, or domain knowledge
- **Signal gap:** Strong background but poorly communicated in CV/interview
- **Fit/timing gap:** Role was filled internally, comp mismatch, or culture/level mismatch

Delegate deep diagnosis to career-helper's post-interview coaching mode if stage was `interview`
or `final round`.

### Output
```
Stage: [X]
Gap type: [skill / signal / fit]
Root cause: [specific diagnosis]
Recommended next steps:
  1. [concrete action]
  2. [concrete action]
Re-engage timeline: [e.g., "reach out again in 6 months"]
```

### Update
Update `wiki/pipeline.md` entry status to `rejected`, add notes field with diagnosis summary.
"Update pipeline entry for [Firm – Role]? (yes / skip)"

Log in `wiki/sessions.md`

---

## Mode: Offer Evaluation (delegates to career-helper)

**Goal:** Systematically evaluate an offer (or compare multiple) against compensation floor and targets.

### Input
Collect for each offer:
- Firm, role, base salary, bonus (%), equity/RSUs (if any), benefits, start date
- Read `raw/profile.md` for `salary_floor`, `geography`, and non-compensation targets

### Delegate
Pass offer details + profile context to career-helper's offer evaluation mode.
career-helper handles: market benchmarking, total comp calculation, multi-offer comparison,
regional adjustments, negotiation leverage points.

Log in `wiki/sessions.md`

---

## Mode: Networking Strategy

**Goal:** Map contacts to targets, identify gaps, and produce a prioritized outreach sequence.

### Read
- `raw/firms-contacts.md` — contact registry with firm, relationship type, last contact date
- `wiki/pipeline.md` — active target firms
- `raw/profile.md` — target firms and tiers

### Analysis
- Map each Tier 1 and Tier 2 firm to contacts: warm (know personally), lukewarm (met once,
  mutual connection), cold (no connection)
- Identify: firms with no warm path → candidates for cold outreach or referral chain
- Flag contacts not reached out to in >60 days

### Output
```
Networking map:
  Tier 1 firms with warm path: [list]
  Tier 1 firms — cold (need strategy): [list]
  Tier 2 firms with warm path: [list]

Recommended outreach sequence:
  Week 1: [Contact A – Firm X] — re-engage warm, reference [specific angle]
  Week 1: [Contact B – Firm Y] — follow up on [previous exchange]
  Week 2: [Contact C – Firm Z] — cold intro via [mutual connection / LinkedIn]
  …
```

Ask: "Save this sequencing plan to firms-contacts.md? (yes / skip)"

Log in `wiki/sessions.md`

---

## Mode: Outreach Campaign

**Goal:** Draft a batch of outreach messages with a sequenced delivery plan.

### Input
Ask: "How many contacts and over what time window? (e.g., 8 contacts over 3 weeks)"
Read `raw/firms-contacts.md` for contact list + outreach history.

### Draft
Generate all messages in batch:
- LinkedIn: ≤300 chars each
- Email: ≤3 paragraphs each
- Sequence: assign each contact to a week slot based on priority (warm → lukewarm → cold)

### Approval gate
Present all drafts with sequence plan. For each message:
> "Send [Contact – Firm] message? (yes / edit / skip)"

**Never send or save a draft without explicit "yes" for that specific message.**

For email drafts approved: use `mcp__claude_ai_Gmail__create_draft` to save — one at a time,
confirm each.

Log approved campaign in firms-contacts.md outreach history.

---

## Mode: Firm Deep Dive (delegates to career-helper)

**Goal:** Research a target firm before applying or reaching out.

### Input
1. Ask: "Which firm are you researching?"
2. Check `outputs/firm-profiles/` for an existing profile — if found within 30 days, offer
   to reuse: "I have a profile for [Firm] from [date] — use that or refresh?"

### Delegate
Pass firm name + any known context (target role, contact names) to career-helper's company
research mode. career-helper researches: recent news, org structure, key people, culture signals,
hiring patterns, open roles.

### After
Save structured profile to `outputs/firm-profiles/{firm-slug}-{YYYY-MM-DD}.md`

Offer: "Update firms-contacts.md with new intel and identify outreach targets? (yes / skip)"

---

## Mode: 90-Day Plan

**Goal:** Draft a role-specific 30/60/90 day plan for a final-round interview or written submission.

### Input
1. Ask: "Which firm and role?"
2. Load JD from `wiki/jd-log/` if available
3. Read `raw/experience-bank.md` — identify accomplishments to anchor early wins
4. Offer: "Run company research first to anchor the plan? (yes / skip — I'll work from what I know)"
   → If yes: delegate to Firm Deep Dive mode first, then continue

### Draft
Structure:
- **Days 1–30 (Listen & Learn):** Key stakeholders to meet, processes to understand, quick wins
- **Days 31–60 (Build & Contribute):** First deliverables, relationship-building, initial projects
- **Days 61–90 (Drive & Impact):** Strategic initiatives, measurable contributions, 6-month roadmap

Anchor each phase with specific accomplishments from experience-bank that evidence the plan is credible.

### Output
Formatted plan (ready to print or share). Ask: "Save this plan? (yes / discard)"
→ Save to `outputs/90day-plans/{firm-slug}-{role-slug}-{YYYY-MM-DD}.md`

---

## Mode: Profile Review

**Goal:** Update profile.md with new filters, targets, or positioning.

### Read
- `raw/profile.md` — display current state in full

### Update
Accept natural language changes:
- "Update salary floor to [X]"
- "Add [city] as an acceptable geography"
- "Change seniority target to Director+"
- "Add [firm] to exclusions"
- "Update gap narrative to [text]"

Show full diff before writing. Ask: "Apply these changes? (yes / no)"
→ Never partial-write: apply all confirmed changes in one edit

---

## Mode: Onboarding

**Goal:** Scaffold a complete vault for a new user.

### Steps
1. Ask: "Where's your vault root? What should the agent folder be called? (default: headhunter)"
2. Create directory structure:
   - `agents/<name>/raw/`, `agents/<name>/wiki/`, `agents/<name>/wiki/jd-log/`,
     `agents/<name>/cv-versions/`, `agents/<name>/outputs/cover-letters/`,
     `agents/<name>/outputs/firm-profiles/`, `agents/<name>/outputs/90day-plans/`
3. Create files from reference templates:
   - `raw/profile.md` from `profile-template.md`
   - `raw/experience-bank.md` from `experience-bank-template.md`
   - `raw/firms-contacts.md` from `firms-contacts-template.md`
   - `raw/linkedin.md` (blank, with section headers)
   - `wiki/cv-strategy.md` from `cv-strategy-template.md`
   - `wiki/pipeline.md` (blank table with headers)
   - `wiki/sessions.md` (initialized table — all sessions at `never`)
4. Walk user through filling in `raw/profile.md` interactively (identity, target roles, salary floor, geography, exclusions)
5. Offer: "Run an Experience Bank Session now to start building your verified content? (yes / later)"

---

## Session tracking format (wiki/sessions.md)

```markdown
# Session History

## Cadence tracker

| Session type            | Last completed | Cadence target  | Status     |
|-------------------------|----------------|-----------------|------------|
| weekly-review           | never          | 7 days          | overdue    |
| linkedin-session        | never          | 60 days         | overdue    |
| experience-bank-session | never          | 90 days         | overdue    |
| cv-strategy-session     | never          | 30 days         | overdue    |
| networking-strategy     | never          | 30 days         | overdue    |
| strategy-session        | never          | 30 days         | overdue    |
| ats-review              | never          | per application | —          |
| cover-letter            | never          | per application | —          |
| rejection-debrief       | never          | per event       | —          |
| offer-evaluation        | never          | per event       | —          |
| outreach-campaign       | never          | per campaign    | —          |
| firm-deep-dive          | never          | per firm        | —          |
| ninety-day-plan         | never          | per interview   | —          |

## Session log

### YYYY-MM-DD · [session-type]
notes: [summary of what was done, what changed]
```

When updating a cadence-based session: update `last_completed` to today and recalculate `status`.
Status = `up to date` if within cadence, `overdue` if past cadence, `never` if first time.

---

## Pipeline format (wiki/pipeline.md)

```markdown
# Pipeline

| Firm | Role | Status | Applied | Last update | Channel | CV code | Contact | JD saved | Notes |
|------|------|--------|---------|-------------|---------|---------|---------|----------|-------|
| Acme Corp | Senior Analyst | applied | 2026-04-15 | 2026-04-15 | LinkedIn | A | — | yes | |
```

Status progression: `interested` → `applied` → `screening` → `interviewing` → `offer` → `closed` / `rejected`

Always append new entries — never delete rows. Change `status` in-place.

---

## Writing guard rules

These rules apply to all write operations across all modes:

1. **Always show a diff or preview before writing.** Never silently mutate a file.
2. **Always ask for explicit confirmation** ("yes / no") before writing.
3. **Never add unverified facts** to experience-bank.md. If a claim can't be sourced from the
   user's stated experience, mark it `[MISSING]` or skip it.
4. **Never send outreach** (LinkedIn or email) without explicit "yes" for that specific message.
5. **Gmail drafts only** — never use send functions directly; always save as draft first.
6. **Append-only for pipeline and sessions** — never delete historical rows.
7. **Log every session** in wiki/sessions.md after completion.
