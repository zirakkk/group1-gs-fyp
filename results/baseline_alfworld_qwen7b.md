# ALFWorld baseline: qwen2.5-7b, AutoGen, empty vs G-Memory

All 134 ALFWorld tasks, run once with each memory setting. `g-memory + projector` is the same as `g-memory` but with `--use_projector`.

| Metric | empty (no memory) | g-memory | g-memory + projector |
|---|---|---|---|
| **Overall success** | **48/134 (35.8%)** | **37/134 (27.6%)** | **51/134 (38.1%)** |
| put | 9/24 (37.5%) | 9/24 (37.5%) | 10/24 (41.7%) |
| clean | 7/31 (22.6%) | 9/31 (29.0%) | 12/31 (38.7%) |
| heat | 13/23 (56.5%) | 10/23 (43.5%) | 11/23 (47.8%) |
| cool | 12/21 (57.1%) | 7/21 (33.3%) | 13/21 (61.9%) |
| examine | 3/18 (16.7%) | 2/18 (11.1%) | 1/18 (5.6%) |
| puttwo | 4/17 (23.5%) | 0/17 (0.0%) | 4/17 (23.5%) |
| Prompt tokens | 7,078,487 | 12,995,340 | 11,972,356 |
| Completion tokens | 124,402 | 223,577 | 278,471 |
| Run time | 2838.16 s (47.3 min) | 5342.34 s (89.0 min) | 6123.54 s (102.1 min) |
| Time per task | 21.2 s | 39.9 s | 45.7 s |

G-Memory stored 36 insights by the end of the run, and 37 with the projector.

## Setup

- **LLM:** `qwen2.5-7b` served by Ollama through its OpenAI-compatible API (`OPENAI_API_BASE=http://localhost:11434/v1`). It is a local model built from `qwen2.5:7b` with `num_ctx 8192`, running fully on an RTX 2060 Super 8 GB.
- **Embeddings:** `sentence-transformers/all-MiniLM-L6-v2`
- **Command:** `python tasks/run.py --task alfworld --reasoning io --mas_type autogen --max_trials 30 --model qwen2.5-7b --mas_memory {empty,g-memory}`, plus `--use_projector` for the projector run
- **G-Memory parameters:** defaults (successful_topk 1, failed_topk 0, insights_topk 3, threshold 0.0, hop 1). The role projector is off unless `--use_projector` is passed.
- **Environment:** ALFWorld `AlfredTWEnv` with the oracle controller, run on WSL2 Ubuntu
- **Seed:** `random.seed(42)`. Each setting was run once, so the run-to-run spread is unknown.

## Notes

- The `price=` value in the run log uses hard-coded GPT-4o rates. It does not apply to a local model.
- All three runs used `TMPDIR` set to a folder on disk. ALFWorld's fast-downward wrapper copies a 31 MB `libdownward.so` into a new temp folder every time it loads a game, and those copies stay mapped. That filled the 7.8 GB `/tmp` tmpfs after about 125 tasks and crashed the first empty run.
- The first G-Memory run crashed at task 20, when `merge_insights` first runs: `FINCH() got an unexpected keyword argument 'metric'`. The cause was that `finch_clust` and `finchpy` both install the `finch` package, and `finch_clust` overwrote `finchpy`. The fix was to keep only `finchpy==0.0.1` (commit `a0a0b0c`).
- The reported empty and g-memory runs both finished cleanly: 134/134 tasks, exit code 0, no tracebacks or API errors. The projector run also reached 134/134 with the end-of-run summary and no tracebacks in its log; its exit code was not recorded.
- With this 7B model, G-Memory scored 8.2 points **below** the no-memory baseline. It also used about 1.8× the tokens and 1.9× the time.
- `--use_projector` is **off by default** in `tasks/run.py` (`store_true`), but the role projector is part of the method described in the paper. The g-memory column above therefore leaves out part of G-Memory; the projector column is the closer match to the paper's setup.
- With the projector, G-Memory scored 38.1%: 10.4 points above G-Memory without it, and 2.2 points (3 tasks) above the no-memory baseline. It is still far below the paper's 67.91%. With one run per setting, a 3-task gap is within plausible noise.
- The projector run used the as-released prompt, the same as the other two columns (not the faithful-prompt branch). "Nothing happens" observations fell from 807 (g-memory) to 667.
