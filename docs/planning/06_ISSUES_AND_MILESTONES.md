# Milestones and Issue Backlog

## 1. Labels

| Label | Meaning |
|---|---|
| `priority:P0` | Bắt buộc để trả lời RQ1/RQ2 |
| `priority:P1` | Chỉ bắt đầu sau core gates |
| `area:product` | Scope, protocol, documentation |
| `area:data` | Dataset, manifests, prompt bank |
| `area:ml` | Training and inference |
| `area:backend` | Automation, registry, artifacts |
| `area:evaluation` | Metrics and analysis |
| `area:frontend` | Gradio and visualization |
| `area:qa` | Validation and reproduction |
| `blocked` | Không thể tiếp tục do dependency/blocker |

## 2. Milestones

### M0 — Scope and Protocol Freeze

**Deadline:** Day 1  
**Exit criteria:** RQ, scope, assumptions, roles, experiment matrix và success criteria được duyệt.

### M1 — Vertical Slice

**Deadline:** Day 3  
**Exit criteria:** Một concept chạy train → load → generate → metric; artifacts truy vết được.

### M2 — Dataset and Configuration Freeze

**Deadline:** Day 4  
**Exit criteria:** Dataset v1, nested subsets, prompts, seeds, baseline config và model revision được khóa.

### M3 — Core Experiments Complete

**Deadline:** Day 8  
**Exit criteria:** 18 target runs hoàn thành; contingency tối thiểu 12 comparable runs nếu compute failure được ghi rõ.

### M4 — Evaluation and Evidence Freeze

**Deadline:** Day 10  
**Exit criteria:** Automated metrics, human ratings, failure cases và source results được khóa.

### M5 — Release Candidate

**Deadline:** Day 12  
**Exit criteria:** Result explorer, charts, grids, report draft, slides và backup demo sẵn sàng.

### M6 — Submission Ready

**Deadline:** Day 14  
**Exit criteria:** Reproduction, QA, rehearsal, attribution và submission package hoàn tất.

## 3. P0 issues

### PROD-01 — Freeze research protocol

- **Owner:** Technical PM/BA
- **Reviewer:** ML Lead, Evaluation Lead
- **Milestone:** M0
- **Estimate:** 0.5 day
- **Depends on:** None
- **Acceptance criteria:**
  - RQ1/RQ2 và hypotheses được ghi.
  - Core matrix là 3 concepts × 6 configs.
  - Controlled variables, metrics, claim boundary và fallback được duyệt.
  - Decision log có protocol version `v1`.

### OPS-01 — Initialize repository and environment contract

- **Owner:** Backend/MLOps
- **Reviewer:** ML Lead
- **Milestone:** M1
- **Estimate:** 0.5 day
- **Depends on:** PROD-01
- **Acceptance criteria:**
  - Repository structure, `.gitignore` và dependency input tồn tại.
  - Base model ID/revision và environment metadata schema được định nghĩa.
  - Không có secret hoặc large artifact trong Git.

### DATA-01 — Select and document concepts

- **Owner:** Data Lead
- **Reviewer:** TPM/QA
- **Milestone:** M0
- **Estimate:** 0.5 day
- **Depends on:** PROD-01
- **Acceptance criteria:**
  - Ba non-human concepts được chọn.
  - Mỗi concept có class noun, unique token và ownership/license record.
  - Không có sensitive/private/copyright-risk concept chưa xử lý.

### DATA-02 — Build dataset manifests and held-out split

- **Owner:** Data Lead
- **Reviewer:** Backend/MLOps
- **Milestone:** M2
- **Estimate:** 1 day
- **Depends on:** DATA-01
- **Acceptance criteria:**
  - 10 train-pool và 3 held-out images/concept.
  - File hash, split, source/license và caption đầy đủ.
  - Held-out files không xuất hiện trong train subsets.

### DATA-03 — Build nested subsets and preprocessing

- **Owner:** Data Lead
- **Reviewer:** Evaluation Lead
- **Milestone:** M2
- **Estimate:** 0.5 day
- **Depends on:** DATA-02
- **Acceptance criteria:**
  - `D1 ⊂ D3 ⊂ D5 ⊂ D10`.
  - Selection order và viewpoint coverage được ghi trước results.
  - EXIF, RGB, crop, resize thống nhất; không random flip.

### DATA-04 — Freeze evaluation prompt bank and seeds

- **Owner:** Evaluation Lead
- **Reviewer:** TPM/BA
- **Milestone:** M2
- **Estimate:** 0.5 day
- **Depends on:** DATA-01, PROD-01
- **Acceptance criteria:**
  - 8 prompt IDs/concept theo categories đã định.
  - Four fixed generation seeds.
  - Prompt version được ghi và dùng cho base/LoRA outputs.

### ML-01 — Run GPU and backbone pilot

- **Owner:** ML Lead
- **Reviewer:** Backend/MLOps
- **Milestone:** M1
- **Estimate:** 0.5 day
- **Depends on:** OPS-01, DATA-02
- **Acceptance criteria:**
  - SD v1.5 load/train/inference không OOM.
  - Wall time và peak VRAM được ghi.
  - Backbone decision được khóa; SDXL chỉ được chọn nếu đạt gate.

### BE-01 — Implement config validation and run IDs

- **Owner:** Backend/MLOps
- **Reviewer:** TPM/BA
- **Milestone:** M1
- **Estimate:** 0.5 day
- **Depends on:** PROD-01, OPS-01
- **Acceptance criteria:**
  - YAML config kiểm tra model, dataset, rank, alpha, steps và seed.
  - Duplicate/completed run ID không bị ghi đè.
  - Core rank sweep enforce `alpha=rank`.

### ML-02 — Implement config-driven LoRA training wrapper

- **Owner:** ML Lead
- **Reviewer:** Backend/MLOps
- **Milestone:** M1
- **Estimate:** 1 day
- **Depends on:** ML-01, BE-01
- **Acceptance criteria:**
  - Một command/config khởi động official Diffusers trainer.
  - Adapter, checkpoint, logs và resolved config được lưu.
  - Adapter load lại thành công.

### BE-02 — Implement artifact registry and status lifecycle

- **Owner:** Backend/MLOps
- **Reviewer:** QA
- **Milestone:** M1
- **Estimate:** 0.5 day
- **Depends on:** BE-01
- **Acceptance criteria:**
  - States `queued/running/completed/failed`.
  - Run error được lưu; successful artifact không bị ghi đè.
  - Environment/model/dataset provenance đầy đủ.

### ML-03 — Implement batch generation

- **Owner:** Backend/MLOps + ML Lead
- **Reviewer:** Evaluation Lead
- **Milestone:** M1
- **Estimate:** 0.5 day
- **Depends on:** ML-02, DATA-04
- **Acceptance criteria:**
  - Sinh prompt × seed matrix cho base và adapter.
  - Tạo generator mới mỗi sample.
  - Metadata contract và missing-output check pass.

### EVAL-01 — Implement DINOv2 fidelity scoring

- **Owner:** Evaluation Lead
- **Reviewer:** ML Lead
- **Milestone:** M1
- **Estimate:** 0.75 day
- **Depends on:** DATA-02, ML-03
- **Acceptance criteria:**
  - Held-out reference centroid được dùng; crop audit trên subset nếu khả thi.
  - Per-sample score không NaN và có unit/smoke test.
  - Per-concept outputs được giữ trước aggregate.

### EVAL-02 — Implement CLIP prompt scoring

- **Owner:** Evaluation Lead
- **Reviewer:** QA
- **Milestone:** M1
- **Estimate:** 0.5 day
- **Depends on:** DATA-04, ML-03
- **Acceptance criteria:**
  - Unique token được normalize thành class noun cho metric text.
  - Per-sample CLIP score và metadata được export.
  - Invalid sample được báo riêng.

### QA-01 — Pass end-to-end vertical slice

- **Owner:** Backend/MLOps
- **Reviewer:** Thành viên không viết trainer
- **Milestone:** M1
- **Estimate:** 0.5 day
- **Depends on:** ML-02, ML-03, EVAL-01, EVAL-02
- **Acceptance criteria:**
  - Train → load → generate → score hoàn tất từ clean config.
  - Artifact contract pass.
  - Runtime/VRAM forecast cho 18 runs được cập nhật.

### ML-04 — Execute data-size sweep

- **Owner:** ML Lead
- **Reviewer:** Backend/MLOps
- **Milestone:** M3
- **Estimate:** GPU-dependent
- **Depends on:** M2, QA-01
- **Acceptance criteria:**
  - `n={1,3,5,10}`, `rank=16` cho ba concepts.
  - Các biến kiểm soát khớp protocol.
  - Failed run có issue/rerun decision riêng.

### ML-05 — Execute rank sweep

- **Owner:** ML Lead
- **Reviewer:** Backend/MLOps
- **Milestone:** M3
- **Estimate:** GPU-dependent
- **Depends on:** M2, QA-01
- **Acceptance criteria:**
  - `rank={4,16,32}`, `n=5` cho ba concepts.
  - Không chạy lại cell `n5-r16` đã hoàn tất.
  - Adapter size/time được ghi.

### EVAL-03 — Score and aggregate all main runs

- **Owner:** Evaluation Lead
- **Reviewer:** QA
- **Milestone:** M4
- **Estimate:** 1 day
- **Depends on:** ML-04, ML-05
- **Acceptance criteria:**
  - Per-sample and aggregate CSV generated.
  - Mean, SD, sample count và missing count có đủ.
  - Không coi generated images là independent subjects trong claims.

### EVAL-04 — Conduct blind human evaluation

- **Owner:** QA/Research Communication
- **Reviewer:** Evaluation Lead
- **Milestone:** M4
- **Estimate:** 1 day
- **Depends on:** ML-04, ML-05
- **Acceptance criteria:**
  - Configuration identity bị ẩn và order random.
  - Rating rubric cố định.
  - Tối thiểu 5 raters; target 8; ghi rõ rater/sample count.
  - Human-evaluation subset gồm 60 samples được lấy theo stratified protocol đã khóa.
  - Source ratings không bị sửa khi aggregate.

### EVAL-05 — Build failure taxonomy and examples

- **Owner:** QA + Evaluation Lead
- **Reviewer:** ML Lead
- **Milestone:** M4
- **Estimate:** 0.5 day
- **Depends on:** EVAL-03
- **Acceptance criteria:**
  - Underfit, identity drift, background leakage, pose copy, prompt refusal, memorization và artifacts được kiểm tra.
  - Mỗi representative case có run/prompt/seed ID.
  - Base-model limitation được phân biệt với LoRA failure.

### FE-01 — Build local result explorer

- **Owner:** FE/Data Visualization
- **Reviewer:** TPM/BA
- **Milestone:** M5
- **Estimate:** 1 day
- **Depends on:** QA-01, EVAL-03
- **Acceptance criteria:**
  - Browse concept/run/config.
  - Load existing adapter và generate hoặc xem precomputed results.
  - Compare same prompt/seed; display provenance.
  - Training không chạy qua UI.

### REP-01 — Build charts and qualitative grids

- **Owner:** FE + Evaluation Lead
- **Reviewer:** TPM/QA
- **Milestone:** M5
- **Estimate:** 0.75 day
- **Depends on:** EVAL-03, EVAL-04
- **Acceptance criteria:**
  - Separate RQ1 and RQ2 plots.
  - Per-concept curves visible.
  - Same-prompt/same-seed grid; no cherry-picking default.
  - Figures cite source data/run IDs.

### DOC-01 — Write findings, limitations and report

- **Owner:** TPM/BA + QA
- **Reviewer:** ML Lead, Evaluation Lead
- **Milestone:** M5
- **Estimate:** 1.5 days
- **Depends on:** EVAL-03, EVAL-04, EVAL-05, REP-01
- **Acceptance criteria:**
  - RQ1/RQ2 answered separately.
  - Null results, confounds and claim boundary included.
  - Every main claim maps to evidence.

### QA-02 — Independent reproduction test

- **Owner:** Member not authoring ML-02
- **Reviewer:** Backend/MLOps
- **Milestone:** M6
- **Estimate:** 0.5 day + runtime
- **Depends on:** DOC-01
- **Acceptance criteria:**
  - Fresh environment or clean setup follows README.
  - One representative run completes.
  - Differences/nondeterminism recorded.

### DOC-02 — Final presentation and backup demo

- **Owner:** QA + all members
- **Reviewer:** TPM/BA
- **Milestone:** M6
- **Estimate:** 1 day
- **Depends on:** FE-01, DOC-01, QA-02
- **Acceptance criteria:**
  - Slides cover problem, protocol, results, failures and limitations.
  - Live demo and offline video/gallery both available.
  - Rehearsal includes expected technical questions.

## 4. P1 issues

| ID | Issue | Start condition | Priority order |
|---|---|---|---:|
| EXP-01 | Repeat anchor configs with second training seed | M3 passes early with GPU budget | 1 |
| EXP-02 | Secondary subset at n=3/n=5 for one concept | Core metrics show strong subset sensitivity or time remains | 2 |
| EXP-03 | LoRA scale `{0.5,0.8,1.0}` on preselected adapter | M4 evidence complete | 3 |
| EXP-04 | Add rank 8 | Replication already complete | 4 |
| FE-02 | Full live-generation controls and export | FE-01 stable, M5 on track | 5 |

Không bắt đầu P1 nếu còn P0 blocker.

## 5. Dependency summary

```text
PROD-01
 ├─ OPS-01 ─ BE-01 ─ ML-02 ─ ML-03 ─ QA-01 ─ ML-04/ML-05
 └─ DATA-01 ─ DATA-02 ─ DATA-03 ──────────────┘
              └─ DATA-04 ─────────────────────┘

ML-04/ML-05
 ├─ EVAL-03 ─ EVAL-05 ─┐
 ├─ EVAL-04 ───────────┼─ REP-01 ─ DOC-01 ─ QA-02 ─ DOC-02
 └─────────────────────┘
                 EVAL-03 ─ FE-01 ────────────────┘
```
