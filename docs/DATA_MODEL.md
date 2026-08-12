# Data Model

## Overview

This document defines the data shapes used across the project: the
candidate profile, a single job record, and the deduplication strategy that
turns "the same job posted on 3 sites" into one row instead of three.

Status: this is the reference data model. The spreadsheet template
(`templates/Job_Hunt_Template.xlsx`) implements the Job Database columns
described below. There is no code in this repo that enforces this schema
automatically - see docs/AUDIT.md.

## Candidate profile

See `templates/candidate_profile.md` for the fillable version. Fields:

- name
- target_roles (list)
- location
- work_authorization
- minimum_salary
- preferred_salary
- remote_preference (remote / hybrid / on-site / no preference)
- maximum_commute
- skills (list, each optionally marked must_have: true/false)
- experience (list of roles: title, company, dates, achievements)
- education (list: degree, institution, dates)
- certifications (list, each optionally marked mandatory: true/false)
- master_cv_reference (link to the source CV document)

## Job record

Matches the Job Database sheet columns:

| Field | Type | Notes |
|---|---|---|
| job_id | string | canonical ID, see "Deduplication" below |
| title | string | |
| company | string | |
| location | string | |
| work_model | enum | remote / hybrid / on-site |
| salary | string | "Not specified" if not stated - never guessed |
| employment_type | string | full-time / part-time / contract |
| url | string | link to the original listing |
| source | string | which board/site it was found on |
| date_posted | date | |
| closing_date | date | nullable |
| date_found | date | |
| last_checked | date | |
| status | enum | see docs/WORKFLOW.md for valid values |
| overall_score | number | 0-100, from the matching prompt |
| skills_score, experience_score, education_score, responsibility_score, seniority_score, location_score | number | 0-100 each, see config/scoring_rules.yaml |
| salary_fit, career_value | number | 0-5 each |
| missing_skills | list | |
| must_have_failures | list | populated when a hard gate triggers |
| recommendation | enum | APPLY_NOW / HIGH_PRIORITY / CONSIDER / LOW_PRIORITY / SKIP |
| cv_status | enum | not_started / drafted / approved |
| cover_letter_status | enum | not_started / drafted / approved |
| application_date | date | nullable |
| follow_up_date | date | nullable |
| notes | string | free text |

## Deduplication

A single job frequently appears on multiple boards (company site, Indeed,
Reed, LinkedIn, CV-Library) with slightly different formatting each time.
Without a deduplication rule, the same job gets scored and tracked multiple
times, which pollutes the pipeline and wastes review time.

**Canonical job ID** is derived by normalizing and combining:

1. `company` - lowercased, stripped of legal suffixes (Ltd, Inc, LLC, plc)
2. `title` - lowercased, stripped of seniority prefixes/suffixes that vary by
   board (e.g. "Senior" vs "Sr.", "(Remote)" tags)
3. `location` - normalized to a city-level string
4. `source_domain` - kept separately, NOT part of the canonical key, so the
   same job on two boards resolves to the same ID

```
canonical_key = normalize(company) + "|" + normalize(title) + "|" + normalize(location)
job_id = short_hash(canonical_key)
```

When a new listing's canonical key matches an existing `job_id`:
- do not create a new row
- update `last_checked`
- if the new listing has a working URL and the existing row's URL is dead,
  update `url` and `source`
- do not re-run scoring unless the listing content materially changed
  (title, requirements, or salary differ from the stored version)

This is a documented strategy, not an implemented deduplication service -
today it means: before adding a row, manually check whether a very similar
company + title + location already exists in the sheet.
