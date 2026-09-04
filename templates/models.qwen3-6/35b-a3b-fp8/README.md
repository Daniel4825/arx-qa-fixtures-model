# Qwen3.6-35B-A3B FP8

Qwen's own FP8 quantization of Qwen3.6-35B-A3B, in transformers/safetensors format, served via vLLM.
Unlike the [GGUF variant](../35b-a3b-gguf/README.md) the vision encoder is part of the same
repository, so there is no separate mmproj file to place.

## Resources Required

| Resource | Requirement |
|----------|-------------|
| CPU | 4 cores |
| Memory | 16 GiB |
| Storage | 40 GiB (repository is ~34.9 GiB) |
| GPU | 1x GPU |
| VRAM | ~48 GiB (weights ~34 GiB + KV cache at 128K + activations) |

FP8 e4m3 is a native weight format on NVIDIA Hopper, Ada and Blackwell. Older GPUs can load the
checkpoint only by upcasting it, which gives up both the memory saving and the speed.

## Quick Start

```bash
arx deploy models.qwen3-6@35b-a3b-fp8
```

## Model Specifications

| Property | Value |
|----------|-------|
| Vendor | Alibaba |
| Family | Qwen3.6 |
| Total Parameters | 35B |
| Activated Parameters | ~3B per token (8 routed + 1 shared of 256 experts) |
| Layers | 40 (10 x (3 x Gated DeltaNet -> MoE, 1 x Gated Attention -> MoE)) |
| Format | safetensors (transformers) |
| Quantization | FP8 e4m3, fine-grained block 128x128, dynamic activation scheme |
| Vision Encoder | in-repository (no mmproj file) |
| Context Length | 131,072 tokens (deployed default), 262,144 native, up to 1,010,000 with YaRN |
| Input Types | Text, Image, Video |
| Output Types | Text |
| Inference Engine | vLLM >= 0.19.0 (`nvidia-vllm`) |

The deployed default is 131,072 rather than the native 262,144: full context needs the KV cache
budget of a larger card, and Qwen advises keeping at least 128K so that thinking mode still has
room to work.

## Configuration

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| Context Window | 131,072 | 8,192 - 262,144 | Maximum tokens in a single request |

## Suggested vLLM Flags

```
--port 8000
--max-model-len 131072
--reasoning-parser qwen3
--enable-auto-tool-choice --tool-call-parser qwen3_coder
```

Two options worth knowing about:

- `--language-model-only` skips the vision encoder and its multimodal profiling, freeing the memory
  for more KV cache. Use it when the deployment only ever sees text.
- `--speculative-config '{"method":"qwen3_next_mtp","num_speculative_tokens":2}'` turns on the
  multi-token-prediction head shipped as `mtp.safetensors`.
- `--media-io-kwargs '{"video": {"num_frames": -1}}'` hands frame sampling to the request, so
  `fps` can be set per call. Video input goes through the same `/v1/chat/completions` endpoint as
  images (`"type": "video_url"`).

Going past 262,144 tokens needs static YaRN, which is a trade rather than a free win: the scaling
factor stays constant regardless of input length and can cost accuracy on short prompts. The model
card has the exact `--hf-overrides` payload.

## Sampling

Qwen's recommended settings differ per mode:

| Mode | temperature | top_p | top_k | presence_penalty |
|------|-------------|-------|-------|------------------|
| Thinking, general | 1.0 | 0.95 | 20 | 1.5 |
| Thinking, precise coding | 0.6 | 0.95 | 20 | 0.0 |
| Instruct (non-thinking) | 0.7 | 0.80 | 20 | 1.5 |

`min_p=0.0` and `repetition_penalty=1.0` in every case.

## Source

- HuggingFace: <https://huggingface.co/Qwen/Qwen3.6-35B-A3B-FP8>
- Base model: <https://huggingface.co/Qwen/Qwen3.6-35B-A3B>
- License: Apache 2.0
