# Problem statement + downstream contract

## Bài toán

Thiết kế annotation cho **traffic-light state + ego relevance trên ảnh tĩnh tại giao lộ có nhiều đầu đèn**, tập trung vào các trường hợp nhiều tín hiệu xuất hiện đồng thời khiến annotator khó xác định đèn nào thực sự điều khiển hướng di chuyển của ego vehicle.

## Downstream contract

1. **Downstream task / model / user là ai?**  
   Module perception của hệ thống hỗ trợ lái/xe tự hành, cung cấp trạng thái đèn giao thông liên quan trực tiếp tới ego vehicle cho tầng ra quyết định và planning.

2. **Output annotation nào thực sự cần?**  
   Mỗi đầu đèn trong scope được gán một **rectangle** bao quanh phần đầu đèn nhìn thấy, với 2 attribute chính:
   - `state`: `red`, `yellow`, `green`, `off`, `unknown`
   - `ego_relevance`: `relevant`, `irrelevant`, `unknown`

3. **Failure nào gây hậu quả lớn nhất?**  
   Critical failure là gán một đèn **không điều khiển ego vehicle** thành `ego_relevance = relevant`, đặc biệt khi đèn đó ở trạng thái `green`, vì downstream có thể sử dụng sai tín hiệu để ra quyết định tiếp tục di chuyển.

4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**  
   Nếu không đủ bằng chứng để xác định state hoặc ego relevance, annotator phải dùng giá trị `unknown`. Nếu bản thân object hoặc quan hệ điều khiển vẫn không thể quyết định theo guideline, đánh dấu `needs_review` để reviewer/QA owner xử lý thay vì tự suy đoán.

## Scope

- **Trong scope (bắt buộc label):** các đầu đèn giao thông nhìn thấy đủ để tạo rectangle và có khả năng liên quan đến giao lộ/làn đường mà ego vehicle đang tiếp cận; bao gồm cả đèn relevant, irrelevant và trường hợp chưa đủ bằng chứng để xác định relevance.
- **Ngoài scope (ignore):** phản chiếu ánh sáng, biển quảng cáo/đèn trang trí, tín hiệu không phải traffic light, object quá nhỏ hoặc bị che đến mức không thể xác định là một đầu đèn giao thông.
- **Geometry tolerance:** rectangle ôm sát phần vỏ đầu đèn nhìn thấy; không mở rộng theo phần bị che giả định. Sai lệch mục tiêu không quá khoảng **2 px mỗi cạnh** đối với object đủ lớn để đánh giá ổn định; object quá nhỏ/xa được xử lý theo rule `unknown`/escalation trong guideline thay vì ép geometry chính xác giả tạo.

## Output chấm được

Blind test sẽ chấm được trực tiếp từ CVAT export các quyết định:

- **LABEL / IGNORE:** có hoặc không có rectangle traffic light theo rule.
- **UNKNOWN:** thể hiện bằng giá trị `unknown` của `state` hoặc `ego_relevance`.
- **ESCALATE:** thể hiện bằng attribute `needs_review`.
- **Attribute correctness:** `state` và `ego_relevance`.
- **Geometry compliance:** rectangle có tuân thủ tight-visible rule và tolerance đã định nghĩa hay không.

## Dữ liệu và giới hạn

Sử dụng **BDD100K** trong `data/bdd100k/`, ưu tiên 12 ảnh có traffic light được repo chuẩn bị cho bài. Dữ liệu gồm cả một số cảnh ban ngày, đêm và chạng vạng nhưng số lượng nhỏ, vì vậy guideline chỉ nhằm chứng minh tính nhất quán và khả năng bàn giao trong phạm vi lab, không đại diện đầy đủ cho mọi giao lộ thực tế. Bài này dùng **ảnh tĩnh**, không sử dụng LISA/video track.