# 4. End-to-End Inference Flow

## 4.1 Reference flow in one place

`acestep/inference.py:310` is the clearest integration seam:

1. Decide if LM is needed (`thinking` or CoT auto-fill requirements).
2. Run LM in chunks (`generate_with_stop_condition`).
3. Merge LM metadata into missing user metadata.
4. Pass caption/lyrics/meta/audio_code_string into DiT handler.
5. Save/return generated audio artifacts.

LM usage decision logic is at `acestep/inference.py:398`.

## 4.2 Thinking semantics

Server/API semantics explicitly separate planner use from executor mode:

- `thinking=false`: no LM code generation for text2music path.
- `thinking=true`: LM generates codes (`lm-dit` behavior).

Documented in `docs/en/API.md:159` and enforced through inference routing.

## 4.3 Minimal pseudocode

```python
def generate_asset(dit_handler, llm_handler, params, config):
    use_lm = should_use_lm(params, llm_handler)

    if use_lm:
        lm_out = llm_handler.generate_with_stop_condition(
            caption=params.caption,
            lyrics=params.lyrics,
            infer_type="llm_dit" if params.thinking else "dit",
            target_duration=params.duration,
        )
        params = merge_missing_metadata(params, lm_out["metadata"])
        if params.thinking:
            params.audio_codes = lm_out["audio_codes"]

    return dit_handler.generate_music(
        captions=params.caption,
        lyrics=params.lyrics,
        bpm=params.bpm,
        key_scale=params.keyscale,
        time_signature=params.timesignature,
        audio_code_string=params.audio_codes,
        task_type=params.task_type,
    )
```

Equivalent concrete calls are in `acestep/inference.py:468` and `acestep/inference.py:581`.

## 4.4 System-level optimizations worth preserving

ACE-Step includes pragmatic production guards that are part of architecture quality:

- GPU-tier adaptive defaults (`acestep/gpu_config.py:130`).
- Backend adaptation (`vllm`, `pt`, `mlx`) in LM and DiT paths.
- Offload/quantization policy by VRAM tier (`acestep/acestep_v15_pipeline.py:92`).
- Batch clamping and seed normalization (`acestep/core/generation/handler/service_generate_request.py:10`).

When generalizing to other domains, keep this “adaptive runtime policy layer” rather than hard-coding one deployment profile.
