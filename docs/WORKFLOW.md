# Workflow

## Human-approval pipeline

This is the core design constraint of the whole project: **nothing is ever
submitted without a human reading and approving it first.** Every prompt in
`prompts/` is written to stop at "here's a draft," never "sent."

```
JOB FOUND
   |
   v
VALIDATE          (prompts/job_validation_prompt.md -> risk tier)
   |
   v
DEDUPLICATE       (docs/DATA_MODEL.md -> canonical job_id)
   |
   v
SCORE             (prompts/matching_prompt.md + config/scoring_rules.yaml)
   |
   v
QUALIFY           (hard gates checked; recommendation assigned)
   |
   v
GENERATE CV              (prompts/cv_tailoring_prompt.md, only if recommendation >= CONSIDER)
   |
   v
GENERATE COVER LETTER    (prompts/cover_letter_prompt.md)
   |
   v
HUMAN REVIEW      <- you read the score, the CV, and the cover letter
   |
   v
YOU APPROVE (or edit, or reject)
   |
   v
YOU APPLY         (this repo never submits anything on your behalf)
   |
   v
TRACK APPLICATION (Job Database sheet, status + dates updated)
```

Every arrow above is currently a manual step: you run the prompt, read the
output, and move the job to the next stage in the sheet yourself. See
docs/AUDIT.md for what is and is not automated today.

## Application lifecycle (status values)

The `status` column in the Job Database should only ever hold one of these
values:

```
DISCOVERED             -> job found, not yet validated
VALIDATED               -> passed job_validation_prompt, risk tier assigned
SCORED                   -> matching_prompt has produced sub-scores + overall_score
QUALIFIED                 -> hard gates checked, recommendation assigned
CV_GENERATED                -> tailored CV drafted (only if recommendation >= CONSIDER)
COVER_LETTER_GENERATED       -> cover letter drafted
HUMAN_REVIEW                  -> waiting on your review
APPROVED                       -> you approved the materials
APPLIED                         -> you submitted the application yourself
FOLLOW_UP_DUE                    -> follow_up_date has passed with no response
INTERVIEW                         -> moved to interview stage
OFFER                              -> offer received
REJECTED                            -> rejected at any stage
CLOSED                               -> listing closed/expired before you applied
WITHDRAWN                            -> you withdrew your application
```

## Valid transitions

- `DISCOVERED -> VALIDATED -> SCORED -> QUALIFIED` is the only path into the
  pipeline; a job cannot skip validation or scoring.
- `QUALIFIED -> CV_GENERATED -> COVER_LETTER_GENERATED -> HUMAN_REVIEW` only
  happens if `recommendation` is `CONSIDER` or higher. Jobs recommended
  `LOW_PRIORITY` or `SKIP` can stay at `QUALIFIED` indefinitely, or move
  straight to `CLOSED`/`REJECTED` if you decide not to pursue them.
- `HUMAN_REVIEW -> APPROVED -> APPLIED` requires an explicit human action at
  each step - there is no automated transition between these three states.
- From `APPLIED`, a job can move to `FOLLOW_UP_DUE` (time-based), then to
  `INTERVIEW`, `OFFER`, `REJECTED`, or `WITHDRAWN`.
- Any state can move to `CLOSED` if the listing itself closes/expires.

## What's automated today vs. planned

- **Automated today**: nothing. This entire pipeline is a set of prompts and
  a spreadsheet you drive by hand.
- **Planned** (see docs/ARCHITECTURE.md's phased build order): an n8n
  workflow that triggers validation + scoring automatically when a new row
  is added to the sheet, and surfaces `FOLLOW_UP_DUE` jobs on a schedule.
  Human review and application submission remain manual by design even in
  the planned automation - that gate is not meant to be removed.
