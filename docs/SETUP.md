# Setup

## What you need

- A Google account (for Google Sheets + Google Drive)
- Access to an LLM you're comfortable pasting your CV and job listings into
  (Claude, ChatGPT, or any capable model) - this repo does not include or
  require an API key of its own
- Your master CV as plain text (for pasting into prompts)

## 1. Get the spreadsheet

Download `templates/Job_Hunt_Template.xlsx` from this repo, then in Google
Drive: right-click -> **Open with -> Google Sheets**. This converts it to a
live, editable Sheet with the Dashboard, Job Database, and How To Use tabs
intact, including the two charts.

## 2. Fill in your candidate profile

Copy `templates/candidate_profile.md` somewhere you'll reuse it (a Google
Doc, a local file - anywhere), and fill in your real details: target roles,
location, salary floor, must-have skills, work authorization, etc. You'll
paste relevant parts of this into the prompts below.

## 3. Run the prompts manually

For each job listing you want to evaluate:

1. **Validate** - paste the listing into `prompts/job_validation_prompt.md`
   and run it through your LLM. Read the risk tier before doing anything
   else with a MEDIUM_RISK or HIGH_RISK listing.
2. **Score** - paste the listing + your CV into `prompts/matching_prompt.md`.
   Use `config/scoring_rules.yaml` as your reference for weights and hard
   gates if your LLM needs the explicit rules spelled out.
3. **Draft materials** (only for CONSIDER and above) - run
   `prompts/cv_tailoring_prompt.md` and `prompts/cover_letter_prompt.md`.
4. **Record** - add or update the row in the Job Database sheet with the
   scores, recommendation, and status (see docs/WORKFLOW.md for valid
   status values).
5. **Review and apply yourself** - nothing in this repo submits an
   application. You read the drafts, edit if needed, and apply through the
   employer's actual application process.

## 4. Environment variables (only needed if you automate)

See `.env.example`. These are only relevant if/when you build the n8n
automation described in docs/ARCHITECTURE.md - running the prompts manually
through a chat interface requires none of them.

## Verifying your setup

- Open the Sheet and confirm you see three tabs: Dashboard, Job Database,
  How To Use.
- Confirm the Dashboard's two charts render (they read from the Job Database
  tab, so they'll be empty/flat until you add rows).
- Run one job listing through `prompts/job_validation_prompt.md` and
  `prompts/matching_prompt.md` end to end before doing this for real, so you
  know what the output should look like.
