# Profile

## Identity

Name: [Full name]
Location: [City, Country]
Languages: [e.g., English (native), Spanish (native), French (conversational)]
Email: [email]
LinkedIn: [URL]

---

## Professional positioning

One-paragraph positioning statement. What you do, what makes you distinctive, and what you're
targeting. Write in first person. No buzzwords. Focus on domain, outcomes, and differentiators.

Example: "I'm targeting senior roles in [domain] with a background in [X] and [Y]. My edge is
[specific combination that's rare or hard to replicate]."

---

## Target roles

List the specific role titles you're targeting. Be precise — this drives job scanning filters.

- [Role title 1] (e.g., VP Product Strategy)
- [Role title 2]
- [Role title 3]

---

## Target sectors

- [Sector 1] (e.g., Asset Management)
- [Sector 2]
- [Sector 3]

---

## Target firms

List firms by priority tier. headhunter uses this for networking strategy and scan scoring.

### Tier 1 (top priority)
- [Firm A]
- [Firm B]

### Tier 2
- [Firm C]
- [Firm D]

### Tier 3
- [Firm E]

---

## Hard filters

These are applied before any role is surfaced. No exceptions without explicit override.

```
seniority_floor: Senior        # minimum title level (Senior / VP / Director / MD)
salary_floor_local: 0          # monthly, local currency (set to 0 if not applicable)
salary_floor_remote_usd: 0     # annual USD for remote roles (set to 0 if not applicable)
geography:
  - [City]
  - remote
exclusions:
  - [firm or role type to never recommend]
```

---

## Employment gap narrative (if applicable)

If you have an employment gap, write the explanation here in 1–2 sentences. headhunter will
include this in cover letters and flag it for screening call prep.

Example: "My previous role was eliminated in a company restructuring — I have documentation
and strong references."

Leave blank if not applicable.

---

## Key differentiators

3–5 bullet points that capture what makes your candidacy distinctive. These inform how
headhunter frames CV bullets and outreach messages.

- [Differentiator 1 — be specific, not generic]
- [Differentiator 2]
- [Differentiator 3]

---

## Certifications and credentials (active or in progress)

| Credential | Status | Expected date |
|---|---|---|
| [e.g., CFA Level III] | [in progress] | [Month YYYY] |
| [e.g., PMP] | [active] | — |

---

## § Config

```
timezone: [e.g., America/Mexico_City]
notion_enabled: false
notion_pipeline_db:            # Notion database name or ID (if notion_enabled: true)
```
