# Qwen3.8-Flash-Next: FlashMix v3, plus AgentX

FlashMix v3 is the benchmark behind our Qwen3.8-Flash-Next results against vLLM. It measures long-context serving up to a 262,144-token window, with 8 to 64 requests at once. Our FP8 checkpoint also ran SemiAnalysis's [AgentX](#agentx) benchmark, described at the end.

| File | Prompts | SHA-256 |
| --- | ---: | --- |
| `flashmix-v3-c64-358.json` | 358 | `f37d932285d255c7a8d6b7cd1adc460d68b45e2da34c65838859a4c143cf16a4` |

## What Is In It

The 358 prompts are 180 long-document tasks, 60 tool-use tasks, 59 short chats and 59 maths and coding tasks with reasoning on. The median prompt is about 8,100 tokens and the longest is 255K.

| Slot | Rows | What |
| --- | ---: | --- |
| `chat` | 59 | Short chat (Dolly, OpenAssistant) |
| `thinking` | 59 | Maths and code with reasoning on (MATH, GSM8K, MBPP, HumanEval) |
| `tool` | 60 | Function calling (Glaive, plus the four built-in rows in [builtin-tools.json](../builtin-tools.json)) |
| `long` | 45 | Questions over 8-28K-token documents |
| `doc_T3` | 45 | Questions over 64-159K-token documents |
| `doc_T4` | 45 | Questions over 160-255K-token documents |
| `doc_summary` | 45 | Summaries of 68-248K-token books |

Long documents come from LongBench, LongBench v2, InfiniteBench and QuALITY.

## How It Is Sent

- **Bursts.** `flashmix_batch` labels 12 bursts: C8, C16, C32 and C64 in each of three reps, `r{rep}-c{C}`. Send each batch at once and wait for all of it.
- **Sustained.** Run all 358 as a closed loop, keeping C requests in flight, at C16 and C64.
- **No output caps.** Rows have no output budget: answers end when the model stops or reach the window.

The set was built with 360 prompts. Two answers ran away to the context window, `short:dolly:04097` and `thinking:mbpp:218`, so they were dropped for both engines. That is why burst `r1-c16` has 15 prompts and `r2-c32` has 31.

## AgentX

Our Qwen3.8-Flash-Next-FP8 configuration was also measured on [AgentX](https://inferencex.semianalysis.com/agentx), SemiAnalysis's agentic coding benchmark, against SemiAnalysis's published SGLang numbers. AgentX replays recorded coding-agent sessions, with their request lengths, prefix reuse, sub-agent branches and timing. Its trace is SemiAnalysis's, so it is not copied here.

| Setting | Value |
| --- | --- |
| Harness | [agentx-harness](https://github.com/SemiAnalysisAI/agentx-harness) 1.0.7, scenario `agentx` |
| Trace | `semianalysis_cc_traces_weka_062126_256k` |
| Points | Concurrency 1, 4, 8, 12 and 16; 3,600 seconds a point, default warmup, a fresh server a point |
