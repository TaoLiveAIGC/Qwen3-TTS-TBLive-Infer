# Qwen3-TTS-TBLive-Infer

Inference companion for **Qwen3-TTS-TBLive** — an extension of
[Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS) for the Chinese live-streaming
domain, built on the upstream `Qwen3-TTS-12Hz-1.7B-Base` model, by the
[TaoLiveAIGC](https://github.com/TaoLiveAIGC) Team (Taobao & Tmall Group, Alibaba).

- 🔊 **Live Demo**: <https://taoliveaigc.github.io/Qwen3-TTS-TBLive-Infer/> — TTS A/B
  samples on live-streaming scripts, instruction-control demos, and digital-human videos
  (served from the `gh-pages` branch of this repository)
- 🤗 **Models**: [TaoLiveAIGC on HuggingFace](https://huggingface.co/TaoLiveAIGC)

Two checkpoints are released:

- `Qwen3-TTS-TBLive-Base` — a domain-adapted base model for Chinese live-streaming
  speech. Recommended starting point for downstream fine-tuning.
- `Qwen3-TTS-TBLive-CustomVoice` — a CustomVoice checkpoint with six built-in
  open-source speakers spanning three live-streaming styles, plus inline
  instruction-control tokens for pause and speaking-rate control.

---

## Contents

- [Demo](#demo)
- [Released Models](#released-models)
- [Instruction Control Tokens](#instruction-control-tokens)
- [Benchmark](#benchmark)
- [Quickstart](#quickstart)
- [Scope boundary](#scope-boundary)
- [License](#license)

---

## Demo

**▶ Online demo: <https://taoliveaigc.github.io/Qwen3-TTS-TBLive-Infer/>**

Base vs. `TBLive` A/B samples across three live-streaming styles (professional
presentation / warm recommendation / energetic promotion), plus instruction-control
and digital-human video demos.

---

## Released Models

The checkpoints are hosted on HuggingFace under the
[TaoLiveAIGC](https://huggingface.co/TaoLiveAIGC) organization.

| Model | Description | Size | Language | HuggingFace |
|---|---|---|---|---|
| `Qwen3-TTS-TBLive-Base` | Domain-adapted base checkpoint for Chinese live-streaming speech. Recommended starting point for downstream fine-tuning. | ~4.2 GB | Chinese (live-streaming) | [TaoLiveAIGC/Qwen3-TTS-TBLive-Base](https://huggingface.co/TaoLiveAIGC/Qwen3-TTS-TBLive-Base) |
| `Qwen3-TTS-TBLive-CustomVoice` | CustomVoice checkpoint built on `TBLive-Base`, with the six built-in speakers listed below. | ~4.2 GB | Chinese | [TaoLiveAIGC/Qwen3-TTS-TBLive-CustomVoice](https://huggingface.co/TaoLiveAIGC/Qwen3-TTS-TBLive-CustomVoice) |

Each release contains the main `model.safetensors` checkpoint, the 12 Hz speech
tokenizer under `speech_tokenizer/`, and the generation / tokenizer configs needed
by the inference runtime.

`Qwen3-TTS-TBLive-CustomVoice` ships six built-in speakers — three live-streaming
styles, each in a female and a male voice. Pass the speaker ID to
`generate_custom_voice(text, speaker=...)`:

| Speaker | Gender | Style | Speaker ID |
|---|---|---|---|
| 专业讲解女 | Female | Professional presentation (专业讲解) | 115 |
| 专业讲解男 | Male | Professional presentation (专业讲解) | 96 |
| 温柔女音 | Female | Warm recommendation (贴心推荐) | 26 |
| 温柔男音 | Male | Warm recommendation (贴心推荐) | 80 |
| 激情促销女 | Female | Energetic promotion (激情促销) | 4 |
| 激情促销男 | Male | Energetic promotion (激情促销) | 1 |

Download from HuggingFace:

```bash
# Base checkpoint (recommended for fine-tuning)
huggingface-cli download TaoLiveAIGC/Qwen3-TTS-TBLive-Base --local-dir ./Qwen3-TTS-TBLive-Base

# CustomVoice checkpoint (six built-in speakers)
huggingface-cli download TaoLiveAIGC/Qwen3-TTS-TBLive-CustomVoice --local-dir ./Qwen3-TTS-TBLive-CustomVoice
```

---

## Instruction Control Tokens

> Available only with `Qwen3-TTS-TBLive-CustomVoice` and its six built-in speakers.

Insert the following tokens directly into the input text to control pauses and
speaking rate; they are interpreted inline at the position where they appear.

**Silence / pause tokens** — each token synthesizes a pause within a fixed duration band:

| Token | Pause duration |
|---|---|
| `<sil_L2>` | 0.55 – 0.70 s |
| `<sil_L3>` | 0.80 – 0.95 s |
| `<sil_L4>` | 1.05 – 1.20 s |
| `<sil_L5>` | 1.30 – 1.45 s |
| `<sil_L6>` | 1.55 – 2.50 s |

**Speed tokens** — like silence tokens, they can be inserted anywhere in the text and
adjust the speaking rate of the speech that follows:

| Token | Effect |
|---|---|
| `[speed_e2]` | Faster speech |
| `[speed_d2]` | Slower speech |

Example:

```text
[speed_e2] 家人们，最后一波福利来了！<sil_L3> 库存不多，喜欢的抓紧下单！
```

---

## Benchmark

Zero-shot voice-cloning results on the public
[seed-tts-eval](https://github.com/BytedanceSpeech/seed-tts-eval) test sets, compared
against open-source baselines. `/` marks numbers not reported.

| Model | test-en SIM-o ↑ | test-en WER ↓ | test-en UTMOS ↑ | test-zh SIM-o ↑ | test-zh WER ↓ | test-zh UTMOS ↑ |
|---|---|---|---|---|---|---|
| Ground-truth | 0.734 | 2.14 | 3.52 | 0.755 | 1.25 | 2.78 |
| IndexTTS2 | 0.706 | 2.33 | 3.65 | 0.764 | 1.05 | 3.00 |
| CosyVoice3 | 0.696 | 2.17 | 3.96 | 0.778 | 1.14 | 3.32 |
| VoxCPM | 0.731 | 1.92 | 3.77 | 0.772 | 0.99 | 2.94 |
| MossTTS Local | 0.732 | 1.93 | / | 0.796 | 1.44 | / |
| Qwen3-TTS | 0.708 | 1.54 | 4.16 | 0.766 | 1.15 | 3.46 |
| **Qwen3-TTS-TBLive-Base** | **0.732** | **1.63** | **4.11** | **0.782** | **1.28** | **3.38** |

---

## Quickstart

### 1. Environment setup

```bash
conda create -n qwen3-tts python=3.12 -y
conda activate qwen3-tts

# Inference runtime (pulls transformers / torch / etc.)
pip install -U qwen-tts

# Optional: FlashAttention-2 for reduced memory footprint
pip install -U flash-attn --no-build-isolation
```

### 2. Zero-shot voice cloning (TBLive-Base)

```python
import torch
from qwen_tts import Qwen3TTSModel

model = Qwen3TTSModel.from_pretrained(
    "TaoLiveAIGC/Qwen3-TTS-TBLive-Base",
    device_map="cuda:0",
    dtype=torch.bfloat16,
)

# Zero-shot voice cloning with a short reference audio
wavs, sr = model.generate_voice_clone(
    text="家人们，今天直播间的福利真的拉满了！",
    language="Chinese",
    ref_audio="path/to/reference.wav",
    ref_text="参考音频对应的文本",
)
```

### 3. Built-in speakers (TBLive-CustomVoice)

```python
import torch
from qwen_tts import Qwen3TTSModel

model = Qwen3TTSModel.from_pretrained(
    "TaoLiveAIGC/Qwen3-TTS-TBLive-CustomVoice",
    device_map="cuda:0",
    dtype=torch.bfloat16,
)

wavs, sr = model.generate_custom_voice(
    text="[speed_e2] 家人们，最后一波福利来了！<sil_L3> 库存不多，喜欢的抓紧下单！",
    speaker="4",  # 激情促销女
)
```

Inference post-selection utilities (two-stage speaker-similarity → CER re-ranking
and automatic long-text segmentation) will be added to this repository soon.

---

## Scope boundary

For upstream features (Voice Clone / Voice Design / CustomVoice APIs, tokenizer
usage, vLLM serving, DashScope API, deployment recipes, etc.) please refer to the
[upstream Qwen3-TTS README](https://github.com/QwenLM/Qwen3-TTS#readme).

---

## License

The model checkpoints are released under the
[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0), consistent with
upstream [Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS).

### Acknowledgements

Our sincere thanks to the Qwen team for open-sourcing
[Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS) — the base model, tokenizer, and
reference inference code that make this work possible.
