Generate a ranked leaderboard of all benchmarked models by pass rate.

## Arguments

$ARGUMENTS

The user may optionally provide:
- `--category CATEGORY_NAME` to filter results to a specific task category (e.g., `BasicGates`, `GroversAlgorithm`)
- `--strategy STRATEGY_NAME` to filter to a specific prompting strategy (e.g., `zero-shot`, `few-shot-3`, `chain-of-thought`)
- If no arguments, show the overall leaderboard using each model's best configuration

## Instructions

1. Glob for `results/*.json` files. **Skip** any file whose name starts with `checkpoint_` or `ablation_summary_`. Also skip any result where `total_tasks < 350` (these are partial/category-only runs).

2. For each JSON file, read: `model_id`, `pass_rate`, `passed_tasks`, `total_tasks`, and `results` (the per-task array).

3. Parse the filename to extract strategy and system prompt. Most files follow the convention:
   `{ModelName}_{strategy}_{system_prompt}_{YYYYMMDD_HHMMSS}.json`
   For example: `DeepSeek-V3-2_zero-shot_default_20260115_105900.json`
   Some files may not have strategy/system_prompt in the name -- for those, label the config as "unknown".

4. If `--strategy` was provided, only keep results whose parsed strategy matches.

5. If `--category` was provided, recompute the pass rate from the per-task `results` array, counting only tasks whose `task_id` starts with that category prefix, and only counting those as passed where `evaluation.passed` is true.

6. Group results by `model_id` (the value inside the JSON, e.g., `deepseek-ai/DeepSeek-V3.2`). For each model, pick the configuration with the **highest pass rate** (this is the "best config").

7. Calculate 95% Wilson score confidence intervals using passed_tasks and total_tasks:
   ```
   p = passed_tasks / total_tasks
   z = 1.96
   denominator = 1 + z^2 / total_tasks
   center = (p + z^2 / (2 * total_tasks)) / denominator
   margin = (z / denominator) * sqrt(p * (1 - p) / total_tasks + z^2 / (4 * total_tasks^2))
   CI = [max(0, center - margin), min(1, center + margin)]
   ```

8. Sort by pass rate descending and display as a formatted table:

```
Leaderboard (best config per model, N tasks)
================================================
 Rank  Model                   Pass Rate    95% CI              Passed   Best Config
    1  model-name               XX.X%       [XX.X%, XX.X%]     NNN/350  strategy/system_prompt
    2  ...
```

Use short model names for display: if the `model_id` contains a `/`, show only the part after the last `/` (e.g., `deepseek-ai/DeepSeek-V3.2` becomes `DeepSeek-V3.2`). Some model_ids have no prefix (e.g., `model-1`, `mistral-small-3.2-24b-qiskit`) -- use them as-is. Show the strategy and system_prompt that produced the best result in the "Best Config" column.
