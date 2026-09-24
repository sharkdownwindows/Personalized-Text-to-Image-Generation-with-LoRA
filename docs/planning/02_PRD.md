# Product Requirements Document

## 1. Product definition

**Tên:** Few-Shot Personalized Text-to-Image Generation with LoRA  
**Loại:** Coursework experiment system  
**Thời gian:** 14 ngày  
**Đội:** 6 thành viên  
**Trạng thái:** Scope baseline v1.0

Hệ thống cho phép nhóm cá nhân hóa một pretrained text-to-image model bằng LoRA từ một tập ảnh tham chiếu nhỏ, sinh ảnh theo bộ prompt kiểm soát và so sánh ảnh hưởng của data size và LoRA rank.

## 2. Problem statement

Nhóm cần đáp ứng hai yêu cầu:

1. Adapt một pretrained text-to-image model để sinh ảnh mới của một visual concept cụ thể từ ít ảnh tham chiếu.
2. Điều tra ảnh hưởng của lượng dữ liệu và LoRA settings đến chất lượng và tính nhất quán của ảnh sinh.

Các thử nghiệm tùy ý, thiếu kiểm soát hoặc chỉ trình bày ảnh đẹp không đủ trả lời yêu cầu. Sản phẩm phải đảm bảo cấu hình so sánh công bằng, artifact truy vết được và kết luận giới hạn đúng theo bằng chứng.

## 3. Product goals

- Hoàn thành pipeline từ dataset đến báo cáo mà một thành viên khác có thể tái chạy.
- Trả lời RQ1 và RQ2 bằng cùng một evaluation protocol.
- Cho phép trình diễn trực quan base model và các LoRA configurations bằng cùng prompt/seed.
- Tạo tài liệu, bảng, biểu đồ và failure cases phù hợp cho báo cáo môn học.

## 4. Non-goals

- Không phát triển foundation model hoặc phương pháp personalization mới.
- Không xây hệ thống SaaS, multi-user hoặc training platform qua web.
- Không tối ưu cho production latency, throughput hoặc high availability.
- Không so sánh nhiều backbone trong core scope.
- Không tuyên bố state of the art hoặc tính tổng quát ngoài thí nghiệm.

## 5. Users and stakeholders

| Persona | Nhu cầu |
|---|---|
| Nhóm nghiên cứu sinh viên | Cấu hình, chạy, theo dõi và tái lập experiment |
| Người phụ trách evaluation | Sinh batch đồng nhất, tính metric và kiểm tra failure cases |
| Giảng viên/người chấm | Hiểu câu hỏi, phương pháp, bằng chứng và giới hạn |
| Người thuyết trình | Demo nhanh, ổn định và đối chiếu cấu hình công bằng |

## 6. Research protocol encoded as product scope

### 6.1 Core matrix

| Sweep | Data size | Rank | Unique configurations/concept |
|---|---|---|---:|
| Data size | 1, 3, 5, 10 | 16 | 4 |
| Rank | 5 | 4, 16, 32 | 3 |

`n=5, rank=16` trùng giữa hai sweep. Tổng cộng 6 configurations/concept và 18 core training runs cho 3 concept. Rank 8 là P1 nếu còn compute.

### 6.2 Evaluation set

- 8 prompt IDs/concept, bao phủ: simple, new background, viewpoint/pose, style và challenging composition.
- 4 fixed generation seeds/prompt/run.
- 3 held-out reference images/concept, không dùng để training.
- Cùng sampler, inference steps, guidance scale, resolution và negative prompt nếu có.
- Base-model output dùng cùng prompt và seed.

## 7. Functional requirements

| ID | Requirement | Priority | Acceptance criterion |
|---|---|---|---|
| FR-01 | Quản lý concept manifest, dataset version và subset 1/3/5/10 | P0 | Mỗi subset có danh sách file cố định; validator phát hiện file thiếu/trùng/hỏng |
| FR-02 | Khai báo experiment bằng config | P0 | Config chứa run ID, concept, dataset version, rank, alpha, steps, LR và seed |
| FR-03 | Train DreamBooth-LoRA bằng CLI | P0 | Một command/config tạo adapter, log, checkpoint và resolved config |
| FR-04 | Lưu checkpoint trung gian | P0 | Có checkpoint tại các mốc được cấu hình và không ghi đè artifact cũ |
| FR-05 | Sinh batch đánh giá tự động | P0 | Mỗi run sinh đủ prompt × seed và kèm metadata |
| FR-06 | Tính DINOv2 và CLIP metrics | P0 | Xuất được per-image records và aggregate theo run/concept |
| FR-07 | Tạo bảng, biểu đồ và qualitative grid | P0 | Có plot cho data size và rank; grid dùng cùng prompt/seed |
| FR-08 | Ghi human ratings mù | P0 | Người đánh giá không thấy tên configuration; dữ liệu lưu theo anonymized sample ID |
| FR-09 | Demo load adapter và generate | P1 | Chọn run, prompt, seed, LoRA scale; sinh và hiển thị metadata |
| FR-10 | Xem kết quả so sánh trong demo | P1 | Lọc theo concept/variable/metric và xem chart/grid |
| FR-11 | Export CSV và figures | P1 | Export không thay đổi source metrics |
| FR-12 | Launch training từ UI | Out | Training chỉ chạy bằng CLI để tránh timeout và state phức tạp |

## 8. Non-functional requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-01 | Reproducibility | Pin environment, base-model revision, config, dataset hash và seeds |
| NFR-02 | Traceability | 100% generated samples có metadata liên kết tới run và prompt |
| NFR-03 | Reliability | Pipeline fail fast khi thiếu model, dataset, config hoặc GPU memory |
| NFR-04 | Usability | Người mới trong nhóm chạy được inference từ README trong ≤15 phút sau khi môi trường sẵn sàng |
| NFR-05 | Performance | Demo xử lý một inference request tại một thời điểm; không yêu cầu real-time SLA |
| NFR-06 | Maintainability | Một nguồn config; không hard-code path/hyperparameter trong notebook |
| NFR-07 | Privacy | Không dùng ảnh người không có đồng ý; không public raw data/checkpoint mặc định |
| NFR-08 | Safety | Không dùng prompt bị cấm; giữ safety checker nếu tương thích pipeline |
| NFR-09 | Portability | Local-first trên NVIDIA CUDA; paths tương đối từ repository root |
| NFR-10 | Auditability | Không xóa run thất bại; ghi trạng thái và nguyên nhân |

## 9. User stories

### US-01 — Chuẩn bị concept dataset

**As a** data owner, **I want** tạo dataset manifest và các subset bất biến **so that** mọi run dùng đúng dữ liệu đã định trước.

Acceptance criteria:

- Given một thư mục ảnh, when chạy validator, then hệ thống báo count, format, duplicate và missing file.
- Mỗi concept có `train_pool`, `eval_refs`, class noun và caption template.
- Dataset version không thay đổi sau protocol freeze; thay đổi tạo version mới.

### US-02 — Khai báo experiment

**As an** experiment owner, **I want** mô tả run bằng YAML **so that** không phải sửa source code giữa các cấu hình.

Acceptance criteria:

- Config schema kiểm tra `rank > 0`, `alpha > 0`, subset hợp lệ và run ID duy nhất.
- Resolved config được copy vào artifact directory trước khi train.
- `alpha = rank` trong rank sweep, trừ khi protocol được version hóa lại.

### US-03 — Train và resume LoRA

**As an** ML engineer, **I want** train LoRA và lưu checkpoint/log **so that** run có thể tiếp tục hoặc được audit.

Acceptance criteria:

- Adapter mở lại được bằng base-model revision đã ghi.
- Checkpoint và final adapter không ghi đè run khác.
- OOM hoặc interruption tạo failed status và error log, không tạo nhãn “complete”.

### US-04 — Sinh evaluation batch

**As an** evaluator, **I want** sinh cùng prompt và seed cho mọi run **so that** comparison công bằng.

Acceptance criteria:

- Evaluation prompt registry là read-only trong core evaluation.
- Tên file hoặc metadata chứa run ID, prompt ID và seed.
- Batch generator phát hiện output thiếu trước khi đánh giá.

### US-05 — Tính automated metrics

**As an** evaluator, **I want** tính subject fidelity và prompt alignment **so that** có bằng chứng định lượng cho RQ1/RQ2.

Acceptance criteria:

- DINOv2 dùng held-out references cố định; subject-crop audit chỉ thực hiện trên một subset nhỏ nếu khả thi.
- CLIP text thay trigger token bằng class noun trước khi encode.
- Per-image metrics được lưu trước aggregation.
- Báo cáo mean, standard deviation và sample count.

### US-06 — Human evaluation mù

**As a** reviewer, **I want** đánh giá ảnh mà không thấy configuration **so that** giảm thiên kiến.

Acceptance criteria:

- Thứ tự ảnh/config được random hóa.
- Thu ba rating: concept fidelity, prompt alignment, visual quality.
- Có failure tags và optional comment.
- Báo cáo số người chấm, số mẫu và protocol.

### US-07 — So sánh kết quả

**As a** student researcher, **I want** xem chart và qualitative grid **so that** trả lời trực tiếp từng research question.

Acceptance criteria:

- Chart RQ1 có data size trên trục x và hiển thị từng concept.
- Chart RQ2 có rank trên trục x và hiển thị adapter size/training time bên cạnh quality metrics.
- Qualitative grid dùng cùng prompt và seed.
- Outlier và failed run được hiển thị, không âm thầm bỏ.

### US-08 — Demo generation

**As a** presenter, **I want** load một adapter đã train và sinh ảnh **so that** minh họa kết quả mà không chạy training trực tiếp.

Acceptance criteria:

- UI hiển thị run ID, config, prompt, seed và LoRA scale.
- Generate button bị khóa khi GPU đang xử lý.
- Lỗi thiếu checkpoint hoặc CUDA OOM được hiển thị rõ.
- Có base-vs-LoRA comparison dùng cùng seed.

### US-09 — Reproduce một run

**As a** team member, **I want** tái chạy một experiment từ repository **so that** kiểm chứng reproducibility.

Acceptance criteria:

- README mô tả environment, data preparation, train, generate và evaluate.
- Một người không phải tác giả pipeline tái chạy thành công vertical slice.
- Sai khác môi trường hoặc model revision được ghi trong reproduction log.

## 10. Success criteria and metrics

### Delivery success

- Vertical slice hoàn thành trước cuối ngày 3.
- Core experiment matrix gồm 18 runs hoàn thành trước cuối ngày 8.
- 100% artifact hoàn chỉnh có resolved config và metadata.
- Có ít nhất một biểu đồ và một qualitative comparison cho mỗi RQ.
- Final report nêu assumptions, confounds, failures và scope of claim.

### Model and experiment metrics

- DINOv2 subject similarity: mean ± SD.
- CLIP prompt alignment: mean ± SD.
- Human concept fidelity, prompt alignment và visual quality.
- Optional pairwise LPIPS diversity.
- Adapter size, training duration, peak GPU memory nếu đo được.
- Missing-output rate và failed-run rate.

Model quality là kết quả nghiên cứu, không phải tiêu chí để che giấu null result.

## 11. Dependencies

- Một NVIDIA GPU được xác nhận qua Day-2 smoke test.
- Quyền tải base model và metric encoders.
- 39 ảnh hợp lệ: 13 ảnh × 3 concept.
- Python environment và storage đủ cho model cache, adapters và generated images.
- Người đánh giá độc lập cho human study; mục tiêu 8, tối thiểu 5 nếu ghi rõ hạn chế.

## 12. Risks and product decisions

| Risk | Probability | Impact | Decision |
|---|---|---|---|
| Scope creep | Cao | Cao | RQ1/RQ2 là P0; RQ3 và metric phụ là stretch |
| Web UI làm chậm pipeline | Trung bình | Cao | Gradio mỏng; không React/FastAPI/database |
| Data count bị confound bởi diversity | Cao | Trung bình | Dùng nested subsets, mô tả selection rule và ghi limitation |
| Một training seed không đo training variance | Cao | Trung bình | Không claim significance; repeat key configs nếu compute còn |
| Automated metric lệch human judgment | Trung bình | Cao | Dùng nhiều metric và human evaluation mù |
| Cherry-picking checkpoint/image | Trung bình | Cao | Checkpoint/prompt/seed protocol khóa trước evaluation |

## 13. Release gates

| Gate | Deadline | Pass condition |
|---|---|---|
| G1 — Scope freeze | Ngày 1 | RQ, concept, matrix, prompts và success criteria được duyệt |
| G2 — Vertical slice | Ngày 3 | Train → load → generate → metric chạy end-to-end |
| G3 — Experiment complete | Ngày 8 | Core runs và output coverage được xác nhận |
| G4 — Evidence freeze | Ngày 10 | Metrics, ratings, plots và failures được khóa |
| G5 — Final release | Ngày 14 | Docs, code, demo, report và backup demo được kiểm tra |
