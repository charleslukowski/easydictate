# Free / Open-Source Speech-to-Text Models Research (Feb 2026)

EasyDictate currently uses **Whisper base.en** (~140 MB, via whisper.cpp / Whisper.net).
This document evaluates newer alternatives that are free, open-source, and can run
locally on CPU without a GPU requirement.

---

## 1. Voxtral Transcribe 2 (Mistral AI) — Released 2026-02-04

Mistral released two models under the Voxtral Transcribe 2 banner:

| Variant | Params | License | Local? | GPU Required? |
|---------|--------|---------|--------|---------------|
| **Mini Transcribe V2** (batch) | ~3B | Proprietary (API only) | No | N/A |
| **Mini 4B Realtime** (streaming) | 4B | **Apache 2.0** | Yes | Yes (~10-12 GB VRAM) |

### Key Details
- **API pricing**: $0.003/min batch, $0.006/min realtime (~$0.18/hr batch)
- **Languages**: 13 (en, zh, hi, es, ar, fr, pt, ru, de, ja, ko, it, nl)
- **Latency**: Configurable down to ~200ms for realtime
- **Accuracy**: Claims lowest WER of any transcription service; outperforms GPT-4o mini
  Transcribe, Gemini 2.5 Flash, AssemblyAI Universal, and Deepgram Nova

### Verdict for EasyDictate: NOT YET VIABLE
- The Realtime model (Apache 2.0) **requires a GPU with >=10 GB VRAM** and only runs
  via vLLM. There is **no GGUF/GGML quantization** and **no llama.cpp or whisper.cpp
  support** yet due to its novel architecture. Mistral has invited community contributions
  to port it to Transformers and llama.cpp.
- The batch API model is not open-source and not free.
- No C#/.NET bindings exist.
- **Watch for**: GGUF ports and CPU inference support from the community. If/when
  llama.cpp adds support, this could become compelling.

---

## 2. Moonshine (Useful Sensors) — Strong Candidate

| Variant | Params | Size | License |
|---------|--------|------|---------|
| **Tiny** | 27M | Very small | MIT |
| **Base** | 61.5M | Small | MIT |

### Key Details
- **5x faster** than Whisper Tiny on CPU with equivalent or better accuracy
- Proportional compute: processes audio length-proportionally (unlike Whisper's fixed
  30-second chunks), making short utterances much faster
- **Runs on CPU**, including Raspberry Pi and edge devices
- ONNX Runtime support for cross-platform inference
- C++ implementation exists: [moonshine.cpp](https://github.com/royshil/moonshine.cpp)
- Multilingual "Flavors of Moonshine" variants achieve 48% lower error rates than
  Whisper Tiny and match/beat Whisper Medium (28x larger) for specific languages

### Verdict for EasyDictate: PROMISING — Needs C# Integration Work
- No existing C#/.NET NuGet package for Moonshine
- Could integrate via:
  1. `Microsoft.ML.OnnxRuntime` NuGet + ONNX model files (most practical)
  2. P/Invoke bindings around moonshine.cpp
- Model is **much smaller** than current Whisper base.en (27-61M params vs 74M params)
- Would likely be faster for the short dictation clips EasyDictate handles
- MIT license is ideal

---

## 3. Whisper Large V3 Turbo (OpenAI) — Upgrade Path

| Variant | Params | GGML Size | License |
|---------|--------|-----------|---------|
| **large-v3-turbo** | 809M (4 decoder layers) | ~1.5 GB | MIT |

### Key Details
- 5.4x faster than Whisper Large V3 with similar accuracy
- Achieved by reducing decoder layers from 32 to 4
- Fully supported in whisper.cpp and Whisper.net (drop-in replacement)
- GGML quantized variants available (Q2_K through F32)
- Much more accurate than base.en, especially for noisy audio

### Verdict for EasyDictate: EASIEST UPGRADE
- **Drop-in replacement**: Change the model URL and filename in `ModelManager.cs`
- Same whisper.cpp/Whisper.net stack, no code changes needed beyond model config
- Trade-off: 1.5 GB download vs current 140 MB, and slower on pure CPU
- Could offer as a "high accuracy" option alongside the current base.en model
- Consider offering both models in Settings: "Fast (base.en, 140 MB)" vs
  "Accurate (large-v3-turbo, 1.5 GB)"

---

## 4. Other Notable Models

### NVIDIA Canary Qwen 2.5B
- Excellent English accuracy (6.67% WER on Open ASR Leaderboard)
- Requires ~8 GB VRAM — **GPU only**, not suitable for EasyDictate's CPU-only design

### NVIDIA Parakeet TDT 1.1B
- Extremely fast (RTFx >2000), designed for streaming
- RNN-Transducer architecture, English-only
- Requires GPU — not suitable for CPU-only

### IBM Granite Speech 3.3 8B
- Lowest WER on clean audio in some benchmarks
- 8B parameters — far too large for local CPU inference

### Distil-Whisper Large V3
- Distilled version of Whisper Large V3
- GGML available, works with whisper.cpp
- Smaller and faster than full large-v3 while retaining most accuracy
- Another viable drop-in option for EasyDictate

---

## Recommendation Summary

| Priority | Model | Effort | Benefit |
|----------|-------|--------|---------|
| **1 (Easy)** | Whisper large-v3-turbo | Minimal — change model URL | Much better accuracy, same stack |
| **2 (Easy)** | Distil-Whisper large-v3 | Minimal — change model URL | Better accuracy, smaller than full large-v3 |
| **3 (Medium)** | Moonshine Base | New integration via ONNX Runtime | Faster for short clips, smaller model |
| **4 (Wait)** | Voxtral Realtime | Blocked on community ports | Best accuracy, but needs GPU + vLLM today |

### Short-term: Offer Whisper large-v3-turbo as a model option
The lowest-effort, highest-value change is adding Whisper large-v3-turbo as an
alternative model in Settings. This requires only changing the download URL and model
filename — the entire whisper.cpp/Whisper.net integration stays the same.

### Medium-term: Evaluate Moonshine integration
Moonshine's tiny footprint and CPU efficiency make it ideal for a dictation app. The
main blocker is building C# bindings (via ONNX Runtime or moonshine.cpp P/Invoke).

### Long-term: Watch Voxtral Realtime
Once community GGUF/llama.cpp ports appear, Voxtral Realtime could become the accuracy
leader that also runs locally. Monitor the Hugging Face model page and whisper.cpp
project for updates.

---

## Sources
- [Mistral: Voxtral Transcribe 2](https://mistral.ai/news/voxtral-transcribe-2)
- [VentureBeat: Mistral drops Voxtral Transcribe 2](https://venturebeat.com/technology/mistral-drops-voxtral-transcribe-2-an-open-source-speech-model-that-runs-on-device-for-pennies/)
- [Voxtral-Mini-4B-Realtime-2602 on Hugging Face](https://huggingface.co/mistralai/Voxtral-Mini-4B-Realtime-2602)
- [Northflank: Best open-source STT model in 2026](https://northflank.com/blog/best-open-source-speech-to-text-stt-model-in-2026-benchmarks)
- [Moonshine on GitHub](https://github.com/moonshine-ai/moonshine)
- [moonshine.cpp on GitHub](https://github.com/royshil/moonshine.cpp)
- [whisper.cpp on GitHub](https://github.com/ggml-org/whisper.cpp)
- [Whisper large-v3-turbo GGML quants](https://huggingface.co/Pomni/whisper-large-v3-turbo-ggml-allquants)
- [Distil-Whisper large-v3 GGML](https://huggingface.co/distil-whisper/distil-large-v3-ggml)
- [AssemblyAI: Top free STT APIs](https://www.assemblyai.com/blog/the-top-free-speech-to-text-apis-and-open-source-engines)
- [Deepgram: Best STT APIs 2026](https://deepgram.com/learn/best-speech-to-text-apis-2026)
- [MarkTechPost: Voxtral Transcribe 2 launch](https://www.marktechpost.com/2026/02/04/mistral-ai-launches-voxtral-transcribe-2-pairing-batch-diarization-and-open-realtime-asr-for-multilingual-production-workloads-at-scale/)
