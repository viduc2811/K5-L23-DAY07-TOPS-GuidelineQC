# Guideline gán nhãn — <phiên bản> — <mã hoặc nhóm>

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v0.1
- Ngày: 16/11
- Người viết / duyệt: Trần Gia Huy

## 1. Mục tiêu
Mô hình nào dùng nhãn này, để làm gì, sai ở đâu thì nguy hiểm.
- **Mô hình dùng nhãn:** bộ phát hiện vật thể (object detection) trên ảnh đường phố, để nhận diện xe thật đang tham gia giao thông.
- **Mục đích:** hỗ trợ lái xe an toàn và đếm lưu lượng xe.
- **Sai ở đâu thì nguy hiểm:**
  - Bỏ sót xe thật (nhất là xe bị che, xe ở mép ảnh) làm mô hình không thấy vật cản.
  - Gán nhãn cho vật không phải xe thật (tượng, hình in, phản chiếu, đồ chơi) làm mô hình phanh nhầm hoặc đếm sai.
  - Box lệch nhiều làm đo IoU sai và nhãn không ổn định.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| car | Xe con 4 bánh chở người, có khoang hành khách và cốp/khoang hành lý riêng, thân thấp hơn van | Sedan, hatchback, SUV, taxi, pickup cabin nhỏ | SUV dễ nhầm với van (Toyota Hiace, Tera V): thân van dài, mui cao, không tách cốp, thì gán `van` |
| truck | Xe tải chở hàng, có đầu cabin tách khỏi thùng hàng | Xe tải thùng, xe ben, xe container, xe đầu kéo, xe cẩu chở xe | Pickup (là `car`); xe tải nhỏ kiểu van kín (là `van`) |
| van | Xe thân hộp dài, mui cao, thường chở khách hoặc hàng nhẹ, cabin liền thân | Toyota Hiace, Ford Transit, Tera V, xe van chở hàng kín | SUV (là `car`); minibus lớn 16+ chỗ (là `bus`) |
| motorcycle | Xe hai bánh có động cơ; người ngồi trên xe là box `person` riêng | Xe máy, xe tay ga, xe côn tay | Xe đạp, xe đạp điện không bàn đạp (xem `bicycle`); xe 3 bánh |
| person | Người thật đang đứng, đi hoặc ngồi trên xe | Người đi bộ, người lái xe máy, người đạp xe | Ma-nơ-canh, tượng, hình in, người trong tranh quảng cáo |
| ignore_region | Vùng ảnh không đáng tin để gán, bị loại khỏi tính điểm | Đám xe/người dày đặc không tách được, vùng quá nhỏ hoặc quá mờ | Vật tách được rõ ràng (phải gán lớp thật, không đưa vào ignore) |
| three_wheeler | Xe 3 bánh có động cơ chở người hoặc hàng | Tuk-tuk, xe lam, xe ba gác máy | Xe máy chở thùng bên cạnh (là `motorcycle`); xe 4 bánh nhỏ (là `car`) |
| bicycle | Xe hai bánh đạp bằng chân | Xe đạp thường, xe đạp địa hình, xe đạp có giỏ | Xe máy (là `motorcycle`); xe đạp điện không bàn đạp (là `motorcycle`) |
| bus | Xe khách/xe buýt cỡ lớn, nhiều hàng ghế, thân dài | Xe buýt thành phố, xe khách đường dài, xe bus 2 tầng | Van/minibus nhỏ (là `van`); xe tải |

Vật **không thuộc lớp nào** thì làm gì (không gán? vùng ignore? lớp mới?).
    Tượng, mô hình, đồ chơi, xe giả tưởng, hình in/áp-phích, phản chiếu trên gương hoặc kính, xe trưng bày không tham gia giao thông. Những vật này **không gán box**. Riêng vùng đông đúc không tách được thì dùng `ignore_region`; vật tách được nhưng chưa chắc là lớp nào thì chọn lớp gần nhất và đánh `can_review`.

### Thuộc tính
Mỗi thuộc tính: tên, **các mức cố định**, nghĩa từng mức, **ngưỡng cụ thể** (ví dụ che bao nhiêu % thì mức nào).

| Thuộc tính | Các mức | Nghĩa và ngưỡng |
|---|---|---|
| occlusion (che khuất) | `0`, `1`, `2`, `3` | `0`: thấy ≥ 90% thân. `1`: bị che 10–35%. `2`: bị che 35–70%. `3`: bị che > 70% |
| truncated (cắt mép ảnh) | `no`, `yes` | `yes` khi > 5% thân xe nằm ngoài ảnh |
| can_review | `no`, `yes` | `yes` khi người gán chưa chắc, đưa ra họp calibration |

## 3. Quy tắc vẽ box
Box ôm sát phần nhìn thấy; vật bị che; vật bị cắt mép ảnh; phản chiếu / hình in / tượng / đồ chơi; xe chủ (ego); vật quá nhỏ; đám đông; một vật có nhiều phần (người và xe)…
- **R-01 Box ôm sát phần nhìn thấy:** cạnh box chạm mép ngoài cùng của thân xe/người nhìn thấy, sai số tối đa ~2 px. Không tính gương chiếu hậu, ăng-ten, hành lý nhô ra khỏi thân.
- **R-02 Vật bị che:** chỉ vẽ phần **nhìn thấy**, không đoán phần bị che. Gán thuộc tính `occlusion` theo bảng ở mục 2.
- **R-03 Vật bị cắt mép ảnh:** vẽ box tới đúng mép ảnh và gán `truncated=yes`. Nếu phần nhìn thấy < 15% thân thì không gán.
- **R-04 Phản chiếu, hình in, tranh vẽ:** xe/người trong gương, kính cửa hàng, áp-phích, bảng quảng cáo, tranh tường thì **không gán**.
- **R-05 Tượng, mô hình, đồ chơi, ma-nơ-canh, xe giả tưởng:** không gán, kể cả tượng xe pickup cỡ lớn. Ca mơ hồ (xe cổ trưng bày, xe tĩnh không rõ có chạy không) thì đánh `can_review`.
- **R-06 Xe 3 bánh:** tuk-tuk, xe lam, xe ba gác máy gán `three_wheeler`. Box ôm cả thân xe, **không** gồm người ngồi (người là box `person` riêng, xem R-11).
- **R-07 Xe chủ (ego):** phần capô hoặc thân xe của chính xe gắn camera ở đáy ảnh thì **không gán**.
- **R-08 Xe chở xe:** xe vận tải gán `truck`. Mỗi xe được chở gán box riêng theo đúng loại (`car`, `van`, `truck`) nếu thấy ≥ 50% thân. Box xe được chở được phép nằm trong box xe vận tải.
- **R-09 Vật quá nhỏ:** box nhỏ hơn 20×20 px thì không gán, hoặc để trong vùng `ignore_region` nếu nằm giữa đám đông.
- **R-10 Đám đông, hàng xe dày đặc:** gán từng vật nếu tách được bằng mắt thường. Nếu không tách được (từ 3 vật trở lên dính nhau, hoặc mờ/nhỏ) thì vẽ **một box `ignore_region`** bao cả cụm và ghi vấn đề cấp ảnh. Không vừa gán từng vật vừa phủ `ignore_region` lên cùng vật đó.
- **R-11 Người và xe đi cùng nhau:**
  - Người lái/ngồi trên `motorcycle` hoặc `bicycle`: vẽ **hai box riêng**, một cho xe, một cho `person`. Box `person` được phép chồng lên box xe.
  - Người ngồi trong `car`, `van`, `bus`, `truck` thì không gán `person` nếu bị kính/thân xe che gần hết (> 70%).
- **R-12 Vật bị chia mảnh bởi vật cản:** cột điện, cành cây chắn ngang một xe thì vẽ **một** box bao cả hai mảnh.
- **R-13 Xe đạp và xe máy điện:** có bàn đạp thì gán `bicycle`; không có bàn đạp (xe tay ga điện, xe máy điện) thì gán `motorcycle`. Không phân biệt được thì chọn `bicycle` và đánh `can_review`.
- **R-14 Van, minibus, xe buýt:** xe thân hộp dài, mui cao, ≤ 16 chỗ thì gán `van`; xe nhiều hàng ghế, thân dài hơn ~ 2 xe con nối nhau thì gán `bus`. Không chắc thì chọn `van` và đánh `can_review`.

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
|img_002.jpg|Xe tải chở nhiều xe con|R-08|gán nhãn cho cả xe con lẫn xe tải|Mỗi xe thật đều cần được phát hiện|
|img_011.jpg|Tượng/mô hình xe pickup lớn ở khu trưng bày|R-05|Không gán|Không tham gia giao thông thật|
|img_020.jpg|Phản chiếu xe trên kính|R-04| Không gán| Không phải vật thật|
|img_024.jpg|Tượng/mô hình xe pickup lớn ở khu trưng bày|R-05|Không gán|Không tham gia giao thông thật|
|img_027.jpg|Tượng/mô hình xe pickup lớn ở khu trưng bày|R-05|Không gán|Không tham gia giao thông thật|
|img_038.jpg|Tượng/mô hình xe pickup lớn ở khu trưng bày|R-05|Không gán|Không tham gia giao thông thật|

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản
Ngưỡng QC áp dụng, ai quyết ca mơ hồ, lịch sử thay đổi:

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| | | |
