# Contributions

Each entry: date, member, what was added, why, and where to find it.
AI assistance (e.g. Claude) is noted where it was used.

| Date | Member | What | Why | Where | AI help |
|---|---|---|---|---|---|
| 2026-10-16 | Gul Zareen | Set up the environment (WSL2, Ollama + Qwen2.5-7B, G-Memory, ALFWorld data) | Run the baseline locally | baseline/gmemory | Yes |
| 2026-10-23 | Gul Zareen, Muhammad Shees | Ran the ALFWorld baseline (134 tasks: no memory, G-Memory, G-Memory + projector) | Reproduce the paper's baseline | results/baseline_alfworld_qwen7b.md | Yes |
| 2026-10-27 | Gul Zareen | Found the prompt mismatch between released code and authors' logs | Explain the gap to the paper | results/reproduction_notes.md | Yes |
| 2026-11-17 | Muhammad Shees | PDDL runs; restored the authors' prompt template (no memory: 15.5% vs paper 16.17%) | Reproduce the paper on PDDL | baseline/gmemory (branch pddl-authors-prompt) | Yes |
| 2026-11-18 | Muhammad Shees | Set up the group repo: G-Memory linked as submodule, folders, this log | Keep team work clear and traceable | this branch | Yes |
