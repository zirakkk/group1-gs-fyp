# GuardMind-H: agents and code

## Principle
A job becomes an agent only if it needs judgement. If an exact rule can
always do it correctly, it is plain code: cheaper, faster, and it cannot
make things up.

## Overview
| Type | Members |
|---|---|
| Agents (daily) | Investigator, Skill writer, Reporter |
| Agent (testing only) | Red-team |
| Code | Data pull, missing-report list, statistical check, verifier, emergency-mode switch, skill tests, escalation router |

## Daily jobs: code or agent
| # | Job | Type |
|---|---|---|
| 1 | Pull yesterday's counts from DHIS2 | Code |
| 2 | List facilities that have not reported | Code |
| 3 | Late report or a real problem? | Agent (Investigator) |
| 4 | Compare counts with each facility's normal level | Code |
| 5 | Outbreak, typing error or ghost data? | Agent (Investigator) |
| 6 | Check every number in an alert against the source | Code (verifier) |
| 7 | Turn confirmed cases into a tested skill | Agent writes, code tests |
| 8 | Write the alert in English and Urdu | Agent wording, code fills numbers |
| 9 | Switch to emergency-mode rules when an emergency is declared (flood, earthquake, epidemic, conflict) | Code |

## Emergency mode
When an emergency is declared for an area, the system switches to that
emergency's rule version (for example, different alert thresholds and
expected reporting delays). Skills and memory tied to normal-mode rules are
not used for cases inside the emergency period, and normal rules return
when the emergency ends. The switch itself is code: declared emergency ->
activate rule version.

## Investigator
| | |
|---|---|
| Job | Decide the cause of a flagged case (late report, real problem, outbreak, typing error, ghost data) |
| Input | The flagged case, plus evidence from code: facility history, neighbours' counts, reporting times, active emergency mode |
| Reads memory | Confirmed past cases of the same case type and district, valid under the current rule version, above a confidence threshold |
| Writes memory | Nothing directly; its answer is stored only after the verifier and the officer confirm it |
| Output | Cause, reasons and a confidence score |
| Checked by | Verifier (code): every number re-checked against source data |
| If unsure | Escalates: small LLM -> large LLM -> officer |

## Skill writer
| | |
|---|---|
| Job | Turn a pattern of confirmed cases into a small, tested code skill |
| Runs when | 3 or more officer-confirmed cases share a pattern |
| Input | Those cases, plus cases where the pattern did not apply |
| Reads memory | Confirmed cases of that pattern, current rule version |
| Writes | A draft skill only, never straight into use |
| Checked by | Code tests (must reproduce every past officer decision), then a shadow period before activation |
| If a test fails | Revise once, otherwise discard |

## Reporter
| | |
|---|---|
| Job | Write the daily alert in English and Urdu |
| Input | Verified cases only |
| Rule | Writes wording with placeholders ({count}, {facility}); code fills in the real numbers, so the LLM never invents a number and real values never leave the server |
| Reads memory | Nothing |
| Checked by | Code: every placeholder filled, every number matches the source |

## Red-team (testing only)
| | |
|---|---|
| Job | Create tricky fake cases (copied data, fake outbreak, hidden late report, a sudden emergency) |
| Runs when | Evaluation only, never in daily use |
| Output | Attacks with known right answers, to measure how many are caught |

## Lessons from the G-Memory baseline built into this design
| What went wrong in G-Memory | Design answer |
|---|---|
| Memory used even when rated 2/10 | Investigator reads memory only above a confidence threshold |
| Example from the wrong game (cocktail for tyres) | Memory scoped by case type, district and rule version |
| Helper agent repeated the solver | The verifier is code and checks against source data |
| Vague text lessons | Lessons become tested code skills |
