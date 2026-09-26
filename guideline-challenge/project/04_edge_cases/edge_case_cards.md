# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

CASE ID: EC-01
Sample: LISA05 (sample_id)
Scene: signal light clear, lane straight visible
Observation: Đèn tròn đỏ rõ ở làn xe đi thẳng; không có che hay phản chiếu rõ.
Decision: LABEL
Expected: `traffic_light` track với `state=red`, `pictogram=circle`, `relevance=relevant`
Rationale: Là trường hợp cơ bản dùng cho downstream contract: hành vi dừng/đi phụ thuộc trực tiếp vào đèn của làn xe.
Common mistake: Gán `state` theo màu lốp đèn/đoán không cần kiểm tra.
Diversity: normal / critical

---

CASE ID: EC-02
Sample: LISA07 (sample_id)
Scene: mũi tên trái nhưng xe đang đi thẳng
Observation: Đèn mũi tên trái ở góc khác nhưng không thuộc hướng xe đang di chuyển.
Decision: LABEL
Expected: `pictogram=arrow_left`, `relevance=not_relevant`, `state` theo màu nhìn thấy
Rationale: `relevance` không dẫn theo màu đèn mà theo hướng/ làn xe.
Common mistake: Gán `relevant` chỉ vì đèn là đầu đèn giao thông.
Diversity: ambiguity / conflict

---

CASE ID: EC-03
Sample: LISA12 (sample_id)
Scene: che tạm thời ở mép khung
Observation: Đèn bị che một phần trong 2-3 frame, không đủ bằng chứng để biết state.
Decision: UNKNOWN
Expected: `state=unknown`; `needs_review=true` nếu cần review
Rationale: Mục tiêu không được đoán bừa khi không thấy rõ.
Common mistake: Gán đỏ/xanh dựa trên frame trước/sau mà không có bằng chứng 
Diversity: occlusion / ambiguity / escalation

---

CASE ID: EC-04
Sample: LISA14 (sample_id)
Scene: chuyển màu trong chuỗi frame
Observation: Màu đóng vai trò thay đổi từ đỏ sang vàng rồi xanh, phải cập nhật per-frame.
Decision: LABEL + temporal update
Expected: `state` đổi đúng thời điểm có bằng chứng, `relevance` và `pictogram` giữ nguyên
Rationale: Đèn có thể đổi màu ngay trên track; nếu không cập nhật đúng, model học sai thời điểm chuyển trạng thái.
Common mistake: Giữ nguyên `state` trong cả chuỗi mà không update khi chuyển
Diversity: temporal / critical

---

CASE ID: EC-05
Sample: LISA15 (sample_id)
Scene: đèn rõ và xa hơn một chút
Observation: Đèn còn nhìn thấy rõ, nhưng box cần được giữ sát thân đèn và không bao cột.
Decision: LABEL
Expected: `traffic_light` rectangle bám sát thân đèn; `relevance` theo hướng xe
Rationale: Geometry tightness ảnh hưởng trực tiếp đến bao phủ đúng object.
Common mistake: Box quá lớn, chứa cột hoặc phần đèn khác.
Diversity: small_far / geometry

---

CASE ID: EC-06
Sample: LISA18 (sample_id)
Scene: đèn bị che ở mép khung và có phản chiếu
Observation: Hình không đủ rõ để nhận diện màu, nhưng vẫn còn nhận diện là đèn giao thông.
Decision: UNKNOWN
Expected: `state=unknown` và `pictogram` nếu không chắc
Rationale: Phản chiếu không được xem là bằng chứng cho màu đèn.
Common mistake: Dùng màu từ hình phản chiếu hoặc suy ra bằng ánh sáng xung quanh.
Diversity: occlusion / low_visibility

---

CASE ID: EC-07
Sample: LISA20 (sample_id)
Scene: đầu đèn mũi tên trái với hướng đi của xe khác nhau
Observation: Thử phân minh `relevance` giữa đèn làn xe và đèn không liên quan.
Decision: LABEL + relevance decision
Expected: `relevance=not_relevant` nếu đèn không thuộc làn xe, hoặc `unknown` nếu không đủ bằng chứng
Rationale: Đây là edge case critical vì ảnh hưởng trực tiếp đến hành vi lái.
Common mistake: Chọn `relevant` chỉ khi đèn là `traffic_light` mà không xét hướng đi
Diversity: critical / ambiguity

---

CASE ID: EC-08
Sample: LISA25 (sample_id)
Scene: đèn rời khỏi khung
Observation: Đèn thoát khỏi khung hình trước cuối đoạn video.
Decision: LABEL then OUTSIDE
Expected: `outside` đặt ở frame đầu tiên mất hẳn, không kéo track đến cuối clip
Rationale: Tránh output có box “ma” kéo dài đến frame cuối và làm sai mô hình.
Common mistake: Không set `outside` khi object biến mất.
Diversity: temporal / critical / truncation

---
