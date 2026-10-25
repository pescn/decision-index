# Model submissions

| Model | Edition | Decision Index | Complete results | Engine / commit | Hardware | Declared limits |
|---|---|---:|---|---|---|---|
| [KnowLine-9B-Gen1](https://huggingface.co/PelaAI/KnowLine-9B-Gen1) | 0.3 | 65.67 | [scores.json](https://huggingface.co/datasets/PelaAI/KnowLine-9B-Gen1-decision-index/blob/5de4790da7009622b7b0a8522a42717e688d312e/runs/KnowLine-9B-Gen1/scores.json) | `http` against `knowline_server.py` @ `a421fc438cebe28e95de6951bc8bc440119db969`; SGLang 0.5.21 bf16, temperature 1 with folded T=1.18, read twice <=3000 tokens; kit `62d2f51` | 2 x H20; 8 shards per server; serial latency separately measured on 1 H20 | 64 questions, 255 options; no truncation; 0 unsupported |
