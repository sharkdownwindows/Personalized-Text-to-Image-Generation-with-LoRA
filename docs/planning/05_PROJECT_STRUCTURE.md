# Recommended Project Structure

## 1. Repository layout

```text
personalized-t2i-lora/
├── README.md
├── LICENSES.md
├── .gitignore
├── pyproject.toml
├── requirements.in
├── requirements.lock
│
├── configs/
│   ├── base.yaml
│   ├── schema.yaml
│   ├── data_sweep/
│   │   ├── toy01_n1_r16.yaml
│   │   ├── toy01_n3_r16.yaml
│   │   ├── toy01_n5_r16.yaml
│   │   └── toy01_n10_r16.yaml
│   └── rank_sweep/
│       ├── toy01_n5_r4.yaml
│       └── toy01_n5_r32.yaml
│
├── data/
│   ├── README.md
│   ├── raw/                       # gitignored/private
│   ├── processed/                 # gitignored or shared separately
│   │   └── <concept_id>/<version>/
│   ├── manifests/
│   │   ├── concepts.csv
│   │   └── <concept_id>_v1.csv
│   └── eval_refs/
│       └── <concept_id>/          # three held-out references; optional crops
│
├── prompt_bank/
│   ├── evaluation_prompts.yaml
│   └── generation_seeds.yaml
│
├── src/
│   └── personalized_t2i/
│       ├── __init__.py
│       ├── config.py
│       ├── registry.py
│       ├── data/
│       │   ├── validate.py
│       │   └── prepare.py
│       ├── training/
│       │   ├── train.py
│       │   └── sweep.py
│       ├── inference/
│       │   └── generate.py
│       ├── evaluation/
│       │   ├── fidelity.py
│       │   ├── alignment.py
│       │   ├── diversity.py       # optional
│       │   └── aggregate.py
│       └── reporting/
│           ├── charts.py
│           └── grids.py
│
├── app/
│   └── demo.py                    # Gradio inference/result explorer
│
├── scripts/
│   ├── validate_data.py
│   ├── train_run.py
│   ├── run_sweep.py
│   ├── generate_eval_set.py
│   ├── evaluate_outputs.py
│   └── build_report_assets.py
│
├── tests/
│   ├── test_config.py
│   ├── test_manifest.py
│   ├── test_registry.py
│   └── test_vertical_slice.py
│
├── artifacts/                     # gitignored
│   └── <run_id>/
│       ├── status.json
│       ├── config.resolved.yaml
│       ├── environment.json
│       ├── logs/
│       ├── checkpoints/
│       ├── adapter/
│       │   └── adapter.safetensors
│       ├── generations/
│       │   └── <prompt_id>/<seed>.png
│       └── metadata.jsonl
│
├── results/
│   ├── metrics_per_sample.csv
│   ├── metrics_aggregate.csv
│   ├── human_ratings.csv
│   ├── failures.csv
│   ├── figures/
│   └── qualitative_grids/
│
├── notebooks/
│   └── exploratory_analysis.ipynb # exploration only; not source of truth
│
└── docs/
    ├── protocol.md
    ├── decision_log.md
    ├── runbook.md
    ├── evaluation_rubric.md
    └── report_outline.md
```

## 2. Design rules

- `src/` chứa logic có thể tái sử dụng và test.
- `scripts/` chỉ là entry points mỏng gọi code trong `src/`.
- `configs/` là nguồn duy nhất của hyperparameters.
- `artifacts/` chứa output từng run và không commit vào Git.
- `results/metrics_per_sample.csv` là nguồn dữ liệu đánh giá; aggregate/figures là derived artifacts.
- Notebook dùng để khám phá, không chứa bước bắt buộc duy nhất của pipeline.
- Raw/private images không tự động upload hoặc commit.

## 3. Naming conventions

### Concept ID

```text
<category><two_digit_index>
```

Ví dụ: `toy01`, `bottle01`, `plush01`.

### Run ID

```text
<concept_id>_n<data_size>_r<rank>_ts<training_seed>
```

Ví dụ:

```text
toy01_n5_r16_ts42
```

Nếu protocol version thay đổi:

```text
toy01_n5_r16_ts42_pv2
```

### Generated sample

```text
<run_id>__<prompt_id>__gs<generation_seed>.png
```

## 4. Configuration contract

```yaml
run:
  id: toy01_n5_r16_ts42
  protocol_version: v1

model:
  id: stable-diffusion-v1-5/stable-diffusion-v1-5
  revision: <pinned-sha>

data:
  concept_id: toy01
  dataset_version: v1
  manifest: data/manifests/toy01_v1.csv
  subset_size: 5
  instance_prompt: "a photo of toktoy toy"

training:
  rank: 16
  alpha: 16
  resolution: 512
  learning_rate: 1.0e-4
  max_train_steps: 500
  checkpointing_steps: 100
  batch_size: 1
  gradient_accumulation_steps: 1
  scheduler: constant
  mixed_precision: fp16
  train_text_encoder: false
  prior_preservation: false
  seed: 42
```

The validator must enforce:

- unique run ID;
- `alpha == rank` trong core rank sweep;
- allowed subset sizes `{1,3,5,10}`;
- allowed core ranks `{4,16,32}`;
- dataset/model revision tồn tại;
- no output overwrite nếu run đã complete.

## 5. Dataset manifest contract

| Field | Type | Required | Meaning |
|---|---|---:|---|
| `image_id` | string | Yes | Stable image identifier |
| `file_path` | string | Yes | Relative path from repository root |
| `sha256` | string | Yes | Integrity and version check |
| `concept_id` | string | Yes | Subject identifier |
| `split` | enum | Yes | `train_pool` or `heldout` |
| `subset_membership` | string | Yes for train | Example: `1,3,5,10` |
| `caption` | string | Yes for train | Consistent instance caption |
| `source` | string | Yes | Self-captured or licensed source |
| `consent_or_license` | string | Yes | Permission record |
| `notes` | string | No | Quality or viewpoint notes |

## 6. Artifact contract

Every completed run directory must contain:

```text
status.json
config.resolved.yaml
environment.json
adapter/adapter.safetensors
logs/
generations/
metadata.jsonl
```

### `status.json`

```json
{
  "run_id": "toy01_n5_r16_ts42",
  "status": "completed",
  "started_at": "<ISO-8601>",
  "completed_at": "<ISO-8601>",
  "error": null
}
```

Allowed states:

```text
queued → running → completed
                 ↘ failed
```

A failed run remains in the registry with its error log.

## 7. Results contract

### Per-sample metrics

Required columns:

```text
sample_id
run_id
concept_id
prompt_id
generation_seed
checkpoint_step
rank
data_size
dino_subject_similarity
clip_prompt_similarity
lpips_diversity_optional
valid
invalid_reason
```

### Human ratings

```text
rating_id
anonymous_rater_id
sample_id
subject_fidelity_1_5
prompt_alignment_1_5
visual_quality_1_5
failure_tags
comment_optional
```

Configuration identity must be hidden from the rating form.

## 8. Minimal commands

The final README should expose one command per stage:

```text
python scripts/validate_data.py --manifest <path>
python scripts/train_run.py --config <path>
python scripts/generate_eval_set.py --run-id <id>
python scripts/evaluate_outputs.py --run-id <id>
python scripts/build_report_assets.py
python app/demo.py
```

Exact flags may change during implementation, but the number of user-facing entry points should remain small.

## 9. Version-control policy

Commit:

- source code;
- configs and schemas;
- dataset manifests without private data;
- prompt bank;
- tests;
- small result tables/figures approved for submission;
- documentation.

Do not commit by default:

- base-model cache;
- raw/private images;
- LoRA checkpoints and large generated batches;
- access tokens;
- local virtual environments;
- temporary metric embeddings.

Store large artifacts in an agreed shared directory or controlled drive. Every shared artifact must preserve the repository-relative run ID structure.

## 10. Test strategy

| Test | Purpose |
|---|---|
| Config schema test | Reject invalid rank, missing model revision and duplicate run ID |
| Manifest test | Detect missing file, bad hash, split leakage and malformed subset |
| Artifact contract test | Confirm required files after a mocked/small run |
| Generation completeness test | Detect missing prompt/seed combinations |
| Metric smoke test | Score one known sample without NaN |
| Vertical-slice test | One tiny train → load → generate → score path |

Do not attempt expensive model-quality unit tests. Validate interfaces, metadata, completeness and deterministic setup instead.
