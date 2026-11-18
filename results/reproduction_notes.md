# G-Memory reproduction notes: ALFWorld, qwen2.5-7b

Summary of the attempt to reproduce the paper's ALFWorld + AutoGen result on Qwen-2.5-7b, and why it falls short.

## Setup

| Item | Value |
|---|---|
| LLM | `qwen2.5-7b`, built from Ollama `qwen2.5:7b` (Qwen2.5 7B Instruct, **Q4_K_M**), `num_ctx 8192`, served through Ollama's OpenAI-compatible API |
| Hardware | RTX 2060 Super 8 GB, WSL2 Ubuntu |
| MAS / reasoning | AutoGen, `io`, `--max_trials 30` |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` |
| Seed | `random.seed(42)`, one run per setting |

The paper says the Qwen models were run locally with Ollama but does not give a quantization or context size.

## As-released results (134 tasks)

| | empty | g-memory | g-memory + projector |
|---|---|---|---|
| Ours | 48/134 (35.8%) | 37/134 (27.6%) | 51/134 (38.1%) |
| Paper (Table 3, Qwen-2.5-7b) | 52.99% | 67.91% | 67.91% (the paper reports one G-Memory figure) |
| Authors' log (`logs/alfworld/`, **qwen2.5-14b**) | — | 115/134 (85.8%) | |

There is no author log for 7b on ALFWorld; the only one is 14b. Details are in `baseline_alfworld_qwen7b.md`.

**Role projector.** `tasks/run.py` leaves the projector off unless `--use_projector` is passed, but it is part of the method in the paper. So the paper's 67.91% should be compared with the projector run, not with plain `g-memory`. The projector run uses the as-released prompt and otherwise the same settings as `g-memory` (successful_topk 1, failed_topk 0, insights_topk 3, threshold 0.0).

| Projector run | Value |
|---|---|
| Success | 51/134 (38.1%) |
| put / clean / heat / cool / examine / puttwo | 10/24, 12/31, 11/23, 13/21, 1/18, 4/17 |
| Prompt / completion tokens | 11,972,356 / 278,471 |
| Run time | 6123.54 s (102.1 min), 45.7 s per task |
| Insights stored | 37 |
| "Nothing happens" observations | 667 (vs 807 without the projector) |

Turning the projector on adds 10.4 points over plain `g-memory` and puts G-Memory 2.2 points (3 tasks) above the no-memory baseline. That is still about 30 points short of the paper's figure.

## Setup changes needed to run the released code

| # | Type | Problem | Fix |
|---|---|---|---|
| 1 | Bug | `requirements.txt` pins nonexistent `skimage==0.0` | Use `scikit-image` (`e16cd6c`) |
| 2 | Bug | `tasks/run.py` forces `HF_ENDPOINT` to hf-mirror | Commented out (`e16cd6c`) |
| 3 | Bug | `finch_clust` and `finchpy` both install `finch`; the first insight merge crashed with `FINCH() got an unexpected keyword argument 'metric'` | Keep only `finchpy==0.0.1` (`a0a0b0c`) |
| 4 | Bug | ALFWorld copies a 31 MB `libdownward.so` into a new temp dir per game, filling the 7.8 GB `/tmp` tmpfs after ~125 tasks | Run with `TMPDIR` on disk (no code change) |
| 5 | Tooling | No way to run a subset of tasks | Added `--num_tasks` (`e16cd6c`) |
| 6 | Repo hygiene | `.env`, `.db/` results and ALFWorld data not ignored | `.gitignore` entries (`e16cd6c`, `9dc0974`, `ff15db9`) |

## Causes checked

| Possible cause | Evidence | Verdict |
|---|---|---|
| Context truncation | Ollama logs: no truncation warnings, every slot `truncated = 0`. Longest prompt 4,833 tokens (g-memory), 3,569 (empty), vs `num_ctx` 8192 | Ruled out |
| Quantization | Q4_K_M is the default Ollama build; paper also used Ollama | Probably the same |
| `insights_topk` | Author logs use 5; `run.py` default is 3 | Minor, g-memory only |
| **Prompt differs from the authors'** | Authors' prompt includes a `think:` note that is defined but unused in the release (`temp` in `tasks/mas_workflow/format.py`), and lacks our `9. think: xxx` rule | See below |
| **Invalid actions** | Prose sent as an action: 11% (empty) and 13% (g-memory) of actions vs ~0% in the authors' log. "Nothing happens": 631 / 807 vs 39 | Main visible failure |

The missing note in the authors' prompt, placed just before the task:

```
NOTE: You must use the command `think: <your thoughts here>` if you want to think!!!
    - Right output: think: To solve the task, ...
    - Wrong output: To solve the task, ...
```

## Faithful-prompt test (branch `baseline-faithful`, commit `d231881`)

Changes: add the `think:` note, remove `9. think: xxx`, default `insights_topk` to 5. Prompt now matches the authors' log byte for byte. Run: empty memory, first 20 tasks.

| Metric (tasks 0–19) | As released | Faithful prompt |
|---|---|---|
| Success | 7/20 (35%) | **1/20 (5%)** |
| Total actions | 489 | 600 |
| Prose actions | 69 | 2 |
| "Nothing happens" | 127 | 5 |
| Repeated identical actions | 139 | **481** (all `think:`) |

Prose actions disappear, but the model now repeats one `think:` line until the 30-step limit, in 18 of 20 tasks. Both prompt changes were made together, so which one triggers the loops is not yet isolated.

## Why the `ground_truth` helper cannot break loops

`_solver_stuck()` (`tasks/mas_workflow/autogen/autogen.py:220`) is a plain string match on the last three actions. It does not exclude `think:`, and it fired on 434 steps. The helper was called each time (Ollama shows 1,056 requests for 600 steps), but it returned the same line because:

| Reason | Where |
|---|---|
| **Same prompt:** the helper's user prompt is built exactly like the solver's; only the system prompt differs | `autogen.py:134–139` vs `158–163` |
| **Temperature 0:** both agents decode greedily | `ReasoningConfig(temperature=0, stop_strs=['\n'])`, `autogen.py:23` |
| **No pointer to the stuck action:** the prompt is only `task_description + task_trajectory` | `memory_base.py:71` |
| **Self-reinforcing:** each repeat adds another copy to the trajectory | — |

In the as-released run the helper did diverge, but as prose ("Let's break out of the loop…"), which became most of the invalid actions. With the note it writes `think:` and copies the solver. The 14b model in the authors' log copes with the same prompt; the 7b model does not.

## Next steps

- Isolate the two prompt changes with two 20-task runs (note only; `9. think` removal only).
- Use `--use_projector` for all future G-Memory runs, since it is part of the paper's method.
- Repeat runs with different seeds to measure the run-to-run spread.
