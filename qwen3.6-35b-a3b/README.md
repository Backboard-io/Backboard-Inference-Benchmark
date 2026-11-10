# Qwen3.6-35B-A3B: C256 Ladder

This is the benchmark behind our Qwen3.6-35B-A3B results against vLLM. It measures serving from 8 to 256 requests at once, in bursts and under steady load. Every prompt fits a 32,768-token window.

| File | Prompts | Sent as | SHA-256 |
| --- | ---: | --- | --- |
| `bursts.json` | 1,512 | Bursts of C8, C16, C32, C64, C128 and C256, three reps | `14aa0516821b8da88af44e6e89e769842a6c5dadaf3216d3416f297876874dec` |
| `s-c8.json` | 64 | Sustained at C8 | `d24c8365952f9c5d830a470e879247f92960eb3fff8fa772b6f79fe23a14b671` |
| `s-c16.json` | 128 | Sustained at C16 | `c412cff5ff7cf807ef65804fec5080278995261a8333d0757a251e0696c14aa5` |
| `s-c32.json` | 256 | Sustained at C32 | `11c2ac3d8b224244fbc3c97ec65824a4b59bd6e7de2024e8c680642ad71f6b4e` |
| `s-c64.json` | 256 | Sustained at C64 | `0e8bf03113271a7dc33f53652081cfdad89b2932e75c22c904603d35d312fb24` |
| `s-c128.json` | 512 | Sustained at C128 | `8b7b0b366aaa103dae9332d61251626ce562ab62c9c916c20b58f039e0873883` |
| `s-c256.json` | 1,024 | Sustained at C256 | `b4b0185bf03f9f3c7ff6a8fc4e04e32af13bbec93141ab8371b7efa9e5ee7a07` |

`manifest.json` records the seed (20261102), the hash of the catalogue the prompts were drawn from, and the number of prompts of each kind in every batch.

## What Is In It

Every batch is built from groups of 8 holding 3 chat, 2 long-document, 2 thinking and 1 tool prompt, so each concurrency sees the same mix.

| Kind | Sources |
| --- | --- |
| Chat | Databricks Dolly 15k, OpenAssistant oasst2 (many languages) |
| Long documents (up to 28K tokens) | LongBench, LongBench v2, QuALITY |
| Thinking | MATH, GSM8K, MBPP, HumanEval |
| Tool | Glaive function calling v2, plus the four built-in rows in [builtin-tools.json](../builtin-tools.json) |

Within a kind, draws rotate over the source datasets, so no single dataset dominates a batch. A C256 burst carries 64 documents and 600-840K prompt tokens. No prompt appears twice across the seven files.

## How It Is Sent

- **Bursts.** Each row's `flashmix_batch` names its burst, `r{rep}-c{C}`. Send every row of a batch at once and wait for all of them before the next. The ladder runs C8 to C256 within each of the three reps.
- **Sustained.** `s-cC.json` is a closed loop: keep C requests in flight, starting the next row as each one finishes, until the file is used up.
- **Output budget.** Send each row's `output_budget` as `max_tokens`: 12,288 tokens for thinking and 4,096 for the rest. A reply that reaches its budget is a normal finish.
