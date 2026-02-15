# 5. Generalized Pattern

## 5.1 Pattern definition

Generalized ACE-Step pattern:

1. Planner model generates a compact, structured blueprint from user intent.
2. Executor model realizes that blueprint into target asset space.
3. A strict interface contract separates planner/executor concerns.

```text
User Intent
  -> Planner (semantic planning)
     -> Blueprint (schema + token stream + constraints)
        -> Executor (domain renderer)
           -> Asset
```

## 5.2 Why this can transfer across domains

The split helps when:

- Natural language is too ambiguous for direct high-fidelity generation.
- There is a latent/token representation the executor can consume reliably.
- You need explicit control knobs and editability.

It is less useful when:

- Executor already performs excellent direct instruction following.
- No meaningful intermediate representation exists.
- Additional planner latency is unacceptable.

## 5.3 The interface contract

A reusable contract should usually include:

- `plan_schema`: strongly typed fields (style, structure, duration, constraints).
- `semantic_stream`: token/code/program sequence for executor priors.
- `hard_constraints`: max length, class vocab, legality bounds.
- `confidence/quality estimates`: optional planner-side diagnostics.

In ACE-Step terms, this maps to `metadata + audio_code_string`.

## 5.4 Training decomposition

Recommended split training:

1. Train executor on paired condition->asset data.
2. Train planner on intent->intermediate outputs.
3. Distill planner outputs that maximize executor quality.
4. Add constrained decoding to guarantee interface validity.

ACE-Step’s constrained processor is a concrete example of step 4 (`acestep/constrained_logits_processor.py:81`).

## 5.5 Runtime decomposition

At runtime, keep three paths:

- Full path: planner + executor.
- Expert path: executor with user-authored blueprint.
- Fallback path: executor-only minimal conditioning.

ACE-Step already exhibits this shape with `thinking` toggles and task-based LM skip logic (`acestep/inference.py:389`).
