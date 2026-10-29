# Model submissions

| Model | Edition | Decision Index | Complete results | Engine / commit | Hardware | Declared limits |
|---|---|---:|---|---|---|---|
| [KnowLine-0.8B-Gen1](https://huggingface.co/PelaAI/KnowLine-0.8B-Gen1) | 0.3 | 44.51 | [scores.json](https://huggingface.co/datasets/PelaAI/KnowLine-0.8B-Gen1-decision-index/blob/4f04c4378d4bda40f34d35a21f36b3566061fc9a/runs/KnowLine-0.8B-Gen1/scores.json) | `http` against `knowline_server.py` @ `b76c79d4c64d35219b7801a66a5f84cc718dce4e`; SGLang 0.5.21 bf16, temperature 1 with folded T=1.54, score questions / 0.6494, read twice <=3000 tokens; kit `62d2f51` | 4 x H20; 8 shards per server; serial latency separately measured on 1 H20 | 64 questions, 255 options; no truncation; 0 unsupported |
