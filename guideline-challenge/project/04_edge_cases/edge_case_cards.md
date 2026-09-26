# Edge-case library

---

CASE ID: EC-01
Sample: GTS06
Scene: Đường cao tốc ngoại ô ban ngày
Observation: Cột biển báo gắn 2 biển xếp chồng (stacked): bên trên là biển Hạn chế tốc độ 80 (hình tròn viền đỏ), ngay bên dưới là biển Cấm vượt (viền đỏ có 2 xe ô tô).
Decision: LABEL
Expected: 2 bbox riêng biệt sát khít từng biển (bbox 1: speed_limit_80; bbox 2: no_overtaking). Không bao gồm cột đỡ.
Rationale: Downstream perception model cần phát hiện độc lập từng lệnh giao thông để đồng thời kích hoạt kiểm soát tốc độ và vô hiệu hóa chế độ tự động vượt xe.
Common mistake: Annotator vẽ 1 bounding box to gộp cả hai biển thành một hoặc vẽ trùm cả thanh kim loại nối hai biển.
Diversity: conflict

---

CASE ID: EC-02
Sample: GTS04
Scene: Tuyến đường nông thôn có sương mù dày đặc (heavy fog)
Observation: Biển báo Danger / Warning hình tam giác viền đỏ nằm bên phải đường nhưng bị sương mù làm suy giảm tương phản nghiêm trọng, chỉ nhìn rõ hình dáng khung tam giác mờ nhạt.
Decision: LABEL
Expected: Bbox bao quanh viền tam giác mờ, class: danger_warning, attribute: occlusion_fog=true.
Rationale: [CRITICAL RISK] Biển cảnh báo nguy hiểm trong điều kiện tầm nhìn kém (sương mù) nếu bị bỏ sót (False Negative) sẽ khiến hệ thống lái tự động không kịp giảm tốc hoặc cảnh báo tài xế, dẫn đến nguy cơ va chạm chết người.
Common mistake: Annotator thấy biển mờ nên tự ý đánh nhãn IGNORE vì nghĩ model không học được.
Diversity: critical

---

CASE ID: EC-03
Sample: GTS28
Scene: Đoạn đường nông thôn ngang qua khu dân cư / nhà hàng
Observation: Bên lề phải xuất hiện biển hiệu thương mại chữ "Biergarten" hình chữ nhật màu xanh/trắng, kích thước và vị trí lắp đặt tương tự biển chỉ dẫn giao thông.
Decision: IGNORE
Expected: Không tạo bounding box cho bảng hiệu này.
Rationale: Biển quảng cáo/thương mại không thuộc quy chuẩn báo hiệu đường bộ (StVO). Gán nhãn sẽ khiến model downstream báo sai biển giao thông (False Positive).
Common mistake: Annotator thấy bảng gắn cạnh đường là tạo bbox hoặc nhầm lẫn thành biển chỉ dẫn khu dân cư.
Diversity: ambiguity

---

CASE ID: EC-04
Sample: GTS09
Scene: Đường quốc lộ ngoài đô thị có dải phân cách
Observation: Cột biển báo tích hợp cụm 3 biển: Biển tròn Hạn chế tốc độ 70, Biển phụ giải thích thời tiết bên dưới và biển Cấm vượt. Các biển nằm sát mép nhau và thanh đỡ che khuất một phần rìa.
Decision: LABEL
Expected: Vẽ 3 bbox độc lập cho từng phần tử theo guideline phân rã biển chính - biển phụ.
Rationale: Giúp model học được tính năng phân tách các biển báo có liên kết ngữ nghĩa (speed limit kèm điều kiện thời tiết).
Common mistake: Vẽ thiếu biển phụ hoặc vẽ box chồng lấn quá 20% diện tích của nhau.
Diversity: occlusion

---

CASE ID: EC-05
Sample: GTS12
Scene: Đoạn đường cao tốc qua khu vực thi công / sửa đường
Observation: Hai biển giới hạn tốc độ xuất hiện gần nhau với giá trị mâu thuẫn (biển 70km/h và biển 80km/h cách nhau cự ly ngắn).
Decision: LABEL
Expected: Gán nhãn cả 2 biển với giá trị tốc độ tương ứng, không tự ý chọn 1 biển để loại bỏ.
Rationale: Hệ thống ADAS planning cấp cao (path planner) cần thông tin của cả hai để áp dụng quy tắc an toàn (chọn giá trị nhỏ hơn 70km/h). Annotator không được phán đoán thay planner.
Common mistake: Annotator chỉ vẽ biển có giá trị nhỏ hơn vì nghĩ biển kia bị gắn nhầm hoặc hết hiệu lực.
Diversity: conflict

---

CASE ID: EC-06
Sample: GTS08
Scene: Đường thẳng tầm nhìn xa, hai bên là hàng cây
Observation: Biển báo hình tròn ở rất xa phía trước (>50m), kích thước trên ảnh chỉ khoảng 12x12 pixel, chi tiết số bên trong không thể đọc rõ bằng mắt thường.
Decision: IGNORE
Expected: Không gán nhãn (bỏ qua theo ngưỡng min-size).
Rationale: Vật thể dưới 15px không đủ đặc trưng trực quan để mạng nơ-ron học đặc trưng hình học, gán vào sẽ tạo nhiễu nhãn (noisy labels) làm giảm mAP.
Common mistake: Annotator cố zoom 400% để đoán nội dung biển và vẽ bbox quá nhỏ (dưới min-size quy định).
Diversity: small_far

---

CASE ID: EC-07
Sample: GTS02
Scene: Đường rừng nhiều cây cối, ánh nắng xiên tạo bóng râm gắt (harsh shadows)
Observation: Biển báo tròn nằm trong vùng bóng râm tối của tán cây, bị cành lá che khuất 45% diện tích mặt biển, không nhận diện được ký hiệu bên trong là cấm vượt hay giới hạn tốc độ.
Decision: ESCALATE
Expected: Gắn cờ ESCALATE kèm comment: "Occluded >40% by foliage under harsh shadow, text unreadable - request expert review".
Rationale: Vượt quá ngưỡng tự phán đoán của annotator thông thường; cần ý kiến của Domain Expert/Lead để thống nhất quy chuẩn che khuất cho toàn bộ batch dữ liệu.
Common mistake: Tự ý phán đoán class theo linh cảm hoặc tự gắn nhãn UNKNOWN mà không báo cáo lên hệ thống.
Diversity: escalation

---

CASE ID: EC-08
Sample: GTS11
Scene: Giao lộ đường nhánh nông thôn
Observation: Biển báo người đi bộ qua đường (Pedestrian Crossing) bị nghiêng góc 30 độ do va quẹt hoặc gió bão, mép biển nằm sát mép ảnh (truncation).
Decision: LABEL
Expected: Vẽ bounding box bao quanh phần nhìn thấy của biển (không ngoại suy phần bị cắt), attribute: truncated=true.
Rationale: Đảm bảo xe tự hành vẫn nhận biết biển hiệu dù trạng thái vật lý của biển bị biến dạng/nghiêng lệch ngoài thực tế.
Common mistake: Cố vẽ bbox mở rộng ra ngoài rìa ảnh để "đoán" đủ kích thước biển.
Diversity: truncation
