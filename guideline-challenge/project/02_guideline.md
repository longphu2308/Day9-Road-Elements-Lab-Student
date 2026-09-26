# Annotation guideline — Traffic sign family classification

**Version:** v2

> Bản v2 cập nhật sau khi rà soát edge-case library EC-01 → EC-08.  
> Mọi rule mà peer cần biết phải nằm trong file này. Không dùng hidden rule chỉ giải thích bằng miệng.

## 1. Objective + scope

Mục tiêu là tạo annotation nhất quán cho **traffic sign ở mức sign family** trên ảnh tĩnh GTSDB trong `guideline-challenge/data/gtsdb/`. Bộ ảnh challenge có cả ảnh không có biển, ảnh chỉ có một biển, ảnh có nhiều biển cùng lúc, cụm biển xếp chồng, biển phụ, biển rất nhỏ/xa, biển bị che/cắt và candidate dễ nhầm với bảng thương mại; vì vậy guideline ưu tiên tính nhất quán trong phát hiện instance, geometry và phân loại family.

Downstream là module perception của hệ thống hỗ trợ lái/xe tự hành. Output cần cho mỗi traffic sign gồm:
- vị trí bằng rectangle;
- `sign_family`;
- cờ `needs_review` cho case cần escalation.

Bài này chỉ phân loại ở mức **family**, không yêu cầu 43 `sign_class` cụ thể của GTSDB. Annotator chỉ ghi nhận những gì nhìn thấy trong ảnh, không quyết định biển nào đang có hiệu lực hơn, biển nào planner nên ưu tiên hoặc biển nào có thể bị lắp sai.

### Trong scope

- Mặt trước của traffic sign thật xuất hiện trong ảnh.
- Biển nhỏ hoặc xa nhưng đạt ngưỡng kích thước ở mục 6 và vẫn có đủ bằng chứng để nhận biết là traffic sign.
- Biển bị che/cắt một phần nhưng phần còn nhìn thấy vẫn đủ để xác định đây là traffic sign.
- Biển nghiêng, xoay hoặc bị cắt bởi mép ảnh nhưng mặt biển vẫn nhận biết được.
- Nhiều biển trên cùng cột hoặc cùng cụm: mỗi physical panel là một instance riêng.
- Biển phụ/supplementary panel có nội dung giao thông riêng: một panel = một instance riêng.
- Các biển thuộc cả bốn family `prohibitory`, `mandatory`, `danger`, `other`.

### Ngoài scope

- Quảng cáo, bảng cửa hàng, bảng tên thương mại hoặc bảng ven đường không phải road traffic sign.
- Mặt sau của biển nếu không có thông tin mặt biển cần cho task.
- Reflection của traffic sign trên kính/gương.
- Candidate quá nhỏ dưới ngưỡng mục 6.
- Candidate quá mờ, quá khuất hoặc quá thiếu thông tin đến mức không thể xác định đáng tin cậy đó là traffic sign và không có lý do hợp lý để escalation.

## 2. Annotation unit

- **Loại dữ liệu:** ảnh tĩnh.
- **Đơn vị annotation:** một physical sign panel = một instance.
- **CVAT tool:** Rectangle / Shape.
- Không dùng Track hoặc interpolation.

Quy tắc instance:

1. Một panel có nội dung riêng = một box.
2. Hai hoặc nhiều panel xếp chồng trên cùng cột = nhiều box riêng.
3. Biển phụ/supplementary panel có nội dung giao thông riêng cũng là một panel riêng; không gộp vào biển chính.
4. Không dùng một box chung cho cả cụm biển.
5. Không gộp các biển giống nhau thành một instance chỉ vì chúng nằm gần nhau.
6. Nếu hai biển đưa ra thông tin có vẻ mâu thuẫn, ví dụ hai speed limit khác nhau, vẫn label **cả hai** nếu cả hai đều là traffic sign hợp lệ. Annotator không được tự chọn biển “đúng hơn” hoặc “còn hiệu lực hơn”.
7. Ảnh không có traffic sign hợp lệ thì để ảnh hoàn toàn không có rectangle; đây là output đúng, không phải annotation thiếu.

## 3. Geometry rule

Sử dụng **rectangle tight-visible**.

1. Box ôm sát **mặt biển nhìn thấy**.
2. Không lấy cột, giá đỡ, dây, nền hoặc khoảng trống quanh biển.
3. Không suy đoán kích thước thật của phần bị che; box chỉ bao phần nhìn thấy.
4. Nếu biển bị cắt bởi mép ảnh, box kết thúc tại mép ảnh. Không mở rộng box ra ngoài ảnh để đoán phần bị mất.
5. Biển nghiêng/xoay vẫn dùng rectangle axis-aligned nhỏ nhất hợp lý bao phần mặt biển nhìn thấy; không xoay box theo biển.
6. Mỗi panel trong cụm biển có box riêng, kể cả panel phụ nếu panel đó thuộc scope.
7. Với object đủ lớn để đánh giá ổn định, mục tiêu là lệch không quá khoảng **2 px mỗi cạnh** giữa các annotator.
8. Với biển nhỏ nhưng còn trong scope, ưu tiên tight box nhất quán theo pixel nhìn thấy; không vẽ rộng để “bù” cho phần chi tiết khó thấy.

## 4. Taxonomy

### 4.1 Object class

Chỉ dùng một object class:

- `traffic_sign`

### 4.2 Attribute `sign_family`

Allowed values:

- `prohibitory`
- `mandatory`
- `danger`
- `other`
- `unknown`

### `prohibitory`

Biển cấm hoặc hạn chế hành vi giao thông, ví dụ nhóm speed limit, no overtaking, no trucks. Trong GTSDB, các class competition-relevant thuộc nhóm này gồm 0, 1, 2, 3, 4, 5, 7, 8, 9, 10, 15, 16.

Nếu hai biển `prohibitory` có giá trị khác nhau xuất hiện cùng ảnh, vẫn label từng biển độc lập. Không dùng logic planner để loại biển có giới hạn cao hơn/thấp hơn.

### `mandatory`

Biển yêu cầu phương tiện thực hiện một hướng/hành vi, thường là các biển tròn nền xanh như đi trái/phải/thẳng, keep left/right, roundabout. GTSDB dùng các class 33–40 cho nhóm này.

### `danger`

Biển cảnh báo nguy hiểm hoặc điều kiện đường phía trước, thường có dạng tam giác. GTSDB dùng các class 11, 18–31 cho nhóm này.

Điều kiện môi trường như sương mù, bóng râm hoặc tương phản thấp **không tự động biến biển thành `unknown` hoặc IGNORE**. Nếu hình dạng/nội dung còn đủ bằng chứng để xác định family `danger`, vẫn chọn `danger`.

### `other`

Traffic sign hợp lệ nhưng không thuộc ba nhóm trên. **`other` vẫn phải label và không đồng nghĩa với “không quan trọng”.** Trong GTSDB, nhóm này có thể bao gồm restriction-end, priority road, give way, stop, no entry, biển phụ/supplementary panel và một số class khác ngoài ba superclass competition-relevant.

### `unknown`

Annotator chắc chắn object là traffic sign nhưng ảnh không cung cấp đủ bằng chứng để chọn một trong bốn family trên.

### Default

Trong CVAT, `sign_family` có default kỹ thuật là `__undefined__`.

- `__undefined__` không phải annotation hoàn chỉnh.
- Trước export, mọi rectangle phải có một giá trị semantic: `prohibitory`, `mandatory`, `danger`, `other` hoặc `unknown`.
- Không đặt `unknown` làm default, để tránh annotator quên phân loại biển nhìn rõ.

### Attribute `needs_review`

- `false`: guideline đủ rõ để annotator tự quyết định.
- `true`: case không thể resolve chắc chắn bằng guideline v2 và cần reviewer/QA owner quyết định.

Phân biệt:
- `sign_family = unknown`, `needs_review = false`: chắc chắn là traffic sign nhưng family không đọc đủ từ ảnh; rule xử lý đã rõ.
- `sign_family = unknown`, `needs_review = true`: bản thân quyết định label/family còn tranh chấp, mức che khuất cao hoặc có ≥2 cách xử lý hợp lý và cần escalation.

**Không tạo thêm attribute** như `occlusion_fog` hoặc `truncated` trong task hiện tại. Hai trạng thái này được xử lý bằng geometry + `sign_family` + `needs_review` theo mục 6–7 để giữ schema CVAT đồng nhất với `03_cvat_labels.json`.

## 5. Inclusion / exclusion

### LABEL

Bắt buộc tạo rectangle khi:

- chắc chắn object là traffic sign thật;
- biển nhỏ/xa nhưng vẫn đạt ngưỡng kích thước ở mục 6 và nhận biết được là traffic sign;
- biển mờ do sương, bóng râm hoặc tương phản thấp nhưng vẫn đủ bằng chứng nhận diện object;
- biển bị che/cắt một phần nhưng còn đủ phần mặt biển để xác định object;
- biển bị nghiêng/xoay nhưng vẫn thấy mặt biển;
- biển thuộc bất kỳ family nào, kể cả `other`;
- biển phụ có nội dung giao thông riêng;
- có nhiều biển cùng cột/cùng cụm hoặc có nội dung mâu thuẫn: label tất cả panel hợp lệ độc lập;
- chắc chắn là traffic sign nhưng chưa đủ bằng chứng phân family → label + `unknown`.

### IGNORE

Không tạo rectangle khi:

- object rõ ràng không phải traffic sign;
- chỉ là quảng cáo/bảng cửa hàng/bảng tên thương mại;
- chỉ thấy reflection;
- chỉ thấy backside không mang thông tin của mặt biển;
- cạnh dài nhất của candidate **< 15 px** trong ảnh gốc;
- bằng chứng quá ít để xác định object là traffic sign và case không đạt tiêu chí escalation ở mục 7.

**Ngưỡng 15 px chỉ dùng cho candidate cực nhỏ.** Các sign khoảng 17–20 px vẫn có thể thuộc scope và phải label nếu nhận biết được.

## 6. Visibility / occlusion / small signs

### Biển nhỏ / xa

Quy tắc v2 dùng ngưỡng vận hành rõ ràng để tránh annotator tự đặt threshold khác nhau:

- nếu **cạnh dài nhất < 15 px** trên ảnh gốc → IGNORE;
- nếu cạnh dài nhất ≥ 15 px và nhận biết được là traffic sign + family rõ → LABEL + family;
- nếu cạnh dài nhất ≥ 15 px, chắc chắn là traffic sign nhưng family không rõ → LABEL + `unknown`;
- nếu cạnh dài nhất ≥ 15 px nhưng việc object có phải traffic sign hay không vẫn có ≥2 cách hiểu hợp lý → ESCALATE.

Không zoom rồi suy diễn ký hiệu/nội dung không thực sự có bằng chứng. Có thể zoom để đặt box chính xác, nhưng không dùng zoom như lý do để “đoán” family.

### Cụm biển / biển xếp chồng / biển phụ

Luôn:

- quét từng physical panel riêng;
- một panel = một rectangle;
- panel phụ có message riêng = rectangle riêng;
- family được quyết định **theo từng panel**, không suy ra từ biển phía trên/dưới hoặc từ cả cột;
- không quan tâm việc các panel có liên quan ngữ nghĩa hay có vẻ mâu thuẫn: annotation vẫn tách độc lập.

### Sương mù / bóng râm / tương phản thấp

- Môi trường xấu **không phải lý do tự động IGNORE**.
- Nếu vẫn chắc chắn đây là traffic sign và family còn đủ bằng chứng → LABEL + family.
- Nếu chắc chắn là traffic sign nhưng family không đọc được → `unknown`.
- Nếu chất lượng ảnh thấp đến mức việc có phải traffic sign hay không còn tranh chấp → ESCALATE hoặc IGNORE theo mục 7.

### Bị che một phần

Ước lượng tỷ lệ phần mặt biển bị che theo diện tích nhìn thấy/không nhìn thấy; đây là ngưỡng vận hành, không yêu cầu đo pixel chính xác.

- che **≤ 40%**: nếu chắc chắn là traffic sign, vẫn LABEL; family rõ thì chọn family, không rõ thì `unknown`;
- che **> 40%** nhưng vẫn chắc chắn object là traffic sign và family vẫn rõ từ bằng chứng còn lại → vẫn LABEL + family;
- che **> 40%** và family không thể resolve đáng tin cậy hoặc có ≥2 family/decision hợp lý → `sign_family=unknown`, `needs_review=true` để ESCALATE;
- nếu phần còn lại không đủ để xác định đây là traffic sign → IGNORE.

Không vẽ amodal box qua vật che.

### Bị cắt mép ảnh / biển nghiêng

Nếu phần còn lại vẫn nhận biết được là traffic sign:

- label phần nhìn thấy;
- box dừng tại mép ảnh;
- không ngoại suy phần nằm ngoài ảnh;
- góc nghiêng của biển không làm thay đổi family và không phải lý do IGNORE.

Nếu không còn đủ bằng chứng để xác định object thì IGNORE/ESCALATE theo mục 7.

### Blur / glare / low confidence

- chắc chắn là traffic sign nhưng family không rõ → `unknown`;
- không chắc object có phải traffic sign → ESCALATE nếu có ≥2 quyết định hợp lý; nếu không đủ bằng chứng tối thiểu thì IGNORE.

## 7. Ambiguity / escalation

| Tình huống | Quyết định | Thể hiện trong CVAT |
|---|---|---|
| Chắc chắn là traffic sign, family rõ | LABEL | Rectangle `traffic_sign`, chọn family, `needs_review=false` |
| Chắc chắn là traffic sign, family không đủ bằng chứng | UNKNOWN | Rectangle, `sign_family=unknown`, `needs_review=false` |
| Occlusion >40%, object chắc chắn là sign nhưng family/decision còn ≥2 cách hiểu hợp lý | ESCALATE | Rectangle, `sign_family=unknown`, `needs_review=true` |
| Candidate có khả năng là sign nhưng guideline/bằng chứng chưa đủ để chốt và vẫn có ≥2 quyết định hợp lý | ESCALATE | Rectangle candidate, `sign_family=unknown`, `needs_review=true` |
| Traffic sign hợp lệ nhưng ngoài 3 family chính hoặc là supplementary panel | LABEL | Rectangle, `sign_family=other` |
| Hai traffic sign có nội dung mâu thuẫn | LABEL cả hai | Hai rectangle độc lập, family của từng panel |
| Candidate có cạnh dài nhất <15 px | IGNORE | Không tạo rectangle |
| Rõ ràng không phải traffic sign / quảng cáo / reflection / backside ngoài scope | IGNORE | Không tạo rectangle |

Nguyên tắc:

- `UNKNOWN` = object chắc chắn là sign, chỉ thiếu thông tin để phân family.
- `ESCALATE` = annotator không thể áp dụng guideline chắc chắn hoặc có ≥2 quyết định hợp lý; trong schema hiện tại thể hiện bằng `unknown + needs_review=true`.
- Không tự tạo family mới.
- Không tự quyết định tính hiệu lực của biển dựa trên bối cảnh giao thông; nhiệm vụ annotation chỉ ghi nhận các panel nhìn thấy.
- Không đổi taxonomy bằng thỏa thuận miệng; thay đổi rule phải cập nhật guideline và revision log.

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.**

Không dùng Track, interpolation hoặc mutable temporal attribute.

## 9. Examples grounded in challenge data

Các sample dưới đây dùng để minh họa rule.

| Tình huống / evidence | Expected output | Rule áp dụng |
|---|---|---|
| Hai hoặc nhiều biển xếp chồng cùng cột | Mỗi physical panel một rectangle; không lấy cột/giá đỡ | EC-01 / multi-instance |
| Biển cảnh báo bị sương/bóng râm làm giảm tương phản nhưng hình dạng/family vẫn đủ rõ | LABEL, chọn family tương ứng; không IGNORE chỉ vì visibility xấu | EC-02 / low visibility |
| Bảng thương mại ven đường có hình thức gần giống biển chỉ dẫn | Không tạo rectangle | EC-03 / exclusion |
| Cụm biển chính + biển phụ | Mỗi panel một rectangle; supplementary panel thường `other` nếu không thuộc ba family chính | EC-04 / supplementary panel |
| Hai speed limit hoặc hai biển có thông tin có vẻ mâu thuẫn | LABEL cả hai; không chọn hộ planner | EC-05 / conflict |
| Candidate cực nhỏ khoảng 12×12 px | IGNORE vì cạnh dài nhất <15 px | EC-06 / small-far threshold |
| Biển bị che >40%, chắc chắn là sign nhưng family không resolve được | Rectangle + `unknown` + `needs_review=true` | EC-07 / escalation |
| Biển nghiêng và bị cắt bởi mép ảnh | Tight-visible box chỉ phần nhìn thấy, dừng ở mép ảnh | EC-08 / truncation |

Các ví dụ dataset đã dùng trong calibration/example vẫn tuân theo cùng rule:

- GTS01/GTS04: không gộp các panel nằm gần hoặc xếp chồng.
- GTS02/GTS03/GTS24/GTS26: sign khoảng 17–20 px vẫn không bị loại chỉ vì nhỏ nếu cạnh dài nhất ≥15 px và object nhận biết được.
- GTS07/GTS28: zero-box output là hợp lệ khi không có traffic sign thuộc scope.

## 10. Common mistakes

1. **Bỏ sót mọi biển nhỏ chỉ vì nghĩ “biển nhỏ thì bỏ”**  
   → Chỉ IGNORE tự động khi cạnh dài nhất <15 px. Sign 17–20 px vẫn có thể phải label.

2. **Cố label candidate 12×12 px bằng cách zoom rồi đoán**  
   → IGNORE theo threshold v2.

3. **Một box ôm cả cụm biển hoặc cả biển chính + biển phụ**  
   → Mỗi physical panel là một instance riêng.

4. **Box cả cột/giá đỡ**  
   → Tight box chỉ quanh mặt biển nhìn thấy.

5. **Bỏ qua biển vì sương mù/bóng râm**  
   → Visibility xấu không phải lý do IGNORE nếu object/family vẫn đủ bằng chứng.

6. **Chỉ giữ một biển khi hai biển có nội dung mâu thuẫn**  
   → LABEL cả hai; annotator không quyết định thay planner.

7. **Suy ra family của cả cột từ một panel**  
   → Một cụm có thể trộn nhiều family; classify từng panel độc lập.

8. **Coi `other` là object ngoài scope**  
   → `other` là một family hợp lệ; supplementary panel hợp lệ thường rơi vào `other` nếu không thuộc ba family chính.

9. **Nhầm `other` với `unknown`**  
   → `other`: biết traffic sign thuộc nhóm ngoài three main families. `unknown`: không đủ bằng chứng để xác định family.

10. **Biển bị che >40% nhưng vẫn tự đoán family**  
    → Nếu family không resolve đáng tin cậy, dùng `unknown + needs_review=true`.

11. **Vẽ box vượt mép ảnh cho biển truncated**  
    → Chỉ box phần nhìn thấy; không amodal/extrapolate.

12. **Tự thêm `occlusion_fog`, `truncated` hoặc attribute khác vào CVAT**  
    → Schema hiện tại chỉ có `sign_family` và `needs_review`; trạng thái visibility/truncation được xử lý bằng rule.

13. **Để `__undefined__` khi export**  
    → Annotation chưa hoàn tất.

14. **Rule chỉ thống nhất bằng miệng**  
    → Nếu peer cần rule đó, phải ghi vào guideline trước freeze.
