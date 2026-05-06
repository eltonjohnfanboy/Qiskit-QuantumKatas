Validate dataset integrity by running all canonical solutions against their tests to find broken tasks.

## Arguments

$ARGUMENTS

Optional:
- A category name (e.g., `BasicGates`) to validate only that category
- `--verbose` to show per-task pass/fail details
- `--fix` to also check if the fixes in `_fix_dataset.py` are still needed
- No arguments: validate all 350 tasks and show a summary

## Instructions

**Time estimate**: Validating all 350 tasks takes several minutes (each task has a 30s timeout). Use the category filter to validate a subset quickly.

1. Read all tasks from `dataset/qiskit_quantumkatas.jsonl`. Each line is a JSON object with: `task_id`, `prompt`, `canonical_solution`, `test`, `entry_point`.

2. If a category argument was provided, filter to only tasks whose `task_id` starts with that category prefix.

3. For each task, validate the canonical solution by writing and executing a Python script that:
   - Imports necessary modules (numpy, qiskit, qiskit_aer, etc.)
   - Executes the `canonical_solution` code
   - Executes the `test` code
   - Calls the test function `test_{entry_point}()`
   - Uses a timeout of 30 seconds per task
   - Captures whether it passed or the exception details

   **Important**: Run each task in a subprocess or with a timeout mechanism, since some tasks may infinite-loop. The actual evaluator (`benchmark/evaluator.py`) uses `multiprocessing.Process` with a 30-second timeout.

   The simplest approach is to write a single Python validation script and run it with `uv run python`. The core logic for each task:
   ```python
   # For each task:
   namespace = {}
   exec(canonical_solution, namespace)
   test_namespace = namespace.copy()
   exec(test_code, test_namespace)
   test_namespace[f"test_{entry_point}"]()
   ```

   Alternatively, you can use the existing evaluator directly:
   ```python
   from benchmark.evaluator import evaluate_solution
   result = evaluate_solution(canonical_solution, test_code, entry_point, timeout=30.0)
   ```

4. Collect results and display a per-category summary:
   ```
   Dataset Validation (N tasks)
   =============================
   Category               Tasks   Passing   Failing
   BasicGates             16      16        0
   Superposition          21      21        0
   GroversAlgorithm       8       7         1
   ...

   Summary: X/Y canonical solutions pass (Z%)
   ```

5. For any failures, show details:
   ```
   Failures:
     GroversAlgorithm/2.2
       Entry point: grover_oracle_2
       Error: ImportError: name 'numpy' is not defined
       Note: This task has a fix in _fix_dataset.py

     GraphColoring/9
       Entry point: graph_coloring_phase_oracle
       Error: CircuitError: duplicate qubit in ccx
       Note: This task has a fix in _fix_dataset.py
   ```

6. If `--fix` was specified, also read `_fix_dataset.py` and check which task_ids have entries in the `FIXES` dictionary. Report whether each fix is still needed (the unfixed canonical solution still fails) or is no longer necessary (the canonical solution now passes without the fix).

7. If `--verbose` was specified, show each task's result as it runs:
   ```
   [1/350] BasicGates/1.1 (state_flip)... PASS
   [2/350] BasicGates/1.2 (basis_change)... PASS
   ...
   [127/350] GroversAlgorithm/2.2 (grover_oracle_2)... FAIL: ImportError
   ```

8. At the end, if there are failures, suggest next steps:
   - Check if the failure is already tracked in `_fix_dataset.py`
   - If not, suggest adding a fix entry
   - If many failures in one category, flag potential systemic issue
