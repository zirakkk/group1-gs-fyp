# Environment setup (G-Memory baseline)

Tested on: Windows 10, WSL2 Ubuntu, RTX 2060 Super 8 GB, 32 GB RAM.

## 1. WSL2 + Ubuntu
- PowerShell (as administrator): wsl --install -d Ubuntu --web-download
- Restart the PC when asked.
- Note: Ubuntu installed from an admin window is only visible to that account.
  Open it with: wsl -d Ubuntu (from an admin PowerShell).
- Work in the Linux home folder (cd ~), not in /mnt/c.

## 2. Miniconda
- bash Miniconda3-latest-Linux-x86_64.sh -b
- ~/miniconda3/bin/conda init bash, then reopen the terminal.
- Accept Anaconda's terms once:
  conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
  conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r

## 3. G-Memory code and packages
- git clone --recurse-submodules https://github.com/zirakkk/group1-gs-fyp.git
- conda create -n GMemory python=3.12 -y && conda activate GMemory
- pip install -r requirements.txt (inside baseline/gmemory)

## 4. Ollama + model
- sudo apt install -y zstd (the installer needs it)
- curl -fsSL https://ollama.com/install.sh | sh
- ollama pull qwen2.5:7b (rerun if the download drops; it resumes)
- Create a copy with a larger context window, named qwen2.5-7b:
  FROM qwen2.5:7b
  PARAMETER num_ctx 8192
  then: ollama create qwen2.5-7b -f Modelfile

## 5. Data and config
- ALFWorld game files go in data/alfworld (alfworld-download).
- .env: OPENAI_API_BASE=http://localhost:11434/v1 and OPENAI_API_KEY=ollama
- PDDL needs: python -c "import nltk; nltk.download('punkt_tab')"

## 6. Before every run
- export TMPDIR=~/.cache/gmemory_tmp
  (ALFWorld fills /tmp, which is a RAM disk in WSL, and crashed at task 125)

## Problems we fixed (already in our fork)
| Problem | Fix |
|---|---|
| requirements.txt lists skimage==0.0 | Use scikit-image |
| Code forces a Chinese download mirror | Disabled in tasks/run.py |
| finch_clust and finchpy clash (crash at task 20) | Keep only finchpy==0.0.1 |
| /tmp fills up (crash at task 125) | Set TMPDIR to disk |
| PDDL crash: punkt_tab missing | nltk.download('punkt_tab') |

## Slow internet tips
- Run one download at a time.
- Use wget -c or retry loops, which resume after a drop.
