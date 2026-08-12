# AI Job Hunt Agent

An explainable AI-powered job hunting framework: a Google Sheets tracker, a
set of carefully-written LLM prompts, and documentation designed to
eventually plug into an n8n-based automation - built with a human-approval
gate that never goes away.

**Current status: manual, prompt-driven framework.** Nothing in this repo
runs automatically today. You run the prompts yourself through an LLM
(Claude, ChatGPT, or any capable model) and update the spreadsheet by hand.
The n8n automation described below is a documented design, not a working
integration - see [docs/AUDIT.md](docs/AUDIT.md) for a full breakdown of
what's implemented vs. planned.

## What it does

- **Validates** job listings for quality/risk signals before you invest time
  in them ([prompts/job_validation_prompt.md](prompts/job_validation_prompt.md))
- **Scores** each listing against your CV on a transparent 6-factor rubric,
  with hard requirement gates so a good weighted score can never hide a
  disqualifying issue like missing work authorization
  ([prompts/matching_prompt.md](prompts/matching_prompt.md),
  [config/scoring_rules.yaml](config/scoring_rules.yaml))
- **Drafts** a tailored CV and cover letter for anything that clears the bar
  - never fabricating experience, and never sent automatically
  ([prompts/cv_tailoring_prompt.md](prompts/cv_tailoring_prompt.md),
  [prompts/cover_letter_prompt.md](prompts/cover_letter_prompt.md))
- **Tracks** the full pipeline from discovery to offer/rejection in a
  spreadsheet with a live Dashboard
  ([templates/Job_Hunt_Template.xlsx](templates/Job_Hunt_Template.xlsx))

## Current capabilities vs. limitations

| | |
|---|---|
| **Works today** | The scoring rubric, validation prompt, CV/cover-letter prompts, and spreadsheet template - all run manually through an LLM of your choice. |
| **Not implemented** | Job discovery/scraping, deduplication, and follow-up reminders are documented but not automated. There is no code in this repository. |
| **Planned** | An n8n workflow to automate discovery, validation, and scoring - with the human-approval gate kept fully intact. See [workflows/README.md](workflows/README.md). |

See [docs/AUDIT.md](docs/AUDIT.md) for the full IMPLEMENTED / PARTIAL /
PLANNED breakdown of every claimed capability.

## Architecture

```
Job Sources -> Validate -> Deduplicate -> Score -> Qualify -> Draft CV/Cover Letter
   -> Human Review -> You Apply -> Track -> Follow-up
```

Full system design: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
Data shapes and deduplication strategy: [docs/DATA_MODEL.md](docs/DATA_MODEL.md)
Step-by-step pipeline and status lifecycle: [docs/WORKFLOW.md](docs/WORKFLOW.md)

## Repository structure

```
ai-job-hunt-agent/
├── README.md
├── LICENSE                          - MIT
├── .gitignore
├── .env.example                     - only relevant if you build the n8n automation
├── docs/
│   ├── AUDIT.md                     - what's implemented vs. planned, and why
│   ├── ARCHITECTURE.md              - system design, agent roles, security & cost notes
│   ├── DATA_MODEL.md                - candidate/job record shapes, deduplication strategy
│   ├── WORKFLOW.md                  - human-approval pipeline, status lifecycle
│   ├── SETUP.md                     - how to actually use this, step by step
│   └── CHANGELOG.md
├── prompts/
│   ├── job_validation_prompt.md     - listing quality/risk assessment
│   ├── matching_prompt.md           - 0-100 scoring rubric
│   ├── cv_tailoring_prompt.md       - tailored CV generation
│   └── cover_letter_prompt.md       - tailored cover letter generation
├── templates/
│   ├── Job_Hunt_Template.xlsx       - Dashboard + Job Database + charts, pre-filled with examples
│   └── candidate_profile.md         - fillable profile you paste into the prompts
├── examples/
│   ├── sample_job.json
│   ├── sample_match.json
│   └── sample_candidate.json
├── config/
│   └── scoring_rules.yaml           - weights, hard gates, output schema
└── workflows/
    └── README.md                    - planned n8n architecture (not yet built)
```

## Setup

See [docs/SETUP.md](docs/SETUP.md) for the full walkthrough. Short version:

1. Download `templates/Job_Hunt_Template.xlsx`, open with Google Sheets.
2. Fill in `templates/candidate_profile.md` with your real details.
3. For each job: run it through `job_validation_prompt.md`, then
   `matching_prompt.md`. If it clears the bar, run `cv_tailoring_prompt.md`
   and `cover_letter_prompt.md`.
4. Record everything in the Job Database sheet. Review the drafts yourself,
   then apply through the employer's actual process - nothing here submits
   anything for you.

## Matching methodology

Six weighted sub-scores (skills 25%, experience 20%, education 10%,
responsibilities 20%, seniority 10%, location 15%), plus two 0-5 flags
(salary/career value, application feasibility) used to break ties - not to
override hard gates. See [config/scoring_rules.yaml](config/scoring_rules.yaml)
for the full rule set, including the hard gates that force a SKIP or
LOW_PRIORITY recommendation regardless of the weighted score (missing work
authorization, mandatory certification gaps, unacceptable commute, salary
far below your floor).

## Human approval

Nothing in this repository ever submits an application, sends an email, or
takes any irreversible action. Every prompt stops at "here's a draft." You
read it, edit it if needed, and apply yourself. This is a design constraint,
not a feature that might get removed later - see
[docs/WORKFLOW.md](docs/WORKFLOW.md).

## Future automation

The `workflows/` and `.env.example` files describe (but do not implement) an
n8n-based pipeline that would automate discovery, deduplication, validation,
and scoring - while keeping human review and application submission fully
manual. If you build this automation, a PR with the resulting n8n workflow
export is welcome.

## Roadmap

- [ ] n8n workflow export for scheduled job-board polling
- [ ] Automated deduplication against the canonical job ID
- [ ] Skill-gap tracker (recurring "missing skills" across low-scoring jobs)
- [ ] Multi-CV support (different master CVs for different role types)

## Contributing

Issues and PRs welcome - especially around adding job-board search patterns,
improving the scoring rubric, building the n8n workflow described in
`workflows/README.md`, or adapting the sheet for non-UK job markets.

## License

MIT - see [LICENSE](LICENSE). Use it, fork it, adapt it for your own job search.
