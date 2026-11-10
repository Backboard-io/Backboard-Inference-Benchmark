# File Format

Every benchmark file is a JSON object with a `schema_version` and a `scenarios` list; the GLM-5.3 file also has a `description` of its run order and a `replaced` list. Each scenario is one row: a prompt plus everything needed to send it and score the answer.

## Fields Every Row Has

| Field | Meaning |
| --- | --- |
| `id` | A unique id. It starts with the kind or tier and names the source, such as `thinking:gsm8k:0997` or `T4:ib-longbook_qa_eng:272`. |
| `kind` | `short`, `long`, `thinking` or `tool`. |
| `prompt` | The user message, as text. Documents are inside the prompt. |
| `enable_thinking` | Whether to run the model with reasoning on. Send it as the chat template's `enable_thinking`. |
| `language` | The prompt's language code, such as `en`, `zh` or `pt-BR`. |
| `domain`, `task_type` | What the task is about and what it asks for, such as `math` and `competition-math:Level 2`. |
| `source` | Where the row came from: `dataset`, `split`, `row_id` and `license`. |
| `grader` | How the answer is scored (see below), or `null` for open-ended rows that aren't scored. |
| `approx_input_tokens` | The prompt's length in tokens, measured with the tokenizer of the model the set was built for. Other tokenizers give somewhat different counts. |

## Fields Some Rows Have

| Field | Where | Meaning |
| --- | --- | --- |
| `flashmix_slot` | Qwen3.6, Flash-Next | The finer kind used to balance batches (the README's Terms table lists them). |
| `flashmix_batch` | Qwen3.6 bursts, Flash-Next | The burst a row belongs to, `r{rep}-c{C}`. Every row of `r0-c16` is sent at the same moment. |
| `output_budget` | Qwen3.6 | The `max_tokens` to send. Rows without it run until the model stops or the context window fills. |
| `tier`, `document_tokens`, `shape` | Flash-Next long documents | `T3` (64-159K) or `T4` (160-255K), the document's length, and whether the task is `prefill-heavy` (long input, short answer) or `decode-heavy` (a long answer). |
| `source_enable_thinking` | GLM-5.3 | The row's original `enable_thinking`, where the set forces it on. |
| `tools`, `tool_results`, `expected_calls`, `dependent_calls`, `tool_family` | Tool rows | See [Tool Rows](#tool-rows). |
| `expected_answer` | Built-in tool rows | The exact JSON the final answer must be. |
| `source_modification` | A few tool rows | How the row differs from its source, such as a recorded tool result replaced with an API error. |

## Graders

A grader reads the final visible answer, after removing any reasoning (`<think>...</think>`) and tool-call markup.

| `grader.type` | Passes when |
| --- | --- |
| `choice` | The answer picks the letter in `answer` (A-D), for example "The answer is C". |
| `boxed_answer` | The last `\boxed{...}` in the answer equals `answer` after light LaTeX normalization (spaces, `\left`, `\dfrac`, `$` and degree signs ignored). |
| `exact_number` | The last number in the answer equals `answer`. |
| `code_tests` | The last Python code block, run after `setup`, passes every assert in `tests`. Run untrusted model code only in a sandbox. |
| `reference_f1` | Word-overlap F1 against the best of `references` is at least 0.5 (characters for Chinese). |
| `rouge_l` | ROUGE-L against the best of `references` is at least 0.2. This is a rough check for summaries. |
| `tool_calls` | The model's tool calls, as `[name, arguments]` pairs, are the calls in `expected_calls`, in any order. Arguments are compared after normalizing: string case and spacing are ignored, and numeric strings equal numbers. If the grader has an `expected_json`, the final answer parsed as JSON (a code fence around it is allowed) must also equal it. |
| `null` | Not scored: open-ended chat and some long-form tasks. |

F1, ROUGE-L and answer overlap are proxies, and public sources may be in a model's training data, so use the scores to check that an engine didn't break the model's answers, not to rank models.

## Tool Rows

A tool row has `tools` (OpenAI function definitions), `tool_results` and `expected_calls`.

1. Send the prompt with `tools` and `tool_choice: "auto"`.
2. When the model calls a tool, answer each call from `tool_results`: find the record with the same `name` and `arguments` (exactly, or failing that after the same normalizing the grader uses) and send its `result` back as a `tool` message. A call with no matching record gets `{"error": "not_found", ...}`, and a call to an unknown tool gets `{"error": "unknown_tool", ...}`.
3. Repeat until the model answers without calling a tool, for at most four rounds.

`dependent_calls: true` means a later call needs an earlier call's result. The four rows whose ids start with `tool:builtin:` were written by us and have no `tools` field. They use the three tools and the fixture in [builtin-tools.json](builtin-tools.json), and their grader's `expected_json` (the same value as `expected_answer`) is the exact JSON the final answer must be. In `tool:builtin:dependent`, each call needs the previous call's result, so the model makes one call a round.
