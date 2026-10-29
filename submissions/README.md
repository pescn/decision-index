# Model submissions

| Model | Edition | Decision Index | Complete results | Engine / commit | Hardware | Declared limits |
|---|---|---:|---|---|---|---|
| [KnowLine-2B-Gen1](https://huggingface.co/PelaAI/KnowLine-2B-Gen1) | 0.3 | 52.23 | [scores.json](https://huggingface.co/datasets/PelaAI/KnowLine-2B-Gen1-decision-index/blob/a75cebf1d1998ae25bf5612b22452dc9efac40c5/runs/KnowLine-2B-Gen1/scores.json) | `http` against `knowline_server.py` @ `9f3cde885f8ecf94254656991e1a726a857d38a5`; SGLang 0.5.21 bf16, temperature 1 with folded T=1.32, score questions / 0.7576, read twice <=3000 tokens; kit `62d2f51` | 4 x H20; 8 shards per server; serial latency separately measured on 1 H20 | 64 questions, 255 options; no truncation; 0 unsupported |
