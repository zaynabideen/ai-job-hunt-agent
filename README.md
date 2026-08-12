# AI Job Hunt Agent

A personal AI-powered job hunting system built on **Google Sheets + LLM prompts (+ optional n8n automation)**. It finds and scores job vacancies against your real CV, ranks them with a transparent 0-100 match rubric, drafts tailored CVs/cover letters for the strong matches, and tracks every application through to offer/rejection - all in a single spreadsheet you already know how to use.

No SaaS subscription, no black-box "AI resume matcher." Everything - the scoring logic, the prompts, the tracker - is yours to read, edit, and run.

## Why this exists

Most job-search tools either spam-apply to everything (low quality, high risk of scams) or give you a black-box match score with no explanation. This project takes the opposite approach:

- **Explainable scoring** - every match score breaks down into 6 sub-scores (skills, experience, education, responsibilities, seniority, location) plus a feasibility rating, so you know why a job scored 82 and not 95.
- - **Truthful by default** - the system is instructed to never fabricate experience, salary, or job status. Anything unverified is labeled "Not specified" / "Unverified" instead of guessed.
  - - **Human approval gate** - nothing is ever auto-applied or auto-emailed. The agent prepares; you approve and send.
    - - **One spreadsheet, not five apps** - the whole tracker lives in Google Sheets so it's inspectable, exportable, and shareable.
     
      - ## What it does
     
      - 1. **Search** - pulls job listings from job boards (Indeed, Glassdoor, CV-Library, Reed, etc.) for the roles you specify.
        2. 2. **Score** - runs each listing against your CV using the Match Score rubric (see prompts/matching_prompt.md) and assigns a 0-100 score plus a priority tier (Apply Now to Skip).
           3. 3. **Draft** - for anything scoring 75+, generates a tailored CV and cover letter (never submitted automatically).
              4. 4. **Track** - logs everything into the Job Database sheet: status, application stage, follow-ups, closing dates.
                 5. 5. **Report** - the Dashboard sheet gives you live KPIs and charts (priority breakdown, match score per job) so you can see your pipeline at a glance.
                   
                    6. ## Repo structure
                   
                    7. ```
                       ai-job-hunt-agent/
                       ├── README.md                        <- you are here
                       ├── LICENSE                          <- MIT
                       ├── docs/
                       │   └── ARCHITECTURE.md              <- full system design: agent roles, data flow, security & cost notes
                       ├── prompts/
                       │   ├── matching_prompt.md           <- the 0-100 scoring rubric prompt
                       │   ├── cv_tailoring_prompt.md       <- generates a tailored CV from your master CV + job listing
                       │   └── cover_letter_prompt.md       <- generates a tailored cover letter
                       └── templates/
                           └── Job_Hunt_Template.xlsx       <- ready-to-use spreadsheet: Dashboard, Job Database, charts, dropdowns, conditional formatting - pre-filled with example rows so you can see how it works before you plug in your own data
                       ```

                       ## Quick start

                       1. Download templates/Job_Hunt_Template.xlsx and upload it to Google Drive (right-click -> Open with -> Google Sheets to convert it to a live, editable Sheet).
                       2. 2. Replace the example rows in the Job Database sheet with your own - either by hand, or by feeding job listings + your CV through the prompts in prompts/ using Claude (or any capable LLM).
                          3. 3. Read docs/ARCHITECTURE.md if you want to wire this up to automatically search job boards on a schedule (the doc includes an n8n-based automation design - optional, the spreadsheet + prompts work perfectly well run manually too).
                             4. 4. Keep the CV Version / Cover Letter Version columns hyperlinked to your actual generated documents (Google Docs or Drive links) so clicking a row takes you straight to the tailored application.
                               
                                5. ## The scoring rubric, in short
                               
                                6. | Component | Weight | What it checks |
                                7. |---|---|---|
                                8. | Skills Match | 25% | Required tools/skills you actually have |
                                9. | Experience Match | 20% | Years + type of experience vs. what's asked |
                                10. | Education Match | 10% | Degree/certification requirements |
                                11. | Responsibilities Match | 20% | Day-to-day tasks overlap with your background |
                                12. | Seniority Match | 10% | Job level vs. your level |
                                13. | Location Match | 15% | Remote/hybrid/on-site + commute fit |
                               
                                14. Plus two non-weighted flags: Salary/Career Value (/5) and Application Feasibility (/5) - used to break ties and flag jobs that score well but aren't worth your time (e.g. unpaid, scam-shaped, or wildly underpaid listings).
                               
                                15. Priority tiers:
                                16. - Apply Now - 85-100
                                    - - High Priority - 75-84
                                      - - Consider - 65-74
                                        - - Low Priority - 50-64
                                          - - Skip - below 50
                                           
                                            - ## Ground rules the agent follows
                                           
                                            - - Never fabricates salary, experience, or application status - uses "Not specified" / "Unverified" instead.
                                              - - Never auto-submits an application or sends an email. Every CV/cover letter is a draft for you to review.
                                                - - Flags likely low-quality or scam-shaped listings (recruiter template-farming, "placement programmes" with unclear cost models, etc.) instead of silently scoring them.
                                                 
                                                  - ## Roadmap ideas
                                                 
                                                  - - [ ] n8n workflow export for scheduled job-board polling
                                                    - [ ] - [ ] Skill-gap tracker (which recurring "missing skills" show up most across your rejected/low-scoring jobs)
                                                    - [ ] - [ ] Multi-CV support (different master CVs for different role types)
                                                   
                                                    - [ ] ## Contributing
                                                   
                                                    - [ ] Issues and PRs welcome - especially around adding more job-board search patterns, improving the scoring rubric, or adapting the sheet for non-UK job markets.
                                                   
                                                    - [ ] ## License
                                                   
                                                    - [ ] MIT - see LICENSE. Use it, fork it, adapt it for your own job search.
                                                    - [ ] 
