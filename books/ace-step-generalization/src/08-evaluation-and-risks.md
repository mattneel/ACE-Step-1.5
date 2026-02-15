# 8. Evaluation and Risks

## 8.1 Evaluate planner and executor separately

Do not only track end-to-end quality. Track:

- planner schema validity rate,
- planner token legality rate,
- planner->executor contract satisfaction,
- executor fidelity conditional on valid plans,
- full system quality.

ACE-Step’s profiling split (LM time vs DiT time vs VAE time) follows this decomposition spirit (`README.md:234`, `docs/en/BENCHMARK.md:21`).

## 8.2 Latency budget model

For planner-executor systems:

- short generations can be planner-time dominated,
- long generations are executor-time dominated.

ACE-Step explicitly documents this trend (`docs/en/BENCHMARK.md:430`).

## 8.3 Common failure modes

1. Planner outputs valid syntax but poor semantics.
2. Planner outputs semantically good but executor-incompatible tokens.
3. Executor overfits planner priors and ignores user edits.
4. Planner/executor drift after independent fine-tuning.

Mitigations:

- enforce interface constraints,
- maintain contract tests,
- keep planner and executor versioned as a pair,
- support bypass/fallback modes.

## 8.4 Safety and misuse

Structured planning can increase controllability, but also misuse precision.

Recommended controls:

- policy checks pre-planner and post-planner,
- bounded decoding for sensitive fields,
- audit logging for generated blueprints,
- domain-specific abuse filters on final outputs.

## 8.5 Research risks when generalizing

Do not assume transfer works because “LM+DiT worked for music.” Validate:

- representation adequacy for new domain,
- executor sensitivity to planner tokens,
- robustness under out-of-distribution prompts,
- cost/latency vs quality tradeoff against direct generation baselines.
