# Architecture

## Overview

The system is deliberately low-infrastructure: a Google Sheet is the database
and UI, an LLM (Claude or similar) does the reasoning (scoring, drafting), and
an optional automation layer (n8n) handles scheduling and job-board polling.
Nothing requires a hosted backend or database of your own.

```
Job Boards (Indeed, etc) --> Search/Scrape (manual or n8n) --> Matching Agent (LLM + rubric prompt)
                                                                       |
                                                                score >= 75
                                                                       v
Job Database (Google Sheet) <-- CV/Cover Letter Docs (Drive) <-- Drafting Agent (LLM + tailoring
       |                                                          prompts, human-reviewed before
       v                                                          sending)
Dashboard (KPIs + charts)
```

## Agent roles

1. **Search Agent** - collects job listings for the target roles/locations.
   Manual (paste listings in) works fine at low volume; n8n + job-board APIs
   or RSS feeds can automate this at scale.
2. **Matching Agent** - runs every new listing through the scoring rubric
   (prompts/matching_prompt.md) against the candidate's master CV. Outputs
   the 6 sub-scores, overall score, priority tier, and match/gap reasoning.
3. **Drafting Agent** - for anything scoring 75+, generates a tailored CV
   (prompts/cv_tailoring_prompt.md) and cover letter
   (prompts/cover_letter_prompt.md). Both are drafts only - never sent
   automatically.
4. **Tracking layer** - the Job Database sheet is the single source of truth
   for application status, follow-ups, and outcomes. The Dashboard sheet
   reads from it live via formulas + two charts (priority breakdown pie
   chart, match-score bar chart).

## Data structure (Job Database sheet)

37 tracked columns per job, grouped into:
- **Identity & sourcing** - Job ID, date found, company, title, location,
  working model, salary, employment type, URL, source, posted/closing dates
- **Scoring** - overall score, priority, the 6 sub-scores, salary/career
  value, application feasibility
- **Reasoning** - must-have skills, missing skills, why-match / why-not-match
  (kept as free text so the reasoning is auditable, not just a number)
- **Application materials** - CV version, cover letter version (hyperlinked
  to the actual generated documents)
- **Pipeline tracking** - application status, date applied, follow-up date,
  last checked, still-open flag, evidence of status, follow-up-required flag,
  notes

Conditional formatting and dropdown validations are all bounded to the actual
data range (not an arbitrary large range) - this matters more than it sounds:
Google Sheets recalculates formulas and validations across their entire
declared range on every edit, so binding a COUNTIF to row 200 when you only
have 14 rows of data makes the whole sheet noticeably laggier as it grows.

## Human-approval gate

This is the one non-negotiable rule in the system: nothing is ever
auto-submitted. The Drafting Agent's output is a Google Doc / Word file
sitting in Drive with a link in the tracker - a human reads it, edits it if
needed, and submits it themselves. Same for any follow-up email drafts. The
agent's job stops at "here's what I'd send," not "sent."

## Security & privacy notes

- The tracker will contain real salary figures, application statuses, and
  personal contact info once you fill it in - treat the Sheet's sharing
  settings the same way you'd treat any personal document.
- If you automate search with n8n or a job-board API, store credentials in
  n8n's credential store (encrypted at rest), never hard-coded in a workflow
  node.
- The scoring/drafting prompts don't need any of your accounts - only the CV
  text and the job listing text, so you can run the LLM steps through
  whatever provider you trust without granting it inbox or Drive access if
  you'd rather keep those steps fully manual.

## Cost notes

- Google Sheets: free.
- LLM calls: the matching prompt is short (~1 job + 1 CV per call); at normal
  job-search volume (10-30 listings/week) this stays well within low-cost or
  free-tier LLM usage. CV/cover-letter drafting only runs for the subset that
  scores 75+, so volume stays low.
- n8n: free if self-hosted; the cloud tier's free allowance is generally
  enough for a personal, non-commercial job search workflow.

## Suggested phased build order

1. **Phase 1 (this template)** - manual search, manual paste into the
   matching prompt, manual tracker updates. Get the rubric and prompts right
   before automating anything.
2. **Phase 2** - automate search with n8n (scheduled job-board polling ->
   Google Sheets row creation) while keeping scoring/drafting as an
   LLM-in-the-loop step you trigger.
3. **Phase 3** - full pipeline: n8n triggers the matching prompt via API for
   every new row automatically, drafts CVs for 75+ scores, and pings you
   (email/Slack) with a daily digest of what needs review - still with the
   human-approval gate intact before anything is sent.
