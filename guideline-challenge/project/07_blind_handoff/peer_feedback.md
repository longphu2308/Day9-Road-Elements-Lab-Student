# Peer feedback + owner response



## 1. Peer trả lời

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?**  
   Quy tắc vẽ hình chữ nhật ôm khít biên nhìn thấy (tight visible bounding box) và tiêu chuẩn phân loại 4 nhóm hình học/màu sắc chính (`danger`: tam giác viền đỏ; `prohibitory`: tròn viền đỏ; `mandatory`: tròn nền xanh; `other`: các loại biển chỉ dẫn/thông tin hình chữ nhật hoặc vuông). Quy tắc này giúp annotator nhận diện và phân loại tức thì với các biển báo rõ nét ở khoảng cách gần và trung bình.

2. **Rule nào mơ hồ hoặc phải tự suy diễn?**  
   - Cách xử lý đối với biển báo phụ (auxiliary plates): Biển hình chữ nhật nhỏ gắn phía dưới biển chính để thuyết minh cự ly hoặc khung giờ không được nói rõ là vẽ chung một bounding box gộp, vẽ riêng từng biển, hay bỏ qua.  
   - Ngưỡng kích thước tối thiểu đối với các biển ở quá xa đường chân trời chưa được định lượng bằng pixel cụ thể trong phiên bản đầu, khiến annotator lúng túng giữa việc bỏ qua hay gắn nhãn `unknown`.

3. **Sample nào khiến guideline "vỡ"?**  
   - `GTS09` và `GTS11`: Có cụm biển gồm biển chính và 2 biển phụ đi kèm, cùng với biển cấm ở vị trí rất xa bị vỡ nét (kích thước chỉ khoảng 6–8px). Nhóm peer mỗi người xử lý một kiểu: người vẽ gộp 1 box lớn, người vẽ 3 box con, người bỏ qua biển phụ.  
   - `GTS10`: Cụm biển báo có biển quay mặt sau (backside) về phía camera; guideline v1 chưa nói rõ mặt sau có phải gán nhãn hay không.

4. **Attribute / default nào trong CVAT dễ gây thao tác sai?**  
   - Giá trị mặc định `__undefined__` cho `sign_family` hoạt động tốt để cảnh báo thiếu sót, nhưng khi annotator vẽ liên tiếp bằng phím tắt `N`, nếu quên không đổi attribute thì nhãn bị lưu ở trạng thái chưa hoàn tất.  
   - Checkbox `needs_review` cần được lưu ý vì khi copy-paste thuộc tính giữa các object, trạng thái tick có thể bị sao chép nhầm.

5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?**  
   - Bổ sung ngay vào mục **Inclusion / Exclusion** quy định rõ: "Biển phụ chỉ chứa chữ thuyết minh bổ trợ thì IGNORE; biển phụ có biểu tượng đồ họa độc lập thì vẽ box riêng với class `other`; mặt sau của biển báo bắt buộc IGNORE".  
   - Quy định ngưỡng kích thước cut-off rõ ràng: "Kích thước < 8px thì IGNORE; từ 8px - 15px không nhận rõ ký hiệu thì gán `unknown`".

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| Lúng túng khi xử lý biển phụ (auxiliary plates) gắn dưới biển chính ở GTS09 | guideline_gap | accept + revise | clarification_log dòng 1: Peer hỏi về biển phụ; đã bổ sung rule chi tiết vào Mục 5 và Mục 7 của Guideline v3 |
| Vẽ nhãn cho mặt sau của biển báo kim loại ở GTS10 | guideline_gap | accept + revise | clarification_log dòng 2: Đã bổ sung rõ vào mục 5 (Exclusion) và mục 7 (IGNORE): Mặt sau biển báo bắt buộc bỏ qua |
| Phân vân giữa ignore và unknown cho biển ở quá xa (< 8px) ở GTS11 | guideline_gap | accept + revise | clarification_log dòng 4: Cập nhật mục 6.3 quy định ngưỡng cut-off 8px (dưới 8px IGNORE, 8-15px mờ gán unknown) |
| Bỏ sót thuộc tính sign_family để nguyên `__undefined__` trên 1 box tại GTS08 | execution_error | reject with evidence | Schema CVAT đã cố ý đặt default là `__undefined__` để phát hiện lỗi thao tác; đã thêm cảnh báo vào Mục 10 (Common mistakes) |
| Nghi ngờ phân loại biển báo LED ma trận điện tử hiển thị tốc độ | guideline_gap | accept + revise | clarification_log dòng 3: Cập nhật mục 4 và mục 7 xác định biển VMS viền đỏ cấm thuộc `prohibitory` |
| Biển bị lóa sáng mạnh do ánh sáng ngược ở GTS12 | data_ambiguity | add escalation rule | Đã bổ sung quy tắc vào Mục 7: Khi overexposure làm mất hoàn toàn biểu tượng bên trong, gán `unknown` và tick `needs_review=true` |
