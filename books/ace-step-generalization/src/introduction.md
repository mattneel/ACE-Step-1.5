# Introduction

This book documents the implemented ACE-Step 1.5 architecture and then generalizes it into a reusable pattern for other generative AI domains.

Scope:

- Grounded in current repository code and docs.
- Focused on architecture and transferability, not UI walkthroughs.
- Uses Python examples aligned with existing implementation style.

Primary repository anchors:

- High-level architecture statement: `README.md:30`, `docs/en/Tutorial.md:123`.
- LM planner implementation: `acestep/llm_inference.py:1111`.
- Constrained decoding FSM: `acestep/constrained_logits_processor.py:81`.
- DiT-side generation orchestration: `acestep/handler.py:950`, `acestep/handler.py:1593`.
- End-to-end LM→DiT routing: `acestep/inference.py:330`.

Important framing:

- ACE-Step’s planner-executor split is a strong pattern, but it is not universally superior for every domain.
- The architecture transfers best when a domain supports a compact intermediate representation (semantic tokens, plans, or programs) that an executor can faithfully realize.
