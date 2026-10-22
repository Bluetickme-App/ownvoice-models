# OwnVoice detector model: desklib DeBERTa-v3-large, weight-only int8 ONNX

This release is a **converted copy** of
[`desklib/ai-text-detector-v1.01`](https://huggingface.co/desklib/ai-text-detector-v1.01) at revision
`5fdea974cd4287c61674951ec78803aa274e2fb7` (`model.safetensors` sha256
`c024a1704c65f5a4bffeda58745c58fc0ed67d6ca07b158b068a257238815265`), itself a fine-tune of
[`microsoft/deberta-v3-large`](https://huggingface.co/microsoft/deberta-v3-large). Both are published under
the **MIT licence** (declared in their model cards; neither upstream repository has a separate LICENSE file).
The MIT text with the copyright lines of Microsoft Corporation (DeBERTa-v3) and Desklib (the fine-tuned
detector) is in `LICENSE`, at the top of this repository and next to the model files, so it travels with every
copy. The upstream model card is included unchanged as `UPSTREAM-MODEL-CARD.md`.

It is not trained or fine-tuned further. Only the format and number precision were changed:

1. `torch.onnx.export` (torch 2.14.1, transformers 5.20.2, opset 17) of DeBERTa + the model card's
   attention-masked mean pooling + 1-logit classifier.
2. ONNX Runtime 1.23.2 weight-only quantisation: every MatMul weight to symmetric 8-bit blocks of 128
   (`MatMulNBits`, `accuracy_level` 4), after pointing the MatMuls that share the query/key weights at the
   weights directly; the word-embedding table to per-tensor int8.

Mean difference from the fp32 export on 188 held-out paragraphs: about 0.003 in probability.

| Asset | Path in the model directory |
| --- | --- |
| `LICENSE`, `config.json`, `tokenizer.json`, `tokenizer_config.json`, `special_tokens_map.json`, `added_tokens.json` | same name |
| `model_quantized.onnx` | `onnx/model_quantized.onnx` |

Used by OwnVoice's `local` detector (`packages/detector`), which downloads these assets at image build time
and checks every file's size and sha256 against its manifest. The scores are shown to students only as
broad bands with reasons, never as a percentage. A flag is not proof that text was AI-generated.

Reproduce: `packages/detector/scripts/export_onnx.py desklib-deberta-v3-large-v1.01 <out>` in the OwnVoice
repository (see `packages/detector/MODEL.md`). Conversions are not bit-identical across machines, so the
manifest pins these exact files.
