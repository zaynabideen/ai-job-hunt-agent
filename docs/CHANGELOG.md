# Changelog

## Audit-driven update (this update)

### Fixed

- **Broken file reference**: README.md and ARCHITECTURE.md referenced
  `templates/Job_Hunt_Template.xlsx`, which did not exist in the repository.
  The file had actually been uploaded but landed as a mangled, truncated
  filename (`JOB_HU~1.XLS`) at the repo root instead of the documented path -
  renamed/moved to `templates/Job_Hunt_Template.xlsx`.
- **Weighted score could hide disqualifying issues**: the matching prompt
  had no mechanism to stop a job with (e.g.) incompatible work authorization
  from still scoring 90/100. Fixed with hard gates in
  `config/scoring_rules.yaml`.
- **No structured output** from any prompt - all three original prompts
  produced free text. Fixed with explicit JSON output schemas.
- **Scam/quality flagging was a single soft instruction** buried in the
  matching prompt's ground rules, with no risk tiers or evidence requirement.
  Split into its own prompt with mandatory evidence.

### Added

- `docs/AUDIT.md` - full repository audit: structure, broken references,
  documentation-vs-implementation table, prompt quality review, architecture
  gaps.
- `docs/DATA_MODEL.md` - candidate profile and job record schemas, plus a
  deterministic deduplication strategy (canonical job ID from normalized
  company + title + location).
- `docs/WORKFLOW.md` - the human-approval pipeline as a diagram, the full
  application lifecycle state machine, and valid transitions between states.
- `docs/SETUP.md` - step-by-step setup and usage instructions.
- `config/scoring_rules.yaml` - deterministic weights, sub-score
  definitions, hard requirement gates, priority tiers, and output schema.
- `prompts/job_validation_prompt.md` - dedicated listing-quality/risk
  assessment prompt with LOW/MEDIUM/HIGH_RISK tiers and mandatory evidence.
- `templates/candidate_profile.md` - fillable candidate profile template.
- `examples/sample_job.json`, `examples/sample_match.json`,
  `examples/sample_candidate.json` - worked examples matching the schemas
  above.
- `.env.example` - documents the configuration a future n8n automation would
  need; explicitly labeled PLANNED, nothing in the repo reads it yet.
- `workflows/README.md` - documents the recommended n8n architecture;
  explicitly labeled PLANNED, no workflow JSON included since none is
  implemented.

### Changed

- `README.md` rewritten to state current capabilities and limitations
  honestly (removed language implying automation that doesn't exist yet),
  and updated the repo structure diagram to match reality.

### Remaining (needs external implementation, not fixable from documentation alone)

- Job-board scraping/API integration for automated discovery - none exists;
  each source would need its own integration and terms-of-service review.
- An actual n8n deployment running the workflow described in
  `workflows/README.md`.
- Google Sheets API / OAuth setup, if the sheet is ever read/written by
  automation instead of by hand.
- An LLM API key, if the prompts are ever run programmatically instead of
  pasted into a chat interface.
- Notification integrations (email/Telegram/Slack) - documented in
  `.env.example` but not built.
- Automated deduplication - the strategy is documented in
  `docs/DATA_MODEL.md`, but applying it today is still a manual check.
