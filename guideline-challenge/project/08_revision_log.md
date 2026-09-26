# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu cho task traffic light: xác định `traffic_light` là class, `state`, `relevance`, `pictogram` là attribute, rule bắt buộc phải dùng Track và đặt `outside` khi đèn biến mất. | Hình thành baseline đầu tiên cho toàn bộ dữ liệu, thân thiện với annotator mới. | Dựa trên problem statement và schema `03_cvat_labels.json` |
| v2 | Chỉnh rõ `unknown` khi che/mờ, thêm quy tắc `outside` và `needs_review=true` cho edge case ambig, nhấn mạnh relevance phải theo làn/đường đi của xe. | Calibration nội bộ cho thấy nhiều nhầm lẫn giữa “đoán theo cảm giác” và “xác định bằng chứng”. | sample_ids `LISA08`, `LISA12`, `LISA14` có độ lệch `state`/`relevance` đáng kể trong calibration report |
| v3 | Finalize consent rule cho peer handoff: nếu không đủ bằng chứng, ưu tiên `unknown` thay vì `not_relevant`; phải ghi quyết định trong CVAT với attribute rõ ràng. | Blind handoff của Sakana và feedback peer cho thấy guideline cần bước escalation rõ hơn. | feedback peer + clarification log; sửa sau khi nhận phản hồi từ nhóm Sakana |


