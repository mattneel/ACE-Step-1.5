# 6. Domain Playbooks

## 6.1 Text-to-image

Planner output:

- Scene graph, camera metadata, palette, composition constraints.
- Optional discrete visual code stream.

Executor:

- Diffusion image model conditioned on graph/tokens.

Why this mirrors ACE-Step:

- Similar ambiguity problem: prompt language vs pixel-level realization.
- Planner can standardize style + composition before denoising.

## 6.2 Text-to-video

Planner output:

- Shot list, timeline beats, motion intents, transitions.
- Temporal token stream with per-shot constraints.

Executor:

- Video diffusion/transformer conditioned on shot-level plan.

Key adaptation:

- Strong temporal schema is mandatory, analogous to duration->code budget handling.

## 6.3 Speech/voice generation

Planner output:

- Prosody plan: speaking rate, pauses, emotional contour, style tags.
- Phoneme/prosody token stream.

Executor:

- Acoustic model + vocoder stack.

Parallel with ACE-Step:

- Metadata constraints and token legality checks are critical for intelligibility.

## 6.4 3D asset generation

Planner output:

- Structural program: parts, proportions, materials, rig constraints.
- Optional mesh/implicit-primitive token program.

Executor:

- Geometry/material generator and renderer.

Pattern fit:

- Planner can produce topology-safe blueprints before expensive geometry synthesis.

## 6.5 Code/UI generation

Planner output:

- Architecture plan, file graph, API contracts, test plan.
- Structured intermediate representation (AST fragments or typed plan schema).

Executor:

- Code generator + validator loop.

Practical note:

- Constrained decoding maps naturally to grammar-constrained generation and static checks.

## 6.6 Where not to force it

Avoid planner+executor split if:

- Executor already has robust symbolic grounding.
- The added stage introduces too much latency for user value.
- Interface tokens are unstable and harder to validate than direct generation.
