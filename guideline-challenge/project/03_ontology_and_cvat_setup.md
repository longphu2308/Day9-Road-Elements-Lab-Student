# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_sign` | `rectangle` | class | N/A | N/A | false | Thể hiện một biển báo giao thông hợp lệ trong ảnh; rectangle ôm sát phần mặt biển nhìn thấy, không bao gồm cột/giá đỡ. |
| `sign_family` | N/A | attribute | `__undefined__`, `prohibitory`, `mandatory`, `danger`, `other`, `unknown` | `__undefined__` | false | Phân loại nhóm chức năng chính của biển báo theo taxonomy; default `__undefined__` để buộc annotator chủ động chọn, tránh bias im lặng. |
| `needs_review` | N/A | attribute | `false`, `true` | `false` | false | Cờ đánh dấu trường hợp mơ hồ cao (ambiguity) hoặc tranh chấp cần chuyển escalation lên reviewer/QA. |

## Class hay attribute

- **`traffic_sign` là class (Geometry: rectangle):** Đây là đối tượng vật thể độc lập cần xác định vị trí và kích thước hình học trên ảnh. Downstream perception trước hết cần phát hiện (detect) sự tồn tại của biển báo giao thông; nếu tách mỗi family thành một class riêng (`traffic_sign_prohibitory`, `traffic_sign_danger`,...) sẽ làm phân mảnh không gian nhãn, khiến annotator dễ chọn nhầm công cụ vẽ và gây khó khăn khi downstream cần mở rộng taxonomy (ví dụ gán thêm mã biển chi tiết sau này).
- **`sign_family` là attribute (kiểu select):** Là thuộc tính phân loại ngữ nghĩa gắn liền với đối tượng `traffic_sign`, cho phép annotator tập trung vẽ hình học chuẩn xác trước rồi mới gán nhóm chức năng ở lượt 2 (Attribute Annotation Mode).
- **`needs_review` là attribute (kiểu checkbox):** Đóng vai trò escalation flag cho từng bounding box cụ thể, giúp tách biệt rõ ràng giữa quyết định chuyên môn và việc nghi ngờ chất lượng để QA/reviewer thẩm định.
- **Default value và bias:** Nếu đặt default của `sign_family` là một giá trị có nghĩa như `prohibitory` hoặc `other`, khi annotator chỉ vẽ box mà quên chọn thuộc tính, hệ thống sẽ âm thầm ghi nhận nhãn đó dẫn đến dữ liệu huấn luyện/đánh giá bị sai lệch nghiêm trọng ("lỗi im lặng"). Chọn `__undefined__` làm default value buộc annotator phải chủ động chọn family phù hợp. Trong file export hoặc pipeline kiểm tra, nếu còn tồn tại `__undefined__` thì QA/script sẽ dễ dàng phát hiện annotation chưa hoàn tất.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `2.74.1` (tại `http://localhost:8080`)
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `team06-calib-v1`
- **Guide của task đã dán `02_guideline.md`?** có
- **Nhóm dùng Track hay Shape, vì sao:** Dùng **Shape** vì bài toán sử dụng tập dữ liệu ảnh tĩnh GTSDB (`GTS01`–`GTS28`), mỗi ảnh là một khung hình độc lập, không phải video stream và không có đối tượng di chuyển liên tục cần tracking qua các frame.

## Setup test

Một thành viên **chưa tham gia setup** (Lại Hoàng Duy - spec owner) mở task và trả lời:
- **Label gì & Tool nào:** Dùng label `traffic_sign` với công cụ Rectangle (chế độ Shape) để vẽ bounding box ôm sát mặt biển nhìn thấy.
- **Gán attribute nào:** Dropdown `sign_family` chọn 1 trong 5 nhóm (`prohibitory`, `mandatory`, `danger`, `other`, hoặc `unknown` nếu chắc chắn là biển báo nhưng không đủ dấu hiệu xác định loại).
- **Khi nào escalate:** Khi gặp trường hợp mơ hồ cao không thể chắc chắn vật thể có phải là biển báo giao thông nằm trong scope hay không, tick chọn checkbox `needs_review = true` để chuyển reviewer/QA thẩm định.
- **Chỗ vấp ban đầu:** Annotator dễ vẽ box xong rồi chuyển ảnh mà quên chọn `sign_family`. Do default là `__undefined__`, annotator được nhắc nhở luôn chọn thuộc tính ngay sau khi vẽ box, và QA sẽ quét lọc giá trị `__undefined__` trước khi nghiệm thu.
