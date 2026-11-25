# PDDL baseline: Qwen2.5-7B, AutoGen

All 60 PDDL tasks (data/pddl/test.jsonl), 30 steps per task, one run per setting.
Score = progress rate (share of goal conditions reached), as in the paper.

## Results

| Run | Prompt | Memory | Progress | Fully solved | Time | Prompt tokens |
|---|---|---|---|---|---|---|
| 1 | Released | None | 11.0% | 0/60 | 15.3 min | 5,325,103 |
| 2 | Released | G-Memory + projector | 17.5% | 1/60 | 36.0 min | 6,054,861 |
| 3 | Authors' | None | 15.5% | 2/60 | 13.7 min | 4,777,252 |
| 4 | Authors' | G-Memory + projector | 9.2% | 2/60 | 41.5 min | 6,936,517 |
| 5 | Authors' | G-Memory, no projector | 7.1% | 1/60 | 30.1 min | 6,114,207 |
| Paper | – | None / G-Memory | 16.17% / 21.01% | – | – | – |
| Authors' log | Authors' | G-Memory | 21.0% | 2/60 | 65.1 min | 5,232,141 |

G-Memory runs used insights_topk 5 (as in the authors' log).
"Authors' prompt" = template restored from the authors' PDDL log (fork branch pddl-authors-prompt).

## Setup fix

- PDDL crashed with "Resource punkt_tab not found". Fix: python -c "import nltk; nltk.download('punkt_tab')"

## What we found

- No memory with the authors' prompt (15.5%) is close to the paper (16.17%).
  Restoring the prompt wording from the authors' log raised it from 11.0%,
  so the released prompt is one reason for the gap.
- With the authors' prompt, G-Memory scored below no memory (9.2% / 7.1%).
  In the logs, a cocktail task was retrieved as the example for a tyre task
  even though the LLM rated it 2/10; nothing below a threshold is rejected.
- Insights stayed empty for tyre tasks: all 7 learned rules were linked to
  cocktail tasks, so rules learned in one game never reached the others.
- The helper agent repeated the solver's stuck line instead of breaking
  the loop, wasting up to 13 steps in one task.
- G-Memory runs took 2-3x longer and used more tokens than no memory.
  Its gain on PDDL is unstable (7.1% to 17.5% depending on prompt and projector).
