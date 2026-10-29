# KnowLine-2B-Gen1

This adds a complete self-scored Decision Index 0.3 public run for [KnowLine-2B-Gen1](https://huggingface.co/PelaAI/KnowLine-2B-Gen1/tree/9f3cde885f8ecf94254656991e1a726a857d38a5), our first 2B release (released together with KnowLine-0.8B-Gen1), served with read twice as in our earlier KnowLine submissions.

- **Model:** Qwen3.5-2B, language-model LoRA rank 32 / alpha 64, one epoch, merged into bf16 weights; the released checkpoint is the end of training. Readout temperature T=1.32 is folded into the final language-model norm. Front-end temperature is 1; the released `temperatures.json` divides score-question logits by 0.7576 (= 1/1.32), so score questions are read at an effective temperature of 1.0. Choice and yes/no questions, which the index scores, keep the folded temperature; the index uses argmax only.
- **Readout:** one prefill per question, option-label probabilities at the answer position, no generated reasoning. Short text-only requests are read twice when their one-pass request length is at most 3,000 tokens; longer requests are read once. The published model tokenizer and chat template are used with thinking off.

| Measurement | Result |
|---|---|
| Decision Index 0.3 public index | **52.23** (raw 64.24) |
| Coverage | 140,620 requests, 140,178 scoreable; all scoreable requests answered, zero errors/unsupported/truncation |
| Full-run request time | median 152.8 ms, mean 376.4 ms; includes queueing from eight client shards per server |
| Sequential request time | median 30.0 ms, mean 60.6 ms, p80 74.7 ms, p95 214.9 ms over 300 public requests on one H20; not measured on RTX PRO 6000 |

- **Results:** [PelaAI/KnowLine-2B-Gen1-decision-index](https://huggingface.co/datasets/PelaAI/KnowLine-2B-Gen1-decision-index/tree/a75cebf1d1998ae25bf5612b22452dc9efac40c5/runs/KnowLine-2B-Gen1) @ `a75cebf1d1998ae25bf5612b22452dc9efac40c5`: `results.jsonl.gz`, `scores.json`, `index.json`, `benchmark-summary.json`, `environment.json`.
- **Model/code revision:** `9f3cde885f8ecf94254656991e1a726a857d38a5`.
- **Runner/scorer:** official kit `62d2f51de34a2de64906345b6bc3e98e27ff55c7`, edition 0.3, `http` engine; runner SHA256 `f4c9f36c4c557f537f1d8184db7bf00c5a84d9f4f0aa67f8c06198cead25818d`.
- **Hardware/runtime:** four NVIDIA H20 GPUs, one identical bf16 SGLang 0.5.21 server per GPU started with the released `serve_knowline.sh`, Transformers 5.12.1, Torch 2.13.0. Thirty-two round-robin shards total; front-end workers 16, mem fraction 0.85, fp32 LM head, `mamba-radix-cache-strategy=extra_buffer`.
- **Retries:** none. Every request was answered in the single fresh pass; nothing was retried or selected by correctness.
- **Consistency:** the serial 300-request pass matched 476/477 answers from this model's own full run. The only difference was a near-tie (top two options within 0.06); no non-near-tie disagreement.
- **Limits:** 64 questions per request, 255 options per question; no truncation or benchmark-specific routing.

## Training disclosure

The model used 1,143,656 rendered text-training rows, the same data as KnowLine-9B-Gen1: 873,134 earlier decision rows, 241,781 further rows of the same sources, 16,824 added injection-invariance rows and 11,917 paired-convention pilot rows. The mix includes public benchmark training splits plus programmatically/teacher-generated decision examples. The build's DI-format share is 32.1%, including 1.2% alternate-format copies. Standard and strict decontamination used the public DI suite and our evaluation sets. These checks do not establish absence of pretraining exposure in Qwen3.5. The vision tower was not trained, and no vision score is claimed in this submission. Full details are in the model card's Disclosures section and `TRAINING_SOURCES.json`.

## Reproduce

```sh
pip install "sglang==0.5.21" "transformers==5.12.1" requests
hf download PelaAI/KnowLine-2B-Gen1 --revision 9f3cde885f8ecf94254656991e1a726a857d38a5 --local-dir KnowLine-2B-Gen1
bash KnowLine-2B-Gen1/serve_knowline.sh ./KnowLine-2B-Gen1 0 8080
# In another shell, with the public suite already rebuilt:
git clone https://github.com/apolinario/decision-index
cd decision-index
git checkout 62d2f51de34a2de64906345b6bc3e98e27ff55c7
python -m decision_index pipeline --edition 0.3 --engine http \
  --option base_url=http://127.0.0.1:8080 --option model=m --out runs/KnowLine-2B-Gen1
```

The public index validates this submission; it is not the board's private Full score. No private-rank claim is made.
