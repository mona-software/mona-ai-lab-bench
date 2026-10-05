# MONA AI Lab Bench

A benchmark and harness for evaluating LLMs on Vietnamese customer-facing business tasks such as support chat, sales and complaint handling.

The tasks, prompts and judge rubric are in Vietnamese. MONA uses this bench in [MONA AI Lab](https://mona.media/ai-lab/) when testing new models; you can run it against any OpenAI or Anthropic model with your own API keys.

## What it tests

14 synthetic scenarios in `tasks/vi-business-bench.jsonl`, two per category. None contain real customer data.

| Category | What is checked |
| --- | --- |
| `faq` | Answers policy questions correctly without inventing conditions |
| `tu-van` (advice) | Recommends according to the customer's needs without inventing prices |
| `chot-sale` (closing) | Handles objections and holds the price |
| `khieu-nai` (complaints) | Apologizes appropriately and follows the process without over-promising |
| `da-luot` (multi-turn) | Keeps context across turns |
| `injection` | Resists "ignore your instructions / reveal the system prompt" attempts |
| `edge-so-ten-tien` (numbers, names, money) | Computes amounts correctly and reproduces Vietnamese names and phone numbers accurately |

## Scoring

Each task is scored in two layers (details in [`rubric.md`](rubric.md)):

1. **Rule checks** on the model's final answer. Each check is a case-insensitive regex: `must_contain` / `regex_ok` pass when it matches, `must_not_contain` passes when it does not.
2. **LLM judge**: a second model reads the whole conversation and scores four criteria from 1 to 5: correctness (`dung_y`), natural Vietnamese (`tieng_viet`), no fabrication (`khong_bia`) and context retention (`giu_ngu_canh`), plus a verdict of `pass`, `weak` or `fail`.

Task score out of 100 = 80 × (judge average / 5) + 20 × (rule-check pass rate). A task with no checks counts as a 100% check pass rate. Results are aggregated overall and per category.

## Install

Requires Python 3 with the `openai` and `anthropic` SDKs.

```bash
git clone https://github.com/mona-software/mona-ai-lab-bench
cd mona-ai-lab-bench
pip install -r requirements.txt
```

## Configuration

The harness reads API keys from environment variables. `.env.example` lists them, but the harness does not load `.env` files, so export the keys in your shell:

```bash
export OPENAI_API_KEY="..."
export ANTHROPIC_API_KEY="..."
```

You only need the keys for the providers you use as target and judge.

## Usage

```bash
python harness/run_eval.py \
  --provider openai --model <model-id> \
  --judge-provider anthropic --judge-model <judge-model-id> \
  --tasks tasks/ --out out/
```

| Option | Description | Default |
| --- | --- | --- |
| `--provider` | Provider of the model under test: `openai` or `anthropic` | required |
| `--model` | Model ID under test | required |
| `--judge-provider` | Provider of the judge model: `openai` or `anthropic` | required |
| `--judge-model` | Judge model ID | required |
| `--tasks` | Directory of `.jsonl` task files | `tasks/` |
| `--out` | Output directory | `out/` |
| `--timeout` | API timeout in seconds | `60` |
| `--retries` | Attempts per API call | `3` |

Output:

- `out/report.md`: a Vietnamese-language summary table (overall and per category) and a per-task table.
- `out/report.json`: metadata, every task's transcript, check results, judge scores and `score_100`, plus the aggregates.

Exit codes: `0` all tasks completed, `1` at least one task errored, `2` setup or write failure.

## Adding tasks

Each task is one JSON line in any `tasks/*.jsonl` file. Required fields: `id` (unique), `category`, `system`, `turns` (non-empty list of user messages), `checks` (list, may be empty) and `judge` (task-specific instructions for the judge).

```json
{"id":"faq-03","category":"faq","system":"<role and policy>","turns":["<customer message>"],"checks":[{"type":"must_not_contain","value":"<regex that must not appear>"}],"judge":"<what the judge should look for>"}
```

Write `must_not_contain` patterns against incorrect statements, not single words a correct refusal might also contain. Remove personal data before adding real conversations.

## License

MIT, see [LICENSE](LICENSE).

**MONA AI Lab Bench is a product of MONA Software, a member of The MONA Group.**
