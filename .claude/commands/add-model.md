Add a new model configuration to `models.json` so it can be used with the benchmark.

## Arguments

$ARGUMENTS

The user may provide:
- `MODEL_NAME PROVIDER` (e.g., `my-model vllm`) for a quick setup
- `MODEL_NAME PROVIDER MODEL_ID BASE_URL` for full specification
- No arguments for fully interactive guidance

## Instructions

1. Read the current `models.json` file to understand existing entries and avoid name collisions.

2. Parse the arguments. If a model name was provided, check it doesn't already exist in `models.json`. If it does, ask if the user wants to update the existing entry instead.

3. Gather the required configuration. If not all info was provided in arguments, ask the user for each missing field:

   **Required fields:**
   - `model_name`: The key used in `models.json` and CLI `--model` flag. Should be short and descriptive (e.g., `llama-4-scout`, `claude-opus-4.5`).
   - `provider`: One of `anthropic`, `openai`, `google`, `litellm`, `vllm`, `qiskit_assistant`
   - `model_id`: The actual model identifier for the API. For `litellm`, this must include a routing prefix indicating the cloud provider (e.g., `aws/claude-opus-4-5`, `Azure/gpt-5.2-2025-12-11`, `gcp/gemini-3-pro-preview`). For `vllm`, use the HuggingFace-style ID (e.g., `meta-llama/Llama-4-Scout-17B-16E-Instruct`). For direct providers (`anthropic`, `openai`, `google`), use the provider's model name (e.g., `claude-opus-4-20250514`, `gpt-4o`).

   **Conditionally required:**
   - `base_url`: Required for `vllm` provider. Must end with `/v1`. For `litellm`, optional (defaults to `http://localhost:4000/v1` from env var `LITELLM_BASE_URL`).
   - `headers`: Common for vLLM deployments behind auth proxies. Format: `{"HEADER_NAME": "ENV_VAR_NAME"}` where the env var value is resolved at runtime. For example, many models in the existing `models.json` use `{"RITS_API_KEY": "VLLM_API_KEY"}`.

   **Optional with defaults:**
   - `max_tokens`: Default 4096
   - `temperature`: Default 0.0

4. Construct the JSON entry following the exact schema used by existing entries:
   ```json
   {
     "model_name": {
       "provider": "vllm",
       "model_id": "org/model-name",
       "base_url": "https://...",
       "max_tokens": 4096,
       "temperature": 0.0,
       "headers": {
         "API_KEY": "VLLM_API_KEY"
       }
     }
   }
   ```

5. Show the proposed entry to the user for confirmation before writing.

6. Add the entry to `models.json` (maintain the existing JSON formatting with 2-space indentation).

7. Verify the addition by reading back `models.json` and confirming the new model is present.

8. Suggest the next step:
   ```
   Model '{model_name}' added to models.json.

   To run the benchmark:
     uv run qk-benchmark --model {model_name}

   To verify the model is reachable:
     uv run qk-benchmark --model {model_name} --categories BasicGates

   Make sure the required API key / env var is set in your .env file.
   ```

9. **Provider-specific guidance:**
   - `vllm`: Remind about `VLLM_API_KEY` env var and that `base_url` must end with `/v1`
   - `litellm`: Remind about `LITELLM_API_KEY` and `LITELLM_BASE_URL`
   - `anthropic`: Remind about `ANTHROPIC_API_KEY`
   - `openai`: Remind about `OPENAI_API_KEY`
   - `google`: Remind about `GOOGLE_API_KEY`
   - `qiskit_assistant`: Remind about `QISKIT_ASSISTANT_TOKEN`
