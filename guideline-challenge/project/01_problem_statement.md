# Problem statement + downstream contract

## Bài toán

Gán nhãn và theo dõi từng đầu đèn giao thông trong chuỗi frame để xác định trạng thái, hình dạng và mức độ liên quan với hướng di chuyển của xe. Khó khăn chính là đèn nhỏ/xa, bị che, mờ hoặc loá và có thể đổi trạng thái theo thời gian.

## Downstream contract

- **Người dùng:** hệ thống hỗ trợ lái xe cần nhận biết tín hiệu ảnh hưởng trực tiếp đến quyết định dừng/đi.
- **Đầu ra:** một rectangle track cho mỗi đầu đèn, class `traffic_light`; attributes `state`, `pictogram`, `relevance`, `needs_review` và trạng thái `outside`.
- **Lỗi nghiêm trọng nhất:** gán sai `state` hoặc `relevance` của đèn áp dụng cho xe, dẫn đến quyết định dừng/đi sai.
- **Escalation:** dùng `unknown` khi thiếu bằng chứng và bật `needs_review=true` khi cần người phụ trách review.

## Scope

- **Trong scope:** các đầu đèn giao thông nhìn thấy đủ rõ trong clip LISA và ảnh calibration/example của CVAT; mỗi đầu đèn là một track liên tục.
- **Ngoài scope:** cột, giá treo, biển báo, phản chiếu, vật không phải tín hiệu giao thông và đèn quá xa/không đủ bằng chứng để xác định là đầu đèn.
- **Geometry:** box ôm sát vỏ đèn, không chứa cột/giá treo hoặc nhiều đầu đèn; đặt `outside` tại frame đầu tiên đèn biến mất hoàn toàn.

## Output chấm được

CVAT export phải thể hiện được: `LABEL` bằng track/box và các attributes hợp lệ; `IGNORE` bằng việc không tạo annotation; `UNKNOWN` bằng giá trị `unknown`; `ESCALATE` bằng `needs_review=true`. `state` thay đổi theo frame, còn `pictogram` và `relevance` giữ cố định theo track.

## Dữ liệu và giới hạn

Dữ liệu gồm clip LISA và ảnh calibration/example được đưa vào CVAT. Số lượng frame/ảnh chưa được guideline xác định; các trường hợp che khuất, mờ, loá hoặc thiếu bằng chứng phải dùng `unknown`, không suy đoán.
