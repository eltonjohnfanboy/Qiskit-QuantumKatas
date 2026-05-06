Show everything about a specific benchmark task -- the prompt, canonical solution, test code, and how every model performed on it.

## Arguments

$ARGUMENTS

The user should provide one of:
- A full task_id like `BasicGates/1.1` for a single task deep-dive
- A category prefix like `GroversAlgorithm` to see all tasks in that category
- A keyword to search for in task prompts (e.g., `teleportation`, `Grover`)

## Instructions

### Single Task (exact task_id match)

1. Read `dataset/qiskit_quantumkatas.jsonl` and find the task with matching `task_id`.

2. Display the task information:
   ```
   Task: {task_id}
   Category: {category (part before /)}
   Entry Point: {entry_point}

   Prompt:
   {prompt}

   Canonical Solution:
   {canonical_solution}

   Test Code:
   {test}
   ```

3. Glob for `results/*.json` files (skip filenames starting with `checkpoint_` or `ablation_summary_`, and skip results with `total_tasks < 350`). For each unique `model_id`, use only its best configuration (highest overall `pass_rate`).

4. In each result file, find the entry in the `results` array where `task_id` matches. Record whether `evaluation.passed` is true, and if not, the `evaluation.error_type` and a short version of `evaluation.error_message`.

5. Display a per-model results table. Parse the strategy/system_prompt from the filename of each model's best-config result file. Use short model names (strip everything before the last `/` in `model_id`):
   ```
   Model Results (best config per model):
     Model                    Passed?   Error Type           Strategy
     claude-opus-4-5          PASS      -                    zero-shot/default
     DeepSeek-V3.2            FAIL      AssertionError       few-shot-3/default
     ...

   Overall: X/Y models passed (Z%)
   ```

### Category listing (prefix match or no exact task_id match)

1. Read the dataset and find all tasks whose `task_id` starts with the given prefix.

2. For each task, load results from all models' best configs and compute: how many models passed this task out of total models.

3. Display a summary table:
   ```
   Category: {category_name} ({N} tasks)

   Task ID                    Entry Point              Models Passed
   BasicGates/1.1             state_flip               20/20 (100%)
   BasicGates/1.2             basis_change             20/20 (100%)
   BasicGates/1.10            bell_state_change_3      18/20 (90%)
   ...
   ```

4. Sort by model pass rate descending (easiest tasks first).

### Keyword search

If the argument doesn't match any task_id or category prefix, search through all task prompts for the keyword (case-insensitive). List matching tasks with their task_id and the first 100 characters of their prompt.
