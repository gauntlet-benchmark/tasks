# GauntletBenchmark

A multi-domain agent benchmark covering five interactive web applications: **Workflow Builder** (n8n-style pipelines), **3D Modeller** (Three.js scenes), **Video Editor** (browser-based timeline), **Circuit Designer** (CircuitJS), and **Flight Analyser** (ADS-B replay).

Each application contributes 27 tasks at a mix of *easy*, *medium*, and *hard* difficulties, for a total of **135 tasks**. Tasks ask the agent to perform a goal-oriented sequence of UI actions and report a structured JSON answer; the answer (and, where applicable, the resulting application state) is then compared against an authoritative ground truth.

This repository contains only the dataset — task prompts, ground truth, and a [Croissant](http://mlcommons.org/croissant/) metadata descriptor. The evaluation harness is hosted separately.

## Applications

All five YAMLs follow the same shape (27 tasks each, split 9 easy / 9 medium / 9 hard):

| Application      | Tasks file                       | Ground truth                 | Task IDs                |
| ---------------- | -------------------------------- | ---------------------------- | ----------------------- |
| Workflow Builder | `tasks/workflow_builder.yaml`    | `assets/graph_ground_truth/` | `tc_graph_001`–`027`    |
| 3D Modeller      | `tasks/3d_modeller.yaml`         | `assets/3d_ground_truth/`    | `tc_3d_001`–`027`       |
| Video Editor     | `tasks/video_editor.yaml`        | `assets/video_ground_truth/` | `tc_vid_001`–`027`      |
| Circuit Designer | `tasks/circuit_designer.yaml`    | inline (`gt` field)          | `tc_circuit_001`–`027`  |
| Flight Analyser  | `tasks/flight_analyser.yaml`     | inline (`gt` field)          | `tc_frad_001`–`027`     |

## Layout

```
.
├── metadata.json                  # Croissant 1.0 dataset descriptor
├── tasks/
│   ├── 3d_modeller.yaml
│   ├── circuit_designer.yaml
│   ├── flight_analyser.yaml
│   ├── video_editor.yaml
│   └── workflow_builder.yaml
└── assets/
    ├── 3d_ground_truth/           # task1.json … task27.json + tolerance_overrides.json
    ├── graph_ground_truth/        # task1.json … task27.json
    └── video_ground_truth/        # tc_vid_001.json … tc_vid_027.json
```

## Task schema

Each YAML file is a single object with a top-level `tasks` list. Every entry has the same shape:

```yaml
tasks:
  - id: tc_frad_001
    difficulty_level: easy
    prompt: |
      # <Task Title>
      ## GOAL ...
      ## STEPS ...
      # RESULT FORMAT
      ```json
      { "field": "..." }
      ```
    gt:
      answer: '{"field": "..."}'
```

| Field              | Type   | Notes                                                                                                  |
| ------------------ | ------ | ------------------------------------------------------------------------------------------------------ |
| `id`               | string | Unique within the benchmark (see Task IDs in the table above).                                         |
| `difficulty_level` | string | One of `easy`, `medium`, `hard`.                                                                       |
| `prompt`           | string | Markdown shown to the agent — goal, numbered steps, and a JSON result template.                       |
| `gt`               | object | Inline ground truth (Circuit, Flight). Empty for the other three apps; the `assets/` files apply.      |
| `eval_config`      | object | Optional. Per-task evaluator hints (e.g. numeric tolerances). Currently used only by Flight Analyser. |

### Ground-truth modes

Two patterns are used depending on what the task asks for:

- **Inline** — Circuit Designer and Flight Analyser. The exact answer (CircuitJS netlist, callsigns, counts, telemetry readings, …) lives in `gt.answer` directly in the YAML.
- **File-based** — Workflow Builder, 3D Modeller, Video Editor. The agent is graded against the final application state, which is too large for inline storage. The `gt` field in the YAML is empty; the authoritative ground truth is the corresponding JSON file in `assets/`. Mapping conventions:

  | App              | YAML id          | Asset path                                  |
  | ---------------- | ---------------- | ------------------------------------------- |
  | Workflow Builder | `tc_graph_NNN`   | `assets/graph_ground_truth/taskN.json`      |
  | 3D Modeller      | `tc_3d_NNN`      | `assets/3d_ground_truth/taskN.json`         |
  | Video Editor     | `tc_vid_NNN`     | `assets/video_ground_truth/tc_vid_NNN.json` |

  (Leading zeros are stripped when forming `taskN.json` for Workflow Builder and 3D Modeller.)

  Ground-truth files are available for all 27 tasks in each of these three applications.

### 3D Modeller tolerances

Floating-point scene properties are compared with a default absolute tolerance. Two tasks (numbers 14 and 23) override that default; the per-task values live in `assets/3d_ground_truth/tolerance_overrides.json`. Tasks not listed there fall back to the evaluator's default.

## Loading

Plain YAML, no preprocessing required:

```python
import yaml
from pathlib import Path

tasks = yaml.safe_load(Path("tasks/flight_analyser.yaml").read_text())["tasks"]
print(len(tasks), tasks[0]["id"], tasks[0]["difficulty_level"])
```

## Croissant metadata

`metadata.json` is a [Croissant 1.0](http://mlcommons.org/croissant/) descriptor. It declares each YAML as a `FileObject` (with SHA-256), each ground-truth folder as a `FileSet`, and each application's tasks as a `RecordSet` with fields extracted via `jsonPath` (`$.tasks[*].id`, `$.tasks[*].prompt`, …). Croissant-aware tooling can ingest the dataset directly from the descriptor.

## Versioning and citation

- Version: `1.0.0`
- License: [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/)
- Cite as: `GauntletBenchmark v1.0.0`
