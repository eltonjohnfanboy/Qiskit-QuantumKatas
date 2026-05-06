Deep-dive comparison between 2 or 3 specific models: overall stats, per-category breakdown, and disagreement analysis.

## Arguments

$ARGUMENTS

The user provides two or three model names separated by spaces. Model names should match either:
- Keys in `models.json` (e.g., `claude-opus-4.5`, `DeepSeek-V3-2`)
- Or substrings of model_id values in result files (e.g., `claude-opus`, `gpt-5.2`)

## Instructions

1. Glob for `results/*.json` files. **Skip** filenames starting with `checkpoint_` or `ablation_summary_`, and skip any result with `total_tasks < 350`.

2. For each model name provided by the user, find the matching result file(s). Try matching in this order:
   - Exact match on the filename prefix (before the first `_zero-shot`, `_few-shot`, `_chain-of-thought`, or `_YYYYMMDD`)
   - Substring match in the filename
   - Substring match in the `model_id` field inside the JSON (note: `model_id` includes a provider prefix like `deepseek-ai/DeepSeek-V3.2`)
   Use the result with the **highest `pass_rate`** as the best config for that model.

3. If a model name doesn't match any result file, report the error and list available model names (extracted from all result filenames).

4. Display the **overall comparison**. Compute avg latency as the mean of `generation.latency_ms` across all tasks. Compute token totals by summing `generation.input_tokens` and `generation.output_tokens` across all tasks. Parse the strategy/system_prompt from the result filename:
   ```
   Head-to-Head Comparison
   ========================
   Model                Pass Rate    95% CI              Passed/Total   Strategy            Avg Latency
   claude-opus-4-5      78.3%        [73.6%, 82.4%]     274/350        zero-shot/default   1234ms
   DeepSeek-V3.2        62.0%        [56.8%, 66.9%]     217/350        zero-shot/default   5678ms

   Token Usage:
   Model                Input Tokens    Output Tokens    Total
   claude-opus-4-5      1,234,567       234,567          1,469,134
   DeepSeek-V3.2        1,345,678       345,678          1,691,356
   ```

5. Display the **per-category breakdown**. For each category, compute pass rates from the per-task `results` array:
   ```
   Category Breakdown:
   Category               Model A          Model B          Delta
   BasicGates             100% (16/16)     93.8% (15/16)    +6.2pp
   Superposition          85.7% (18/21)    71.4% (15/21)   +14.3pp
   GroversAlgorithm       87.5% (7/8)      50.0% (4/8)     +37.5pp
   ...
   ```
   Sort by the absolute delta (largest differences first) to highlight where the models diverge most.

6. Perform **disagreement analysis**. For each task, check if one model passed and the other failed:
   ```
   Disagreement Analysis:
     Model A solves but Model B fails: 72 tasks
     Model B solves but Model A fails: 15 tasks
     Both pass: 202 tasks
     Both fail: 61 tasks

   Top categories where they diverge (A passes, B fails):
     MarkingOracles       12 tasks
     GraphColoring         8 tasks
     BoundedKnapsack       7 tasks
     ...

   Top categories where they diverge (B passes, A fails):
     tutorials             3 tasks
     ...
   ```

7. Show **specific disagreement tasks** (up to 10 from each direction):
   ```
   Tasks only Model A solves (showing 10 of 72):
     MarkingOracles/1.3, MarkingOracles/1.5, GraphColoring/3, ...

   Tasks only Model B solves (showing 10 of 15):
     tutorials/2.5, BasicGates/1.9, ...
   ```

8. If exactly 2 models are compared, include a brief McNemar-style significance note:
   ```
   Statistical Note:
     Discordant pairs: A+B- = 72, A-B+ = 15
     McNemar chi-sq = (|72-15|-1)^2 / (72+15) = 36.0
     The difference is statistically significant (p < 0.001).
   ```

9. If 3 models are provided, show all pairwise comparisons in the tables but skip the McNemar test.
