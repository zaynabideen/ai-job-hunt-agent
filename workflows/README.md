# Workflows

## Status: PLANNED

There is no n8n workflow JSON in this repository yet. This folder documents
the recommended architecture so that when the automation is actually built,
it follows a design that has already been thought through - not so that you
can import something that doesn't exist.

Today, every step described below is performed manually by running the
relevant prompt from `prompts/` through an LLM and updating the spreadsheet
by hand. See docs/WORKFLOW.md for the human-approval pipeline this
automation would sit inside of.

## Recommended architecture (not yet built)

```
Job Sources (Indeed, Reed, LinkedIn, CV-Library, company sites)
   |
   v
n8n (scheduled trigger, polls sources)
   |
   v
Normalize Job          - map each source's format to the docs/DATA_MODEL.md job record shape
   |
   v
Deduplicate             - compute canonical job_id (docs/DATA_MODEL.md), skip if already tracked
   |
   v
Validate                 - run prompts/job_validation_prompt.md via LLM node, get risk_tier
   |
   v
AI Matching                - run prompts/matching_prompt.md via LLM node, get scores + recommendation
   |
   v
Google Sheets                - write/update the row in the Job Database (status: SCORED/QUALIFIED)
   |
   v
CV Generator                   - run prompts/cv_tailoring_prompt.md, only if recommendation >= CONSIDER
   |
   v
Cover Letter Generator            - run prompts/cover_letter_prompt.md
   |
   v
Human Approval                      - notify the candidate; wait for explicit approval (not automated)
   |
   v
Application Tracking                  - candidate applies manually; status updated to APPLIED
   |
   v
Follow-up                               - scheduled check against follow_up_date
   |
   v
Notification                              - email/Telegram/Slack (see .env.example - none wired up yet)
```

## Why n8n specifically

n8n is suggested because it can run on a free/self-hosted tier, has native
Google Sheets nodes, and can call an LLM API directly from a workflow node -
so the whole pipeline above can run without a custom backend. This is a
recommendation, not a requirement: the same architecture works with any
workflow tool that can poll a source, call an LLM, and write to Sheets.

## What would need to be true before this is real

- A working n8n instance (self-hosted or cloud)
- Google Sheets API credentials (see `.env.example`)
- An LLM API key
- Actual job-board scraping/API access for each source (this repo does not
  include or endorse any specific scraping method - check each site's terms
  of service)
- A notification channel wired up (email/Telegram/Slack)

None of the above exists in this repository today. If you build this, please
consider opening a PR with the resulting workflow so this section can be
updated from PLANNED to IMPLEMENTED.
