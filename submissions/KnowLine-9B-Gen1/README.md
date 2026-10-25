# KnowLine-9B-Gen1

This adds a complete self-scored Decision Index 0.3 public run for [KnowLine-9B-Gen1](https://huggingface.co/PelaAI/KnowLine-9B-Gen1/tree/a421fc438cebe28e95de6951bc8bc440119db969), our first 9B release, served with read twice as in our earlier KnowLine submissions.

- **Model:** Qwen3.5-9B, language-model LoRA rank 32 / alpha 64, one epoch, merged into bf16 weights; WiSE 0.90 (90% fine-tuned checkpoint and 10% original base). Readout temperature T=1.18 is folded into the final language-model norm. Front-end temperature is 1; there is no per-type temperature file.
- **Readout:** one prefill per question, option-label probabilities at the answer position, no generated reasoning. Short text-only requests are read twice when their one-pass request length is at most 3,000 tokens; longer requests are read once. The published model tokenizer and chat template are used with thinking off.

| Measurement | Result |
|---|---|
| Decision Index 0.3 public index | **65.67** (raw 73.88) |
| Coverage | 140,620 requests, 140,178 scoreable; all scoreable requests answered, zero errors/unsupported/truncation |
| Full-run request time | median 642.0 ms, mean 1,606.7 ms; includes queueing from eight client shards per server |
| Sequential request time | median 96.7 ms, mean 207.7 ms, p80 261.9 ms, p95 826.7 ms over 300 public requests on one H20; not measured on RTX PRO 6000 |

- **Results:** [PelaAI/KnowLine-9B-Gen1-decision-index](https://huggingface.co/datasets/PelaAI/KnowLine-9B-Gen1-decision-index/tree/5de4790da7009622b7b0a8522a42717e688d312e/runs/KnowLine-9B-Gen1) @ `5de4790da7009622b7b0a8522a42717e688d312e`: `results.jsonl.gz`, `scores.json`, `index.json`, `benchmark-summary.json`, `environment.json`.
- **Model/code revision:** `a421fc438cebe28e95de6951bc8bc440119db969`.
- **Runner/scorer:** official kit `62d2f51de34a2de64906345b6bc3e98e27ff55c7`, edition 0.3, `http` engine; runner SHA256 `f4c9f36c4c557f537f1d8184db7bf00c5a84d9f4f0aa67f8c06198cead25818d`.
- **Hardware/runtime:** two NVIDIA H20 GPUs, one identical bf16 SGLang 0.5.21 server per GPU, Transformers 5.12.1, Torch 2.13.0. Sixteen round-robin shards total; front-end workers 16, mem fraction 0.85, fp32 LM head, `mamba-radix-cache-strategy=extra_buffer`.
- **Transport retries:** the first pass had two HTTP 504 failures (SATA-Bench and RouterBench). Only those two failed requests were retried through the official runner, with identical payload hashes and unchanged model/settings. The other 140,618 successful responses were not retried or selected by correctness. Retry provenance is in `environment.json`; the full merged result was rescored with the official scorer.
- **Consistency:** the serial 300-request pass matched 476/477 answers from this model's own full run. The only difference was a near-tie, margins 0.006 and 0.021; no non-near-tie disagreement.
- **Limits:** 64 questions per request, 255 options per question; no truncation or benchmark-specific routing.

## Training disclosure

The model used 1,143,656 rendered text-training rows: 873,134 or1d, 241,781 Kf examples not present in or1d, 16,824 added injection-invariance rows and 11,917 paired-convention pilot rows. The mix includes public benchmark training splits plus programmatically/teacher-generated decision examples. The build's DI-format share is 32.1%, including 1.2% alternate-format copies. Standard and strict decontamination used the public DI suite and the project's evaluation roots, with explicit held-out-probe checks. These checks do not establish absence of pretraining exposure in Qwen3.5. The vision tower was not trained, and no vision score is claimed in this submission. Full details are in the pinned model card's Disclosures section.

## Reproduce

```sh
pip install "sglang==0.5.21" "transformers==5.12.1" requests
hf download PelaAI/KnowLine-9B-Gen1 --revision a421fc438cebe28e95de6951bc8bc440119db969 --local-dir KnowLine-9B-Gen1
bash KnowLine-9B-Gen1/serve_knowline.sh ./KnowLine-9B-Gen1 0 8080
# In another shell, with the public suite already rebuilt:
git clone https://github.com/apolinario/decision-index
cd decision-index
git checkout 62d2f51de34a2de64906345b6bc3e98e27ff55c7
python -m decision_index pipeline --edition 0.3 --engine http \
  --option base_url=http://127.0.0.1:8080 --option model=m --out runs/KnowLine-9B-Gen1
```

The public index validates this submission; it is not the board's private Full score. No private-rank claim is made.
