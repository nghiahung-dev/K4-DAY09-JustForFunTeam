# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT; `03_cvat_labels.json` phải khớp với bảng này.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_light` | Rectangle | Class | n/a | n/a | No | Đối tượng cần phát hiện và theo dõi bằng một box cho mỗi đầu đèn vật lý. |
| `state` | Rectangle | Attribute (`select`) | `__undefined__`, `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | Yes | Trạng thái có thể đổi theo frame; `unknown` dùng khi không đủ bằng chứng. |
| `relevance` | Rectangle | Attribute (`select`) | `__undefined__`, `relevant`, `not_relevant`, `unknown` | `__undefined__` | No | Mức ảnh hưởng đến hướng/làn của xe, cố định cho toàn track. |
| `pictogram` | Rectangle | Attribute (`select`) | `__undefined__`, `circle`, `arrow_left`, `arrow_straight`, `arrow_right`, `other`, `unknown` | `__undefined__` | No | Hình dạng là đặc tính của đầu đèn và cố định cho toàn track. |
| `needs_review` | Rectangle | Attribute (`checkbox`) | `false` (bỏ chọn) / `true` (chọn) | `false` | Yes | Đánh dấu frame/object cần người phụ trách kiểm tra. CVAT khai báo checkbox bằng `values: ["false"]`. |

## Class hay attribute

`traffic_light` là class duy nhất vì mọi instance có cùng geometry và quy tắc QA. `state`, `relevance` và `pictogram` là attributes để tránh tạo nhiều class tổ hợp; chỉ `state` thay đổi theo thời gian. `needs_review` là checkbox phục vụ escalation, không phải một loại object.

Các select dùng `__undefined__` làm default để buộc annotator chọn. `__undefined__` còn trong export nghĩa là chưa hoàn thành, khác với `unknown` là quyết định hợp lệ khi đã xem nhưng thiếu bằng chứng. Cách này tránh default như `green` hoặc `relevant` tạo nhãn sai im lặng.

## CVAT

- **Phiên bản CVAT:** `2.76.1` tại `http://localhost:8080` (đã kiểm tra bằng `make cvat-status`).
- **Tên task calibration dự kiến:** `just-for-fun-calib-v1`.
- **Guide của task:** dán toàn bộ `02_guideline.md` khi tạo task; chưa xác nhận trên CVAT.
- **Công cụ:** dùng **Track** với rectangle vì cùng một đầu đèn phải giữ chung ID qua các frame, hỗ trợ keyframe, thuộc tính mutable và `outside`. Không dùng Shape rời.

## Setup test

Đã test sơ bộ bởi Nguyễn Nghĩa Hùng trên task calibration dựng sẵn với 5 sample đầu của clip LISA. Người test xác nhận được:

- class `traffic_light` xuất hiện đúng
- Rectangle + Track là cách dùng chính
- attributes `state`, `relevance`, `pictogram`, `needs_review` hiện ra như mong đợi
- `unknown` nên dùng khi trời mờ/che và không đủ bằng chứng
- `outside` phải đặt ở frame đầu tiên mất hẳn

Vấn đề gặp phải trong lần test đầu:

- `__undefined__` dễ bị quên và gây label chưa hoàn tất
- trên một số frame đèn bị che ngắn, team lúc đầu muốn đoán màu; sau đó đã chỉnh thành `unknown`
- `relevance` ban đầu dễ gán theo cảm giác, cần phải dựa vào làn/đường đi của xe

Cần công việc tiếp theo: tạo sample_pack rõ ràng, thêm edge cases và chạy calibration nội bộ trước khi freeze.
