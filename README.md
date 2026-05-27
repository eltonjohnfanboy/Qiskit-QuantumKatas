# Quantum Katas Benchmark

![Qiskit/Qiskit-QuantumKatas](resources/hf-banner-Qiskit-QuantumKatas.png)

Benchmark LLMs on quantum computing tasks using the Quantum Katas dataset translated to Qiskit.

## Dataset

The dataset contains **350 quantum computing tasks** across 26 categories, derived from Microsoft's [QuantumKatas](https://github.com/microsoft/QuantumKatas) and translated to Qiskit.

### Categories

| Category | Tasks | Description |
|----------|-------|-------------|
| BasicGates | 16 | Fundamental quantum gates |
| Superposition | 21 | Superposition state preparation |
| Measurements | 18 | Quantum measurements |
| DeutschJozsa | 15 | Deutsch-Jozsa algorithm |
| GroversAlgorithm | 8 | Grover's search algorithm |
| QFT | 16 | Quantum Fourier Transform |
| PhaseEstimation | 7 | Quantum Phase Estimation |
| ... | ... | And 19 more categories |

## Installation

```bash
# Using uv (recommended)
uv sync

# Or with pip
pip install -e .
```

### Dependencies

- Python 3.10+
- qiskit >= 2.0.0
- qiskit-aer >= 0.15.0
- anthropic >= 0.40.0 (for Claude models)
- openai >= 1.0.0 (for GPT/vLLM models)
- google-generativeai >= 0.8.0 (for Gemini models)

## Quick Start

```bash
# Run benchmark with a model
uv run qk-benchmark --model claude-sonnet-4.6

# List available models
uv run qk-benchmark --list-models
```

## Configuration

### Environment Variables

API keys are configured via environment variables. Create a `.env` file:

```bash
cp .env.example .env
# Edit .env with your credentials
```

| Variable | Provider |
|----------|----------|
| `ANTHROPIC_API_KEY` | Anthropic Claude |
| `OPENAI_API_KEY` | OpenAI GPT |
| `GOOGLE_API_KEY` | Google Gemini |
| `VLLM_API_KEY` | vLLM (optional, defaults to "dummy") |
| `LITELLM_API_KEY` | LiteLLM proxy |
| `QISKIT_ASSISTANT_TOKEN` | IBM Qiskit Code Assistant |

The `.env` file is automatically loaded when running the benchmark.

### JSON Configuration

Models can be configured in `models.json` at the project root. The CLI loads this file automatically.

```json
{
  "my-claude": {
    "provider": "anthropic",
    "model_id": "claude-sonnet-4-6",
    "max_tokens": 4096,
    "temperature": 0.0
  },
  "my-vllm": {
    "provider": "vllm",
    "model_id": "Qwen/Qwen2.5-Coder-32B-Instruct",
    "base_url": "http://localhost:8000/v1",
    "max_tokens": 4096,
    "temperature": 0.0
  }
}
```

Supported providers: `anthropic`, `openai`, `google`, `vllm`, `litellm`, `qiskit_assistant`

### Custom Headers

Some endpoints require custom HTTP headers for authentication. Header values can reference environment variables:

```json
{
  "my-model": {
    "provider": "vllm",
    "model_id": "ibm-granite/granite-4.0-h-small",
    "base_url": "https://inference.example.com/granite-4-h-small/v1",
    "max_tokens": 4096,
    "temperature": 0.0,
    "headers": {
      "API_KEY": "VLLM_API_KEY"
    }
  }
}
```

The `VLLM_API_KEY` environment variable is resolved at runtime and sent as the `API_KEY` header.

**vLLM Example:**

```json
{
  "granite-4.0-h-small": {
    "provider": "vllm",
    "model_id": "ibm-granite/granite-4.0-h-small",
    "base_url": "https://<vllm-url>/granite-4-h-small/v1",
    "max_tokens": 4096,
    "temperature": 0.0,
    "headers": {
      "API_KEY": "VLLM_API_KEY"
    }
  }
}
```

**Note:** Base URLs for OpenAI-compatible endpoints must end with `/v1`.

## Available Models

[models.json](models.json) ships with 18 preconfigured model keys. Run `uv run qk-benchmark --list-models` for the live list.

### Hosted (routed via LiteLLM)

| Family | Keys |
|--------|------|
| Anthropic Claude | `claude-opus-4.7`, `claude-sonnet-4.6`, `claude-haiku-4.5` |
| OpenAI GPT | `gpt-5.5`, `gpt-5.3-codex` |
| Google Gemini | `gemini-3.1-pro-preview` |

### Self-hosted (via vLLM)

| Family | Keys |
|--------|------|
| Meta Llama | `llama-4-maverick`, `llama-4-scout` |
| IBM Granite | `granite-4.1-8b`, `granite-4.1-30b` |
| Alibaba Qwen | `qwen3-next-80b-a3b-thinking`, `Qwen3.5-397B` |
| DeepSeek | `deepseek-v3.2` |
| Mistral | `mistral-large-3`, `mistral-small-3.2-24b` |
| OpenAI OSS | `gpt-oss-120b`, `gpt-oss-20b` |
| Moonshot | `Kimi-K2.5` |

To run any other model, add an entry to [models.json](models.json) or construct a `ModelConfig` via the Python API.

## Usage

### Command Line

```bash
# Basic usage
uv run qk-benchmark --model claude-sonnet-4.6

# Filter by category
uv run qk-benchmark --model claude-sonnet-4.6 --categories BasicGates Superposition

# Filter by task IDs
uv run qk-benchmark --model claude-sonnet-4.6 --task-ids "BasicGates/1.1" "BasicGates/1.2"

# Custom output directory
uv run qk-benchmark --model claude-sonnet-4.6 --output results/experiment1

# Use custom config file
uv run qk-benchmark --model my-model --config custom_models.json
```

### Debug & Quiet Modes

```bash
# Debug: show raw responses for failed tasks
uv run qk-benchmark --model claude-sonnet-4.6 --debug

# Quiet: suppress task-by-task output
uv run qk-benchmark --model claude-sonnet-4.6 --quiet

# Combine both
uv run qk-benchmark --model claude-sonnet-4.6 -q -d
```

### Parallel Execution

```bash
# Run all models in parallel
uv run qk-benchmark --all --parallel

# Limit workers (useful for rate limits)
uv run qk-benchmark --all --parallel 2

# Parallel with ablation study (each model runs all 7 configs in its own worker)
uv run qk-benchmark --all --parallel --ablation

# Parallel ablation with worker limit and multiple runs
uv run qk-benchmark --all --parallel 4 --ablation --num-runs 3
```

### Python API

```python
from benchmark import BenchmarkRunner, ModelConfig, ProviderType, get_model_config

# Using a built-in model
config = get_model_config("claude-sonnet-4.6")
runner = BenchmarkRunner(model_config=config)
results = runner.run(verbose=True)
results.save("results/claude-sonnet-4.6.json")

# Custom configuration
config = ModelConfig(
    provider=ProviderType.OPENAI,
    model_id="gpt-4o",
    temperature=0.0,
)
runner = BenchmarkRunner(model_config=config)
results = runner.run()

# vLLM with custom headers
config = ModelConfig(
    provider=ProviderType.VLLM,
    model_id="ibm-granite/granite-4.0-h-small",
    base_url="https://inference.example.com/granite-4-h-small/v1",
    headers={"API_KEY": "VLLM_API_KEY"},
)

# Load from JSON config
from benchmark import load_models_from_json
models = load_models_from_json("models.json")
runner = BenchmarkRunner(model_config=models["my-model"])
```

## Statistical Analysis

### Multiple Runs

For statistically rigorous evaluation, run each task multiple times:

```bash
# Majority voting (recommended)
uv run qk-benchmark --model claude-sonnet-4.6 --num-runs 3 --aggregate majority

# Any pass (for pass@k metrics)
uv run qk-benchmark --model claude-sonnet-4.6 --num-runs 5 --aggregate any

# All must pass (strict)
uv run qk-benchmark --model claude-sonnet-4.6 --num-runs 3 --aggregate all
```

| Aggregation | Behavior | Use Case |
|-------------|----------|----------|
| `majority` | Pass if >50% succeed | Robust estimates |
| `any` | Pass if any succeeds | pass@k metrics |
| `all` | Pass only if all succeed | Strict evaluation |

### Recommended Settings

| Use Case | `--num-runs` | `--aggregate` |
|----------|--------------|---------------|
| Quick exploration | 1 | - |
| Published results | 3-5 | `majority` |
| pass@k metrics | 10+ | `any` |

### Confidence Intervals

Results include 95% Wilson score confidence intervals:

```python
from benchmark import load_results, generate_report

results = load_results("results/claude-sonnet-4.6.json")
report = generate_report(results)

print(f"Pass rate: {report.pass_rate:.1%}")
print(f"95% CI: [{report.stats.ci_lower:.1%}, {report.stats.ci_upper:.1%}]")
```

## Prompting Strategies

### Strategy Options

```bash
# Zero-shot (default)
uv run qk-benchmark --model claude-sonnet-4.6 --prompt-strategy zero_shot

# Few-shot with examples
uv run qk-benchmark --model claude-sonnet-4.6 --prompt-strategy few_shot_3

# Chain-of-thought
uv run qk-benchmark --model claude-sonnet-4.6 --prompt-strategy chain_of_thought
```

### System Prompts

```bash
# Default: balanced instructions
uv run qk-benchmark --model claude-sonnet-4.6 --system-prompt default

# Minimal: brief instructions
uv run qk-benchmark --model claude-sonnet-4.6 --system-prompt minimal

# Detailed: comprehensive Qiskit guidance
uv run qk-benchmark --model claude-sonnet-4.6 --system-prompt detailed
```

### Ablation Studies

Run all prompting combinations automatically:

```bash
# Full ablation (7 configurations)
uv run qk-benchmark --model claude-sonnet-4.6 --ablation

# With multiple runs
uv run qk-benchmark --model claude-sonnet-4.6 --ablation --num-runs 3

# Specific strategies only
uv run qk-benchmark --model claude-sonnet-4.6 --ablation --ablation-strategies zero_shot few_shot_3

# Parallel ablation across all models
uv run qk-benchmark --all --parallel --ablation

# Parallel ablation with worker limit
uv run qk-benchmark --all --parallel 4 --ablation --num-runs 3
```

Configurations tested:
1. Zero-shot + default/minimal/detailed prompts
2. Few-shot (1, 3, 5 examples) + default prompt
3. Chain-of-thought + CoT prompt

When using `--parallel` with `--ablation`, each model runs its full ablation study in a separate worker process.

## Results & Reporting

Results are saved as JSON with pass/fail status, generated code, evaluation details, and metrics.

### Compare Models

```bash
# List available results
uv run qk-compare --list

# Generate comparison table
uv run qk-compare

# Save to file
uv run qk-compare --output results/comparison.md
```

### Generate Reports

```python
from benchmark import (
    load_results,
    load_all_results,
    generate_report,
    format_markdown_report,
    format_statistical_comparison,
)

# Single model report
results = load_results("results/claude-sonnet-4.6.json")
report = generate_report(results)
print(format_markdown_report(report))

# Compare all models
all_results = load_all_results("results")
reports = [generate_report(data) for _, data in all_results]
print(format_statistical_comparison(reports))
```

## Dataset Format

Each task in `dataset/qiskit_quantumkatas.jsonl`:

```json
{
  "task_id": "BasicGates/1.1",
  "prompt": "# Task: State flip\n# Input: A qubit in state |ψ⟩ = α|0⟩ + β|1⟩\n# Goal: Change the state to α|1⟩ + β|0⟩...",
  "canonical_solution": "def state_flip(qc, q):\n    qc.x(q)\n    return qc",
  "test": "def test_state_flip():\n    qc = QuantumCircuit(1)\n    ...",
  "entry_point": "state_flip"
}
```

## Citation

If you use this benchmark or dataset, please cite our paper ([arXiv:2605.27210](https://arxiv.org/abs/2605.27210)):

```bibtex
@article{cruzbenito2026qiskitquantumkatas,
  title={Qiskit QuantumKatas: Adapting Microsoft's Quantum Computing Exercises for LLM Evaluation},
  author={Cruz-Benito, Juan and Faro, Ismael},
  journal={arXiv preprint arXiv:2605.27210},
  year={2026},
  eprint={2605.27210},
  archivePrefix={arXiv},
  primaryClass={quant-ph},
  url={https://arxiv.org/abs/2605.27210}
}
```

## License

This work is licensed under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC-BY-NC-SA-4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/) - see [LICENSE](LICENSE) file.

Based on Microsoft's [QuantumKatas](https://github.com/microsoft/QuantumKatas).
