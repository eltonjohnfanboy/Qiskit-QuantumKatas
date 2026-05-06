# Maintainers' Notes — `qiskit_quantumkatas.jsonl`

Carry-forward knowledge for future edits. Fix history lives in `git log`.

## Do not "fix" these — they are correct by construction

**Trivial returning stubs pass these tests, but that reflects the task, not a bug.** Any stub-probe audit will flag them; leave them.

| Task ID | Why the stub is correct |
|---------|------------------------|
| `DeutschJozsa/1.1` | Oracle for `f(x) = 0` is the identity |
| `tutorials/ExploringDeutschJozsaAlgorithm/2` | Constant `f(x) = 0` oracle is the identity |
| `QEC_BitFlipCode/1` | `|0⟩_L = |000⟩` is already the initial state |
| `UnitaryPatterns/1` | Task is explicitly the identity pattern |
| `CHSHGame/1.2`, `1.3` | `return False` is the optimal classical CHSH strategy |

## Intentional non-`pass` prompt stubs

These 5 prompts end with a typed return rather than `pass` to signal the return shape. Don't normalize them.

- `Teleportation/1.6` — `return False` (bool)
- `Teleportation/1.7` — `return True` (bool)
- `TruthTables/1` — `return (x1, x2, x3)` with placeholders
- `TruthTables/2`, `3` — `return f` with placeholder

## Known remaining weaknesses (accepted)

- **Probability-only tests** in a handful of arithmetic / `UnitaryPatterns/18` / `RippleCarryAdder` / `BoundedKnapsack` tasks where phase isn't semantically meaningful. Tightening these further yields no benchmark signal.
- **Hardcoded-only test values** (~94 tasks) and **single-size coverage** (~49 tasks) — flagged for possible future parameterization, not bugs.

## Harness contract

`entry_point` names are **not unique** across tasks (e.g. `bell_state`, `qft_1qubit`, `inverse_qft`, `win_condition`, `alice_classical`, `create_entangled_pair` all appear in 2+ tasks). Benchmark runners must key on `task_id`.

## Coverage gaps (roadmap, not defects)

Not covered: VQE / QAOA, quantum ML, noise models / error mitigation, Hamiltonian simulation, quantum chemistry, Shor's / period finding, surface codes / advanced QEC, density matrices / channels, adiabatic quantum computing.

## Validation

Run `/validate-dataset` (or call `benchmark.evaluator.evaluate_solution` directly per task). All 350 canonical solutions must pass their own tests. A "trivial-stub probe" (synthesize `return qc` / `return 0` / `return False` from the entry-point's type hint and run the test) should reject all tasks except the 6 listed above.
