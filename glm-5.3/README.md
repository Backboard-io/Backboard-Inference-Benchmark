# GLM-5.3: eval10, Thinking On

eval10 is the benchmark behind our GLM-5.3 results against vLLM. It is a mixed workload of short chat, questions over 1-25K-token documents, maths and coding, and function calling. Every prompt fits a 32,768-token window, and it is sent 8, 16 and 32 requests at once.

| File | Prompts | SHA-256 |
| --- | ---: | --- |
| `eval10-336-thinking-on.json` | 336 | `3a46ee17722278d8a2182c15dcd381ce9b7b0c496d0d1f5e2caf6ca9bb8eacd4` |

## What Is In It

| Kind | Rows | Prompt tokens (min / median / max) | Sources |
| --- | ---: | --- | --- |
| `short` | 126 | 24 / 37 / 1,117 | OpenAssistant oasst2, Databricks Dolly 15k (15 languages) |
| `long` | 84 | 1,161 / 8,070.5 / 25,365 | LongBench, LongBench v2, QuALITY |
| `thinking` | 84 | 38 / 87.5 / 298 | MATH, GSM8K, MBPP, HumanEval |
| `tool` | 42 | 75 / 150 / 268 | Glaive function calling v2 |

The file's `replaced` list records 10 prompts that were swapped for others of the same kind before the set was frozen.

## How It Is Sent

- **In run order.** No labels are needed. A run is three bursts: C8 sends the next 8 prompts at once, C16 the next 16 and C32 the next 32, each burst waiting for all its answers. Run r uses prompts 56r to 56r+55, so six runs use all 336 prompts, each once.
- **The same mix in every burst.** Every group of 8 holds 3 short, 2 long, 2 thinking and 1 tool prompt.
- **Thinking on for every row.** GLM-5.3 ran with reasoning on for every request, so every row has `enable_thinking` set to true and every engine gets identical requests. Each row's `source_enable_thinking` keeps the original flag, which was off in 231 rows.
- **No output caps.** Rows have no output budget: answers end when the model stops or reach the window.
