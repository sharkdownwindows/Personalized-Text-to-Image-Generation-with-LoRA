# Repository Guidelines

## Project Structure & Module Organization

This repository is currently planning-first. Requirements and the proposed layout live in `docs/planning/`; start with `01_PROJECT_BRIEF.md` and treat `03_PROJECT_OVERVIEW_AND_TECHNICAL_SPEC.md` as the technical reference.

Implementation should follow the documented layout:

- `src/personalized_t2i/`: reusable training, inference, evaluation, and reporting code.
- `scripts/`: thin command-line entry points; keep business logic in `src/`.
- `configs/` and `prompt_bank/`: versioned experiment settings, prompts, and seeds.
- `tests/`: fast interface, metadata, and pipeline smoke tests.
- `artifacts/`, raw data, checkpoints, and caches: generated/private content; do not commit.
- `results/`: reviewed metric tables, figures, and qualitative grids.

## Build, Test, and Development Commands

No executable pipeline or lockfile is committed yet. Planned user-facing commands are:

```text
python scripts/validate_data.py --manifest <path>
python scripts/train_run.py --config <path>
python scripts/generate_eval_set.py --run-id <id>
python scripts/evaluate_outputs.py --run-id <id>
python scripts/build_report_assets.py
python app/demo.py
pytest -q
```

Keep entry points small and config-driven. Record Python, CUDA, model, and package versions for every experiment.

## Coding Style & Naming Conventions

Use Python 3.10 or 3.11 after the pilot selects one version. Follow PEP 8, four-space indentation, `snake_case` for functions/modules, and `PascalCase` for classes. Add type hints to public interfaces. Use repository-relative paths and YAML for hyperparameters; never hard-code local paths, secrets, prompts, or seeds. No formatter or linter is configured yet.

Use concept IDs such as `toy01` and run IDs such as `toy01_n5_r16_ts42`.

## Testing Guidelines

Use pytest and name files `test_*.py`. Prioritize config, manifest, artifact-contract, generation-completeness, metric smoke, and vertical-slice tests. Avoid expensive model-quality unit tests. Bug fixes need a regression test or documented smoke test.

## Commit & Pull Request Guidelines

Git history currently contains only `Initial commit`, so no established message convention exists. Use short imperative messages with prefixes such as `feat:`, `fix:`, `test:`, or `docs:`. Keep changes narrowly scoped.

Pull requests should state the objective, affected run/config IDs, verification, and protocol or artifact impact. Link the issue or decision record. Include screenshots for result-view changes; never omit failed runs or unfavorable evidence.

## Security & Reproducibility

Never commit access tokens, private images, model caches, LoRA checkpoints, or large generated batches. Preserve dataset hashes, resolved configs, seeds, logs, environment metadata, and failure status. Protocol changes require a versioned decision record and reruns of affected experiment cells.
Refine the root AGENTS.md for this repository.

Keep it concise and project-specific. It must contain these rules:

- This is a 14-day university project for six students, not a production platform.
- Read docs/planning/01_PROJECT_BRIEF.md through
  docs/planning/06_ISSUES_AND_MILESTONES.md before structural or experimental changes.
- P0 scope is a reproducible CLI experiment pipeline plus a thin Gradio result explorer.
- Core experiment: 3 concepts, data sizes {1,3,5,10}, ranks {4,16,32},
  with 18 unique core runs.
- configs/ is the source of truth for experiment parameters.
- src/ contains reusable logic; scripts/ contains thin entry points.
- Never commit raw/private data, model caches, checkpoints, secrets or large generated batches.
- Do not download models, install large ML packages, start training, or run GPU experiments unless explicitly requested.
- Do not delete or move legacy project files without explicit approval.
- Do not commit, push, merge, create PRs, or mutate GitHub issues unless explicitly requested.
- Never rewrite Git history or force-push.
- Do not fabricate experiment results, successful commands or dependency locks.
- After local changes, run appropriate cheap validation, git diff --check, and report remaining limitations.

Keep AGENTS.md under 100 lines. Do not change any other file.