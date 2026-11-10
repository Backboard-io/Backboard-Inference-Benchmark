# Backboard Inference Benchmark

These are the prompt sets behind the [Backboard](https://backboard.io) inference engine's benchmark results against vLLM and SGLang, so anyone can see exactly what our numbers were measured on and run the same prompts on their own setup.

The files hold **workloads, not results**. Each one is a fixed list of requests (chat questions, long documents, maths problems and tool calls) built from public datasets. Every row records the dataset, split, row and licence it came from, and most rows carry what is needed to score the answer.

## Benchmarks by Model

Each model has one benchmark: the one its best configuration was measured on.

| Model | Benchmark | What it measures | Prompts |
| --- | --- | --- | ---: |
| [Qwen3.6-35B-A3B](qwen3.6-35b-a3b/) | C256 ladder | Serving from 8 to 256 requests at once, in bursts and under steady load, within a 32K window | 3,752 |
| [Qwen3.8-Flash-Next](qwen3.8-flash-next/) | FlashMix v3 | Long-context serving up to a 262K window, 8 to 64 requests at once | 358 |
| [Qwen3.8-Flash-Next-FP8](qwen3.8-flash-next/#agentx) | AgentX, in addition | Replayed agentic coding sessions, from SemiAnalysis's public trace | [external](https://inferencex.semianalysis.com/agentx) |
| [GLM-5.3](glm-5.3/) | eval10, thinking on | A mixed chat, documents, maths and tools workload with reasoning on, 8, 16 and 32 requests at once, within a 32K window | 336 |

AgentX is SemiAnalysis's benchmark, so it is linked rather than copied here. Each model's folder has a README with what is in its set, how it is sent, its files and their hashes. None of the sets is tied to its model: any chat model with a large enough context window can run them.

## Quick Start

The files are plain JSON, about 130 MB in total, most of it long documents.

```sh
git clone https://github.com/Backboard-io/Backboard-Inference-Benchmark.git
cd Backboard-Inference-Benchmark
shasum -a 256 -c SHA256SUMS      # or: sha256sum -c SHA256SUMS
```

Each file is an object with a `scenarios` list. This turns a row into an OpenAI-compatible chat request:

```python
import json

with open("qwen3.6-35b-a3b/bursts.json") as f:
    scenarios = json.load(f)["scenarios"]

row = scenarios[0]
request = {
    "messages": [{"role": "user", "content": row["prompt"]}],
    # Reasoning on or off is part of the workload; vLLM and SGLang take it as a chat template argument.
    "chat_template_kwargs": {"enable_thinking": row["enable_thinking"]},
    "stream": True,
}
if "output_budget" in row:
    request["max_tokens"] = row["output_budget"]
if row.get("tools"):
    request["tools"] = row["tools"]
```

Tool rows take more than one request. [FORMAT.md](FORMAT.md) explains how to replay them, every field and how each answer is graded.

## Terms

| Term | Meaning |
| --- | --- |
| Scenario | One row: a prompt and everything needed to send and score it. |
| Kind | `short` (chat), `long` (a question or summary over a document), `thinking` (maths or code with reasoning on) or `tool` (function calling). |
| Slot | A finer kind used to balance batches: `chat`, `thinking`, `tool`, `long` (documents up to 28K tokens), `doc_T3` (64-159K), `doc_T4` (160-255K) and `doc_summary` (book summaries). |
| Concurrency (C) | How many requests are in flight at once. C8 means 8. |
| Burst | C requests sent at the same moment; the batch ends when the last one finishes. |
| Sustained | A closed loop: C requests in flight, and a new one starts as soon as one finishes. |
| Rep | One full pass of a set's burst ladder. Rows labelled `r1-c16` are rep 1's C16 burst. |
| Output budget | The `max_tokens` a row is sent with, the same on every engine. |
| Thinking | Whether the model reasons before answering (`enable_thinking`). |
| Runaway | An answer that keeps generating until it fills the context window. |

## How We Run a Comparison

- **Same everything except the engine.** Both engines get the same file, checkpoint, tokenizer, chat template and generation settings, on fresh or reset servers.
- **Use every row.** A measured pass uses all of a file's scenarios. Nothing is clipped, padded or dropped after looking at results.
- **One named exception.** An answer that runs away on either engine is dropped for both engines, and we name it in the model's README.
- **Check before running.** Every file is checked against its hash before a run.

## Licence

The collection is released under [CC BY-SA 4.0](LICENSE). Every row keeps its own source licence in its `source` field. Some sources are share-alike (HotpotQA under CC BY-SA 4.0, Databricks Dolly 15k under CC BY-SA 3.0), which is why the whole collection is share-alike.

[NOTICE.md](NOTICE.md) lists every source dataset with its licence, copyright notice and link, and what we changed. The MIT and Apache-2.0 licence texts those sources require are in [LICENSES/](LICENSES/). The sources are LongBench (with its HotpotQA, 2WikiMultihopQA, MuSiQue, Qasper, NarrativeQA, DuReader, GovReport, QMSum and MultiFieldQA subsets), LongBench v2, InfiniteBench, QuALITY, Databricks Dolly 15k, OpenAssistant oasst2, Glaive function calling v2, MATH, GSM8K, HumanEval and MBPP.

Public datasets may be in a model's training data, so treat quality scores as a sanity check rather than a capability measurement.
