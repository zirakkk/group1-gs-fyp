# 0003: Memory design

Date: 2026-11-27
Author: Gul Zareen

## Decision
- Four stores: raw data (code only), case store, skill store, rule registry;
  plus an audit log.
- Only officer-confirmed cases are stored. Nothing is deleted.
- Reading memory: skills first; then hard filters (role, confirmed, case
  type, rule version, mode); then ranking; below the confidence threshold,
  nothing is shown and the case escalates.
- Rule changes and emergencies are handled through rule versions: old
  memory is kept but not used outside its valid period.

## Why
- In our G-Memory runs, memory used an example it had itself rated 2/10,
  and showed a cocktail example for a tyre task. Hard filters and a
  threshold prevent both.
- Raw data apart from everything else keeps private data structurally
  away from LLMs and the cloud.
- Rule versions make it safe to keep learning while BHMIS changes rules
  or an emergency is declared.

## Alternatives considered
- A separate memory per agent: rejected, duplicates data; one memory with
  per-request filters does the same job.
- Deleting outdated memory: rejected, old data must still be judged by
  its own rules, and history is needed for audit.

## Consequences
- guardmind/memory/ implements the stores, filters and ranking.
- Every case and skill must carry a rule version and case type.
