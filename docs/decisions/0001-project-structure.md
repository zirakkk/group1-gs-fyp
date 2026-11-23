# 0001: Project structure and data rules

Date: 2026-11-23
Author: Gul Zareen

## Decision
- Code lives in guardmind/, split by component: rules, agents, memory,
  skills, escalation, data, pipeline.
- Fixed checks go in rules/ (plain code); only jobs that need judgement
  become agents.
- tests/ has unit, integration and fixtures.
- Three data sources: synthetic (committed if small), DHIS2 demo (not
  committed), real BHMIS data (never committed, local machine only).
- Git flow: feature branches -> develop (tested together) -> main (stable).

## Why
- Each folder matches one part of the design, so code is easy to find,
  review and divide between team members.
- Keeping fixed rules out of LLM agents makes them cheaper and more reliable.
- Real data must never leave our machine (our promise to BHMIS).
  data/.gitignore enforces this; tested with git check-ignore.

## Consequences
- New components need a new folder under guardmind/ and a matching test folder.
- Changing this structure needs a new decision file.
