# 0002: Agents and code

Date: 2026-11-27
Author: Gul Zareen

## Decision
- Three daily agents: Investigator, Skill writer, Reporter.
- One testing-only agent: Red-team.
- Everything else is plain code, including the verifier, the escalation
  router and the emergency-mode switch.
- Investigator handles both missing-report and anomaly cases; its memory is
  scoped by case type.
- "Emergency mode" (any declared disaster: flood, earthquake, epidemic,
  conflict) replaces the earlier "flood mode".

## Why
- Only jobs that need judgement become agents; exact jobs are cheaper and
  more reliable as code.
- In our G-Memory runs, the helper agent repeated the solver because it saw
  the same input. A code verifier cannot be fooled that way.
- In PDDL, memory leaked between different games. Scoping memory by case
  type fixes this without adding agents.
- Disasters in Balochistan are not only floods; one general emergency mode
  covers all of them with the same mechanism (a rule-version switch).

## Alternatives considered
- Option B: separate Completeness and Anomaly agents (4 agents). Rejected:
  mostly duplicated logic, more to build and explain.
- Separate modes per disaster type. Rejected: same mechanism each time;
  the emergency type is stored as data, not as separate code.

## Consequences
- guardmind/agents/ holds three agents (plus red-team for evaluation).
- guardmind/rules/ and guardmind/escalation/ hold the code parts.
- Rule versions carry an emergency type and an active period.
