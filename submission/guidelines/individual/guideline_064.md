# Guideline gán nhãn — v0.1 đề xuất — CHƯA_CUNG_CẤP_MÃ

- **Phiên bản:** v0.1 (bản guideline cá nhân đề xuất, chưa được reviewer/nhóm phê duyệt)
- **Ngày:** 16/11/2026
- **Người viết / duyệt:** Người viết: thành viên thực hiện (chưa cung cấp mã); Người duyệt: reviewer/lead của nhóm (chưa xác định)
- **Phạm vi dữ liệu khảo sát:** `data/images/img_001.jpg` đến `img_040.jpg` (40 ảnh trong ZIP người dùng gửi)
- **Trạng thái:** Bản làm việc. Các quy tắc gắn nhãn **[ĐỀ XUẤT]** chưa được coi là quy định chính thức của dự án. Tên class và thuộc tính cuối cùng phải đối chiếu danh sách class/schema của dự án trước khi xuất annotation.

> **Quy ước trạng thái**
> - **[QUAN SÁT]**: nội dung có thể nhận diện trực tiếp từ ảnh.
> - **[ĐỀ XUẤT]**: quy tắc vận hành được đề xuất để hai annotator làm nhất quán; cần nhóm/reviewer xác nhận.
> - **[CAN_REVIEW]**: không đủ căn cứ để tự chốt class hoặc quyết định ảnh hưởng taxonomy; ghi cờ `can_review` trên box hoặc issue cấp ảnh.
> - Không được tự tạo class mới, đổi tên class hoặc tự ý ghi nhãn ngoài schema đang được dự án phê duyệt.

## 1. Mục tiêu

### 1.1. Mục tiêu sử dụng

Guideline này định hướng gán nhãn bounding box cho ảnh giao thông đường phố, tập trung vào các đối tượng người tham gia giao thông và phương tiện. Nhãn dự kiến được dùng để huấn luyện/đánh giá mô hình phát hiện đối tượng (object detection), giúp mô hình xác định **đối tượng thuộc lớp nào và vị trí của đối tượng trong ảnh**.

**Lưu ý:** Tên mô hình, phiên bản model, bộ class chính thức và mục tiêu đánh giá cụ thể chưa có trong tài liệu đầu vào hiện tại. Vì vậy, phần này mô tả mục đích ở mức bài toán detection, không khẳng định dự án đã chốt YOLO hay một kiến trúc cụ thể.

### 1.2. Sai ở đâu thì nguy hiểm

1. **Sai class:** gán tuk-tuk thành motorcycle/car, bus thành truck hoặc tượng thành person làm nhiễu ranh giới giữa các lớp.
2. **Thiếu đối tượng:** bỏ sót người lái/người ngồi sau hoặc phương tiện trong cảnh đông làm mô hình học thiếu trường hợp thực tế.
3. **Box gộp nhiều vật:** gộp người và xe máy thành một box khiến mô hình không học được vị trí riêng của person và motorcycle.
4. **Box quá rộng/quá chặt:** box ăn nhiều nền hoặc cắt mất phần nhìn thấy làm sai tọa độ mục tiêu.
5. **Gán vật không phải đối tượng thật:** tượng, ma-nơ-canh, đồ chơi, ảnh in hoặc phản chiếu bị gán như người/xe thật sẽ tạo false positive trong dữ liệu huấn luyện.
6. **Không thống nhất ngưỡng vật nhỏ/che khuất:** hai annotator xử lý khác nhau gây nhãn không đồng nhất và làm metric đánh giá khó diễn giải.

### 1.3. Nguyên tắc ưu tiên

- Chỉ gán nhãn cho đối tượng thuộc class được schema dự án cho phép.
- Một đối tượng vật lý = một box cho class tương ứng; không gộp hai cá thể cùng lớp vào một box.
- Box ôm sát phần đối tượng nhìn thấy, không ước lượng phần bị che để kéo box ra ngoài vùng quan sát.
- Khi không chắc class hoặc không thể xác định ranh giới, dùng `can_review`; không đoán.
- Không vẽ box trong giai đoạn khảo sát guideline. Việc tạo annotation chỉ bắt đầu sau khi nhóm thống nhất quy tắc.

## 2. Lớp và thuộc tính

### 2.1. Lớp

Bảng dưới là **bản đồ class đề xuất để đối chiếu**, không thay thế danh sách class chính thức. Nếu tên class trong dự án khác, phải dùng đúng tên trong schema, không tự tạo class mới.

| Lớp dự kiến | Định nghĩa (một câu) | Ví dụ quan sát trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Một người thật đang hiện diện trong cảnh, bất kể đang đi bộ, lái xe hay ngồi trên phương tiện. | Người đi bộ ở `img_014`, người lái/ngồi sau xe máy ở `img_005`, `img_006`, `img_010`, `img_040`. | Tượng người (`img_017`), ma-nơ-canh (`img_033`), hình người trên poster/ảnh in. |
| `motorcycle` | Một xe máy/xe mô tô có thể nhận diện là phương tiện thật trong cảnh. | Xe máy trong `img_005`, `img_006`, `img_008`, `img_010`, `img_021`. | Tuk-tuk ba bánh (`img_004`, `img_035`), xe đạp, đồ chơi xe (`img_038`), xe trong gương (`img_020`). |
| `car` | Ô tô con/xe con có khoang chở người, không bao gồm xe tải hoặc bus. | Ô tô con trên đường và xe con chở trên xe chuyên dụng ở `img_002`. | Xe bán tải, xe tải, bus, xe đồ chơi trong `img_038`; ranh giới pickup cần reviewer nếu schema không định nghĩa. |
| `truck` | Phương tiện chở hàng có kết cấu xe tải/khung chở hàng rõ ràng. | Xe chở xe chuyên dụng ở `img_002`; xe tải nhỏ ở `img_036`; xe tải/van hàng ở `img_037` nếu taxonomy xếp vào truck. | Ô tô con; pickup có nắp thùng cao (`img_037`) nếu dự án có class riêng; bus. |
| `bus` | Phương tiện chở nhiều hành khách, thân xe dạng bus. | Bus trong `img_009`; bus ở hậu cảnh `img_037` nếu đủ kích thước và rõ ràng. | Van/minibus hoặc xe tải có thùng kín; cần áp dụng định nghĩa chính thức nếu có. |
| `bicycle` | Xe đạp hai bánh vận hành bằng bàn đạp, nếu class này có trong schema. | Xe đạp xuất hiện cùng giao thông đường phố, ví dụ `img_006`, `img_023`. | Motorcycle; xe ba bánh/tuk-tuk. |
| `other_vehicle` | **Không tự tạo class này.** Chỉ dùng nếu schema đã có class tổng quát này và reviewer xác nhận áp dụng cho loại phương tiện chưa có class riêng. | Tuk-tuk ở `img_004`, `img_035` có thể cần class tổng quát nếu taxonomy hỗ trợ. | Không được tự gán tuk-tuk vào motorcycle/car chỉ vì hình dáng có phần giống. |

**Class chính thức cần xác nhận:** danh sách trên được dựng từ nội dung ảnh và bài toán, chưa được đối chiếu file `classes.json`/label schema. Trước khi annotation, lead/reviewer phải xác nhận class nào thực sự tồn tại và cách phân loại `pickup`, `tuk-tuk`, `van`, `minibus`, xe chở xe.

### 2.2. Vật không thuộc lớp nào

- **[ĐỀ XUẤT]** Không tạo class mới tại cấp annotator.
- Nếu vật thể không thuộc bất kỳ class nào trong schema: **không gán box**.
- Nếu vật thể có vẻ thuộc một class nhưng ranh giới class chưa rõ (ví dụ tuk-tuk, pickup, van/minibus): ghi issue cấp ảnh hoặc `can_review`, không ép vào class gần giống.
- Nếu công cụ/dataset có vùng ignore chính thức, chỉ dùng theo hướng dẫn của dự án. Hiện chưa có bằng chứng schema hỗ trợ ignore region, do đó không tự tạo vùng ignore.
- Không dùng nhãn `person` cho tượng, ma-nơ-canh, đồ chơi hoặc hình phản chiếu chỉ vì chúng có hình dáng giống người.

### 2.3. Thuộc tính

Chưa nhận được schema thuộc tính chính thức. Bảng sau là **đề xuất tối thiểu**; chỉ tạo thuộc tính trong CVAT/export nếu schema dự án cho phép.

| Thuộc tính đề xuất | Mức cố định | Định nghĩa / ngưỡng đề xuất |
|---|---|---|
| `occlusion` | `none`, `partial`, `heavy`, `unknown` | Ước lượng tỷ lệ diện tích đối tượng bị che so với hình dạng đầy đủ dự kiến: `none` = 0%; `partial` = trên 0% đến 50%; `heavy` = trên 50% nhưng vẫn nhận diện được đối tượng; `unknown` = không thể ước lượng đáng tin. Ngưỡng 50% là đề xuất, cần reviewer phê duyệt. |
| `truncation` | `no`, `yes` | `yes` khi đối tượng bị mép ảnh cắt mất một phần; `no` khi toàn bộ đối tượng nằm trong ảnh. Không dùng `truncation` thay cho `occlusion`. |
| `visibility` | `visible`, `partly_visible`, `not_assessable` | `visible`: phần chính đủ rõ; `partly_visible`: bị che nhưng còn phần nhìn thấy đủ nhận dạng; `not_assessable`: không đủ bằng chứng xác định đối tượng/ranh giới. Đây là thuộc tính đề xuất, không bắt buộc nếu schema không có. |
| `review_status` | `ok`, `can_review` | `can_review` khi chưa chốt class, đối tượng có nhiều cách diễn giải, hoặc cần quyết định cấp nhóm. Chỉ dùng nếu công cụ hỗ trợ thuộc tính/cờ này; nếu không, ghi issue cấp ảnh trong nhật ký. |

**Không gán mức che khuất chỉ bằng cảm giác.** Nếu dùng `occlusion`, annotator ước lượng theo tỷ lệ diện tích bị che của toàn đối tượng, không theo số bộ phận bị che. Trường hợp không thể ước lượng đáng tin phải dùng `unknown`/`can_review`, không tự chọn `partial` hay `heavy`.

## 3. Quy tắc vẽ box

### 3.1. Quy tắc chung

**[ĐỀ XUẤT]** Box là hình chữ nhật nhỏ nhất bao trọn phần đối tượng nhìn thấy, song song với trục ảnh.

- Mép box chạm sát đường bao ngoài cùng của phần đối tượng nhìn thấy; không lấy thêm nền/đường phố không thuộc đối tượng.
- Không vẽ polygon hoặc box xoay nếu task yêu cầu bounding box axis-aligned.
- Không kéo box qua vùng bị che để bao trọn hình dạng tưởng tượng của đối tượng.
- Không gộp nhiều người, nhiều xe vào một box, kể cả khi chúng đứng sát nhau.
- Một người và chiếc xe họ đang sử dụng là hai đối tượng khác class: nếu cả hai class có trong schema, vẽ hai box riêng.
- Nếu nhiều người chồng lấp, mỗi người vẫn có box riêng khi có thể phân biệt cá thể. Nếu không thể xác định số cá thể/ranh giới, ghi `can_review`.

### 3.2. Người đi xe máy: class, số box và phạm vi box

**Quy tắc đề xuất `R-PER-MOTO-01`:**

1. Mỗi người lái xe hoặc ngồi sau xe máy là **một đối tượng `person` riêng** nếu class `person` tồn tại.
2. Mỗi xe máy là **một đối tượng `motorcycle` riêng** nếu class `motorcycle` tồn tại.
3. Không tạo một box chung bao cả người và xe.
4. Box `person` ôm phần cơ thể người nhìn thấy, bao gồm mũ bảo hiểm, đầu, thân, tay, chân và giày nếu nhìn thấy; không bao gồm yên xe, thân xe, bánh xe hoặc gương xe.
5. Box `motorcycle` ôm phần xe máy nhìn thấy, gồm thân xe, bánh xe và các bộ phận gắn liền có thể nhận diện; không bao gồm người ngồi trên xe.
6. Người lái và người ngồi sau là hai người: hai box `person`, không gộp.
7. Nếu người bị người khác/xe khác che một phần, chỉ ôm phần nhìn thấy; không suy diễn phần khuất.
8. Nếu chỉ thấy một phần rất nhỏ không đủ xác định đó là người hay vật khác, không tự gán `person`; ghi `can_review`.

**Kiểm tra quy tắc trên ảnh khác:**
- `img_006`: nhiều người lái/ngồi trên nhiều xe máy, chụp từ phía sau. Áp dụng nguyên tắc tách từng người và từng xe; không gộp cả dòng xe thành một box. Người bị che một phần bởi xe/người khác chỉ lấy phần cơ thể nhìn thấy.
- Đối chiếu thêm `img_005`: nhiều người ngồi trên xe máy ở cự ly gần, có người ngồi sau và chồng lấp. Mỗi người nhận một box riêng nếu phân biệt được đầu/thân hoặc phần cơ thể đủ nhận dạng; xe máy nhận box riêng. Trường hợp không xác định được ranh giới giữa hai người thì `can_review`.

### 3.3. Vật bị che khuất

- Nếu vẫn xác định chắc class và còn phần nhìn thấy đủ nhận diện: vẽ box quanh phần nhìn thấy.
- Không vẽ box cho phần bị che bằng cách nội suy hình dáng đầy đủ.
- Nếu hai người/xe che nhau nhưng mỗi cá thể vẫn nhận diện được: mỗi cá thể một box.
- Nếu chỉ còn một phần rất nhỏ, không đủ phân biệt class: không đoán; đánh dấu `can_review`.
- Nếu schema có thuộc tính `occlusion`, dùng ngưỡng ở Mục 2.3; nếu schema không có, không tự thêm thuộc tính.

### 3.4. Đối tượng bị cắt mép ảnh

- Nếu đối tượng bị mép ảnh cắt nhưng phần còn lại đủ nhận diện: vẽ box từ mép ảnh đến hết phần nhìn thấy; box được phép chạm mép ảnh.
- Không mở rộng box ra ngoài ảnh.
- Ghi `truncation=yes` nếu schema có thuộc tính này; nếu không, ghi chú QC khi tình huống ảnh hưởng quyết định.
- Nếu phần còn lại quá ít, không xác định được class: `can_review` hoặc không gán theo quy tắc vật quá nhỏ.

### 3.5. Tượng, ma-nơ-canh, đồ chơi, ảnh in và phản chiếu

**[ĐỀ XUẤT]** Task hướng tới phát hiện người/phương tiện thật trong cảnh giao thông, vì vậy không gán các vật thể mô phỏng hoặc hình ảnh phản chiếu như đối tượng thật:

- Tượng người/tượng cưỡi ngựa: không gán `person` hoặc `motorcycle` (ví dụ `img_017`).
- Ma-nơ-canh trưng bày: không gán `person` (ví dụ `img_033`).
- Xe đồ chơi/mô hình trưng bày: không gán `car`/`motorcycle` (ví dụ `img_038`).
- Tượng/mô hình xe đặt ngoài trời: không gán như xe thật (ví dụ `img_027`).
- Đối tượng chỉ xuất hiện trong gương chiếu hậu/phản chiếu: không gán box cho ảnh phản chiếu (ví dụ `img_020`).
- Hình người/xe trên poster, màn hình, ảnh in hoặc quảng cáo: không gán như đối tượng thật.

Nếu mục tiêu dự án thực tế là phát hiện mọi hình dạng bất kể thật/giả (không phải phát hiện thực thể thật trong cảnh), quy tắc này phải được reviewer thay đổi ở cấp guideline; annotator không tự diễn giải khác nhau.

### 3.6. Vật thể quá nhỏ dưới 20 px

**[ĐỀ XUẤT `R-SIZE-20`]** Dùng kích thước box trên ảnh gốc, không dùng kích thước ảnh đã thu nhỏ trên màn hình:

- Nếu **cạnh ngắn hơn của box < 20 pixel**, tạm thời không gán box.
- Nếu cạnh ngắn đúng 20 px hoặc lớn hơn, vẫn xem xét gán nếu class nhận diện rõ.
- Nếu vật thể là đối tượng thuộc class quan trọng nhưng rất mảnh/dài (ví dụ người/xe ở xa có một chiều dưới 20 px), đưa `can_review` thay vì tự động loại.
- Không phóng to ảnh rồi dùng số pixel trên ảnh phóng to để vượt ngưỡng.
- Ngưỡng 20 px là đề xuất từ yêu cầu khảo sát, cần reviewer xác nhận cách đo (cạnh ngắn hay cả chiều rộng và chiều cao) trước khi áp dụng hàng loạt.

### 3.7. Đám đông và nhiều đối tượng sát nhau

- Nếu từng người/xe phân biệt được, gán từng box riêng.
- Không gán một box bao toàn bộ đám đông thay cho các cá thể nếu class yêu cầu person/vehicle instance.
- Với người ở xa, áp dụng ngưỡng 20 px và quy tắc nhận diện.
- Nếu các cá thể chồng lấp mạnh đến mức không xác định được số lượng hoặc ranh giới, ghi issue cấp ảnh/`can_review`.
- `img_021` là cảnh đông xe máy: không gộp toàn bộ dòng xe thành một đối tượng; cần cân nhắc ngưỡng kích thước và khả năng tách từng người/xe.

### 3.8. Một phương tiện có nhiều phần hoặc chở phương tiện khác

- Xe chở xe ở `img_002`: **đề xuất tách xe tải chuyên dụng và từng ô tô được chở thành các instance riêng** nếu `truck` và `car` đều là class được phép. Box truck ôm phần xe tải; mỗi car trên giá chở xe có box riêng quanh phần car nhìn thấy.
- Không gán một box chung cho cả xe tải và toàn bộ các xe được chở.
- Nếu một xe trên giá bị che nhiều nhưng vẫn nhận diện được là car, box quanh phần nhìn thấy và ghi occlusion nếu schema có.
- Nếu không thể xác định các xe riêng biệt hoặc taxonomy không cho phép gán xe đang được vận chuyển, `can_review`.

### 3.9. Xe đặc biệt: tuk-tuk, bus, pickup, van

- `img_004`, `img_035` (tuk-tuk/xe ba bánh có khoang chở khách): **[CAN_REVIEW]** về class. Không tự ép vào `motorcycle` hoặc `car`; reviewer phải xác nhận class đích hoặc quy tắc gộp.
- `img_009` (bus): nếu schema có `bus`, gán `bus`; nếu chỉ có `car`/`truck` mà không có định nghĩa bao quát bus, không tự chọn class gần nhất; `can_review`.
- `img_037` (pickup/xe có khoang thùng cao hoặc van hàng): **[CAN_REVIEW]** vì ranh giới giữa pickup, van và truck phụ thuộc taxonomy. Chỉ gán `truck` khi định nghĩa chính thức của dự án bao gồm pickup/van loại này.
- Xe tải nhỏ ở `img_036`: áp dụng định nghĩa `truck` đã được reviewer xác nhận; nếu chưa rõ ranh giới truck/van thì `can_review`.

### 3.10. Xe chủ (ego vehicle)

- Nếu ảnh chụp từ bên trong xe và thấy gương, taplo, cửa hoặc một phần thân xe chủ: **[ĐỀ XUẤT]** không gán các bộ phận nội thất/xe chủ nếu chúng không phải đối tượng mục tiêu trong schema.
- Không gán đối tượng chỉ xuất hiện qua gương chiếu hậu; xem quy tắc phản chiếu.
- Nếu một phần xe chủ nằm trong ảnh nhưng schema yêu cầu gán tất cả xe thật, cần reviewer xác nhận trước khi xử lý thống nhất.

## 4. Ca khó

Bảng dưới là quyết định làm việc dựa trên khảo sát hình ảnh. `can_review` nghĩa là cần reviewer/nhóm chốt; không đồng nghĩa chắc chắn loại bỏ đối tượng. Các quyết định gắn **[ĐỀ XUẤT]** chưa phải quy định được phê duyệt.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chuyên dụng chở nhiều ô tô | `R-MULTI-01` | **[ĐỀ XUẤT]** Tách truck và từng car thành các box riêng nếu cả hai class có trong schema; nếu taxonomy không cho phép gán xe đang được chở thì `can_review`. | Đây là nhiều phương tiện vật lý khác nhau; box gộp làm mất instance và class riêng. Cần xác nhận xe được vận chuyển có thuộc phạm vi gán nhãn không. |
| `img_005` | Nhiều người đi/ngồi trên xe máy, người chồng lấp | `R-PER-MOTO-01`, `R-OCC-01` | **[ĐỀ XUẤT]** Mỗi người phân biệt được một box `person`, mỗi xe máy một box `motorcycle`; phần bị che không suy diễn. Ranh giới không rõ thì `can_review`. | Kiểm chứng cách tách người lái/người ngồi sau và xe trong cảnh gần, chồng lấp. |
| `img_006` | Nhiều người và xe máy nhìn từ phía sau | `R-PER-MOTO-01`, `R-CROWD-01` | **[ĐỀ XUẤT]** Tách từng người/xe nhận diện được; không gộp cả dòng xe. Đối tượng quá nhỏ xử lý theo `R-SIZE-20`. | Dùng làm ảnh kiểm chứng độc lập cho quy tắc người đi xe máy. |
| `img_008` | Nhiều xe máy và người trong khu vực đông | `R-PER-MOTO-01`, `R-CROWD-01` | **[ĐỀ XUẤT]** Mỗi cá thể đủ phân biệt một box; người và xe tách riêng. Ca chồng lấp không rõ ranh giới thì `can_review`. | Tránh chỉ gán một đối tượng trong ảnh có nhiều instance. |
| `img_010` | Người đi xe máy tại khu vực chợ, có người và hàng hóa che nhau | `R-PER-MOTO-01`, `R-OCC-01` | **[ĐỀ XUẤT]** Tách person/motorcycle nếu nhận diện được; box chỉ bao phần nhìn thấy. | Người, xe và vật thể chợ có thể che khuất nhau. |
| `img_011` | Vật thể hình người/động vật dạng tượng hoặc tác phẩm trưng bày ở tiền cảnh | `R-REAL-01` | **[ĐỀ XUẤT]** Không gán như `person`/động vật thật; nếu reviewer xác định đây là đối tượng thật chứ không phải tác phẩm thì cập nhật. | Hình thái giống sinh vật nhưng bối cảnh thể hiện vật trưng bày/tượng; cần giữ ranh giới thực thể thật. |
| `img_017` | Tượng người cưỡi ngựa | `R-REAL-01` | **[ĐỀ XUẤT]** Không gán `person` hoặc phương tiện/động vật thật. | Đây là tượng cố định, không phải người/đối tượng giao thông thật. |
| `img_020` | Phương tiện/cảnh vật nhìn qua gương chiếu hậu | `R-REFLECT-01` | **[ĐỀ XUẤT]** Không gán box cho hình ảnh phản chiếu trong gương. | Đối tượng chỉ xuất hiện qua phản chiếu, không phải instance trực tiếp trong cảnh camera. Cần reviewer xác nhận mục tiêu dữ liệu có loại trừ phản chiếu không. |
| `img_027` | Mô hình/tác phẩm xe bán tải trưng bày ngoài trời | `R-REAL-01` | **[ĐỀ XUẤT]** Không gán như `car`/`truck` thật. | Vật thể là mô hình/tác phẩm trưng bày, không phải xe tham gia giao thông. |
| `img_033` | Ma-nơ-canh trong cửa hàng | `R-REAL-01` | **[ĐỀ XUẤT]** Không gán `person`. | Ma-nơ-canh có hình dáng người nhưng không phải người thật. |
| `img_035` | Tuk-tuk/xe ba bánh chở khách trên đường | `R-CLASS-EDGE-01` | **[CAN_REVIEW]** Chưa gán class cuối cùng; reviewer quyết định có class riêng, class tổng quát hay loại khỏi phạm vi. | Không được tự coi tuk-tuk là motorcycle hoặc car nếu taxonomy chưa định nghĩa. |
| `img_037` | Xe pickup/van có khoang thùng cao, cạnh một bus | `R-CLASS-EDGE-02` | **[CAN_REVIEW]** Xác nhận pickup/van thuộc `truck`, `car` hay class riêng; bus chỉ gán `bus` nếu schema có class này. | Ranh giới loại phương tiện phụ thuộc định nghĩa dự án, không thể quyết chỉ từ hình dáng. |
| `img_038` | Nhiều xe đồ chơi/mô hình trong tủ trưng bày | `R-REAL-01` | **[ĐỀ XUẤT]** Không gán `car`/`motorcycle` cho xe đồ chơi. | Các xe là mô hình nhỏ trong tủ, không phải phương tiện thật trong cảnh. |
| `img_004` | Hai tuk-tuk/xe ba bánh | `R-CLASS-EDGE-01` | **[CAN_REVIEW]** Reviewer chốt class; không tự gán motorcycle/car. | Có cấu trúc ba bánh và khoang chở khách, ranh giới class chưa được cung cấp. |
| `img_009` | Bus chạy trên đường | `R-CLASS-EDGE-03` | **[CAN_REVIEW]** Nếu schema có `bus`, gán bus; nếu không có, chờ reviewer hướng dẫn. | Không được ép bus vào truck/car khi taxonomy không nói rõ. |
| `img_014` | Người đi bộ bị nhòe chuyển động | `R-PER-01` | **[ĐỀ XUẤT]** Gán `person` nếu vẫn nhận diện chắc; box ôm phần nhìn thấy, không mở rộng theo vệt nhòe nền. Nếu không xác định được hình dạng, `can_review`. | Nhòe chuyển động không tự động làm đối tượng thành không hợp lệ. |
| `img_021` | Đám đông xe máy, nhiều người/xe ở xa | `R-CROWD-01`, `R-SIZE-20` | **[ĐỀ XUẤT]** Tách từng instance đủ rõ; không gộp đám đông. Loại theo ngưỡng kích thước chỉ sau khi reviewer duyệt ngưỡng 20 px. | Cảnh đông dễ bỏ sót và dễ gộp nhiều cá thể. |
| `img_026` | Người ngồi trên xe máy, ảnh dọc, có phản sáng/độ tương phản mạnh | `R-PER-MOTO-01` | **[ĐỀ XUẤT]** Tách person và motorcycle nếu cả hai đủ nhận diện; không gộp người với xe. | Kiểm tra quy tắc người lái trong ảnh có bố cục/cự ly khác. |
| `img_034` | Người đi bộ qua đường, nhiều xe máy ở nền | `R-PER-01`, `R-CROWD-01` | **[ĐỀ XUẤT]** Gán từng người/xe đủ rõ theo class có trong schema; vật ở xa dưới ngưỡng xử lý theo `R-SIZE-20`. | Có đối tượng chính và nhiều instance nền, cần áp dụng cùng tiêu chí. |
| `img_040` | Người đội mũ bảo hiểm trên xe máy ở tiền cảnh, xe khác ở nền | `R-PER-MOTO-01` | **[ĐỀ XUẤT]** Box người và xe riêng; người lái là một person, không gộp với xe/taxi cạnh đó. | Kiểm chứng quy tắc ở cảnh cận, có nhiều phương tiện lân cận. |

### 4.1. Phân biệt ca loại theo quy tắc và ca cần hỏi reviewer

**Có thể loại theo quy tắc đề xuất (không tạo box):**
- `img_017`: tượng người cưỡi ngựa.
- `img_020`: hình ảnh phản chiếu trong gương.
- `img_027`: mô hình/tác phẩm xe trưng bày.
- `img_033`: ma-nơ-canh.
- `img_038`: xe đồ chơi trong tủ.

Các trường hợp trên chỉ được loại nếu mục tiêu dự án là phát hiện người/phương tiện thật trong cảnh. Đây là đề xuất cần reviewer xác nhận một lần ở cấp guideline; sau khi được duyệt, áp dụng nhất quán.

**Cần reviewer chốt trước khi gán class:**
- `img_002`: có cho phép gán những chiếc xe đang được chở trên xe chuyên dụng không?
- `img_004`, `img_035`: tuk-tuk thuộc class nào?
- `img_009`: schema có class bus hay không?
- `img_037`: pickup/van thùng cao thuộc car, truck hay class riêng?
- `img_011`: xác nhận chính xác vật thể tiền cảnh là tác phẩm/tượng hay một đối tượng khác; nếu đúng là tượng/mô hình thì áp dụng `R-REAL-01`.
- Mọi đối tượng quá nhỏ hoặc bị che đến mức không thể xác định class/số lượng.
- Mọi trường hợp không rõ người lái và người ngồi sau là một hay hai cá thể.

## 5. QC và phiên bản

### 5.1. Checklist QC cho từng ảnh

- [ ] Đã rà toàn ảnh, không chỉ vùng trung tâm; kiểm tra cả rìa ảnh và nền.
- [ ] Mỗi người/xe đủ điều kiện đều có instance riêng, không bỏ sót do cảnh đông.
- [ ] Người và phương tiện được tách box nếu cả hai class tồn tại.
- [ ] Box ôm sát phần nhìn thấy; không bao gồm nền hoặc phần bị che tưởng tượng.
- [ ] Không gán tượng, ma-nơ-canh, đồ chơi, mô hình, hình in hoặc phản chiếu như thực thể thật (sau khi quy tắc này được reviewer phê duyệt).
- [ ] Vật nhỏ được đo trên ảnh gốc và áp dụng cùng ngưỡng.
- [ ] Tuk-tuk, bus, pickup/van và xe chở xe được xử lý theo quyết định taxonomy đã thống nhất.
- [ ] Ca chưa rõ có `can_review` hoặc issue cấp ảnh; không tự đoán.
- [ ] Kiểm tra box trùng lặp, box gộp nhiều cá thể, box sai class và box vượt ra ngoài ảnh.

### 5.2. Ngưỡng QC đề xuất

Các ngưỡng dưới đây là **đề xuất khởi tạo**, chưa phải ngưỡng chính thức của khóa học/dự án:

| Chỉ tiêu QC | Ngưỡng đề xuất | Cách xử lý khi không đạt |
|---|---:|---|
| Sai class trong mẫu QC | 0 ca nghiêm trọng | Sửa class; rà lại các ảnh có cùng loại tình huống. |
| Bỏ sót đối tượng rõ ràng thuộc class | 0 ca trong mẫu kiểm tra | Bổ sung nhãn; kiểm tra lại toàn ảnh/các ảnh cùng kiểu cảnh. |
| Box gộp người với xe hoặc gộp nhiều instance | 0 ca | Tách box; nếu không thể tách thì `can_review`. |
| Box bao nền quá mức/cắt mất phần nhìn thấy rõ | Mục tiêu 100% box được rà | Chỉnh box theo mép đối tượng nhìn thấy. |
| Ca taxonomy chưa chốt | 100% phải có `can_review`/issue | Không tự gán class cho đến khi có quyết định. |
| Ca dưới ngưỡng 20 px | 100% đo trên ảnh gốc | Ghi nhận theo `R-SIZE-20`; ngoại lệ vật mảnh chuyển reviewer. |

**Đề xuất lấy mẫu:** reviewer kiểm tra toàn bộ ca `can_review` và toàn bộ nhóm ảnh đặc biệt trong Mục 4; ngoài ra kiểm tra ngẫu nhiên tối thiểu 10% ảnh đã gán của mỗi annotator. Tỷ lệ 10% là đề xuất, cần nhóm thống nhất.

### 5.3. Ai quyết ca mơ hồ

- Annotator: áp dụng quy tắc đã được duyệt, ghi `can_review` khi vượt phạm vi tự quyết.
- Reviewer/coach: quyết định các ca mơ hồ về box, che khuất, kích thước và nhận diện.
- Lead/nhóm: quyết định taxonomy/class mới, class gộp, phạm vi đối tượng được gán (ví dụ xe đang được chở, tuk-tuk, pickup).
- Mọi quyết định mới phải được ghi thành Rule ID và cập nhật guideline để các thành viên áp dụng cùng một cách.

### 5.4. Nhật ký Rule ID

| Rule ID | Nội dung |
|---|---|
| `R-BOX-01` | Box ôm sát phần đối tượng nhìn thấy, không ước lượng phần bị che. |
| `R-PER-MOTO-01` | Người lái/ngồi sau và xe máy là các instance riêng; không gộp person + motorcycle. |
| `R-PER-01` | Mỗi người thật là một instance riêng nếu phân biệt được. |
| `R-OCC-01` | Đối tượng bị che: chỉ box phần nhìn thấy; không rõ class/ranh giới thì `can_review`. |
| `R-TRUNC-01` | Đối tượng bị cắt mép: box phần còn trong ảnh; không mở rộng ra ngoài ảnh. |
| `R-REAL-01` | Không gán tượng, ma-nơ-canh, đồ chơi/mô hình, hình in/phản chiếu như thực thể thật (chờ duyệt). |
| `R-REFLECT-01` | Không gán đối tượng chỉ xuất hiện trong gương/phản chiếu (chờ duyệt). |
| `R-SIZE-20` | Đề xuất loại box có cạnh ngắn dưới 20 px; vật mảnh/khó phân loại chuyển reviewer. |
| `R-CROWD-01` | Tách từng instance nhận diện được; không gộp đám đông. |
| `R-MULTI-01` | Xe chở xe: tách từng phương tiện nếu taxonomy cho phép. |
| `R-CLASS-EDGE-01` | Tuk-tuk: chờ reviewer xác định class. |
| `R-CLASS-EDGE-02` | Pickup/van thùng cao: chờ reviewer xác định class. |
| `R-CLASS-EDGE-03` | Bus: dùng class bus nếu schema có; nếu không, chờ reviewer. |

### 5.5. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0.1` — 16/11/2026 | Tạo bản guideline cá nhân đề xuất; bổ sung quy tắc tách person/motorcycle, vật thật so với tượng/đồ chơi/phản chiếu, ngưỡng vật nhỏ 20 px, xe chở xe và các ranh giới tuk-tuk/bus/pickup. | Khảo sát 40 ảnh `img_001`–`img_040`; tập trung các ca `img_002`, `img_004`, `img_009`, `img_011`, `img_017`, `img_020`, `img_027`, `img_033`, `img_035`, `img_037`, `img_038`. |
| `v0.2` — chờ cập nhật | Ghi các quyết định reviewer/nhóm, thay thế mọi đề xuất đã được chốt bằng quy tắc chính thức. | Sau cuộc họp nhóm Pha 2. |

---

## Phụ lục A. Nhật ký khảo sát 40 ảnh

Đã rà soát contact sheet của đủ 40 ảnh và phóng xem riêng các ảnh tình huống đặc biệt được nêu trong yêu cầu. Danh sách dưới đây là ghi chú định hướng nội dung ở mức khảo sát, không phải kết quả annotation.

| Ảnh | Ghi chú khảo sát sơ bộ |
|---|---|
| `img_001` | Cảnh đường phố, người đi bộ và xe tải/xe chở hàng. |
| `img_002` | Xe chuyên dụng chở nhiều ô tô; ca xe chở xe. |
| `img_003` | Người đi bộ/xe đạp hoặc phương tiện nhỏ trong cảnh phố; cần rà từng instance khi annotation. |
| `img_004` | Tuk-tuk/xe ba bánh chở khách. |
| `img_005` | Nhiều người trên xe máy, chồng lấp ở cự ly gần. |
| `img_006` | Nhiều người và xe máy nhìn từ phía sau. |
| `img_007` | Xe tải/van trong không gian bãi/nhà xe. |
| `img_008` | Nhiều xe máy và người trong khu vực đông. |
| `img_009` | Bus trên đường phố. |
| `img_010` | Người đi xe máy tại khu vực chợ, có che khuất. |
| `img_011` | Vật thể lớn dạng tượng/tác phẩm trưng bày ở tiền cảnh; cần xác nhận bản chất nếu còn nghi ngờ. |
| `img_012` | Xe tải/xe chở hàng và ô tô. |
| `img_013` | Cảnh đường phố ban đêm, nhiều phương tiện/đèn. |
| `img_014` | Người đi bộ có nhòe chuyển động. |
| `img_015` | Ô tô trong bãi/đường ướt. |
| `img_016` | Cảnh giao thông có xe máy và phương tiện khác. |
| `img_017` | Tượng người cưỡi ngựa. |
| `img_018` | Cảnh đường và bãi xe ở xa; cần chú ý vật nhỏ. |
| `img_019` | Xe đồ chơi/trang trí hình xe. |
| `img_020` | Hình ảnh phản chiếu trong gương chiếu hậu. |
| `img_021` | Đám đông xe máy, nhiều instance ở xa. |
| `img_022` | Ảnh giao thông đơn sắc/đen trắng, có phương tiện nhỏ ở xa. |
| `img_023` | Người đi xe đạp/xe máy và ô tô trên đường. |
| `img_024` | Xe thật trưng bày cùng xe/mô hình trên nóc; cần phân biệt từng đối tượng và mục tiêu gán nhãn. |
| `img_025` | Không gian công cộng có người/xe nhỏ ở xa. |
| `img_026` | Người ngồi trên xe máy trong ảnh dọc. |
| `img_027` | Mô hình/tác phẩm xe bán tải trưng bày ngoài trời. |
| `img_028` | Xe cổ/xe bán tải trong không gian trưng bày. |
| `img_029` | Người đi xe đạp/xe máy trong cảnh đường phố. |
| `img_030` | Nhiều xe máy đỗ sát nhau. |
| `img_031` | Cảnh giao thông ban đêm, nhiều nguồn sáng. |
| `img_032` | Đường phố, người đi bộ và phương tiện ở xa. |
| `img_033` | Ma-nơ-canh trong cửa hàng. |
| `img_034` | Người đi bộ qua đường và xe máy ở nền. |
| `img_035` | Tuk-tuk/xe ba bánh và xe máy. |
| `img_036` | Xe tải nhỏ/van giao hàng. |
| `img_037` | Pickup/van có khoang thùng cao và bus. |
| `img_038` | Nhiều xe đồ chơi/mô hình trong tủ trưng bày. |
| `img_039` | Cảnh phố có nhiều người/xe máy ở xa. |
| `img_040` | Người đội mũ bảo hiểm trên xe máy ở tiền cảnh, phương tiện khác ở gần. |

## Phụ lục B. Những điểm phải chốt trong cuộc họp reviewer

1. Xác nhận danh sách class chính thức và mapping cho `truck`, `car`, `bus`, `pickup`, `van`, `tuk-tuk`, `bicycle`.
2. Xác nhận có gán xe được chở trên xe chuyên dụng (`img_002`) hay không.
3. Xác nhận loại tượng/ma-nơ-canh/đồ chơi/mô hình/ảnh in/phản chiếu khỏi tập đối tượng mục tiêu.
4. Phê duyệt hoặc thay đổi ngưỡng 20 px; thống nhất đo theo cạnh ngắn hay theo cả hai chiều.
5. Phê duyệt ngưỡng che khuất 50% và quyết định có cần thuộc tính `occlusion` hay không.
6. Xác nhận cách ghi `can_review` trong CVAT (attribute, tag hay issue cấp ảnh).
7. Xác nhận có bỏ qua xe chủ/chi tiết nội thất xe trong ảnh chụp từ bên trong phương tiện hay không.

**Trạng thái cuối:** Chưa tạo bounding box; chưa chỉnh sửa ảnh; chưa xác nhận taxonomy/schema; chưa có phê duyệt reviewer. Bản này là guideline cá nhân đề xuất để mang vào cuộc họp nhóm, không phải guideline v0 chính thức đã được duyệt.
