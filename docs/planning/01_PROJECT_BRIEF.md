# Project Brief — Few-Shot Personalized Text-to-Image Generation with LoRA

| Thuộc tính | Giá trị |
|---|---|
| Loại | Project môn học; nghiên cứu thực nghiệm |
| Thời gian / đội | 2 tuần / 6 thành viên |
| Backbone mặc định | Stable Diffusion v1.5 |
| Phương pháp | DreamBooth-LoRA; text encoder frozen |
| Sản phẩm | Pipeline tái lập, kết quả đánh giá, báo cáo và demo cục bộ |

## Problem

Pretrained text-to-image models có thể sinh một lớp đối tượng phổ biến nhưng không tái hiện chính xác một chủ thể cụ thể từ vài ảnh tham chiếu. Dự án dùng LoRA để cá nhân hóa mô hình, sinh chủ thể trong các bối cảnh mới và đo ảnh hưởng của dữ liệu cùng LoRA settings.

Đây không phải dự án đề xuất thuật toán mới hoặc hướng tới state of the art. Giá trị nằm ở so sánh có kiểm soát, khả năng truy vết và kết luận đúng với bằng chứng.

## Objective and research questions

Xây dựng một pipeline có thể train LoRA, sinh batch ảnh bằng prompt/seed cố định, tính metrics và tạo comparison artifacts để trả lời:

- **RQ1:** Với ngân sách training update cố định, số ảnh tham chiếu ảnh hưởng thế nào đến subject fidelity, prompt alignment và consistency?
- **RQ2:** Với tập ảnh cố định, LoRA rank ảnh hưởng thế nào đến subject fidelity, prompt alignment, overfitting và adapter size?

RQ3 về checkpoint hoặc inference LoRA scale chỉ là stretch goal.

## MVP scope

- 3 non-human concepts; mỗi concept có 10 ảnh train pool và 3 held-out references.
- Data-size sweep: `n={1,3,5,10}`, cố định `rank=16`.
- Rank sweep: `rank={4,16,32}`, cố định `n=5`, giữ `alpha/rank=1`.
- 6 unique configurations/concept; tổng cộng 18 main training runs.
- 8 evaluation prompts và 4 generation seeds/run.
- Base-model baseline không gắn LoRA.
- DINOv2 similarity, CLIP prompt similarity, blind human rating và failure analysis.
- Gradio result explorer/inference demo; training chạy bằng CLI.

## Out of scope

- Multiple backbones, multi-subject generation, style personalization và LoRA merging.
- Full-model fine-tuning, metric mới hoặc automatic hyperparameter search.
- Cloud deployment, database, accounts, job queue hoặc training qua UI.
- Tuyên bố kết quả tổng quát ngoài model, concepts và settings đã thử.

## Deliverables

1. Versioned dataset manifests, prompts, configs và source code.
2. LoRA adapters, checkpoints, logs và generated evaluation images.
3. Per-sample metrics, aggregate tables, plots và same-prompt/same-seed grids.
4. Human-rating records và representative failure cases.
5. Local demo, report, slides và offline demo backup.

## Success criteria

- Vertical slice `train → load → generate → metric` hoàn thành trước cuối Day 3.
- Main matrix hoàn thành trước Day 8; fallback tối thiểu 2 concepts/12 comparable runs nếu compute không đủ.
- Mọi reported sample truy được tới run, config, dataset version, prompt và seed.
- RQ1/RQ2 mỗi câu có chart, qualitative evidence, limitations và kết luận riêng.
- Một thành viên không viết trainer tái chạy được một representative run.
- Null result được chấp nhận; không đặt ngưỡng DINO/CLIP tùy ý làm điều kiện thành công.

## Critical risks and controls

| Rủi ro | Kiểm soát |
|---|---|
| GPU/OOM hoặc training chậm | Pilot Day 1–2; SD v1.5, fp16, memory optimizations; cắt P1 trước |
| Data count bị confound bởi lựa chọn ảnh | Nested subsets khóa trước; mô tả study là fixed-compute/exploratory |
| Metric bị ảnh hưởng bởi background | Held-out references rõ/centered; human review và crop audit nhỏ |
| Cherry-picking | Khóa prompt/seed/config; giữ failed runs và ảnh xấu |
| Scope creep | RQ1/RQ2 và 18 runs là P0; không thêm backbone/feature trước core gate |

## Verified recommendation

Giữ sản phẩm là **reproducible experiment pipeline + thin inference/result explorer**. Kiến trúc này đáp ứng đúng đề bài, phù hợp 14 ngày và dành phần lớn thời gian cho bằng chứng thay vì hạ tầng không cần thiết.
