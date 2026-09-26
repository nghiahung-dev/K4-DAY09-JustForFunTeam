# Annotation guideline — Traffic light state + ego relevance

**Version:** v1

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

Mục tiêu là gán nhãn các đầu đèn giao thông để xác định:

- trạng thái đèn ở từng frame: đỏ / vàng / xanh / tắt / không rõ
- hình dạng đèn: tròn / mũi tên trái / mũi tên thẳng / mũi tên phải / khác / không rõ
- mức độ liên quan của đèn với hành vi của xe: có liên quan / không liên quan / không đủ bằng chứng

Scope của task:

- Label tất cả đầu đèn giao thông xuất hiện trong clip LISA hoặc trong các ảnh calibration/example được đưa vào task CVAT.
- Mỗi đầu đèn là một instance riêng và được gán theo track xuyên suốt chuỗi frame.
- Chỉ gán các đối tượng thuộc phạm vi đèn giao thông; không label các cột, giá treo, biển báo, vạch kẻ, hoặc phần không phải đèn.
- Không label vùng nền hoặc ảnh tổng quát; chỉ label đầu đèn có thể liên quan đến hành vi lái xe.

Cái ngoài scope:

- cột đèn, giá treo, mảng điện, bóng đèn không có hình dạng quan sát được
- các vật thể không là đầu đèn giao thông
- các region không có chức năng tín hiệu giao thông

## 2. Annotation unit

Annotation unit là một track của một đầu đèn giao thông, không phải là một hình ảnh tĩnh độc lập.

- Một đầu đèn vật lý = một track duy nhất trong toàn bộ video.
- Mỗi track có một bounding box rectangle sát với cơ thể đèn.
- Nếu đèn đi ra khỏi khung hình hoặc bị che hẳn, track được đóng bằng `outside` ở frame đầu tiên mất hẳn.
- Nếu đèn xuất hiện lại sau đó, track có thể tiếp tục ở frame mới, nhưng không được tạo two tracks cho cùng một đầu đèn.
- Trạng thái `state` là thuộc tính mutable theo frame.
- `relevance`, `pictogram` là thuộc tính cố định cho cả track, trừ khi hình dạng thực sự không đủ bằng chứng và cần gán `unknown`.

Một object được tính là instance mới khi:

- đó là một đầu đèn khác về mặt vật lý, không phải cùng đèn đang đi qua frame trước đó
- hai đèn gần nhau nhưng tách biệt rõ ràng về vị trí và tương ứng với hai nguồn tín hiệu khác nhau

Nếu không chắc là cùng đầu đèn hay không, ưu tiên kiểm tra vị trí, cấu trúc và mũi tên trước khi tạo instance mới.

## 3. Geometry rule

### 3.1 Geometry type

- Sử dụng `rectangle` cho từng đầu đèn.
- Không dùng polyline hoặc polygon cho task traffic light.

### 3.2 Position

Box phải:

- bám sát vỏ đèn, không bao gồm cột và giá treo
- bao phủ diện tích sáng rõ nhất của đèn nhưng không lớn hơn cần thiết
- giữ tâm box gần với vị trí vật lý của đèn trong từng frame

### 3.3 Tolerance

- Cho phép sai lệch nhỏ khi box bị xê dịch vì chuyển động camera hoặc khe thời gian giữa các frame.
- Nếu sai lệch rõ ràng, kéo box về đúng vị trí ở keyframe tương ứng.
- Không chấp nhận box quá lớn bao phủ nhiều đầu đèn hoặc cả cột.

### 3.4 Outside

- Khi đèn biến mất hoàn toàn khỏi khung hình hoặc bị che hẳn, chọn track rồi đặt `outside` ở frame đầu tiên không còn thấy đèn.
- Nếu đèn xuất hiện trở lại sau đó, đặt lại `outside` ở frame đó nếu cần, rồi tiếp tục track.
- Không để track kéo dài tới frame cuối với box “ma” không còn đèn thật.

## 4. Taxonomy

Bảng ontology đầy đủ nằm ở `03_ontology_and_cvat_setup.md`; các giá trị dưới đây là source của CVAT schema và phải khớp nhau.

### 4.1 Class

| Name | Geometry | Type | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_light` | Rectangle | class | n/a | n/a | no | Đầu đèn giao thông là đối tượng chính cần xác định | 

### 4.2 Attributes

| Name | Geometry | Type | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `state` | Rectangle | attribute | `red`, `yellow`, `green`, `off`, `unknown` | `unknown` | yes | Trạng thái đèn thay đổi theo thời gian | 
| `relevance` | Rectangle | attribute | `relevant`, `not_relevant`, `unknown` | `unknown` | no | Xác định đèn có liên quan với hành vi của xe hay không | 
| `pictogram` | Rectangle | attribute | `circle`, `arrow_left`, `arrow_straight`, `arrow_right`, `other`, `unknown` | `unknown` | no | phân loại hình dạng đầu đèn | 
| `needs_review` | Rectangle | attribute | `false` | `false` | yes | Dùng khi cần xem lại do mơ hồ hoặc không chắc |

### 4.3 Quy tắc dùng `unknown`

Dùng `unknown` khi:

- đèn bị che hoặc mờ không nhìn rõ màu
- hình dạng đèn không đủ bằng chứng
- không đủ thông tin để xác định `relevance`
- cần escalade do mơ hồ nhưng chưa chắc chắn

`unknown` là giá trị hợp lệ. Không sử dụng đoán mò.

## 5. Inclusion / exclusion

### 5.1 Bắt buộc label

Bắt buộc label các đầu đèn giao thông rõ ràng trong video/ảnh calibration, bao gồm cả:

- đèn tròn
- đèn mũi tên trái / thẳng / phải
- đèn gần/xa nếu nhìn thấy rõ và có thể xác định
- đèn thuộc làn giao thông của xe hoặc đèn có liên quan trực tiếp đến thao tác lái

### 5.2 Không label

Không label các vật thể sau:

- cột, giá treo, trụ điện
- bóng phản chiếu hoặc đèn không phải tín hiệu giao thông
- đèn không rõ là đầu đèn giao thông
- đèn ở vùng quá xa và không đủ bằng chứng để xác định rằng nó thuộc ngã tư/đường đi của xe

### 5.3 Về relevance

- `relevant`: đèn thuộc làn hoặc hướng đi của xe có thể trực tiếp tác động lên quyết định dừng/đi
- `not_relevant`: đèn cho làn khác, cho người đi bộ, hoặc đèn ở ngã tư xa không liên quan trực tiếp
- `unknown`: không đủ dữ kiện để xác định

## 6. Visibility / occlusion

### 6.1 Khi đèn bị che một phần

- Nếu phần đèn còn đủ để nhận dạng hình dạng và màu, label như bình thường.
- Nếu chỉ thấy một phần rất nhỏ mà không đủ xác định, dùng `unknown` cho `state` và `pictogram` nếu cần.
- Không “đoán” màu dựa trên frame trước/sau khi không có bằng chứng trực tiếp.

### 6.2 Khi đèn bị che hẳn

- Nếu đèn biến mất hoàn toàn khỏi ảnh, đánh `outside` ở frame đầu tiên mất hẳn.
- Không giữ box kéo dài qua các frame sau khi không còn đèn thực.

### 6.3 Khi ở mép khung hình

- Nếu đèn chỉ lộ một phần và đủ để biết đó là đầu đèn giao thông, vẫn label nếu có thể xác định rõ thao tác.
- Nếu không đủ bằng chứng để biết là đèn nào, dùng `unknown` và có thể đánh `needs_review`.

### 6.4 Loá, phản chiếu, mờ

- Loá và phản chiếu không được xem là bằng chứng rõ ràng để gán màu.
- Nếu hình ảnh quá mờ hoặc chói, ưu tiên `unknown`.

## 7. Ambiguity / escalation

Khi bằng chứng không đủ, annotator phải quyết định theo flow sau:

1. Nếu object rõ là đèn giao thông: label với `state = unknown` hoặc `pictogram = unknown` nếu cần
2. Nếu đèn không đủ bằng chứng để xác định `relevance`: đặt `relevance = unknown`
3. Nếu cần review do nhiều cách hiểu khác nhau: bật `needs_review = true`
4. Nếu đối tượng không đủ để xác định thuộc phạm vi task hoặc không chắc là đèn giao thông: không label

### Quyết định phải thể hiện trong CVAT như thế nào

- `state`: chọn giá trị đúng trong dropdown
- `relevance`: chọn `unknown` hoặc `not_relevant` tương ứng
- `pictogram`: chọn `unknown` nếu không rõ
- `needs_review`: bật checkbox khi cần review
- `outside`: dùng khi đèn biến mất hoàn toàn

Không có hidden rule nào chỉ nói bằng miệng. Nếu không rõ, phải có giá trị khả dụng trong CVAT để thể hiện quyết định.

## 8. Temporal rule

Task này là video/track-based. Vì vậy:

- mỗi đầu đèn phải có track liên tục trong suốt thời gian xuất hiện
- `state` là attribute mutable theo frame
- khi đèn đổi màu, đổi `state` ở đúng frame đổi màu
- CVAT sẽ tạo keyframe ở frame đó; nếu box lệch, kéo lại để bắt đúng đèn
- `relevance` và `pictogram` giữ nguyên cho cả track, không thay đổi theo frame
- `needs_review` có thể chuyển đổi theo frame nếu cần review ở một frame cụ thể, nhưng không thay đổi quá mức nếu không cần

### 8.1 Xử lý chuyển màu

- `red → yellow → green` hoặc các chuỗi tương tự phải xảy ra ở đúng frame có bằng chứng trực quan
- nếu không thấy quá rõ do che hoặc loá, dùng `unknown` ở frame đó
- không ghi đèn “nhảy” quá nhanh mà không có dữ kiện trực quan

### 8.2 Xử lý bị che ngắn

- nếu đèn bị che ngắn, dùng `unknown` ở các frame che hẳn; không suy đoán
- nếu che ngắn nhưng không đủ để chắc chắn, xem frame trước/sau để hỗ trợ nhưng không đoán bừa

## 9. Examples

Các ví dụ dưới đây dùng sample trong split example/calibration, không dùng blind sample.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| EX01 | Đèn tròn phía làn xe đi thẳng, sáng đỏ rõ ràng | 1 track, `pictogram=circle`, `relevance=relevant`, `state=red` | direct label, relevance by lane | 
| EX02 | Đèn mũi tên trái, xe đi thẳng, đèn trái không liên quan | 1 track, `pictogram=arrow_left`, `relevance=not_relevant`, `state` theo màu nhìn thấy | relevance based on ego direction |
| EX03 | Đèn biến mất khỏi khung hình ở frame 12 | `outside` đặt ở frame 12, không giữ box tới cuối video | outside rule |
| EX04 | Đèn bị che tạm thời trong 2 frame, không nhìn rõ màu | `state=unknown` ở các frame bị che | visibility/occlusion |
| EX05 | Đèn có mũi tên phải và màu xanh rõ | 1 track, `pictogram=arrow_right`, `state=green`, `relevance` matched with maneuvers | pictogram + state |
| CAL01 | đầu đèn gần nhưng giá treo và cột hiện rất rõ | box chỉ bao đèn, không bao giá treo | geometry tightness |
| CAL02 | đèn xa, nhỏ nhưng vẫn nhìn rõ là đầu đèn | label nếu có đủ bằng chứng; nếu không đủ thì `unknown` | small/far visibility |
| CAL03 | đèn cho người đi bộ | `relevance=not_relevant` hoặc `unknown` nếu không đủ thông tin | inclusion/exclusion |

## 10. Common mistakes

Các lỗi thường gặp nhất và cách tránh:

1. Vẽ box quá lớn, chứa cột hoặc giá treo
   - Cách tránh: box chỉ bao phần đèn, không bao các phụ kiện.

2. Dùng Shape thay vì Track
   - Cách tránh: chọn `Track` cho mỗi đầu đèn, không vẽ nhiều object rời rạc.

3. Không đặt `outside` khi đèn mất hẳn
   - Cách tránh: đặt `outside` ngay ở frame đầu tiên mất hẳn.

4. Gán `state` theo đoán khi đèn che hoặc mờ
   - Cách tránh: dùng `unknown` khi không có bằng chứng trực tiếp.

5. Gán `relevance` theo cảm tính thay vì theo làn và hướng của xe
   - Cách tránh: xác định is lane / direction trước rồi gán `relevant` hoặc `not_relevant`.

6. Chọn pictogram sai vì nhìn nhầm hướng mũi tên
   - Cách tránh: kiểm tra hướng mũi tên thật rõ trước khi chọn `arrow_left/right/straight`.

7. Không cập nhật state khi đèn đổi màu
   - Cách tránh: đổi ngay ở frame đổi màu và kiểm tra frame trước/sau.

8. Giữ track quá dài dù đèn đã thật sự biến mất
   - Cách tránh: kiểm tra các frame cuối và đặt `outside` đúng thời điểm.

9. Dùng `not_relevant` khi chưa đủ bằng chứng
   - Cách tránh: nếu không rõ, dùng `unknown` và bật `needs_review`.

10. Bỏ sót đèn có mũi tên
   - Cách tránh: duyệt toàn bộ chuỗi frame và confirm mỗi đầu đèn được cover.

