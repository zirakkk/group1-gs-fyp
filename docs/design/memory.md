# GuardMind-H: memory design

## Stores
| Store | Holds | Who can read it | G-Memory equivalent |
|---|---|---|---|
| Raw data store | DHIS2 counts and facility list, encrypted | Code only; never an LLM or the cloud | None |
| Case store | Every closed, officer-confirmed case | Investigator, Skill writer (filtered) | Query graph + interaction graph |
| Skill store | Tested code skills, their tests, source cases, rule version and state | Code runs skills; Skill writer drafts | Insight graph, but code instead of text |
| Rule registry | Rule versions: thresholds, case definitions, valid dates, emergency type | Code | None |

An audit log records every read, write and model call.
One shared memory, filtered per request; no separate memory per agent.

## One case item
| Group | Fields | Why |
|---|---|---|
| Identity | case ID, date stored | Traceable |
| What | case type, facility, district | Scoping by case type and place |
| When and rules | data date, rule version, emergency mode | Rule-version scoping |
| Evidence | aggregate numbers used, pointer to raw data | Investigator never sees private raw data; verifier can re-check |
| System's answer | cause, reasons, confidence, tier that answered | Cost and accuracy per tier |
| Truth | officer verdict, who confirmed, when | Only confirmed cases are trusted; used for calibration |
| Learning record | times retrieved, times it helped | Track record of usefulness |
| Access | sensitivity, allowed roles | Used by the gate |

Rules: a case is stored only after an officer verdict. Nothing is ever deleted.

## Reading memory
Skills first: an active skill for the same case type and current rule version
answers with no LLM call if its confidence is high enough.

Otherwise, stage 1, hard filters (in order):
1. Role (what this agent may see)
2. Confirmed cases only
3. Same case type
4. Rule version valid on the case's date
5. Same mode (emergency or normal)

Stage 2, ranking: relevance, place (facility > district > province),
recency, track record.
If even the best item is below the confidence threshold, nothing is shown
and the case moves up the ladder (larger model or officer).

## When a rule changes
1. Registry closes the old version and adds the new one with its start date.
2. Cases from the new date use only memory and skills valid under the new version.
3. Old cases are kept and still judged by their own rules.
4. Skills tied to the old version are paused for new dates.
5. After 3 new confirmed cases, paused skills are re-tested: pass -> also
   valid for the new version; fail -> rewritten.
6. Until then, new cases go up the ladder.

## When an emergency is declared
1. Registry switches that area to the emergency rule version from its start date.
2. Cases inside the emergency use only emergency memory and skills.
3. Early on, more cases go to officers (little emergency memory yet).
4. When it ends, normal rules resume automatically.
5. Emergency cases are kept, so the next emergency of the same type starts with experience.

## Skill states
Draft -> Shadow (answers beside the normal route, not trusted) -> Active ->
Paused (rules changed, awaiting re-test) -> Retired (failed too often;
archived, never deleted)

## Lessons from the G-Memory baseline
| What went wrong in G-Memory | Memory design answer |
|---|---|
| Used an example it rated 2/10 | Below-threshold memory is never shown; escalate instead |
| Cocktail example shown for a tyre task | Hard filter on case type |
| Rules learned in one game never reached others, yet wrong examples did | Scoping is explicit and tested, not left to similarity |
| No dates or versions on memory | Rule registry and rule-version filter |
| Every agent reads everything | Role filter (the gate) |
| Simple +1/-2 score on rules only | Track record on every item, plus officer verdicts for calibration |
