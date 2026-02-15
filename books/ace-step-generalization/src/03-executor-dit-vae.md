# 3. Executor: DiT + VAE

## 3.1 Executor contract

Executor consumes normalized conditions and emits target latents, then waveform.

Primary entry:

- `AceStepHandler.service_generate(...)` at `acestep/handler.py:950`.

The executor is decomposed into focused handler mixins (`acestep/core/generation/handler/__init__.py:1`), which makes responsibilities explicit.

## 3.2 Condition building pipeline

`service_generate` performs:

1. Input normalization: `_normalize_service_generate_inputs`.
2. Batch construction: `_prepare_batch`.
3. Embedding preparation: `preprocess_batch`.
4. Condition assembly + diffusion call.

Code anchors:

- `acestep/core/generation/handler/service_generate_request.py:36`.
- `acestep/core/generation/handler/conditioning_batch.py:21`.
- `acestep/core/generation/handler/conditioning_embed.py:81`.
- `acestep/core/generation/handler/service_generate_execute.py:107`.

## 3.3 How LM codes become DiT hints

If LM (or user) provides serialized audio codes:

1. Parse code tokens from string.
2. Quantizer lookup + detokenize to 25Hz latent hints.
3. Inject into cover path via `prepare_condition`.

Code anchors:

- Parse/decode: `acestep/core/generation/handler/audio_codes.py:20` and `acestep/core/generation/handler/audio_codes.py:47`.
- Injection: `acestep/models/turbo/modeling_acestep_v15_turbo.py:1635`.

## 3.4 DiT condition encoder internals

`AceStepConditionEncoder` combines modalities:

- text projector,
- lyric encoder,
- timbre encoder,
- sequence packer.

See `acestep/models/turbo/modeling_acestep_v15_turbo.py:1506`.

The packed condition is then cross-attended by the diffusion decoder.

## 3.5 Diffusion execution

`generate_audio` supports:

- ODE and SDE stepping,
- shift-based timestep schedules (or mapped custom timesteps),
- cover/noise blending controls.

See `acestep/models/turbo/modeling_acestep_v15_turbo.py:1780` and step loop at `acestep/models/turbo/modeling_acestep_v15_turbo.py:1947`.

## 3.6 Decode and return

After diffusion:

- latents are VAE decoded,
- anti-clipping normalization is applied,
- tensors + intermediate state are returned for downstream tooling.

Flow in `acestep/handler.py:1717` through `acestep/handler.py:1919`.
