# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:** Sakana
- **Người label blind:** reviewer 1 / reviewer 2 (được phân công bởi Sakana trong quá trình blind handoff)

## 1. Peer trả lời

1. Rule nào rõ nhất / giúp quyết định nhanh nhất? Quy tắc `Track` cho mỗi đầu đèn và `outside` khi đèn biến mất là dễ nhất để làm đúng ngay từ đầu.
2. Rule nào mơ hồ hoặc phải tự suy diễn? `relevance` khi đèn mũi tên hoặc đèn ở xa; cần phải dựa vào is-lane/hướng xe thay vì suy đoán theo màu.
3. Sample nào khiến guideline "vỡ"? `LISA20` và `LISA22` vì có đèn mũi tên và hình nhỏ, dễ nhầm `relevance` hoặc `pictogram` nếu không xem kỹ.
4. Attribute / default nào trong CVAT dễ gây thao tác sai? `__undefined__` nếu quên cập nhật sau khi vẽ, và `state` khi frame chuyển màu mà không dùng `unknown` ở frame che mờ.
5. Một thay đổi cụ thể giúp annotator mới ít hỏi hơn? Thêm ví dụ rõ about `relevance` theo làn xe và ví dụ `outside` ở frame đầu tiên mất hẳn.

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| Sai `relevance` ở đèn mũi tên | guideline gap: cần ghi rõ `relevance` theo hướng xe | accept + revise | sample `LISA20`, frame chuyển màu và hướng xe trong clip |
| Dùng `not_relevant` thay vì `unknown` khi đèn bị che | data ambiguity | add escalation rule | sample `LISA18`, frame bị che trong 2-3 frame |
| Quên đặt `outside` ở cuối track | execution error | reject with evidence | sample `LISA25`, frame cuối không còn đèn |
