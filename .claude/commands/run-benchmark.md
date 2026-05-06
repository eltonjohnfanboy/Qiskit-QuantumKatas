Run the benchmark for a specific model, constructing the correct `qk-benchmark` command with all needed flags.

## Arguments

$ARGUMENTS

The user provides:
- **Model name** (required): A model key from `models.json`
- Optional flags (these map to the actual `qk-benchmark` CLI flags):
  - `--strategy` or `--prompt-strategy`: `zero_shot|few_shot_1|few_shot_3|few_shot_5|chain_of_thought` (CLI flag: `--prompt-strategy`)
  - `--system-prompt default|minimal|detailed|chain_of_thought`
  - `--categories CATEGORY1 CATEGORY2 ...` to run only specific categories
  - `--task-ids TASK1 TASK2 ...` to run specific tasks
  - `--ablation` to run all 7 prompting strategy combinations
  - `--parallel [N]` to run in parallel (useful with `--all`)
  - `--num-runs N` for multiple runs per task (for statistical analysis)
  - `--aggregate majority|any|all` for multi-run aggregation
  - `--debug` / `-d` to show raw responses for failures
  - `--quiet` / `-q` to suppress per-task output

## Instructions

1. Parse the model name from the first argument. Read `models.json` to verify the model exists as a key. Note: when `models.json` exists in the project root (which it does), the CLI loads ONLY those models -- built-in models from `benchmark/config.py` are NOT merged in. If the model name isn't found in `models.json`, show the available model keys from that file and stop.

2. Display the model configuration:
   ```
   Model: {model_name}
   Provider: {provider}
   Model ID: {model_id}
   Base URL: {base_url or "default"}
   ```

3. Construct the `uv run qk-benchmark` command. Start with:
   ```
   uv run qk-benchmark --model {model_name}
   ```
   Add any optional flags the user specified, mapping `--strategy` to the CLI's `--prompt-strategy` flag. If no strategy is specified, the default is `zero_shot` with `default` system prompt.

4. Show the constructed command to the user and ask for confirmation before running:
   ```
   Command to run:
     uv run qk-benchmark --model {model_name} --prompt-strategy zero_shot --system-prompt default

   This will evaluate {model_name} on 350 tasks (or N if categories filtered).
   Proceed? (The user will confirm or modify)
   ```

5. Execute the command. Monitor the output and relay progress updates to the user.

6. When complete, summarize:
   ```
   Benchmark Complete
   ===================
   Model: {model_name}
   Pass Rate: XX.X% (NNN/350)
   Failed: NN, Errors: NN
   Duration: Xm Ys
   Results saved to: results/{filename}.json
   ```

7. If the benchmark fails (e.g., API key missing, model unreachable), diagnose the error:
   - Missing API key: suggest checking `.env` file and the provider-specific env var (see `benchmark/config.py` for the mapping)
   - Connection error: suggest checking `base_url` in `models.json` and network connectivity
   - Rate limiting (429 errors): the benchmark already has built-in retry with exponential backoff (3 retries by default). If it's still failing, suggest running on a subset with `--categories` to reduce total requests, or waiting and retrying later
