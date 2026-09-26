# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| `v1` | Xây dựng bản guideline nháp đầu tiên | Khởi tạo taxonomy sign_family, quy tắc tight box và phân loại 4 nhóm biển chính | Problem statement + catalog 28 ảnh GTSDB |
| `v2` | Bổ sung quy tắc loại trừ mặt sau biển báo kim loại (IGNORE) và quy tắc cắt mép (truncated) | Đo bất đồng calibration giữa các thành viên: annotator nhầm lẫn khi gặp mặt sau biển báo và biển bị nghiêng sát rìa ảnh | `06_calibration_report.csv` dòng GTS06, GTS11; `06_calibration_measure.csv` |
| `v3` | Làm rõ quy tắc loại trừ biển quảng cáo thương mại ven đường (IGNORE), biển phụ thuyết minh và hướng dẫn nhận diện biển nhỏ xa qua hình khối | Phản hồi và câu hỏi clarification từ nhóm peer (team20): lúng túng khi gặp biển hiệu Biergarten ở GTS28 và biển cấm nhỏ ở GTS24 | `07_blind_handoff/clarification_log.csv` câu 1, 2; `peer_feedback.md` mục 1 và 2 |
