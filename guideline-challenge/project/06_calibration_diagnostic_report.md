# Báo cáo calibration — Traffic sign (Mục 06)

## Trạng thái

Đã chạy phép đo calibration trên 6 ảnh trong split calibration của `sample_pack.csv`: GTS02, GTS04, GTS05, GTS07, GTS08, GTS14. Hai export CVAT mới nhất có cùng 28 ảnh và schema (`traffic_sign`, rectangle); khác nhau về annotation (Phu 83 box, Trong 82 box), nên phép so sánh này dùng được. Để khớp đúng split, lệnh được chạy trên các bản ZIP làm việc chỉ giữ 6 ảnh calibration; annotation được giữ nguyên từ export gốc.

## Kết quả công cụ

- Đồng thuận số lượng object: **66.7%** (4/6 sample-label measures).
- Đồng thuận attributes/tags: **40.0%** (4/10 measures).
- Tổng object trong 6 ảnh calibration: Phu **33**, Trong **31**.
- Tool chỉ đo count, attribute và tag; không tính độ khớp hình học (IoU). Geometry cần QA trực quan trong CVAT.

File máy sinh: `project/06_calibration_measure.csv`.
Bảng phân tích các bất đồng và hành động đề xuất: `project/06_calibration_report.csv`.

## Bất đồng đáng chú ý

1. **GTS02 — sign_family:** cùng box ở mép trái ảnh, Phu chọn `unknown`, Trong chọn `mandatory`. Cần thêm ví dụ cho biển bị cắt ở mép và tiêu chí đủ bằng chứng để chọn family.
2. **GTS05 — object count:** Phu có 9 box, Trong có 8. Box thêm của Phu nằm ở mặt sau của một biển; cần đưa quy tắc loại trừ mặt sau vào mục Inclusion/Exclusion của guideline.
3. **GTS07 — biển nhỏ/xa:** Phu gán một biển khoảng 9×9 px là `unknown`, Trong bỏ qua. Rule hiện tại cho vùng 8–15 px cần được áp dụng nhất quán; dùng coaching và kiểm tra nhận diện object theo ảnh gốc.
4. **GTS08 — sign_family:** Phu để `__undefined__`, Trong chọn `prohibitory` trên bảng lớn thiếu sáng. Cần quy định dùng `unknown` khi chắc chắn là biển nhưng không đọc được family, và không để giá trị mặc định chưa chọn.

## Chẩn đoán và giới hạn

Các bất đồng được phân loại trong CSV theo `guideline_gap`, `execution_error` hoặc `data_ambiguity`, cùng hành động đề xuất. Các chẩn đoán này là kết quả review của owner dựa trên export và ảnh gốc; tool không quyết định annotator nào đúng. Chưa có phép đo geometry và chưa cập nhật version guideline ở báo cáo này. Sau khi nhóm duyệt các rule change, cập nhật `02_guideline.md` lên v2 và ghi bằng chứng vào `08_revision_log.md`.
