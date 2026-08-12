# Candidate Profile

Fill this in with your real information, then reuse it (paste the relevant
parts) whenever you run the prompts in `prompts/`. Nothing here is submitted
anywhere automatically - it's a working reference document.

## Basic info

- **Name**:
- **Target roles**: (e.g. Data Analyst, BI Analyst, Analytics Engineer)
- **Location**:
- **Work authorization**: (e.g. UK citizen, requires sponsorship, etc.)

## Compensation & logistics

- **Minimum acceptable salary**:
- **Preferred salary**:
- **Remote preference**: remote / hybrid / on-site / no preference
- **Maximum commute**: (distance or time, and whether it's a hard limit)

## Skills

List your skills. Mark the ones that are genuinely non-negotiable
requirements for you as `must_have: true` - these feed the hard gates in
`config/scoring_rules.yaml`.

```
- skill: SQL
  must_have: true
- skill: Python
  must_have: false
- skill: Tableau
  must_have: false
```

## Experience

For each role:

```
- title:
  company:
  dates:
  achievements:
    -
    -
```

## Education

```
- degree:
  institution:
  dates:
```

## Certifications

```
- name:
  mandatory: false   # set true only if a role without it is a dealbreaker
```

## Master CV

- **Reference**: link to your actual master CV document (Google Doc, Word
  file, etc.) - the CV tailoring prompt should never write a CV from
  scratch, only tailor this one.
