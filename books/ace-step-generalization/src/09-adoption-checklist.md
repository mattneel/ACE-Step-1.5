# 9. Adoption Checklist

Use this checklist when porting the architecture to a new asset domain.

## 9.1 Representation

- Is there a compact intermediate representation the executor can consume?
- Can you define strict validity constraints for that representation?
- Can user edits be expressed at that intermediate layer?

## 9.2 Model boundaries

- Planner responsibility is explicit and limited.
- Executor responsibility is explicit and limited.
- Interface schema is versioned and tested.

## 9.3 Runtime policy

- Hardware-aware defaults (batch, duration, model size, offload) are defined.
- Fallback paths exist when planner is unavailable.
- Batch behavior and seeds are deterministic where needed.

## 9.4 Quality and observability

- Planner validity metrics are tracked.
- Executor fidelity metrics are tracked.
- End-to-end user metrics are tracked.
- Profiling separates planner, executor, and decode/render stages.

## 9.5 Rollout plan

1. Start with executor-only baseline.
2. Introduce planner for metadata/schema only.
3. Add semantic stream generation.
4. Turn on constrained decoding.
5. A/B test against baseline on quality, latency, and controllability.

## 9.6 Final guidance

Generalize the *interface discipline* and *responsibility split*, not just the component names.

“LLM + DiT” is one implementation instance. The transferable architecture is:

- `Planner model` + `strict intermediate contract` + `high-fidelity executor` + `adaptive runtime policy`.
