# Project Overview and Technical Specification

## 1. Document purpose

Tài liệu này là nguồn thống nhất cho product scope, research protocol, functional/non-functional requirements, data, model, architecture, evaluation, reproducibility và risk controls của dự án.

## 2. Executive summary

Dự án xây dựng một pipeline thực nghiệm để:

1. cá nhân hóa Stable Diffusion v1.5 bằng DreamBooth-LoRA từ ít ảnh tham chiếu;
2. sinh ảnh mới của concept trong nhiều prompt và seed;
3. đánh giá ảnh hưởng của data size và LoRA rank;
4. trình bày kết quả bằng metric, human rating, biểu đồ và failure cases.

Quyết định kiến trúc cốt lõi:

- Training, batch generation và evaluation chạy offline bằng CLI/config.
- Gradio chỉ dùng để inference, duyệt và so sánh kết quả.
- Local filesystem là artifact store; không dùng database hoặc job queue.
- Một backbone, một training script, hai biến nghiên cứu chính.
- Core matrix gồm 18 runs. RQ phụ chỉ bắt đầu sau khi core hoàn tất.

## 3. Verified assumptions and constraints

### Facts

- Thời gian thực hiện: 14 ngày.
- Nhân sự: 6 thành viên.
- Đây là project môn học, không phải research publication hoặc production service.
- Yêu cầu bắt buộc gồm adaptation, image generation và khảo sát data/LoRA settings.

### Assumptions to confirm on Day 1

- Nhóm có ít nhất một NVIDIA CUDA GPU; VRAM và GPU-hour budget chưa được giả định trước khi pilot.
- Nhóm có quyền tải base model và frozen metric encoders.
- Dataset do nhóm tự chụp hoặc có quyền sử dụng.
- Đầu ra cần source code, report, slides và demo; UI không bắt buộc phải là frontend/backend tách biệt.

### Constraints

- Chỉ subject personalization; không trộn với style personalization.
- Không dùng khuôn mặt người trong core dataset.
- Không thay backbone sau protocol freeze.
- Không tối ưu riêng hyperparameter cho từng concept.
- Kết luận chỉ áp dụng cho model, concepts, prompts, seeds và settings đã thử.

## 4. Problem formulation

Cho pretrained generator \(f_\theta\), concept \(c\), tập ảnh tham chiếu nhỏ \(D_c\) và LoRA updates:

\[
W' = W + \frac{\alpha}{r}BA,
\]

dự án thay đổi:

- số ảnh \(n\);
- LoRA rank \(r\);

trong khi giữ các biến còn lại cố định. Ảnh sinh được đánh giá theo subject fidelity, prompt adherence, consistency, diversity, failure modes và efficiency.

### Research questions

- **RQ1:** Với ngân sách training update cố định, số ảnh tham chiếu ảnh hưởng thế nào đến subject fidelity, prompt adherence và consistency?
- **RQ2:** Với tập ảnh cố định, LoRA rank ảnh hưởng thế nào đến subject fidelity, prompt adherence, overfitting và adapter size?

### Hypotheses

- **H1:** Tăng từ 1 lên 3–5 ảnh cải thiện fidelity rõ hơn tăng từ 5 lên 10 ảnh.
- **H2:** Rank 4 có nguy cơ underfit; rank 32 có thể tăng fidelity nhưng cũng tăng overfitting hoặc giảm prompt adherence.
- **H3:** Consistency tốt phải đi cùng fidelity đủ cao; output-output similarity cao do mode collapse không phải kết quả tốt.

Các giả thuyết này không phải acceptance criteria và có thể bị bác bỏ.

## 5. Scope

### P0

- Stable Diffusion v1.5, pinned model revision.
- DreamBooth-LoRA trên UNet attention; text encoder frozen.
- 3 concepts, 18 main runs.
- Versioned manifests và nested subsets.
- Batch generation bằng prompt/seed khóa trước.
- DINOv2, CLIP, human rating và failure taxonomy.
- Charts, qualitative grids, report và local result explorer.

### P1

- Rank 8.
- Inference LoRA-scale sweep.
- Một training seed bổ sung cho anchor configurations.
- Secondary subset selection cho một concept.
- Live generation trong UI nếu chưa có.

### Out of scope

- Multiple backbones, full fine-tuning, text-encoder tuning.
- Multi-subject generation, LoRA fusion, hyperparameter search.
- Production API, authentication, database, cloud deployment.
- New model, loss, metric hoặc benchmark.

## 6. Functional requirements

| ID | Requirement |
|---|---|
| FR-01 | Validate image files, manifests, splits và subset membership |
| FR-02 | Train LoRA từ một resolved YAML config |
| FR-03 | Save checkpoints, final adapter, logs, environment và status theo run ID |
| FR-04 | Resume interrupted run từ checkpoint nếu trainer hỗ trợ ổn định |
| FR-05 | Batch-generate cùng evaluation prompts và seeds cho mọi run |
| FR-06 | Gắn machine-readable provenance với mọi generated sample |
| FR-07 | Tính DINOv2 subject fidelity và CLIP prompt adherence |
| FR-08 | Import blind human ratings và failure tags |
| FR-09 | Aggregate metrics theo concept, prompt, seed và configuration |
| FR-10 | Tạo charts, tables và same-prompt/same-seed qualitative grids |
| FR-11 | Load adapter trong demo, thay prompt/seed/LoRA scale và generate |
| FR-12 | Export results mà không thay đổi source records |

## 7. Non-functional requirements

| ID | Requirement | Verification |
|---|---|---|
| NFR-01 | Reproducible | Pin dependencies/model revision; lưu config, hashes và seeds |
| NFR-02 | Traceable | Truy từ image → prompt/seed → run → config → dataset version |
| NFR-03 | Comparable | Trong mỗi sweep chỉ thay đổi biến được nghiên cứu |
| NFR-04 | Recoverable | Run lỗi giữ log/status; không ghi đè run hoàn thành |
| NFR-05 | Maintainable | Training/generation/evaluation tách module; config không hard-code trong notebook |
| NFR-06 | Private by default | Không tự upload raw images, adapters hoặc ratings |
| NFR-07 | License-aware | Ghi nguồn dữ liệu, model license và dependency notices |
| NFR-08 | Usable | Reviewer tới comparison chính trong tối đa ba thao tác UI |
| NFR-09 | Resource-aware | Đo time/VRAM trong pilot; khóa scope theo budget thực tế |
| NFR-10 | Auditable | Failed/missing runs được báo riêng, không bị loại âm thầm |

## 8. Model selection

| Option | Advantage | Risk | Decision |
|---|---|---|---|
| Stable Diffusion v1.5 | Pipeline phổ biến, 512 px, chi phí thấp, phù hợp nhiều runs | Chất lượng nền thấp hơn model mới | **Chọn làm default** |
| SDXL | Chất lượng hình ảnh và prompt understanding tốt hơn | VRAM/time cao, hai text encoders, tăng rủi ro tiến độ | Chỉ chọn nếu Day-1 pilot thỏa gate |
| FLUX/modern DiT | Chất lượng nền mạnh | Nặng, workflow/compute phức tạp, không cần để trả lời đề bài | Loại khỏi core scope |

### SDXL gate

Chỉ thay SD v1.5 bằng SDXL trước scope freeze nếu:

1. không OOM trong vertical slice;
2. train → load → generate → metric chạy ổn định;
3. dự báo core matrix dùng dưới 40% GPU-hour budget;
4. ít nhất hai thành viên tái chạy được environment.

Nếu một điều kiện không đạt, khóa SD v1.5. Không đổi backbone giữa các runs.

## 9. Training method and baseline configuration

### Base model

- Model ID: `stable-diffusion-v1-5/stable-diffusion-v1-5`.
- Revision SHA phải được ghi sau lần download đầu tiên.
- License: CreativeML OpenRAIL-M; nhóm phải đọc và ghi notice trong repository.

### DreamBooth-LoRA

- LoRA targets: UNet attention projections `to_k`, `to_q`, `to_v`, `to_out.0`.
- Text encoder: frozen.
- Prior preservation: off trong protocol tối giản.
- `lora_alpha = rank`, do đó \(\alpha/r=1\).

### Baseline config

Config này được xác nhận hoặc điều chỉnh một lần trong pilot, sau đó khóa cho tất cả main runs:

```yaml
base_model: stable-diffusion-v1-5/stable-diffusion-v1-5
resolution: 512
train_batch_size: 1
gradient_accumulation_steps: 1
learning_rate: 1.0e-4
lr_scheduler: constant
lr_warmup_steps: 0
max_train_steps: 500
checkpointing_steps: 100
mixed_precision: fp16
train_text_encoder: false
lora_dropout: 0.0
center_crop: true
random_flip: false
prior_preservation: false
training_seed: 42
```

Cho phép dùng gradient checkpointing, xFormers hoặc 8-bit Adam nếu cần, nhưng lựa chọn phải giống nhau cho toàn bộ main runs và được ghi trong environment metadata.

### Base-model baseline

Base model được sinh với class prompt tương ứng và LoRA scale bằng 0. Baseline này cho biết mức personalization trước adaptation; nó không được kỳ vọng nhận đúng subject riêng biệt.

## 10. Data plan

### Concept selection

Ba concept không phải người:

1. một rigid object;
2. một plush/toy object;
3. một object có hình dạng, màu hoặc texture đặc trưng.

Tránh logo, nhân vật có bản quyền, ảnh cá nhân và vật thể quá giống class generic.

### Per-concept dataset

- 10 training-pool images.
- 3 held-out images, không dùng train hoặc checkpoint selection.
- Góc nhìn, khoảng cách và background có chủ đích.
- Subject đủ lớn, không bị che khuất; loại ảnh mờ và near-duplicate.

### Manifest schema

```text
file_path
sha256
concept_id
split: train_pool | heldout
subset_membership
caption
source
consent_or_license
```

### Nested subsets

\[
D_1 \subset D_3 \subset D_5 \subset D_{10}
\]

Subset được khóa trước khi xem kết quả. Selection rule ưu tiên tăng coverage góc nhìn theo thứ tự đã ghi. Đây là fixed-compute study: với cùng 500 steps, ảnh ở subset nhỏ được lặp nhiều hơn. Vì vậy claim hợp lệ là “ảnh hưởng của số ảnh dưới cùng ngân sách update”, không phải causal effect thuần của lượng thông tin.

### Preprocessing

- Sửa EXIF orientation và chuyển RGB.
- Manual square crop theo rule thống nhất; resize 512×512 cùng interpolation.
- Không random flip vì có thể phá đặc trưng bất đối xứng.
- Caption template: `a photo of <unique_token> <class_noun>`.
- Held-out references dùng background đơn giản, subject rõ và centered. Có thể lưu subject crop để audit nhưng không bắt buộc crop toàn bộ generated set.

## 11. Experiment design

### Sweep A — data size

| Variable | Values |
|---|---|
| Data size | 1, 3, 5, 10 |
| Rank | 16 |
| Other settings | Fixed |

### Sweep B — rank

| Variable | Values |
|---|---|
| Data size | 5 |
| Rank | 4, 16, 32 |
| Alpha | Equal to rank |
| Other settings | Fixed |

Một configuration trùng nhau: `n5-r16`. Tổng cộng:

\[
(4+3-1)\times 3 = 18\text{ main runs}.
\]

### Replication priority

Nếu còn GPU budget, thêm training seed cho ba anchors `n1-r16`, `n5-r16`, `n10-r16` trước khi thêm rank mới hoặc backbone mới. Training-seed replication có giá trị khoa học cao hơn rank 8.

## 12. Generation protocol

### Prompt bank

Mỗi concept có 8 prompt IDs:

- 2 simple/subject-only;
- 2 new-background;
- 2 viewpoint/action;
- 1 style;
- 1 challenging composition.

Prompt bank được khóa trước main evaluation và không dùng để chọn checkpoint theo kết quả cuối.

### Inference config

```yaml
width: 512
height: 512
num_inference_steps: 30
guidance_scale: 7.5
scheduler: fixed_for_all_runs
negative_prompt: null
generation_seeds: [11, 22, 33, 44]
lora_scale: 1.0
```

Tạo một `torch.Generator` mới cho từng sample. Same seed không bảo đảm bit-identical giữa mọi GPU/library version; hardware và dependencies phải được ghi.

### Sample metadata contract

```json
{
  "run_id": "toy01_n5_r16_ts42",
  "concept_id": "toy01",
  "dataset_version": "v1",
  "checkpoint_step": 500,
  "rank": 16,
  "alpha": 16,
  "prompt_id": "p04",
  "prompt": "...",
  "seed": 22,
  "lora_scale": 1.0,
  "base_model_revision": "<sha>",
  "created_at": "<ISO-8601>"
}
```

## 13. Evaluation plan

### 13.1 Subject fidelity

Dùng frozen `facebook/dinov2-base` embeddings giữa generated image và centroid của ba held-out references; pin model revision trong protocol. Với generated image \(g\) và centroid held-out \(\bar{h}\):

\[
S_{subject}(g)=\cos(\phi_{DINO}(g),\bar{h}).
\]

Báo cáo per-concept trước khi aggregate. Không dùng training image làm reference chính. DINO trên toàn ảnh có thể bị background/layout chi phối; vì vậy dùng held-out references có background đơn giản và audit một subset nhỏ bằng subject crops nếu thời gian cho phép. Đây là limitation bắt buộc, không được gọi DINO là ground-truth identity metric.

### 13.2 Prompt adherence

Dùng `openai/clip-vit-base-patch32` image-text cosine similarity và pin model revision. Trước text encoding, thay unique token bằng class noun để metric không bị token lạ chi phối. CLIP là proxy; relation/action nhỏ phải được kiểm tra bằng human rating.

### 13.3 Consistency

Cho mỗi run, báo cáo mean và standard deviation của subject fidelity qua prompt và seed. Standard deviation thấp chỉ tốt khi mean fidelity không thấp. Không dùng output-output similarity như metric duy nhất.

### 13.4 Diversity

Optional: mean pairwise LPIPS giữa các seed trong cùng prompt. Diễn giải cùng subject fidelity; diversity tăng do identity drift không phải cải thiện.

### 13.5 Overfitting and memorization

- Nearest-reference perceptual similarity.
- Manual tags: copied pose, copied background, copied crop, prompt refusal.
- So sánh checkpoint trung gian chỉ dùng như failure analysis hoặc theo selection rule khóa trước.

### 13.6 Efficiency

- Adapter file size.
- Trainable parameter count.
- Training wall time.
- Peak VRAM nếu có thể thu thập đáng tin cậy.

### 13.7 Human evaluation

- Blind, randomized configuration order.
- Hiển thị held-out references cạnh generated output.
- Rating 1–5: subject fidelity, prompt adherence, visual quality/artifacts.
- Tối thiểu 5 independent raters; mục tiêu 8 người, ưu tiên ngoài nhóm.
- Chấm năm cấu hình đại diện: `n1-r16`, `n5-r16`, `n10-r16`, `n5-r4`, `n5-r32`.
- Dùng stratified subset 60 samples: 3 concepts × 5 configs × 4 prompt categories × 1 fixed seed. Mỗi rater chấm toàn bộ hoặc được phân block cân bằng.

### 13.8 Analysis rules

- Generated images không được coi là independent subjects để tuyên bố statistical significance.
- Với 3 concepts, báo cáo effect size mô tả, per-concept curves và failure cases.
- Không gộp mọi metric thành một weighted score duy nhất.
- Không dùng FID hoặc Inception Score.
- Missing samples được báo riêng, không gán score 0 và không loại âm thầm.

## 14. Success criteria and metrics

### Scientific success

- RQ1/RQ2 mỗi câu có chart, table, qualitative grid và limitations.
- Mọi comparison dùng cùng prompt/seed và protocol inference.
- Findings âm hoặc không khác biệt vẫn được báo.
- Claims không vượt quá phạm vi thí nghiệm.

### Engineering success

- Mục tiêu 18/18 valid main runs; release gate tối thiểu 17/18 nếu một run thất bại có nguyên nhân và không làm mất ô so sánh quan trọng.
- 100% dataset files có manifest/hash.
- 100% valid artifacts có config, environment, seed, log và adapter.
- 100% reported samples có provenance metadata.
- Một thành viên không viết trainer tái chạy thành công representative run.
- Demo load adapter đã train và generate mà không phụ thuộc live training.

### Metrics dashboard

- Run completion và missing-output rate.
- DINOv2 mean ± SD.
- CLIP mean ± SD.
- Human rating mean, distribution và rater count.
- LPIPS nếu P1.
- Train time, peak VRAM và adapter size.

Không đặt ngưỡng DINO/CLIP tuyệt đối trước khi có baseline calibration.

## 15. System architecture

```text
Dataset manifests ──┐
Experiment YAML ────┼─> Config validator ─> Training orchestrator
Base model cache ───┘                         │
                                              ├─> LoRA adapter/checkpoints
                                              ├─> resolved config/environment
                                              └─> logs/status
                                                       │
Prompt bank + seeds ─────────────────────────────> Batch generation
                                                       │
                                                       v
                              generated images + metadata JSONL
                                                       │
Held-out crops ──> DINO scorer ────────────────────────┤
Prompts ─────────> CLIP scorer ────────────────────────┤
Human ratings ─────────────────────────────────────────┤
                                                       v
                                             Metrics aggregation
                                                       │
                                    CSV + charts + qualitative grids
                                                       │
                                             Report + Gradio demo
```

### Components

| Component | Responsibility |
|---|---|
| Dataset registry | Manifests, hashes, split và subset definitions |
| Config validator | Schema, invariant và unique run ID checks |
| Training runner | Wrap official Diffusers DreamBooth-LoRA execution |
| Run registry | Status và artifact locations; file-based CSV/JSONL |
| Batch generator | Fixed prompts/seeds, metadata, completeness check |
| Evaluation engine | DINO, CLIP, optional LPIPS, aggregation |
| Reporting | Tables, plots và qualitative grids |
| Gradio explorer | Inference và result inspection; không training |

## 16. Data flow

1. Validate raw images and metadata.
2. Preprocess and freeze dataset version.
3. Resolve experiment config and create unique run directory.
4. Train LoRA and store adapter/logs/environment.
5. Generate fixed prompt × seed matrix.
6. Validate output coverage and metadata.
7. Score outputs; import blind human ratings.
8. Aggregate by run/concept/prompt/seed.
9. Export immutable source CSV plus derived tables/figures.
10. Use artifacts in report and demo.

## 17. UI scope

### Technology

Gradio Blocks + Plotly. Không dùng React/FastAPI riêng trừ khi rubric bắt buộc.

### Screens

- Dataset/experiment browser: reference images, subset, run config.
- Generate: existing adapter, prompt, seed, guidance và LoRA scale.
- Evaluate: optional blind rating form.
- Results: filters, KPI cards, plots và qualitative grid.

### UI constraints

- GPU inference concurrency bằng 1.
- Disable Generate khi request đang chạy.
- Hiển thị run ID/config với mỗi output.
- Không ghi đè ảnh cũ.
- Training launch từ UI là out of scope.
- Chuẩn bị gallery/video backup cho live demo.

## 18. Technology stack

| Area | Choice |
|---|---|
| Language | Python 3.10 hoặc 3.11, khóa sau pilot |
| Training/inference | PyTorch, Diffusers, Transformers, PEFT, Accelerate |
| Weights | Safetensors |
| Image/data | Pillow, NumPy, pandas |
| Metrics | DINOv2, CLIP, optional LPIPS |
| Visualization | Matplotlib/Seaborn hoặc Plotly |
| Demo | Gradio Blocks |
| Config | YAML + schema validation |
| Logging | JSONL/CSV và TensorBoard; W&amp;B chỉ là optional |
| Testing | Pytest; smoke, config và manifest tests |
| Version control | Git; large artifacts excluded hoặc managed separately |

Pin versions sau pilot; không phụ thuộc Diffusers `main` branch đang thay đổi. Docker không bắt buộc nếu nhóm chưa có CUDA image ổn định.

## 19. Reproducibility contract

Mỗi run phải lưu:

- Git commit.
- Resolved config.
- Dataset version và file hashes.
- Base-model ID và revision.
- Python/package/CUDA/driver/GPU information.
- Training seed và generation seeds.
- Adapter, checkpoints, logs và status.
- Prompt-bank version.
- Generated-image metadata.

Reproducibility target là hoàn thành lại pipeline và tái hiện xu hướng trong tolerance, không yêu cầu bitwise-identical output giữa phần cứng khác nhau.

## 20. Failure taxonomy

| Failure | Observable symptom | Possible cause | Required evidence |
|---|---|---|---|
| Underfitting | Subject generic, đặc điểm riêng mất | Ít ảnh/rank thấp/steps thiếu | Training curve, rank/data comparison |
| Identity drift | Subject thay đổi theo prompt/seed | Reference coverage yếu | DINO distribution và examples |
| Background leakage | Training background tái xuất hiện | Dataset bias | Nearest training image và prompt contrast |
| Pose copying | Pose ít thay đổi | Overfit/redundant images | Same-seed grid, nearest reference |
| Prompt refusal | Subject đúng nhưng context sai | Adaptation quá mạnh | CLIP/human adherence, rank comparison |
| Memorization | Output gần duplicate training image | Repetition/overtraining | Perceptual nearest neighbor |
| Artifacts | Anatomy/texture/edge lỗi | Base model hoặc training instability | Base-vs-LoRA same prompt/seed |
| Base-model limitation | Cả base và LoRA đều thất bại | Weak pretrained prior | Base-model control output |

## 21. Risks and fallback

| Risk | Trigger | Fallback |
|---|---|---|
| CUDA OOM | Day-1/2 pilot fails | SD v1.5, fp16, gradient checkpointing, xFormers, optional 8-bit Adam |
| Training exceeds budget | Forecast misses Day-8 gate | Giữ 3 concepts/18 runs; bỏ P1 và UI extras |
| GPU outage | >1 working day unavailable | Prioritize anchor configs; use precomputed demo artifacts |
| Dataset confound | Trend changes by selected images | Report exploratory; secondary subset only if budget permits |
| Metric mismatch | Automated and human scores disagree | Report disagreement; prioritize multi-view evidence, not one score |
| Single-person dependency | Only one person can run trainer | Reproduction handoff before Day 4 |
| Live demo failure | Model/cache/GPU unavailable | Offline gallery and recorded demo |

### Contingency floor

Nếu compute không đủ, dùng 2 concepts với cùng 6 configurations: 12 runs. Không giảm số conditions khác nhau giữa concept vì sẽ phá comparability.

## 22. Primary references

- [Hugging Face Diffusers — LoRA training](https://huggingface.co/docs/diffusers/training/lora)
- [Hugging Face Diffusers — DreamBooth](https://huggingface.co/docs/diffusers/main/training/dreambooth)
- [Stable Diffusion v1.5 model card](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-v1-5)
- [DreamBooth](https://arxiv.org/abs/2208.12242)
- [LoRA](https://arxiv.org/abs/2106.09685)
- [DINOv2](https://arxiv.org/abs/2304.07193)
- [CLIP](https://arxiv.org/abs/2103.00020)
- [DINOv2 Base model card](https://huggingface.co/facebook/dinov2-base)
- [CLIP ViT-B/32 model card](https://huggingface.co/openai/clip-vit-base-patch32)

## 23. Final technical decision

Giữ dự án là **reproducible experiment pipeline + inference/result explorer**. Đây là thiết kế nhỏ nhất đáp ứng đầy đủ yêu cầu, tạo bằng chứng đáng tin cậy và có xác suất hoàn thành cao trong 14 ngày.
