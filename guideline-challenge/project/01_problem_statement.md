# Problem statement + downstream contract

## Bài toán

Thiết kế guideline annotation cho **traffic sign theo taxonomy ở mức sign family**, tập trung vào các biển **nhỏ, xa, bị che khuất hoặc khó phân loại** trong ảnh đường phố. Mục tiêu là giúp annotator xác định nhất quán khi nào cần label, khi nào ignore, khi nào dùng `unknown`, và khi nào phải chuyển sang review.

## Downstream contract

1. **Downstream task / model / user là ai?**  
   Module perception của hệ thống hỗ trợ lái/xe tự hành, dùng kết quả nhận diện biển báo để cung cấp thông tin cho tầng nhận thức tình huống và planning.

2. **Output annotation nào thực sự cần?**  
   Mỗi traffic sign hợp lệ được gán một **rectangle** ôm sát phần mặt biển nhìn thấy và một thuộc tính phân loại chính:
   - `sign_family`: `prohibitory`, `mandatory`, `danger`, `other`, `unknown`

   Ngoài ra dùng:
   - `needs_review`: `true/false` để thể hiện trường hợp cần escalation.

   Bài này chỉ phân loại ở **mức family**, không yêu cầu nhận diện từng mã biển cụ thể.

3. **Failure nào gây hậu quả lớn nhất?**  
   Critical failure là **bỏ sót hoặc phân loại sai một biển điều tiết giao thông còn đủ nhìn thấy**, đặc biệt khi biển thuộc nhóm `prohibitory` hoặc `mandatory`, khiến downstream không nhận được đúng thông tin hạn chế hoặc yêu cầu mà phương tiện cần tuân theo.

4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**  
   Nếu xác định chắc chắn đó là traffic sign nhưng không đủ thông tin để quyết định family, annotator vẫn tạo rectangle và gán `sign_family = unknown`. Nếu ngay cả việc có nên label hay không vẫn không thể quyết định theo guideline, đặt `needs_review = true` và chuyển cho reviewer/QA owner xử lý; annotator không tự suy đoán.

## Scope

- **Trong scope (bắt buộc label):** traffic sign thực tế xuất hiện trong ảnh và còn đủ đặc trưng hình dạng/mặt biển để nhận biết là một biển báo giao thông, kể cả khi nhỏ, xa hoặc bị che một phần.
- **Ngoài scope (ignore):** biển quảng cáo, bảng tên đường/cửa hàng không thuộc taxonomy của bài, vật thể có hình dạng giống biển nhưng không phải traffic sign, hoặc object quá nhỏ/mờ/bị che đến mức không thể xác định đáng tin cậy là traffic sign.
- **Geometry tolerance:** dùng rectangle ôm sát **phần mặt biển nhìn thấy**, không mở rộng theo phần bị che khuất giả định và không bao gồm cột/giá đỡ. Với biển có kích thước đủ lớn để đánh giá ổn định, sai lệch mục tiêu không quá khoảng **2 px mỗi cạnh**; các object quá nhỏ để áp dụng tolerance ổn định sẽ được xử lý bằng rule `unknown`/`needs_review` trong guideline.

## Output chấm được

Blind test sẽ đánh giá trực tiếp từ CVAT export các loại quyết định sau:

- **LABEL:** có rectangle cho traffic sign nằm trong scope.
- **IGNORE:** không tạo annotation cho object ngoài scope.
- **UNKNOWN:** rectangle vẫn được tạo nhưng `sign_family = unknown`.
- **ESCALATE:** annotation có `needs_review = true`.
- **Class/attribute correctness:** `sign_family` đúng với rule của guideline.
- **Geometry compliance:** rectangle tuân thủ tight-box rule và geometry tolerance đã quy định.

## Dữ liệu và giới hạn

Sử dụng **GTSDB** trong `data/gtsdb/` làm nguồn dữ liệu chính. Repo cung cấp **28 ảnh GTSDB** (`GTS01`–`GTS28`), độ phân giải 1360×800. Dự kiến chọn khoảng **15 ảnh không trùng nhau** cho toàn bộ workflow, gồm khoảng **4 example, 6 calibration và 5 blind**; ưu tiên các ảnh tạo được sự đa dạng giữa trường hợp rõ ràng và trường hợp nhỏ/xa/bị che hoặc khó phân loại.

Giới hạn của bài là tập dữ liệu trong repo có quy mô nhỏ và không đại diện đầy đủ cho mọi loại biển báo, điều kiện thời tiết, quốc gia hay góc nhìn thực tế. Vì vậy guideline được đánh giá chủ yếu theo **tính nhất quán, khả năng bàn giao và xử lý ambiguity trong phạm vi lab**, không nhằm xây dựng taxonomy traffic sign hoàn chỉnh cho triển khai thực tế.
