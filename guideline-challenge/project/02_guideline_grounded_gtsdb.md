# Annotation guideline — Traffic sign family classification

**Version:** v1

> Bản v1 dùng cho calibration. Sau calibration cập nhật lên v2; sau blind handoff cập nhật lên v3.  
> Mọi rule mà peer cần biết phải nằm trong file này. Không dùng hidden rule chỉ giải thích bằng miệng.

## 1. Objective + scope

Mục tiêu là tạo annotation nhất quán cho **traffic sign ở mức sign family** trên ảnh tĩnh GTSDB trong `guideline-challenge/data/gtsdb/`. Bộ ảnh challenge có cả ảnh không có biển, ảnh chỉ có một biển, ảnh có nhiều biển cùng lúc, cụm biển xếp chồng và biển rất nhỏ/xa; vì vậy guideline ưu tiên tính nhất quán trong phát hiện instance và phân loại family.

Downstream là module perception của hệ thống hỗ trợ lái/xe tự hành. Output cần cho mỗi traffic sign gồm:
- vị trí bằng rectangle;
- `sign_family`;
- cờ `needs_review` cho case cần escalation.

Bài này chỉ phân loại ở mức **family**, không yêu cầu 43 `sign_class` cụ thể của GTSDB.

### Trong scope

- Mặt trước của traffic sign thật xuất hiện trong ảnh.
- Biển nhỏ hoặc xa nhưng vẫn có đủ bằng chứng để nhận biết là traffic sign.
- Biển bị che/cắt một phần nhưng phần còn nhìn thấy vẫn đủ để xác định đây là traffic sign.
- Nhiều biển trên cùng cột hoặc cùng cụm: mỗi panel là một instance riêng.
- Các biển thuộc cả bốn family `prohibitory`, `mandatory`, `danger`, `other`.

### Ngoài scope

- Quảng cáo, bảng cửa hàng, bảng tên không phải road traffic sign.
- Mặt sau của biển nếu không có thông tin mặt biển cần cho task.
- Reflection của traffic sign trên kính/gương.
- Candidate quá mờ, quá khuất hoặc quá thiếu thông tin đến mức không thể xác định đáng tin cậy đó là traffic sign.

## 2. Annotation unit

- **Loại dữ liệu:** ảnh tĩnh.
- **Đơn vị annotation:** một physical sign panel = một instance.
- **CVAT tool:** Rectangle / Shape.
- Không dùng Track hoặc interpolation.

Quy tắc instance:
1. Một panel có nội dung riêng = một box.
2. Hai hoặc nhiều panel xếp chồng trên cùng cột = nhiều box riêng.
3. Không dùng một box chung cho cả cụm biển.
4. Không gộp các biển giống nhau thành một instance chỉ vì chúng nằm gần nhau.
5. Ảnh không có traffic sign hợp lệ thì để ảnh hoàn toàn không có rectangle; đây là output đúng, không phải annotation thiếu.

## 3. Geometry rule

Sử dụng **rectangle tight-visible**.

1. Box ôm sát **mặt biển nhìn thấy**.
2. Không lấy cột, giá đỡ, dây, nền hoặc khoảng trống quanh biển.
3. Không suy đoán kích thước thật của phần bị che; box chỉ bao phần nhìn thấy.
4. Nếu biển bị cắt bởi mép ảnh, box kết thúc tại mép ảnh.
5. Mỗi panel trong cụm biển có box riêng.
6. Không có minimum-size threshold để tự động ignore. Trong bộ challenge có biển chỉ khoảng 20 px nhưng vẫn có ground truth, nên biển nhỏ vẫn phải label nếu còn nhận biết được là traffic sign.
7. Với object đủ lớn để đánh giá ổn định, mục tiêu là lệch không quá khoảng **2 px mỗi cạnh** giữa các annotator.
8. Với biển rất nhỏ, ưu tiên tight box nhất quán theo pixel nhìn thấy; không loại biển chỉ vì sai lệch vài pixel làm IoU thay đổi mạnh.

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

### `mandatory`

Biển yêu cầu phương tiện thực hiện một hướng/hành vi, thường là các biển tròn nền xanh như đi trái/phải/thẳng, keep left/right, roundabout. GTSDB dùng các class 33–40 cho nhóm này.

### `danger`

Biển cảnh báo nguy hiểm hoặc điều kiện đường phía trước, thường có dạng tam giác. GTSDB dùng các class 11, 18–31 cho nhóm này.

### `other`

Traffic sign hợp lệ nhưng không thuộc ba nhóm trên. **`other` vẫn phải label và không đồng nghĩa với “không quan trọng”.** Trong GTSDB, nhóm này có thể bao gồm restriction-end, priority road, give way, stop, no entry và một số class khác ngoài ba superclass competition-relevant.

### `unknown`

Annotator chắc chắn object là traffic sign nhưng ảnh không cung cấp đủ bằng chứng để chọn một trong bốn family trên.

### Default

Trong CVAT, `sign_family` nên có default kỹ thuật là `__undefined__`.

- `__undefined__` không phải annotation hoàn chỉnh.
- Trước export, mọi rectangle phải có một giá trị semantic: `prohibitory`, `mandatory`, `danger`, `other` hoặc `unknown`.
- Không đặt `unknown` làm default, để tránh annotator quên phân loại biển nhìn rõ.

### Attribute `needs_review`

- `false`: guideline đủ rõ để annotator tự quyết định.
- `true`: case không thể resolve chắc chắn bằng guideline v1 và cần reviewer/QA owner quyết định.

Phân biệt:
- `sign_family = unknown`, `needs_review = false`: chắc chắn là traffic sign nhưng family không đọc đủ từ ảnh; rule xử lý đã rõ.
- `sign_family = unknown`, `needs_review = true`: bản thân quyết định label/family còn tranh chấp và cần escalation.

## 5. Inclusion / exclusion

### LABEL

Bắt buộc tạo rectangle khi:

- chắc chắn object là traffic sign thật;
- biển nhỏ/xa nhưng vẫn nhận biết được là traffic sign;
- biển bị che/cắt một phần nhưng còn đủ phần mặt biển để xác định object;
- biển thuộc bất kỳ family nào, kể cả `other`;
- chắc chắn là traffic sign nhưng chưa đủ bằng chứng phân family → label + `unknown`.

### IGNORE

Không tạo rectangle khi:

- object rõ ràng không phải traffic sign;
- chỉ là quảng cáo/bảng cửa hàng/bảng tên;
- chỉ thấy reflection;
- chỉ thấy backside không mang thông tin của mặt biển;
- bằng chứng quá ít để xác định object là traffic sign.

**Không dùng kích thước nhỏ làm lý do duy nhất để IGNORE.**

## 6. Visibility / occlusion / small signs

### Biển nhỏ / xa

Bộ challenge thực tế có các box khoảng 20×19, 20×20, 20×22 px và vẫn được GTSDB annotate. Vì vậy:

- nhỏ nhưng nhận biết được là traffic sign + family rõ → LABEL + family;
- nhỏ nhưng chỉ chắc chắn đây là traffic sign, family không rõ → LABEL + `unknown`;
- nhỏ đến mức không thể xác định có phải traffic sign hay không → IGNORE hoặc ESCALATE tùy mức tranh chấp.

Không zoom rồi suy diễn chi tiết không có đủ bằng chứng trong ảnh.

### Cụm biển / biển xếp chồng

Dataset có nhiều ảnh chứa 4–6 traffic signs và nhiều panel nằm rất gần nhau. Luôn:

- quét từng panel riêng;
- một panel = một rectangle;
- family được quyết định **theo từng panel**, không suy ra từ biển phía trên/dưới hoặc từ cả cột.

### Bị che một phần

Nếu vẫn chắc chắn là traffic sign:
- box phần mặt biển nhìn thấy;
- nếu family đủ bằng chứng → chọn family;
- nếu family không đủ bằng chứng → `unknown`.

Không vẽ amodal box qua vật che.

### Bị cắt mép ảnh

Nếu phần còn lại vẫn nhận biết được là traffic sign:
- label phần nhìn thấy;
- box dừng tại mép ảnh.

Nếu không còn đủ bằng chứng để xác định object thì IGNORE/ESCALATE theo mục 7.

### Blur / glare / low confidence

- chắc chắn là traffic sign nhưng family không rõ → `unknown`;
- không chắc object có phải traffic sign → ESCALATE nếu case có hai quyết định hợp lý, nếu không đủ bằng chứng tối thiểu thì IGNORE.

## 7. Ambiguity / escalation

| Tình huống | Quyết định | Thể hiện trong CVAT |
|---|---|---|
| Chắc chắn là traffic sign, family rõ | LABEL | Rectangle `traffic_sign`, chọn family, `needs_review=false` |
| Chắc chắn là traffic sign, family không đủ bằng chứng | UNKNOWN | Rectangle, `sign_family=unknown`, `needs_review=false` |
| Candidate có khả năng là sign nhưng guideline/bằng chứng chưa đủ để chốt | ESCALATE | Rectangle candidate, `sign_family=unknown`, `needs_review=true` |
| Traffic sign hợp lệ nhưng ngoài 3 family chính | LABEL | Rectangle, `sign_family=other` |
| Rõ ràng không phải traffic sign / reflection / backside ngoài scope | IGNORE | Không tạo rectangle |

Nguyên tắc:
- `UNKNOWN` = object chắc chắn là sign, chỉ thiếu thông tin để phân family.
- `ESCALATE` = annotator không thể áp dụng guideline chắc chắn hoặc có ≥2 quyết định hợp lý.
- Không tự tạo family mới.
- Không đổi taxonomy bằng thỏa thuận miệng; thay đổi rule phải cập nhật guideline và revision log.

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.**

Không dùng Track, interpolation hoặc mutable temporal attribute.

## 9. Examples grounded in challenge data

Các sample dưới đây nên được đưa vào split `example` hoặc `calibration`, **không đưa vào blind**, nếu giữ chúng trong guideline v1/v2.

| sample_id | Quan sát từ ground truth của đúng ảnh gốc GTSDB | Expected output | Rule áp dụng |
|---|---|---|---|
| **GTS07** (`00108`) | Không có traffic sign ground-truth | Không tạo rectangle | Negative image / IGNORE |
| **GTS01** (`00073`) | 6 signs; hai cụm nhiều panel; có `danger` và `prohibitory` trong cùng ảnh | 6 rectangle riêng; không gộp cụm; family từng panel | Multi-instance + stacked panels |
| **GTS02** (`00206`) | 5 signs; trộn `mandatory` và `other`; có một sign khoảng 20×22 px | 5 rectangle; sign nhỏ vẫn label | Mixed-family + small sign |
| **GTS03** (`00054`) | 4 signs và có đủ `danger`, `prohibitory`, `mandatory`, `other`; một sign khoảng 20×20 px | 4 rectangle; classify từng panel độc lập | Mixed family + small sign |
| **GTS04** (`00088`) | 4 sign nhỏ, tạo thành hai cặp gần/xếp nhau; đều `prohibitory` | 4 tight box riêng | Stacked / repeated family |
| **GTS24** (`00116`) | 2 signs; một `mandatory` cỡ vừa và một `other` rất nhỏ/hẹp khoảng 17×29 px | Cả hai đều label nếu nhìn thấy theo ảnh | Do not drop small signs |
| **GTS26** (`00014`) | Một `danger` khoảng 20×19 px | Vẫn LABEL; tight box | Extreme small-sign case |
| **GTS28** (`00139`) | Không có traffic sign ground-truth | Không tạo rectangle | Negative image / IGNORE |

Các ảnh còn lại có thể dùng cho calibration/blind để kiểm tra khả năng áp dụng rule mà không cần example trực tiếp.

## 10. Common mistakes

1. **Bỏ sót biển rất nhỏ vì nghĩ dưới một ngưỡng pixel thì không cần label**  
   → Sai với subset này. Có sign ~20 px vẫn là ground truth.

2. **Một box ôm cả cụm biển**  
   → Mỗi panel là một instance riêng.

3. **Box cả cột/giá đỡ**  
   → Tight box chỉ quanh mặt biển nhìn thấy.

4. **Suy ra family của cả cột từ một panel**  
   → Một ảnh/cụm có thể trộn nhiều family; classify từng panel độc lập.

5. **Coi `other` là object ngoài scope**  
   → `other` là một family hợp lệ và vẫn phải label.

6. **Nhầm `other` với `unknown`**  
   → `other`: biết traffic sign thuộc nhóm ngoài three main families. `unknown`: không đủ bằng chứng để xác định family.

7. **Ép chọn family khi biển quá nhỏ/mờ**  
   → Chắc chắn là sign nhưng family không đủ → `unknown`.

8. **Dùng `unknown` làm lựa chọn lười**  
   → Nếu family đủ rõ phải chọn đúng family.

9. **Để `__undefined__` khi export**  
   → Annotation chưa hoàn tất.

10. **Nghĩ ảnh không có rectangle là annotation lỗi**  
    → GTS07 và GTS28 là negative samples; zero-box output có thể hoàn toàn đúng.

11. **Dựa hoàn toàn vào màu/hình dạng rồi đoán nội dung**  
    → Hình dạng/màu là evidence hỗ trợ; không suy diễn vượt quá thông tin nhìn thấy.

12. **Rule chỉ thống nhất bằng miệng**  
    → Nếu peer cần rule đó, phải ghi vào guideline trước freeze.
