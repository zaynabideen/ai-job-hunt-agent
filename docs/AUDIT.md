# Repository Audit

Date: 2026-08-12
Scope: full repository as of this commit.

## A. Repository structure (before this audit)

```
ai-job-hunt-agent/
├── README.md            - project overview, quick start, scoring summary
├── LICENSE               - MIT
├── .gitignore
├── docs/
│   └── ARCHITECTURE.md   - system design, agent roles, data model, security/cost notes
└── prompts/
    ├── matching_prompt.md        - 0-100 scoring rubric prompt
    ├── cv_tailoring_prompt.md    - CV tailoring prompt
    └── cover_letter_prompt.md    - cover letter prompt
```

There is no application code in this repository. Everything here is
documentation, LLM prompts, and (once added) a spreadsheet template. That is
a deliberate design choice (see ARCHITECTURE.md), not an oversight - but it
means every "capability" in this project is really "a prompt + a manual
process," not a running service. The rest of this audit is written with that
distinction in mind.

## B. Broken references found

| Reference | Referenced in | Status |
|---|---|---|
| `templates/Job_Hunt_Template.xlsx` | README.md ("Quick start", "Repo structure"), ARCHITECTURE.md | **Missing.** No `templates/` directory exists in the repository yet. This is the single most important gap: the README tells a new user to download a file that isn't there. |

No other dead links or missing file references were found in README.md or
ARCHITECTURE.md as of this audit.

## C. Documentation vs. implementation

| Claimed capability | Status | Notes |
|---|---|---|
| Match scoring rubric (6 sub-scores + weights) | **IMPLEMENTED (as a prompt)** | `prompts/matching_prompt.md` exists and defines the rubric. Scoring itself is performed by whichever LLM you run the prompt through - there is no deterministic/automated scorer in this repo. |
| Hard requirement gates (must-have skills, work authorization, salary floor, location) | **PLANNED** | Not present before this audit. A weighted score can currently rank a job highly even if it fails a non-negotiable requirement. Added in `config/scoring_rules.yaml` as part of this update. |
| CV tailoring | **IMPLEMENTED (as a prompt)** | `prompts/cv_tailoring_prompt.md` exists with clear no-fabrication rules. No structured output schema before this update. |
| Cover letter drafting | **IMPLEMENTED (as a prompt)** | `prompts/cover_letter_prompt.md` exists. Same caveat as above. |
| Job Database / Dashboard spreadsheet | **DOCUMENTATION ONLY** | Described in README and ARCHITECTURE.md, but `templates/Job_Hunt_Template.xlsx` was never committed. Anyone cloning the repo before this fix could not actually use the tracker. |
| Job search / discovery | **DOCUMENTATION ONLY** | README describes pulling listings from Indeed, Glassdoor, CV-Library, Reed. There is no scraper, API integration, or automation in this repo - this step is entirely manual (you paste a listing into the matching prompt yourself). |
| Deduplication across sources | **PLANNED** | Not addressed anywhere before this audit. A job posted on 3 boards would currently be scored 3 separate times with no canonical identity. Addressed in `docs/DATA_MODEL.md`. |
| Job/scam validation | **PARTIALLY IMPLEMENTED** | `matching_prompt.md` has an informal "flag likely scam-shaped listings" instruction bundled into its ground rules, but no dedicated risk-tiered validation step. Formalized in `prompts/job_validation_prompt.md`. |
| Human-approval gate | **IMPLEMENTED (as a policy, not code)** | Every prompt in this repo explicitly instructs "never auto-submit." Because nothing in this repo executes automatically, the gate is currently enforced by the fact that a human has to manually run every step - not by any code-level control. Documented explicitly in `docs/WORKFLOW.md`. |
| Application status tracking / lifecycle | **PARTIALLY IMPLEMENTED** | The spreadsheet has status columns, but no defined set of valid states or transitions existed before this audit. Defined in `docs/WORKFLOW.md`. |
| Follow-up reminders | **PLANNED** | Column placeholders only (`Follow-up Date`, `Last Checked`); nothing computes or surfaces them automatically. |
| Notifications (Email / Telegram / Slack) | **PLANNED** | Not implemented, and was not previously documented at all. |
| n8n automation | **PLANNED** | Mentioned as "optional" in ARCHITECTURE.md's roadmap. No workflow JSON exists in this repo, and none is claimed to exist. |

## D. Prompt quality review

**`matching_prompt.md`**
- The rubric weights are clear and sum to 100%, which is good.
- Output format was free text ("1. Overall Match Score... 2. ...") rather than
  a structured schema, which makes the prompt hard to feed into any
  downstream automation. A JSON output schema has been added as part of this
  update (see `config/scoring_rules.yaml` and the audit-driven prompt
  revision).
- No hard-gate logic: a job could score 90/100 overall while still failing a
  non-negotiable requirement (wrong work authorization, missing mandatory
  certification). This is a real risk for anyone relying on the score alone -
  addressed with `config/scoring_rules.yaml`.
- Scam/quality flagging is a single soft instruction rather than a
  structured, evidenced risk rating. Split out into its own prompt with
  LOW/MEDIUM/HIGH_RISK tiers and required evidence, to reduce the chance of
  the model asserting "this is a scam" without justification.

**`cv_tailoring_prompt.md`** and **`cover_letter_prompt.md`**
- Both have strong, explicit no-fabrication rules - this is the most
  important property of these prompts and it was already done well.
- Neither had a structured output schema before this update, and neither
  asked the model to explicitly flag anything it could not support from the
  source CV. Both gaps are addressed in this update.

## E. Architecture problems

- **Job discovery**: fully manual; no automation exists in-repo. Documented
  honestly as PLANNED in the new README rather than implied to be automatic.
- **Deduplication**: no canonical job identity existed. Addressed in
  `docs/DATA_MODEL.md` with a normalized-key approach (company + title +
  location + source domain).
- **Scoring**: previously a single LLM judgment call with no deterministic
  floor/ceiling rules or hard gates. Addressed with `config/scoring_rules.yaml`.
- **Salary / location filtering**: previously soft "career value" flags only,
  no hard filtering for below-minimum salary or unacceptable commute.
  Addressed with hard-gate rules.
- **Application lifecycle**: no defined state machine. Addressed in
  `docs/WORKFLOW.md`.
- **Follow-ups / notifications**: no design existed. Both are now documented
  explicitly as PLANNED, with a description of what would be needed to
  implement them (n8n schedule + Sheets read + notification webhook), rather
  than left unmentioned or implied to already work.
- **Error handling**: not applicable in the current form of the project since
  there is no running automation to fail - flagged here so it is not
  forgotten once any part of this becomes an actual n8n workflow.

## Summary

The core idea and the three existing prompts are solid, and the
no-fabrication / human-approval principles were already well designed. The
main problems fixed by this update are: (1) a missing file the README
depended on, (2) scoring that could hide disqualifying issues behind a good
weighted average, (3) no deduplication strategy, and (4) documentation that
did not clearly separate "this works today" from "this is the plan."
