# Annotation guideline — Traffic light state + ego relevance

**Version:** v1

<!--
Tăng lên v2 sau calibration và v3 sau blind handoff; mỗi lần tăng version phải cập nhật 08_revision_log.md.
Peer chỉ nhận file này, vì vậy mọi rule cần thiết phải được viết tại đây. Không dùng ảnh blind làm ví dụ.
-->

## 1. Objective + scope

Gán nhãn từng đầu đèn giao thông trong dữ liệu CVAT để xác định:

- `state`: màu/trạng thái ở từng frame;
- `pictogram`: hình dạng đầu đèn;
- `relevance`: mức ảnh hưởng đến quyết định dừng/đi của xe.

Label mọi đầu đèn giao thông có thể nhận diện trong clip LISA và ảnh calibration/example, kể cả đèn không liên quan đến xe. Không label cột, giá treo, biển báo, vạch kẻ, phản chiếu, vùng nền hoặc vật không rõ là đèn giao thông.

## 2. Annotation unit

- Một đầu đèn vật lý = một rectangle track duy nhất trong toàn bộ video.
- Hai đầu đèn tách biệt về vị trí hoặc nguồn tín hiệu = hai track.
- Khi không chắc có phải cùng đèn, kiểm tra vị trí, cấu trúc và pictogram trước khi tạo track mới.
- Nếu đèn xuất hiện lại sau khi mất, tiếp tục track cũ; không tạo track trùng.

## 3. Geometry rule

- Chỉ dùng `rectangle`; box ôm sát vỏ đèn, không chứa cột, giá treo hoặc đầu đèn khác.
- Điều chỉnh keyframe nếu box lệch rõ do chuyển động camera.
- Đặt `outside` tại frame đầu tiên đèn biến mất hoàn toàn hoặc bị che hẳn. Khi đèn xuất hiện lại, tiếp tục cùng track từ frame đó.

## 4. Taxonomy

Class: `traffic_light` với geometry `rectangle`.

| Attribute | Allowed values | Default | Mutable |
|---|---|---|---|
| `state` | `__undefined__`, `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | Có, theo frame |
| `pictogram` | `__undefined__`, `circle`, `arrow_left`, `arrow_straight`, `arrow_right`, `other`, `unknown` | `__undefined__` | Không |
| `relevance` | `__undefined__`, `relevant`, `not_relevant`, `unknown` | `__undefined__` | Không |
| `needs_review` | checkbox: bỏ chọn `false`, chọn `true` | `false` | Có |

`__undefined__` nghĩa là chưa gán và không hợp lệ khi hoàn tất annotation; dùng `unknown` khi đã xem nhưng thiếu bằng chứng. CVAT khai báo checkbox bằng `values: ["false"]`, nhưng annotator có thể bật thành `true`. Các giá trị phải khớp với `03_ontology_and_cvat_setup.md` và CVAT schema.

## 5. Inclusion / exclusion

- Label đèn tròn, đèn mũi tên và đèn xa nếu vẫn nhận diện được là đầu đèn giao thông.
- Không label vật quá xa/mờ hoặc bị che đến mức không xác định được là đèn giao thông.
- `relevant`: áp dụng cho làn/hướng của xe và có thể ảnh hưởng trực tiếp đến dừng/đi.
- `not_relevant`: dành cho làn/hướng khác, người đi bộ hoặc giao lộ khác.
- `unknown`: không đủ bằng chứng xác định relevance.

## 6. Visibility / occlusion

- Nếu vẫn nhận diện được đầu đèn, tiếp tục label; đặt thuộc tính không nhìn rõ thành `unknown`.
- Không suy đoán màu từ frame trước/sau khi frame hiện tại không có bằng chứng trực quan.
- Dùng `outside` nếu đèn bị che hoàn toàn. Loá, phản chiếu hoặc mờ không phải bằng chứng để gán màu.

## 7. Ambiguity / escalation

1. Chắc chắn là đèn nhưng không rõ thuộc tính: label và dùng `unknown` cho thuộc tính đó.
2. Có nhiều cách hiểu hợp lý hoặc cần người kiểm tra: đặt `needs_review=true`.
3. Không đủ bằng chứng object là đèn giao thông: không label.

Mọi quyết định phải thể hiện trong CVAT; không dùng quy tắc chỉ truyền đạt bằng miệng.

## 8. Temporal rule

- Giữ track liên tục khi đèn còn xuất hiện.
- Đổi `state` tại đúng frame có bằng chứng đổi màu; tạo/chỉnh keyframe và box nếu cần.
- Giữ `pictogram` và `relevance` cố định cho toàn track.
- Nếu đèn biến mất hoàn toàn dù chỉ trong thời gian ngắn, dùng `outside`; khi vẫn thấy đèn nhưng không rõ màu, dùng `state=unknown`.

## 9. Examples

| Trường hợp | Expected output |
|---|---|
| Đèn tròn đỏ cho làn xe | `pictogram=circle`, `state=red`, `relevance=relevant` |
| Mũi tên trái khi xe đi thẳng | `pictogram=arrow_left`, `relevance=not_relevant` |
| Đèn mất khỏi frame 12 | `outside` tại frame 12 |
| Vẫn thấy đèn nhưng màu bị che/mờ | `state=unknown`; thêm `needs_review=true` nếu cần |
| Đèn xa nhưng nhận diện được | Vẫn label; dùng `unknown` cho thuộc tính không rõ |
| Đèn cho người đi bộ | `relevance=not_relevant`, hoặc `unknown` nếu thiếu ngữ cảnh |

## 10. Common mistakes

- Dùng Shape hoặc tạo nhiều track cho cùng một đèn: luôn dùng một Track.
- Box chứa cột/giá treo/nhiều đèn: thu box sát một đầu đèn.
- Đoán `state`, `pictogram` hoặc `relevance`: dùng `unknown` khi thiếu bằng chứng.
- Quên cập nhật `state` khi đổi màu hoặc quên `outside` khi đèn mất.
- Dùng `not_relevant` khi chưa đủ ngữ cảnh: dùng `unknown` và `needs_review=true` nếu cần.
