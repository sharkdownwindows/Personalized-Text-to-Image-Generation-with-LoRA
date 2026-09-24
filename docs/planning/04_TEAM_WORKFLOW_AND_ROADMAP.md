# Team Workflow and 14-Day Roadmap

## 1. Operating model

Nhóm làm việc theo một experiment protocol chung, không chia thành sáu phần độc lập rồi ghép ở cuối. Mỗi artifact có một owner và một reviewer. Tối thiểu hai người phải chạy được pipeline chính.

### Vai trò

| Thành viên | Vai trò chính | Trách nhiệm | Backup/reviewer |
|---|---|---|---|
| 1 | Technical Product Manager / BA | Scope, RQ, backlog, decision log, milestone, report logic | Review UI/report |
| 2 | AI/ML Training Lead | Trainer, hyperparameters, core runs, checkpoint integrity | Thành viên 4 |
| 3 | Data & Evaluation Lead | Dataset, manifests, prompts, DINO/CLIP protocol | Thành viên 6 |
| 4 | Backend/MLOps Engineer | Config runner, artifact registry, batch generation, reproducibility | Thành viên 2 |
| 5 | Frontend/Data Visualization Engineer | Gradio explorer, charts, qualitative grids | Thành viên 1 |
| 6 | QA/Research Communication | Human evaluation, failure taxonomy, report/slides, demo backup | Thành viên 3 |

Vai trò là ownership, không phải silo. Thành viên 2/4 pair trên vertical slice; thành viên 3/6 pair trên evaluation; thành viên 1/5 pair trên evidence presentation.

## 2. Working agreements

1. RQ1/RQ2 và 18 core runs là P0.
2. Không thêm model, metric hoặc feature trước khi P0 đạt gate.
3. Thay đổi protocol phải có decision record và áp dụng nhất quán.
4. Không chỉnh config trực tiếp trong source code hoặc notebook.
5. Không xóa failed run, outlier hoặc ảnh xấu.
6. Không chọn riêng output đẹp để làm bằng chứng chính.
7. Mỗi conclusion phải trỏ tới table, figure hoặc run IDs.
8. Không để raw data, model credentials hoặc cách chạy pipeline chỉ nằm trên máy một người.

## 3. Communication cadence

### Daily sync — 15 phút

Mỗi người báo cáo đúng bốn mục:

```text
Done:
Next:
Blocker:
Decision needed:
```

Không debug chi tiết trong daily. Các thành viên liên quan tạo phiên pair-debug sau cuộc họp.

### Async end-of-day update

```text
Owner:
Completed artifacts/links:
Run IDs affected:
Open blocker:
Decision or review required:
Plan for next day:
```

### Blocker escalation

| Thời gian blocked | Hành động |
|---:|---|
| 2 giờ | Báo kênh nhóm và cập nhật issue |
| 4 giờ | Pair với backup/reviewer |
| 1 ngày | TPM cắt P1, đổi owner hoặc kích hoạt fallback |

## 4. Decision process

Khi có bất đồng:

1. Viết hai phương án và quyết định cần đưa ra.
2. So sánh theo correctness, deadline, compute, reproducibility và scope.
3. Nếu spike dưới một giờ có thể tạo bằng chứng, chạy spike.
4. Nếu chưa có bằng chứng, chọn phương án đơn giản và ít rủi ro hơn.
5. Ghi quyết định, ngày, owner, rationale và tác động vào `docs/decision_log.md`.

Không thay protocol sau Day-4 freeze trừ lỗi làm mất tính hợp lệ. Nếu bắt buộc thay, version protocol và rerun tất cả cells bị ảnh hưởng.

## 5. Artifact handoff

```text
Data/Evaluation Lead
  → dataset manifest + subsets + prompt bank
AI/ML Lead
  → adapters + checkpoints + training logs
Backend/MLOps
  → run registry + generated images + metadata
Evaluation Lead
  → per-sample metrics + aggregates + ratings
Frontend/Visualization
  → charts + grids + result explorer
TPM/QA
  → findings + limitations + report + slides
```

Mỗi handoff phải có artifact path, version, owner, completeness status và known issues.

## 6. Definition of Ready

Một issue chỉ được bắt đầu khi có:

- objective;
- input và dependency;
- acceptance criteria;
- owner và reviewer;
- expected artifact;
- milestone;
- estimate hoặc timebox.

## 7. Definition of Done

### Code issue

- Code đã review.
- Test hoặc smoke run đã pass.
- Không hard-code local path/secrets.
- README/config schema được cập nhật nếu interface thay đổi.

### Training run

- Status `completed`.
- Resolved config, environment, log, adapter và checkpoint tồn tại.
- Dataset version/hash và seeds được ghi.
- Adapter load lại và sinh được ít nhất một validation image.

### Evaluation run

- Output coverage được kiểm tra.
- Per-sample records và aggregate cùng tồn tại.
- Missing/invalid samples được liệt kê.
- Chart/grid có source run IDs.

### Documentation

- Nội dung khớp config và kết quả thực tế.
- Claim có evidence.
- Assumptions, confounds và limitations được ghi.
- Reviewer ngoài owner đã đọc.

## 8. Quality gates

| Gate | Deadline | Owner | Pass condition | Fallback nếu fail |
|---|---:|---|---|---|
| G0 Scope freeze | Day 1 | TPM/BA | RQ, scope, concepts, matrix, metrics, roles chốt | Cắt RQ3/P1 |
| G1 GPU/model pilot | Day 2 | AI/ML | Model load, one short train và one inference không OOM | Khóa SD v1.5 + memory optimizations |
| G2 Vertical slice | Day 3 | AI/ML + BE | Train → load → batch generate → one metric → artifact registry | Dừng UI; tập trung sửa pipeline |
| G3 Data/config freeze | Day 4 | Data + TPM | Manifests, subsets, prompts, seeds, baseline config khóa | Version lại trước main runs |
| G4 Core experiments | Day 8 | AI/ML | 18 target runs hoặc documented contingency | 2 concepts/12 comparable runs |
| G5 Evidence freeze | Day 10 | Eval + QA | Automated metrics, ratings, failure cases hoàn tất | Human eval chỉ trên anchor configs |
| G6 Release candidate | Day 12 | FE + TPM | Charts, grids, demo, report draft cùng một source data | Bỏ UI extras, dùng static explorer |
| G7 Submission ready | Day 14 | All | Reproduction, QA, rehearsal và backup demo pass | Nộp precomputed demo |

## 9. Day-by-day roadmap

### Day 1 — Scope and environment

- Chốt RQ1/RQ2, P0/P1, model candidates và GPU budget.
- Chọn ba concepts, data ownership và naming convention.
- Tạo repository, board, decision log và artifact contract.
- Owner: 1, 2, 3, 4.

### Day 2 — Pilot

- Chuẩn bị một concept mẫu và manifest.
- Chạy short train trên SD v1.5; chỉ thử SDXL nếu gate cho phép.
- Đo VRAM/time; khóa backbone.
- Tạo prompt bank draft và metric smoke test.
- Owner: 2, 3, 4.

### Day 3 — Vertical slice

- Hoàn thành one-config train → load → generate → score.
- Thành viên 4 tái chạy pipeline do thành viên 2 thiết lập.
- FE đọc artifact mẫu và dựng result-view skeleton.
- G2 review cuối ngày.

### Day 4 — Protocol freeze

- Freeze dataset v1, subsets, prompts, seeds và config baseline.
- Generate toàn bộ 18 configs.
- Không thêm biến nghiên cứu mới.

### Days 5–7 — Main experiments

- Chạy data-size và rank sweeps.
- Evaluation chạy ngay khi mỗi run hoàn tất.
- Track failed/missing cells hằng ngày.
- UI chỉ phát triển bằng mock/precomputed artifacts.

### Day 8 — Experiment completeness review

- Audit 18-cell matrix.
- Rerun lỗi do hệ thống; không rerun vì kết quả “không đẹp”.
- Kích hoạt contingency nếu compute thiếu.
- Freeze danh sách runs dùng trong report.

### Day 9 — Evaluation completion

- Hoàn thành DINO/CLIP, optional LPIPS và efficiency records.
- Tạo same-prompt/same-seed grids.
- Chuẩn bị blind human evaluation trên anchor configs.

### Day 10 — Evidence freeze

- Hoàn thành human ratings và failure taxonomy.
- Review metric disagreement và outliers.
- Freeze source metrics CSV/JSONL.

### Day 11 — Analysis and findings

- Vẽ final RQ1/RQ2 charts.
- Viết findings, counterexamples, limitations và scope of claim.
- UI đọc final artifacts.

### Day 12 — Release candidate

- Hoàn thiện report draft, demo, slides và appendix.
- Cross-review logic giữa findings và figures.
- Record backup demo.

### Day 13 — Reproduction and rehearsal

- Thành viên không viết trainer tái chạy representative config.
- Test demo offline/local.
- Rehearse presentation và Q&A.

### Day 14 — Buffer and submission

- Chỉ sửa lỗi/blocker.
- Kiểm tra file, link, attribution và archive.
- Nộp source, report, slides và demo backup.

## 10. Work-in-progress limits

- Mỗi người tối đa hai active issues.
- Chỉ một protocol version được dùng cho main runs.
- Không quá một UI feature được phát triển trước G4.
- Không chạy unregistered experiment khi GPU đang cần cho core matrix.

## 11. Review checklist

### Experiment fairness

- [ ] Chỉ một biến chính thay đổi trong sweep.
- [ ] `alpha/rank` cố định.
- [ ] Same prompts, seeds và inference config.
- [ ] Same training budget.
- [ ] Subsets khóa trước khi xem kết quả.

### Evidence integrity

- [ ] Không thiếu run/sample mà không giải thích.
- [ ] Không cherry-pick image/checkpoint.
- [ ] Per-concept results xuất hiện trước aggregate.
- [ ] Automated metrics được đối chiếu với human review.
- [ ] Failure cases và null results được giữ lại.

### Release

- [ ] README chạy được.
- [ ] Large/private artifacts không commit nhầm.
- [ ] Model/data licenses được ghi.
- [ ] Demo có offline backup.
- [ ] Báo cáo không claim vượt scope.
