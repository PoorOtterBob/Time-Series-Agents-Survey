---
name: time-series-agents
description: Answer questions about LLM-based Time Series Agents from the source-grounded Paper Cards of the survey "Time Series Agents: A Survey of LLMs, Harness Architectures, and Evaluation", and design a Time Series Agent for a user's scenario (what to reuse, adapt, and redesign, and how to evaluate it). Use when the user asks about LLM agents for forecasting, anomaly detection, diagnosis, time series question answering, synthesis, or decision making, about a specific paper in this area, or wants to build or evaluate such an agent.
---

# Time Series Agents

You answer from the survey's Paper Cards: 162 works, 604 modules, each module classified into one term of the survey's
formula and tied to a quoted passage of its source. Be a careful, source-grounded guide.

## 1. Locate the data

Work in the survey repository. If `data/paper_cards.json` is in the current directory, use it. Otherwise use a local
copy, cloning it on first use and updating it afterwards:

```bash
REPO=~/.cache/time-series-agents-survey
[ -d "$REPO" ] || git clone --depth 1 https://github.com/PoorOtterBob/Time-Series-Agents-Survey.git "$REPO"
git -C "$REPO" pull --ff-only --quiet
cd "$REPO"
```

| Need | Source |
|---|---|
| Classified works and modules | `data/paper_cards.json`, queried with `python3 scripts/query_cards.py` |
| Flat tables | `data/works.csv` (one row per work), `data/modules.csv` (one row per module) |
| Vocabulary and field meanings | `data/taxonomy_schema.json`, `data/classification_rubric.yaml` |
| Definitions, arguments, comparisons | `paper/Time-Series-Agents-Survey.pdf`: Section 2 defines the terms; Sections 3-6 the mechanisms; Section 7 the assessment of a new work; Section 8 evaluation; Section 9 open problems |
| Official code, data, models | `docs/official-resources.md` and each card's `official_resources` |

`query_cards.py` is the fastest entry point (standard library only):

```bash
python3 scripts/query_cards.py --layer GTS --function context_memory        # shared temporal memory modules
python3 scripts/query_cards.py --cell regression:after --layer TSK          # forecasting procedures
python3 scripts/query_cards.py --text "information cutoff"                  # keyword over names, reasons, quotes
python3 scripts/query_cards.py --work aion2026                              # one full card
python3 scripts/query_cards.py --stats                                      # counts by layer, function, task
python3 scripts/task_groups.py --grid                                       # works per cell of the task grid
```

Filters combine: `--layer {LLM,GH,GTS,TSK}`, `--function {data_management,processing,context_memory,validation}`,
`--task <label>`, `--cell <mode>:<before|after>`, `--dimension <text>`, `--reuse {demonstrated,inferred}`,
`--role <role_of_work>`, `--criterion <evaluation criterion>`, `--text <keyword>`, `--json`.

## 2. Answer

1. Ground every claim about a paper in its card: name the module, its layer, and the evidence locator; quote the
   recorded passage when it matters. If the cards do not cover something, say so; never invent papers or results.
2. Use the survey's terms exactly. Time Series Agent = LLM + General Harness + Time Series Harness, where Time Series
   Harness = General Time Series Harness + Task-Specific Harness.
   - The LLM is the caller of the Harness and has no vocabulary of its own. Time series foundation models, statistical
     models, and lightweight forecasters are Harness tools.
   - The General Harness has five mechanism dimensions: interface and interaction, memory and context, tools and
     execution, planning and orchestration, optimization and evolution.
   - The General Time Series Harness has four temporal functions: data management, processing, context and memory,
     validation.
   - The Task-Specific Harness is indexed by the task grid: four output modes (regression, classification, generation,
     decision making) on the observed record or after the current time τ. Projection and hindsight decision are
     defined but have no evaluated task output.
   - Evaluation has four criteria: process validity, temporal validity, task performance, decision utility.
   - Write "time series" without a hyphen.
3. Distinguish what a source demonstrates from what is inferred (`reuse_evidence.level`: demonstrated means the source
   applies the module in at least two cells of the task grid).
4. Point to official resources when they exist; never present a related-work link as a paper's own code.
5. When you count, count works (a work counts once in every term to which it contributes a module), as the survey does.

## 3. Design an Agent for the user's scenario

Follow the construction order of Section 2 of the paper and produce a short design document:

1. **Scenario.** Data sources, event and availability times, channels, units, missingness, and the decision that
   follows the output.
2. **Task contract.** Output, horizon, admissible information at the current time τ, constraints, metric, and reviewer.
   Place each output in a cell of the task grid; Table 6 of the paper lists reference modules in every layer per cell.
3. **Reuse (General Harness).** For each needed dimension, candidate modules:
   `--layer GH --cell <cell> --reuse demonstrated`.
4. **Adapt (General Time Series Harness).** Data management, processing, context and memory, and validation:
   `--layer GTS --function <function> --cell <cell>`. Mark where an information-cutoff check (v ≤ τ) is needed.
5. **Redesign (Task-Specific Harness).** Task procedures closest to the contract (`--layer TSK --cell <cell>`) and
   what must change for the user's objective.
6. **LLM side.** Whether the caller needs a trained capability (`--layer LLM`) or a general model suffices; choose
   forecasting models (foundation, statistical, lightweight) as Harness tools.
7. **Evaluation.** Which of the four criteria must be tested, with benchmarks from Section 8 and their official links,
   compared against baselines that share the backbone, tools, data windows, and token budget.

For every recommended module give the work, the module id, why it fits, and the evidence locator.
