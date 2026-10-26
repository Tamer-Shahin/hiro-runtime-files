# Hiro runtime files

Releases in this repository hold the runtime files for Hiro's on-device voice engine: a WebAssembly
build of [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) and the speech models it runs. Hiro
downloads them at build time, checks each file's SHA-256 and serves them from its own origin. They
are never loaded from here at run time.

The files are built and published by a workflow in Hiro's (private) application repository, from
the sources below. Nothing in this repository is private: every file is open source and is published
unchanged, apart from the format changes listed.

## Release assets

Release assets are flat, so an asset's name is its path with `/` replaced by `--`
(`kws-gigaspeech-3.3M-2024-01-01--tokens.txt` is `kws-gigaspeech-3.3M-2024-01-01/tokens.txt`).
`SHA256SUMS` lists every file.

| Path | What it is | Source | Licence |
| :-- | :-- | :-- | :-- |
| `wasm/sherpa-onnx-wasm-web.{js,wasm}` | sherpa-onnx v1.13.8, built with Emscripten 4.0.23 as an ES module for a worker, exporting only the keyword spotter, streaming recogniser and voice-activity functions. No models are packed in. | [k2-fsa/sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) `v1.13.8`, `build-wasm-simd-web.sh` | Apache-2.0 (`LICENSE-sherpa-onnx.txt`). Links ONNX Runtime 1.28.2 (MIT, [microsoft/onnxruntime](https://github.com/microsoft/onnxruntime)) and the Emscripten runtime (MIT / University of Illinois NCSA). |
| `kws-gigaspeech-3.3M-2024-01-01/` | Keyword spotter: zipformer transducer, 3.3M parameters, int8 encoder and joiner. | [sherpa-onnx kws-models release](https://github.com/k2-fsa/sherpa-onnx/releases/tag/kws-models), `sherpa-onnx-kws-zipformer-gigaspeech-3.3M-2024-01-01.tar.bz2` (the `-mobile` build fails in sherpa-onnx 1.13.8); also on [ModelScope](https://www.modelscope.cn/models/pkufool/sherpa-onnx-kws-zipformer-gigaspeech-3.3M-2024-01-01) | Apache-2.0, as stated in the model's `README.md` (included). Trained on [GigaSpeech](https://github.com/SpeechColab/GigaSpeech) XL; see the note below. |
| `asr-zipformer-en-20M-2023-02-17/` | Streaming English recogniser: zipformer transducer, 20M parameters, int8 encoder and joiner. | [sherpa-onnx asr-models release](https://github.com/k2-fsa/sherpa-onnx/releases/tag/asr-models), `sherpa-onnx-streaming-zipformer-en-20M-2023-02-17-mobile.tar.bz2`, exported from [desh2608/icefall-asr-librispeech-pruned-transducer-stateless7-streaming-small](https://huggingface.co/desh2608/icefall-asr-librispeech-pruned-transducer-stateless7-streaming-small) | Apache-2.0. Trained on [LibriSpeech](https://www.openslr.org/12) (CC BY 4.0). |
| `*/bpe.vocab` | Each model's SentencePiece vocabulary as text (piece and score per line), so the browser can split keywords and hotwords into the model's tokens. | Exported with `sentencepiece` 0.2.2 from the KWS tarball's `bpe.model` and from the recogniser's `data/lang_bpe_500/bpe.model` on Hugging Face (link above) | Same as the model it belongs to. |
| `silero-vad-v5/silero_vad.onnx` | Silero VAD v5, voice-activity detection. | [sherpa-onnx asr-models release](https://github.com/k2-fsa/sherpa-onnx/releases/tag/asr-models) `silero_vad_v5.onnx`, from [snakers4/silero-vad](https://github.com/snakers4/silero-vad) | MIT |

## Note on the keyword spotter's training data

The keyword-spotter model is published under Apache-2.0, but it was trained on GigaSpeech, whose
dataset terms limit the audio itself to non-commercial research and education. Whether those terms
carry over to a model trained on the data is a legal question that the Hiro team still has to settle
before a commercial release. The model is a starting choice and can be replaced (for example, by a
custom wake-phrase model) without changing the rest of the engine.
