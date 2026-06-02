# Experiment Logs & Reproducibility Artifacts

This folder houses the raw training trajectories, playbook checkpoints, and evaluation traces for the **ACE baseline** and **Reflective Context Learning (RCL)** across the AppWorld, BrowseComp+, and RewardBench2 benchmarks.

---

### ⚠️ Note on Codebase Version & Provenance

The raw logs, playbooks, and traces in this folder were captured during active development. 

To prepare the project for public release, the codebase was refactored to improve code quality, modularity, and readability (e.g., consolidating files and removing legacy debug hooks). The underlying optimization loop, primitives, and prompts remain identical to the implementation in this repository. Due to these refactorings, there may be some minor terminology, variable, or configuration parameter mismatches between the raw developmental logs/config files and the finalized codebase.

---

### 📁 Directory Structure Overview

The folder is structured systematically to allow easy navigation:

* **`appworld/`**: AppWorld Normal (`eval/normal/`) and Challenge (`eval/challenge/`) runs.
* **`browsecomp/`**: BrowseComp+ task execution traces and validation histories.
* **`rewardbench/`**: RewardBench2 evaluation trajectories and leaderboard subsets.

Under each benchmark, runs are separated by method:
* **`ace/`**: The ACE baseline runs.
* **`rcl/`**: The Reflective Context Learning (RCL) runs.

Each method directory contains:
* **`training/config.yaml`**: The exact runtime hyperparameters and model presets used.
* **`training/history.json`**: Iteration-by-iteration learning statistics.
* **`training/iterations/`**: Full snapshots of every single training step, containing intermediate playbooks (`playbook.json`), mutator recommendations (`mutations.json`), and rollouts (`traces/`).
* **`eval/`**: Score summaries (`results.json`) and the final task execution logs (`traces/`).
