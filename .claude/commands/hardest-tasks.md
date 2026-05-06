Identify the hardest tasks in the benchmark -- the ones that most models fail -- revealing where LLMs struggle with quantum computing.

## Arguments

$ARGUMENTS

Optional flags:
- `--top N` to control how many tasks to show (default: 20)
- `--category CATEGORY_NAME` to focus on a single category
- `--easiest` to flip and show the easiest tasks instead
- No arguments: show top 20 hardest tasks across all categories

## Instructions

1. Glob for `results/*.json` files. **Skip** filenames starting with `checkpoint_` or `ablation_summary_`, and skip any result with `total_tasks < 350`.

2. Group results by `model_id` (the value inside the JSON, e.g., `deepseek-ai/DeepSeek-V3.2`). For each model, pick the configuration with the **highest overall pass_rate** (best config).

3. Build a task-level pass matrix: for each of the 350 task_ids, count how many models passed and how many failed. A model passes a task if `evaluation.passed` is true in its best-config result file.

4. If `--category` was provided, filter to only tasks whose `task_id` starts with that category prefix.

5. For each task, also collect the most common `evaluation.error_type` among the failures (the mode).

6. Sort tasks by ascending model pass rate (hardest tasks first). If `--easiest` is specified, sort descending instead.

7. Display the top N results. Use short model names in the "Solved By" column (strip everything before the last `/` in `model_id` if present):

```
Hardest Tasks (N models evaluated)
====================================
 Rank  Task ID                        Models Passed    Common Error         Solved By
    1  SolveSATWithGrover/2.3         1/20 (5%)        Timeout              claude-opus-4-5
    2  BoundedKnapsack/1.4            2/20 (10%)       AssertionError       claude-opus-4-5, gpt-5.2
    3  MarkingOracles/1.7             2/20 (10%)       MissingEntryPoint    ...
   ...
```

8. After the main table, also show a **category breakdown** of the hardest tasks:

```
Category Distribution of Hardest Tasks:
  SolveSATWithGrover    5 tasks in top 20
  BoundedKnapsack       4 tasks in top 20
  GraphColoring         3 tasks in top 20
  ...
```

9. At the bottom, show 5 tasks from the opposite end (easiest if showing hardest, and vice versa) for context:

```
For contrast -- Easiest 5 tasks (all models pass):
  BasicGates/1.1       state_flip           20/20
  BasicGates/1.2       basis_change         20/20
  ...
```

10. If a specific task_id has 0 models passing it, flag it prominently -- this may indicate a dataset issue rather than genuine difficulty.
