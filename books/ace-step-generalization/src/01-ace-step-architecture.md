# 1. ACE-Step 1.5 Architecture

## 1.1 Core idea

ACE-Step 1.5 is implemented as a two-stage system:

1. Planner: a 5Hz LM that can generate structured metadata and semantic audio codes.
2. Executor: a DiT-based conditional diffusion model that turns conditions into audio latents, then VAE decodes to waveform.

This is explicitly reflected in the docs and code:

- `User Input -> [5Hz LM] -> Semantic Blueprint -> [DiT] -> Audio` in `docs/en/Tutorial.md:126`.
- Optional LM behavior (`thinking`) in `docs/en/API.md:161` and `acestep/inference.py:398`.

## 1.2 Runtime architecture map

```text
Client/UI/API
  -> inference.generate_music(...)
      -> (optional) LLMHandler.generate_with_stop_condition(...)
           - Phase 1: CoT metadata
           - Phase 2: audio code token stream
      -> AceStepHandler.generate_music(...)
           -> service_generate(...)
              -> _prepare_batch(...)
              -> preprocess_batch(...)
              -> model.prepare_condition(...)
              -> model.generate_audio(...)
              -> VAE decode
```

Key code edges:

- Orchestration: `acestep/inference.py:310`, `acestep/inference.py:468`, `acestep/inference.py:581`.
- DiT request execution: `acestep/handler.py:1593`.
- Service core: `acestep/handler.py:950`.
- Model condition + diffusion: `acestep/models/turbo/modeling_acestep_v15_turbo.py:1604`, `acestep/models/turbo/modeling_acestep_v15_turbo.py:1780`.

## 1.3 Conditioning channels fused by executor

DiT is not conditioned by one text string. It fuses multiple channels:

- Text embeddings (caption/instruction/meta prompt).
- Lyric embeddings.
- Reference-audio timbre embeddings.
- Source/target latent masks for edit tasks.
- Optional LM-derived 25Hz hints decoded from audio code tokens.

Evidence:

- Batch conditioning assembly: `acestep/core/generation/handler/conditioning_batch.py:21`.
- Text/lyric token prep: `acestep/core/generation/handler/conditioning_text.py:57`.
- Condition encoder packing: `acestep/models/turbo/modeling_acestep_v15_turbo.py:1549`.
- LM hint injection into source latents: `acestep/models/turbo/modeling_acestep_v15_turbo.py:1635`.

## 1.4 Why the split matters

The split decouples responsibilities:

- Planner handles intent resolution, metadata normalization, and semantic blueprinting.
- Executor handles high-fidelity realization and domain-specific rendering.

Practically, this creates:

- Better controllability than direct text->audio only.
- An explicit interface (`metadata + audio_code_string`) that can be swapped or externally authored.
- Optional planner bypass for expert or edit-heavy modes.

Example bypass behavior:

- `thinking=false` leads to DiT text2music path and ignores provided `audio_code_string` per API docs (`docs/en/API.md:161`).
- Cover/repaint can rely on direct conditioning from audio/masks (`docs/en/Tutorial.md:146`).
