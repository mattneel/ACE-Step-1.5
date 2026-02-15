# 7. Python Reference Design

This section gives a reusable Python architecture inspired by ACE-Step’s runtime flow.

## 7.1 Core interfaces

```python
from dataclasses import dataclass
from typing import Any, Dict, Optional


@dataclass
class Blueprint:
    metadata: Dict[str, Any]
    semantic_stream: str
    diagnostics: Dict[str, Any]


class Planner:
    def plan(self, user_input: Dict[str, Any]) -> Blueprint:
        raise NotImplementedError


class Executor:
    def render(self, user_input: Dict[str, Any], blueprint: Optional[Blueprint]) -> Dict[str, Any]:
        raise NotImplementedError
```

## 7.2 ACE-Step-shaped orchestrator

```python
class PlannerExecutorPipeline:
    def __init__(self, planner: Planner, executor: Executor):
        self.planner = planner
        self.executor = executor

    def run(self, request: Dict[str, Any]) -> Dict[str, Any]:
        thinking = bool(request.get("thinking", False))
        need_auto_meta = bool(request.get("use_cot_metas", False))

        blueprint = None
        if thinking or need_auto_meta:
            blueprint = self.planner.plan(request)

            # merge metadata only for missing fields
            for field in ["bpm", "duration", "keyscale", "timesignature", "language"]:
                if not request.get(field) and field in blueprint.metadata:
                    request[field] = blueprint.metadata[field]

            # only force semantic stream when planner mode is enabled
            if thinking:
                request["semantic_stream"] = blueprint.semantic_stream

        return self.executor.render(request, blueprint)
```

This matches the real flow in `acestep/inference.py:330` to `acestep/inference.py:614`.

## 7.3 Example adapter using ACE-Step handlers

```python
class AceStepPlanner(Planner):
    def __init__(self, llm_handler):
        self.llm = llm_handler

    def plan(self, user_input: Dict[str, Any]) -> Blueprint:
        infer_type = "llm_dit" if user_input.get("thinking") else "dit"
        out = self.llm.generate_with_stop_condition(
            caption=user_input.get("caption", ""),
            lyrics=user_input.get("lyrics", ""),
            infer_type=infer_type,
            target_duration=user_input.get("duration"),
            use_constrained_decoding=True,
        )
        return Blueprint(
            metadata=out.get("metadata", {}),
            semantic_stream=out.get("audio_codes", ""),
            diagnostics=out.get("extra_outputs", {}),
        )


class AceStepExecutor(Executor):
    def __init__(self, dit_handler):
        self.dit = dit_handler

    def render(self, user_input: Dict[str, Any], blueprint: Optional[Blueprint]) -> Dict[str, Any]:
        return self.dit.generate_music(
            captions=user_input.get("caption", ""),
            lyrics=user_input.get("lyrics", ""),
            bpm=user_input.get("bpm"),
            key_scale=user_input.get("keyscale", ""),
            time_signature=user_input.get("timesignature", ""),
            audio_code_string=user_input.get("semantic_stream", ""),
            task_type=user_input.get("task_type", "text2music"),
        )
```

## 7.4 Strong recommendation for generalized systems

Before invoking the executor, validate planner output against a strict schema:

- required fields,
- value ranges,
- token legality,
- expected sequence budget.

This is the transferable lesson from ACE-Step’s constrained logits processor.
