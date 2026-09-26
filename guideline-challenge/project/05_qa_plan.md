# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:** QA owner (Đỗ Lý Minh Hải) review toàn bộ các sample critical và 20% sample còn lại; CVAT owner (Lê Trung Toán) review tất cả cases có `outside` và mọi frame đổi màu; mỗi annotator tự self-QC 100% track.
- **Chọn sample theo rule nào** (random, theo tag rủi ro, theo annotator mới…): review theo tag rủi ro trước (`critical`, `edge`, `ambiguity`, `occlusion`, `small_far`), sau đó random 20% sample còn lại để tránh bias.
- **Issue được ghi ở đâu, đóng thế nào:** issue ghi vào `06_calibration_report.csv` hoặc `07_blind_handoff/clarification_log.csv`; đóng bằng `accept + revise`, `reject with evidence` hoặc `add escalation rule` tùy mức độ.
- **Khi phát hiện guideline gap thì update và version ra sao:** nếu bug là do rule thiếu/ambiguous, chỉnh `02_guideline.md`, cập nhật version từ v1 → v2 (sau calibration) hoặc v2 → v3 (sau blind handoff), và ghi vào `08_revision_log.md`.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Sai `state` hoặc `relevance` trực tiếp ảnh hưởng đến quyết định dừng/đi của xe | đèn đỏ/đèn xanh nhầm trong frame quan trọng, `relevance` sai cho làn xe | Rework ngay; không được release |
| Major | Vị trí box sai hoặc track bị đứt trong khoảng dài, gây lệch hầu hết người dùng | box chứa cột, track tách thành 2 track, mất `outside` | Rework + check lại toàn bộ track |
| Minor | Vấn đề không làm sai quyết định chính nhưng làm mẫu không nhất quán | pictogram sai một trường hợp nhỏ, `unknown` dùng không đồng nhất | Fix trong vòng review ngắn |
| Question | Không đủ bằng chứng để kết luận, cần escalade | `relevance` không chắc, che kín nhiều frame nhưng không đủ dữ kiện | `needs_review=true` và quyết định theo guideline |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Critical defect rate | số lỗi Critical / tổng track review | downstream contract có hậu quả lớn nhất |
| Relevance mismatch rate | số track `relevance` sai / tổng track có relevance | trực tiếp ảnh hưởng đến dừng/đi |
| Outside omission rate | số track thiếu `outside` / tổng track kết thúc sớm | tránh box “ma” kéo dài đến cuối video |
| Temporal state consistency | số frame state nhảy vô lý / tổng frame có đèn | đảm bảo sự nhất quán về thời gian |
| Unknown usage rate | số frame dùng `unknown` / tổng frame có thể xác định | dùng để phát hiện over-guessing |

Metric high-risk tách riêng: critical defect escape rate = số lỗi Critical thoát qua QA / tổng track được release; mục tiêu là 0 trong sản phẩm final.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  - 0 Critical defect
  - 0 track thiếu outside khi đèn biến mất
  - 100% track có `relevance` được gán hoặc `unknown` rõ ràng
  - <= 2% frame state nhảy vô lý
  - tất cả issue critical/major đã được fix hoặc đã có escalation rule rõ
REWORK if:
  - có 1-2 Major defect hoặc 3-5 Minor defect
  - có các sample rủi ro chưa được review
REJECT / ESCALATE if:
  - phát hiện 1 Critical defect chưa xử lý
  - guideline không đủ để quyết định ở 1 edge case critical
  - có conflict giữa peer và owner không giải quyết bằng bằng chứng rõ
```

Trade-off: Với traffic light, cost của một lỗi Critical cao hơn rất nhiều so với thời gian review thêm, nên nhóm ưu tiên review các sample có chuyển màu, đèn bị che và đèn mũi tên. Dù vậy, chúng ta vẫn giữ giới hạn hợp lý để không mất quá nhiều thời gian trên những sample dễ.
