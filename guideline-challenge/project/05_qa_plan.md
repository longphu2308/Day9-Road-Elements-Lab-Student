# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:**
  - **Ai review:** QA owners (**Trần Đức Quân** phụ trách chính file `05_qa_plan.md`, **Nguyễn Văn Trọng** phụ trách `06_calibration_report.csv` và `07_blind_handoff/`) đóng vai trò QA Reviewer độc lập. Áp dụng nguyên tắc kiểm tra chéo (cross-review): annotator tuyệt đối không tự review bài dán nhãn của chính mình để tránh điểm mù chủ quan. Spec owner (**Lại Hoàng Duy**) đóng vai trò trọng tài chuyên môn giải quyết các trường hợp bất đồng quy tắc hoặc nghi vấn lỗ hổng guideline.
  - **Review bao nhiêu (Sampling Rate):**
    - **Self-QC (100%):** Mọi annotator bắt buộc tự kiểm tra 100% số ảnh và bounding box trước khi bàn giao qua Self-QC checklist: không còn thuộc tính `__undefined__`, không gộp cụm biển xếp chồng, không dính cột/giá đỡ, biển nhỏ <15 px đã loại trừ đúng quy định.
    - **Vòng Calibration nội bộ:** Review **100% số ảnh** (toàn bộ 7 ảnh calibration GTS06–GTS12) giữa tất cả các thành viên để tính độ tương đồng (Inter-Annotator Agreement - IAA), rà soát bất đồng và căn chỉnh tư duy trước khi chốt gold standard.
    - **Vòng Production / Blind Handoff:**
      - **100% mẫu rủi ro cao (High-risk):** Rà soát toàn bộ các ảnh có cờ `needs_review = true`, ảnh mang tag rủi ro (`critical`, `conflict`, `occlusion`, `small_far`, `ambiguity`) và ảnh negative test (ảnh zero-box).
      - **30% – 50% mẫu thông thường (Normal):** Lấy mẫu ngẫu nhiên từ các ảnh có điều kiện quan sát chuẩn để đánh giá chất lượng hình học và độ tuân thủ lề.
      - **100% mẫu của annotator mới hoặc người có lịch sử lỗi cao:** Nếu phát hiện ≥1 lỗi Critical hoặc tỷ lệ lỗi Major >5% trong đợt kiểm tra trước, toàn bộ batch tiếp theo của thành viên đó sẽ bị kiểm tra 100%.

- **Chọn sample theo rule nào** (Stratified Risk-based Sampling):
  - **Tầng 1 - High-risk & Edge-cases (100% audit):**
    - Các ảnh chứa cụm biển xếp chồng (stacked panels như GTS06, GTS09), biển phụ (supplementary panel), biển có nội dung mâu thuẫn về tốc độ (như GTS12).
    - Các ảnh có biển nhỏ/xa ở vùng ranh giới 15–20 px (như GTS02, GTS24) hoặc điều kiện môi trường suy giảm (sương mù GTS04, bóng râm gắt GTS02).
    - Toàn bộ bounding box có gán nhãn `sign_family = unknown` hoặc cờ `needs_review = true`.
  - **Tầng 2 - Negative & Ambiguity verification (100% audit):**
    - Toàn bộ ảnh không có biển báo hợp lệ (như GTS07) hoặc ảnh chứa vật thể dễ gây nhầm lẫn như bảng hiệu thương mại/quảng cáo ven đường (như bảng "Biergarten" trong GTS28) nhằm kiểm tra lỗi False Positive (vẽ thừa ngoài scope) và False Negative (bỏ sót).
  - **Tầng 3 - Random Sampling (Tối thiểu 30% batch bình thường):**
    - Chọn ngẫu nhiên có kiểm soát từ các ảnh `normal` (như GTS05, GTS21) để kiểm tra độ chính xác hình học (IoU) và phòng ngừa lỗi lơ đễnh/chủ quan.

- **Issue được ghi ở đâu, đóng thế nào:**
  - **Nơi ghi nhận:** Issue được ghi nhận tập trung tại bảng theo dõi lỗi QA nội bộ (`qa_issue_log` trong thư mục quản lý dự án) và comment trực tiếp trên từng bounding box/frame của CVAT task. Cấu trúc mỗi issue gồm: `Issue_ID`, `Sample_ID`, `Object_ID / Box_Coordinates`, `Annotator`, `Reviewer`, `Defect_Severity` (Critical / Major / Minor / Question), `Error_Category` (Missing, Misclass, Geometry, False Positive, Schema), `Description / Feedback`, `Status` (Open / Reworked / Verified_Closed).
  - **Quy trình đóng issue (Issue Lifecycle):**
    1. **Log & Assign (Open):** Reviewer phát hiện lỗi, tạo issue, gắn severity, mô tả cụ thể điều khoản guideline bị vi phạm và chuyển lại cho annotator thực hiện.
    2. **Rework:** Annotator mở task trên CVAT, chỉnh sửa theo feedback, ghi chú giải trình (nếu có) và cập nhật trạng thái issue thành `Reworked`.
    3. **Re-inspection (Đóng issue):** Duy nhất Reviewer (QA owner) mới có quyền kiểm tra lại (Re-inspection). Nếu đạt yêu cầu, QA chuyển trạng thái sang `Verified_Closed`. Nếu sửa chưa đạt hoặc phát sinh lỗi mới, issue bị trả về `Open` kèm ghi nhận lỗi lặp lại. Annotator tuyệt đối không được tự ý đóng issue.

- **Khi phát hiện guideline gap thì update và version ra sao:**
  - **Nhận diện gap:** Khi xuất hiện issue dạng `Question` hoặc có tranh chấp phân loại giữa annotator và reviewer mà `02_guideline.md` chưa có rule rõ ràng hoặc rule hiện hành gây hiểu đa nghĩa khi áp dụng thực tế (ví dụ: cách xác định ranh giới che khuất >40% hay xử lý biển phụ đặc biệt).
  - **Quy trình cập nhật & Đánh số phiên bản:**
    1. Annotator tạm thời đặt `sign_family = unknown` và bật `needs_review = true` để không tắc nghẽn luồng dán nhãn.
    2. QA owner ghi nhận tình huống vào `07_blind_handoff/clarification_log.csv` hoặc bảng thảo luận kỹ thuật.
    3. Tổ chức họp nhanh 10–15 phút giữa Spec owner (Lại Hoàng Duy), QA owner (Trần Đức Quân) và Gold owner (Lê Hữu Nghĩa) để thống nhất giải pháp dựa trên downstream contract (ưu tiên tuyệt đối an toàn vận hành xe tự hành).
    4. Spec owner sửa đổi trực tiếp vào `02_guideline.md`, bổ sung edge case vào `04_edge_cases/edge_case_cards.md` (nếu là tình huống điển hình mới).
    5. Nâng phiên bản guideline theo nguyên tắc:
       - `v1` → `v2`: Sau khi kết thúc đợt Calibration nội bộ và cập nhật edge cases EC-01 đến EC-08.
       - `v2` → `v3`: Sau khi nhận bài blind handoff từ nhóm peer và giải quyết các câu hỏi/feedback thực tế.
    6. Ghi chép chi tiết nguyên nhân, bằng chứng thực tế (sample_id, dòng calibration report, feedback) và thay đổi quy tắc vào `08_revision_log.md`.
    7. Broadcast thông báo cập nhật quy tắc cho toàn bộ thành viên và cập nhật lại guideline trong phần task guide trên CVAT.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Sai sót đe dọa trực tiếp đến an toàn vận hành xe tự hành (Safety-critical violation). Bỏ sót hoàn toàn (False Negative) hoặc phân loại sai nhóm biển điều tiết giao thông có tính cưỡng chế/cảnh báo nguy hiểm cao (`prohibitory`, `mandatory`, `danger`) khi biển còn đủ điều kiện quan sát (cạnh dài nhất ≥15 px, che khuất ≤40%). | - Bỏ sót biển hạn chế tốc độ 80 (`speed_limit_80`) hoặc cấm vượt trong GTS06.<br>- Nhầm biển hướng đi bắt buộc (`mandatory` vòng xuyến trong GTS20) thành biển `other` hoặc `unknown`.<br>- Nhầm biển cảnh báo nguy hiểm (`danger` trong GTS04) thành IGNORE do sương mù làm mờ tương phản.<br>- Nhầm biển báo giao thông hợp lệ thành biển quảng cáo ngoài scope rồi bỏ qua. | **REJECT toàn bộ batch của annotator đó.** Dừng luồng dán nhãn (Stop-the-line) để tìm nguyên nhân gốc; yêu cầu annotator rà soát 100% số ảnh đã làm; QA tiến hành re-audit 100% toàn bộ ảnh của annotator đó trước khi cho phép tiếp tục. |
| Major | Sai sót làm suy giảm hiệu năng nhận diện hoặc sai lệch cấu trúc dữ liệu downstream nhưng chưa trực tiếp gây ra tai nạn khẩn cấp, hoặc vi phạm nghiêm trọng quy chuẩn schema dữ liệu. | - Gộp 2–3 biển xếp chồng hoặc biển chính + biển phụ thành 1 bounding box duy nhất (vi phạm EC-01, EC-04).<br>- Bỏ sót biển phụ (supplementary panel) có nội dung cảnh báo độc lập.<br>- Tạo bounding box cho biển quảng cáo thương mại ngoài scope (False Positive, ví dụ bảng "Biergarten" trong GTS28).<br>- Bỏ sót giá trị thuộc tính kỹ thuật, để nguyên `sign_family = __undefined__` khi bàn giao.<br>- Cố tình đoán mò phân loại thay vì gán `sign_family = unknown` khi biển bị che >40% không đọc được nội dung (vi phạm EC-07).<br>- Vẽ bounding box cho candidate cực nhỏ dưới 15 px (vi phạm ngưỡng min-size EC-06, gây nhiễu label).<br>- Bounding box sai lệch nghiêm trọng: IoU < 0.60 hoặc trùm cả cột đỡ kim loại lớn. | **Bắt buộc REWORK trên từng instance bị lỗi.** Annotator phải sửa chữa và nộp lại trong vòng 30 phút. Nếu tỷ lệ lỗi Major vượt quá 5% tổng số instance trong batch kiểm tra ngẫu nhiên, nâng tỷ lệ kiểm tra lên 100% toàn bộ batch đó. |
| Minor | Sai lệch nhỏ về chất lượng hình học (geometry tolerance) hoặc định dạng kỹ thuật không làm thay đổi ngữ nghĩa phân loại và không làm mất đối tượng. | - Bounding box bị lỏng hoặc cắt nhẹ viền biển: sai lệch từ 3 px đến 5 px mỗi cạnh so với mặt biển nhìn thấy (nhưng IoU vẫn đạt từ 0.70 đến 0.85).<br>- Biển bị cắt mép ảnh (truncated như GTS11) nhưng bounding box vẽ lấn ra ngoài khung ảnh 1–2 px thay vì dừng chuẩn tại mép ảnh.<br>- Lỗi định dạng nhẹ trong phần comment ghi chú hoặc đặt tên nhãn phụ không chuẩn. | Annotator nhận feedback và sửa nhanh (Quick fix) trên instance lỗi trong đợt chỉnh sửa tiếp theo; cho phép tích hợp sửa hàng loạt mà không cần dừng batch kiểm tra. |
| Question | Tình huống mơ hồ cao (ambiguity), góc nhìn/thời tiết quá khắc nghiệt hoặc dữ liệu đa nghĩa mà guideline hiện hành chưa có tiền lệ rõ ràng, cần ý kiến hội chẩn của chuyên gia (Expert Escalation). | - Biển báo bị cành cây che khuất ranh giới ~40–45% diện tích mặt biển và nằm sâu trong bóng râm tối gắt (harsh shadow như GTS02/EC-07), không thể xác định chắc chắn ký hiệu bên trong.<br>- Candidate ở khoảng cách xa mờ nhòe có kích thước đúng ranh giới 14–16 px, khó khẳng định là biển báo hay nắp cống ven đường.<br>- Hai biển báo cự ly gần có thông tin xung đột bất thường tại đoạn đường công trường tạm thời (như GTS12). | Annotator tạo bounding box, đặt `sign_family = unknown` và tích chọn checkbox `needs_review = true`. Chuyển ngay cho QA Owner và Spec Owner phân giải trong vòng 15 phút. Quyết định thống nhất sẽ được bổ sung vào Edge-case library và cập nhật vào guideline version tiếp theo. |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| **Critical Defect Rate (CDR)** | $\text{CDR} = \frac{\text{Số lỗi Critical phát hiện}}{\text{Tổng số instance đã QA review}} \times 100\%$ | Đo lường mức độ vi phạm an toàn nghiêm trọng đối với hệ thống xe tự hành. Bài toán GTSDB phân loại biển báo phục vụ trực tiếp cho module lập kế hoạch di chuyển (motion planner), do đó tỷ lệ lỗi Critical bắt buộc phải tiệm cận 0 để đảm bảo xe không vi phạm luật giao thông. |
| **Decision Accuracy (DA) / Transfer Agreement** | $\text{DA} = \frac{\text{Số decision đúng (Label, Ignore, Family, Escalate)}}{\text{Tổng số decision đánh giá trong Gold / Peer Review}} \times 100\%$ | Đo lường độ chuẩn xác toàn diện của annotator theo đúng downstream contract được quy định tại mục "Output chấm được" trong `01_problem_statement.md`, bao gồm cả khả năng loại trừ vật thể âm (negative test như GTS07, GTS28). |
| **Geometry Compliance Rate (GCR) & Mean IoU** | $\text{IoU} = \frac{\text{Area}(B_{\text{annotated}} \cap B_{\text{gold}})}{\text{Area}(B_{\text{annotated}} \cup B_{\text{gold}})}$<br>$\text{GCR} = \frac{\text{Số box có } \text{IoU} \ge 0.85 \text{ và sai lệch cạnh } \le 2\text{px}}{\text{Tổng số box được kiểm tra}} \times 100\%$ | Đánh giá việc tuân thủ quy tắc tight-visible box (ôm sát mặt biển nhìn thấy, không lấy cột đỡ). Đây là tiêu chuẩn cần thiết để thuật toán object detection học đúng biên đặc trưng hình học mà không bị nhiễu nền. |
| **Undefined Attribute Escape Rate (UAER)** | $\text{UAER} = \frac{\text{Số box còn giữ giá trị thuộc tính } \text{'__undefined__'}}{\text{Tổng số box xuất xưởng (Export)}} \times 100\%$ | Bắt buộc kiểm soát lỗi im lặng do cơ chế default của công cụ CVAT. Giúp phát hiện các box dán nhãn dở dang trước khi đưa dữ liệu vào pipeline huấn luyện mô hình. |
| **False Positive Rate on Non-signs ($\text{FPR}_{\text{neg}}$)** | $\text{FPR}_{\text{neg}} = \frac{\text{Số box vẽ nhầm vào biển quảng cáo/thương mại ngoài scope}}{\text{Tổng số candidate ngoài scope được kiểm tra}} \times 100\%$ | Đảm bảo xe tự hành không nhận diện nhầm biển quảng cáo (như bảng "Biergarten" trong GTS28) thành biển báo giao thông, tránh các hành vi phanh gấp hoặc đổi làn đột ngột gây nguy hiểm (phantom braking). |

Metric high-risk tách riêng (ví dụ critical defect escape rate):
- **Critical Defect Escape Rate (CDER):**
  - *Công thức:*
    $$\text{CDER} = \frac{\text{Số lỗi Critical lọt qua Self-QC và QA được phát hiện tại Quality Gate hoặc Blind Test}}{\text{Tổng số đối tượng High-Risk trong ground truth}} \times 100\%$$
  - *Mục tiêu bắt buộc:* **0.0%** (Chính sách Zero Tolerance). Bất kỳ một lỗi bỏ sót biển cấm/nguy hiểm nào lọt xuống downstream đều coi như Quality Gate thất bại, kích hoạt cơ chế phong tỏa dữ liệu và audit toàn diện.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  - Critical Defect Rate (CDR) = 0% (Tuyệt đối không có lỗi Critical)
  - Critical Defect Escape Rate (CDER) = 0% (Không lọt bất kỳ lỗi nghiêm trọng nào)
  - Undefined Attribute Escape Rate (UAER) = 0% (100% box có semantic label hợp lệ)
  - Decision Accuracy (DA) >= 90% trên tập đánh giá
  - Mean IoU >= 0.85 và Geometry Compliance Rate >= 90% (sai lệch <= 2px mỗi cạnh cho sign thông thường)
  - Tỷ lệ lỗi Major <= 3% trên tổng số instance được review
  - Tỷ lệ lỗi Minor <= 8% trên tổng số instance được review
  - 100% trường hợp Question / needs_review đã được xử lý và đóng trạng thái

REWORK if:
  - Xuất hiện đúng 1 lỗi Critical trong batch lấy mẫu (cần cô lập và sửa ngay) HOẶC
  - Tỷ lệ lỗi Major nằm trong khoảng > 3% và <= 8% HOẶC
  - Decision Accuracy (DA) nằm trong khoảng 75% <= DA < 90% HOẶC
  - Mean IoU nằm trong khoảng 0.70 <= IoU < 0.85 HOẶC
  - Tỷ lệ lỗi Minor nằm trong khoảng > 8% và <= 15% HOẶC
  - Còn từ 1 đến 2 box sót giá trị '__undefined__'
  Hành động: Trả batch về cho annotator gốc; gửi kèm danh sách ID và tọa độ instance lỗi; annotator phải sửa xong và nộp lại trong vòng 45 phút; QA reviewer tiến hành re-audit 100% các instance đã rework.

REJECT / ESCALATE if:
  - Xuất hiện >= 2 lỗi Critical trong cùng một batch kiểm tra HOẶC
  - Tỷ lệ lỗi Major > 8% HOẶC
  - Decision Accuracy (DA) < 75% HOẶC
  - Mean IoU < 0.70 (box vẽ cẩu thả, bao cả cột đỡ hoặc trùm nhiều biển) HOẶC
  - Tỷ lệ lỗi Minor > 15% HOẶC
  - Tái phát lỗi Critical sau 1 lượt REWORK HOẶC
  - Phát sinh bất đồng quan điểm chuyên môn không thể giải quyết giữa Annotator và Reviewer
  Hành động: Từ chối nghiệm thu toàn bộ batch dữ liệu. Đình chỉ bàn giao (Stop-the-line). Họp khẩn cấp Spec owner, QA owner và Lab Coach để xác định nguyên nhân cốt lõi (annotator chưa hiểu guideline hay guideline tồn tại lỗ hổng); cập nhật lại guideline nếu cần; tiến hành đào tạo lại annotator và yêu cầu dán nhãn lại từ đầu toàn bộ batch.
```

Trade-off:
- **Đánh đổi Giám sát An toàn (Safety/Risk) vs Tốc độ & Chi phí Dán nhãn (Cost/Throughput):**
  - Thiết lập ngưỡng **Zero Tolerance (0%) cho Critical Defects** đòi hỏi quy trình QA phải kiểm tra 100% các mẫu rủi ro cao và thực hiện cross-review độc lập. Điều này làm tăng thời gian kiểm thử khoảng 25–30% so với phương pháp kiểm tra ngẫu nhiên thông thường. Tuy nhiên, đối với bài toán nhận thức xe tự hành (ADAS/Autonomous Driving), cái giá phải trả cho một lỗi bỏ sót biển báo cấm vượt hay giới hạn tốc độ ngoài đời thực là tai nạn giao thông nghiêm trọng. Do đó, việc chịu chi phí nhân lực QA cao hơn ở khâu tiền kỳ là hoàn toàn xứng đáng và cần thiết để loại trừ rủi ro an toàn downstream.
- **Đánh đổi Độ chính xác hình học (Geometry Tolerance) vs Năng suất thực thi:**
  - Nhóm đặt ngưỡng sai lệch hình học cho phép là 2 px mỗi cạnh và chấp nhận tỷ lệ lỗi Minor lên đến 8% cho các biển nhỏ (15–20 px) hoặc biển bị cắt mép ảnh. Việc yêu cầu độ chính xác tuyệt đối từng điểm ảnh (pixel-perfect) trên các đối tượng kích thước nhỏ ở khoảng cách xa sẽ khiến thời gian dán nhãn tăng gấp đôi mà không mang lại cải thiện đáng kể cho các mô hình object detection (vốn chỉ cần IoU ≥ 0.70 để phát hiện đối tượng chuẩn xác). Nhóm ưu tiên nguồn lực để kiểm soát 100% tính toàn vẹn của nhãn phân loại `sign_family` và cấu trúc phân rã từng panel độc lập.
- **Đánh đổi giữa Unknown / Needs Review và Tự suy đoán (Label Precision vs Coverage):**
  - Guideline khuyến khích annotator sử dụng nhãn `unknown` kết hợp cờ `needs_review = true` khi biển bị che khuất >40% hoặc điều kiện quan sát quá kém thay vì cố gắng suy đoán phân loại. Sự đánh đổi này có thể làm giảm nhẹ tỷ lệ gán nhãn chi tiết (coverage) tại một vài trường hợp biên, nhưng giúp ngăn chặn hoàn toàn việc "bơm nhãn giả / nhãn ảo" (label hallucination/noise) vào dữ liệu huấn luyện, bảo vệ độ tin cậy tuyệt đối của module điều khiển tự hành.
