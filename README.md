# GuardMind-H (Group 1, FYP)

Final Year Project, Department of Computer Sciences, NUST Balochistan Campus.
Team: Gul Zareen, Muhammad Shees
Supervisor: Dr Zirak Khan · Co-supervisor: Mr Daniyal

GuardMind-H builds on G-Memory (NeurIPS 2025), a memory system for teams of LLM agents.
We first reproduce G-Memory on our own hardware, then extend it for health-data
surveillance (DHIS2, Balochistan).

## Repository layout

| Folder | Contents |
|---|---|
| baseline/gmemory | G-Memory code (linked from our fork, with our fixes) |
| results | Result tables and reproduction notes |
| experiments | Exact commands used for each run |
| docs | Design notes and decisions |

## Getting the code

    git clone --recurse-submodules https://github.com/zirakkk/group1-gs-fyp.git

`--recurse-submodules` also downloads the G-Memory code into baseline/gmemory.

## How we work

- One branch per task, named member/task (e.g. shees/pddl-results)
- Commit messages say what changed and why
- Changes reach main only through a pull request
- Every pull request adds a line to CONTRIBUTIONS.md

## Credits
G-Memory: Zhang et al., "G-Memory: Tracing Hierarchical Memory for Multi-Agent Systems",
NeurIPS 2025. Original code: https://github.com/bingreeky/GMemory


# group1-gs-fyp

