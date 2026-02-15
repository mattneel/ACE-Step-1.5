# 2. Planner: 5Hz LM

## 2.1 Planner contract

Planner output is a compact semantic blueprint:

- Structured metadata fields (`bpm`, `duration`, `keyscale`, `language`, `timesignature`, optionally caption/genres).
- Audio semantic code stream: tokens like `<|audio_code_12345|>`.

The parser and expected format live in `acestep/llm_inference.py:2523`.

## 2.2 Two-phase generation protocol

`LLMHandler.generate_with_stop_condition` (`acestep/llm_inference.py:1111`) runs:

1. Phase 1 (`cot`): generate CoT metadata inside `<think>...</think>`.
2. Phase 2 (`codes`, only for `infer_type="llm_dit"`): continue generation into audio code tokens.

`infer_type` values:

- `dit`: metadata only.
- `llm_dit`: metadata + audio codes.

See branch logic at `acestep/llm_inference.py:1257` and `acestep/llm_inference.py:1287`.

## 2.3 Constrained decoding as interface guardrail

ACE-Step does not rely on unconstrained text completion for metadata schema.

`MetadataConstrainedLogitsProcessor` implements an FSM (`acestep/constrained_logits_processor.py:53`) to enforce:

- field order and syntax,
- numeric ranges (BPM, duration),
- valid timesignatures/languages/keyscales,
- audio code token legality in code-generation phase.

Important details:

- Audio code range is clamped/validated against `0..63999` (`acestep/constrained_logits_processor.py:47`, `acestep/core/generation/handler/audio_codes.py:25`).
- Duration is converted to code budget with `5 codes = 1 second` (`acestep/constrained_logits_processor.py:155`, `acestep/llm_inference.py:197`).

This is a key architectural insight: planner output is constrained to the executor’s accepted interface.

## 2.4 Prompt structure

Prompts are built as chat templates and include instruction + caption + lyric; phase 2 reuses phase-1 CoT in assistant context:

- `build_formatted_prompt`: `acestep/llm_inference.py:1475`.
- `build_formatted_prompt_with_cot`: `acestep/llm_inference.py:1523`.

This lets phase 2 generate tokens conditioned on an explicit, machine-readable plan.

## 2.5 Python pattern from repo

```python
lm_res = llm_handler.generate_with_stop_condition(
    caption=caption,
    lyrics=lyrics,
    infer_type="llm_dit",  # or "dit"
    target_duration=duration_sec,
    user_metadata=user_metadata,
    use_cot_metas=True,
    use_cot_caption=True,
    use_cot_language=True,
    use_constrained_decoding=True,
)

metadata = lm_res["metadata"]
audio_codes = lm_res["audio_codes"]
```

The equivalent call path is in `acestep/inference.py:468`.
