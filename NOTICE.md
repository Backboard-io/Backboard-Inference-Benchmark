# Third-Party Notices

The benchmark files contain rows taken from the public datasets below. Each row's `source` field names its dataset, split, row and licence. The collection as a whole is released under [CC BY-SA 4.0](LICENSE). The rows keep the licences listed here, and nothing in this repository changes them.

**What we changed.** We selected rows from each dataset, wrote them out as chat prompts (adding answer-format instructions such as "Put your final answer in \boxed{}"), recorded each row's grader and token count, and grouped the rows into batches. In 31 Glaive scenarios, the last recorded tool result is replaced with an API error, and each one's `source_modification` field records it.

## Licences and Attribution

| Dataset | Licence | Copyright notice, where the licence file gives one | Source |
| --- | --- | --- | --- |
| LongBench | MIT ([text](LICENSES/MIT-LongBench.txt)) | Copyright (c) 2023 THU-KEG & Zhipu AI | https://github.com/THUDM/LongBench |
| LongBench subset: HotpotQA | CC BY-SA 4.0 | | https://hotpotqa.github.io/ |
| LongBench subset: 2WikiMultihopQA | Apache-2.0 ([text](LICENSES/Apache-2.0.txt)) | | https://github.com/Alab-NII/2wikimultihop |
| LongBench subset: MuSiQue | CC BY 4.0 | | https://github.com/StonyBrookNLP/musique |
| LongBench subset: Qasper | CC BY 4.0 | | https://huggingface.co/datasets/allenai/qasper |
| LongBench subset: NarrativeQA | Apache-2.0 ([text](LICENSES/Apache-2.0.txt)) | | https://github.com/google-deepmind/narrativeqa |
| LongBench subset: DuReader | Apache-2.0 ([text](LICENSES/Apache-2.0.txt)) | Copyright 2017 Baidu.com, Inc. | https://github.com/baidu/DuReader |
| LongBench subset: GovReport | CC BY 4.0 | | https://gov-report-data.github.io/ |
| LongBench subset: QMSum | MIT ([text](LICENSES/MIT-QMSum.txt)) | Copyright (c) 2021 Yale-LILY | https://github.com/Yale-LILY/QMSum |
| LongBench subset: MultiFieldQA (en, zh), built by LongBench | MIT ([text](LICENSES/MIT-LongBench.txt)) | Copyright (c) 2023 THU-KEG & Zhipu AI | https://github.com/THUDM/LongBench |
| LongBench v2 | Apache-2.0 ([text](LICENSES/Apache-2.0.txt)) | | https://huggingface.co/datasets/zai-org/LongBench-v2 |
| InfiniteBench | Apache-2.0 ([text](LICENSES/Apache-2.0.txt)) | | https://huggingface.co/datasets/xinrongzhang2022/InfiniteBench |
| QuALITY | CC BY 4.0 | | https://nyu-mll.github.io/quality/ |
| Databricks Dolly 15k | CC BY-SA 3.0 | | https://huggingface.co/datasets/databricks/databricks-dolly-15k |
| OpenAssistant oasst2 | Apache-2.0 ([text](LICENSES/Apache-2.0.txt)) | | https://huggingface.co/datasets/OpenAssistant/oasst2 |
| Glaive function calling v2 | Apache-2.0 ([text](LICENSES/Apache-2.0.txt)) | | https://huggingface.co/datasets/glaiveai/glaive-function-calling-v2 |
| MATH | MIT ([text](LICENSES/MIT-MATH.txt)) | Copyright (c) 2021 Dan Hendrycks | https://github.com/hendrycks/math, mirrored at https://huggingface.co/datasets/EleutherAI/hendrycks_math |
| GSM8K | MIT ([text](LICENSES/MIT-GSM8K.txt)) | Copyright (c) 2021 OpenAI | https://github.com/openai/grade-school-math, mirrored at https://huggingface.co/datasets/openai/gsm8k |
| HumanEval | MIT ([text](LICENSES/MIT-HumanEval.txt)) | Copyright (c) OpenAI (https://openai.com) | https://github.com/openai/human-eval |
| MBPP | CC BY 4.0 | | https://huggingface.co/datasets/google-research-datasets/mbpp |

The source links above credit each dataset's creators, as the CC BY and CC BY-SA licences allow. The CC licences are at https://creativecommons.org/licenses/by/4.0/, https://creativecommons.org/licenses/by-sa/4.0/ and https://creativecommons.org/licenses/by-sa/3.0/. CC BY-SA 3.0 permits adaptations under a later version of the same licence (section 4(b)), which is why the Dolly rows can be part of a CC BY-SA 4.0 collection.

## Written by Backboard

The four rows whose ids start with `tool:builtin:` and the fixture in [builtin-tools.json](builtin-tools.json) were written by Backboard. Their `source.license` reads `repository-authored`, and they are released under CC BY-SA 4.0 with the rest of the collection.
