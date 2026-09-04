# Qwen3.6

Qwen3.6 is Alibaba's next-generation multimodal MoE family, providing efficient inference with sparsely-activated parameters. The flagship variant is `Qwen3.6-35B-A3B` - 35B total parameters with ~3B activated per token, delivering 30B-class quality at near-3B inference cost.

## Variants

| Variant | Params | Activated | Quantization | Notes |
|---------|--------|-----------|--------------|-------|
| 35b-a3b-gguf | 35B | 3B | UD-Q5_K_XL | llama.cpp GGUF format, UD = "ultra-dense" mixed quantization |
| 35b-a3b-fp8 | 35B | 3B | FP8 (e4m3, block 128x128) | Qwen's own safetensors release, served by vLLM; vision encoder in-repository |

## Key Features

- **MoE Architecture**: ~3B parameters activated per token from a 35B total pool
- **Multimodal**: Text and vision throughout, plus video on the FP8 variant (in-repository encoder; the GGUF variant uses an mmproj projector)
- **Long Context**: 262,144 tokens native (up to ~1M with YaRN); both variants deploy at 131,072
- **Tool Calling**: Jinja chat template supports OpenAI-compatible tool calls
- **Apache 2.0 License**

## References

- HuggingFace (FP8): <https://huggingface.co/Qwen/Qwen3.6-35B-A3B-FP8>
- HuggingFace (GGUF): <https://huggingface.co/unsloth/Qwen3.6-35B-A3B-GGUF>
- Qwen project: <https://qwenlm.github.io>
