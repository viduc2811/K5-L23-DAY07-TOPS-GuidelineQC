# Guideline gán nhãn — v1.0 — Nhóm TOPS

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v1.0
- Ngày: 16-11-2026
- Người viết / duyệt: Nhóm TOPS (049, 051, 061, 064, 217)

## 1. Mục tiêu
Mô hình nào dùng nhãn này, để làm gì, sai ở đâu thì nguy hiểm.
Nhãn dùng huấn luyện mô hình phát hiện người và phương tiện thật trên ảnh đường phố 2D (camera giao thông, xe tự lái). Mô hình chỉ thấy ảnh nên nhãn chỉ ghi điều nhìn thấy được.

Hai kiểu sai nguy hiểm nhất:
- Bỏ sót người hoặc phương tiện thật: mô hình không thấy vật cản.
- Gán nhãn cho vật giả (tượng, đồ chơi, hình in, phản chiếu): mô hình báo động giả, phanh nhầm.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô thân liền một khối, không có thùng hàng riêng: sedan, hatchback, SUV, xe cổ, taxi. | img_023, img_012 (Mazda bạc) | Có thùng hàng hở hoặc cabin tách khỏi thùng là `truck`; thân hộp mui cao là `van`. |
| `truck` | Xe có cabin tách riêng khỏi thùng hàng: xe tải, bán tải, xe chở xe. | img_001, img_007, img_036, img_037 | Thân hộp một khối là `van`. |
| `van` | Xe thân hộp kín một khối, mui cao, tối đa 16 chỗ: minivan, xe giao hàng nhỏ. | img_005 (xe trắng 51LD) | Từ 17 chỗ trở lên là `bus`; SUV là `car`. |
| `bus` | Xe buýt, xe khách cỡ lớn, từ 17 chỗ trở lên. | img_009, img_037 (xa) | Xe 16 chỗ trở xuống là `van`. |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, tay ga, xe điện không bàn đạp), kể cả có sidecar; chỉ phần xe, không gồm người. | img_006, img_008, img_010 | Xe đạp là `bicycle`; xe ba bánh là `three_wheeler`. |
| `three_wheeler` | Xe ba bánh chở người hoặc hàng: tuk-tuk, xe lam, ba gác máy, xích lô đạp. | img_004, img_016, img_035 | Xe máy có sidecar vẫn là `motorcycle`. |
| `bicycle` | Xe đạp hai bánh có bàn đạp, kể cả khi đỗ. | img_003, img_029 | Xe máy điện không bàn đạp là `motorcycle`. |
| `person` | Người thật nhìn thấy trên ảnh: đi bộ, đứng, ngồi, ngồi hoặc đạp trên xe. | img_034, img_014 | Tượng, ma-nơ-canh, hình in, người trong phản chiếu. |
| `ignore_region` | Vùng không gán từng vật (đám đông, bãi xe dày không tách được); bỏ sót ở đây không tính lỗi. | img_021, img_030 | Không dùng để né vật tách rõ. |

Vật **không thuộc lớp nào** thì làm gì (không gán? vùng ignore? lớp mới?).
không vẽ box, không tự thêm lớp mới. Người đi trên các xe đó vẫn gán `person`.

### Thuộc tính
Mỗi thuộc tính: tên, **các mức cố định**, nghĩa từng mức, **ngưỡng cụ thể** (ví dụ che bao nhiêu % thì mức nào).
| Thuộc tính | Mức | Nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Bị che dưới 10% diện tích. |
| | `1` | Bị che từ 10% đến dưới 35%. |
| | `2` | Bị che từ 35% đến dưới 70%, vẫn nhận ra lớp. |
| | `3` | Bị che từ 70% trở lên nhưng vẫn nhận ra chắc lớp. Nếu có đủ căn cứ vẽ box nhưng không ước lượng được mức che thì cũng chọn `3` và tick `can_review`. |
| `truncated` | `no` | Vật nằm trọn trong ảnh, hoặc chỉ chạm mép mà không mất phần nào. |
| | `yes` | Mép ảnh cắt mất một phần vật. Bị vật khác che thì không tính. |

## 3. Quy tắc vẽ box
Box ôm sát phần nhìn thấy; vật bị che; vật bị cắt mép ảnh; phản chiếu / hình in / tượng / đồ chơi; xe chủ (ego); vật quá nhỏ; đám đông; một vật có nhiều phần (người và xe)
- **R-01 — Ôm sát phần nhìn thấy.** Hình chữ nhật nhỏ nhất bao hết phần nhìn thấy, sai số tối đa 2 px mỗi cạnh. Không tính gương chiếu hậu nhô ra, ăng-ten, hàng hoá nhô khỏi thân. Không kéo box để đoán phần bị che.
- **R-02 — Vật bị che.** Chỉ ôm phần thấy được; điền `occluded_level` theo bảng ở mục 2. Hai vật che nhau mà mỗi vật vẫn nhận ra thì mỗi vật một box.
- **R-03 — Vật cắt mép ảnh.** Box dừng ở mép ảnh, đặt `truncated=yes`. Phần nhìn thấy dưới 15% thân vật hoặc không nhận ra lớp thì không gán (xem R-11).
- **R-04 — Vật giả và phản chiếu không gán.** Tượng, ma-nơ-canh, đồ chơi, mô hình xe, hình in trên biển quảng cáo, phản chiếu trong gương, kính, mặt nước. Xe thật có đồ trang trí thì gán xe, không gán đồ trang trí. Xe cổ thật trưng bày vẫn gán; không rõ là xe thật hay mô hình thì gán và tick `can_review`.
- **R-05 — Người và xe hai/ba bánh.** Mỗi xe một box (`motorcycle`, `bicycle` hoặc `three_wheeler`) và mỗi người ngồi, đạp hay lái một box `person` riêng (kể cả người ngồi sau, trẻ em). Hai box được chồng nhau, không gộp. Box xe không gồm người, box người không gồm thân xe. Người ngồi trong ô tô, van, bus, truck thì không gán `person` khi bị kính và thân xe che từ 70% trở lên. Người ngồi trong tuk-tuk gán `person` nếu nhìn rõ.
- **R-06 — Xe chở xe.** Một box `truck` ôm đầu kéo và khung chở. Mỗi xe được chở thấy từ 50% thân xe trở lên và cao từ 20 px có một box riêng theo đúng lớp, được phép nằm trong box `truck`. Xe được chở thấy dưới 50% thì không gán, tick `can_review` ở box `truck`.
- **R-07 — Hàng hoá.** Gánh hàng, ống dài, bao tải chất cao không nằm trong box người hay xe. Hàng chất trong thùng xe tải hay bán tải thì thuộc box xe.
- **R-08 — Vật quá nhỏ.** Chiều cao box dưới 20 px (đo trên ảnh gốc, theo `min_box_height_px` trong `taxonomy.json`) thì không gán và không tính là lỗi. Nếu vật nằm trong cụm không tách được thì thuộc `ignore_region` theo R-09. Vật lẻ, tách rõ nhưng dưới 20 px vẫn không gán; ai cần đánh dấu thì ghi vấn đề cấp ảnh trong `image_issues.csv`.
- **R-09 — Đám đông.** Tách được từng vật bằng mắt thì gán từng vật. Không tách được (hai người gán khác nhau sẽ đếm ra số vật khác nhau) thì vẽ một `ignore_region` ôm cả cụm và không gán từng vật bên trong; không vừa gán vừa phủ ignore lên cùng một vật. Vật tách rõ ngoài cụm vẫn gán.
- **R-10 — Xe chủ (ego).** Nắp ca-pô, nội thất, gương của chính xe gắn camera không gán. Chỉ thấy gương hay kính mà không nhận ra xe cũng không gán (img_020).
- **R-11 — Chỉ gán khi nhận ra lớp.** Ảnh mờ do chuyển động hoặc thiếu sáng mà nhận ra được thì vẫn gán và tick `can_review` nếu rìa nhoè. Không nhận ra thì không gán, không đoán lớp.
- **R-12 — Ranh giới lớp.** `van` đến 16 chỗ, mui cao, thân hộp một khối; `bus` từ 17 chỗ, hoặc thân dài hơn khoảng hai ô tô con nối nhau. Bán tải gán `truck`. Xe máy điện không bàn đạp gán `motorcycle`; xe có bàn đạp gán `bicycle`; không phân biệt được thì chọn `bicycle` và tick `can_review`. Xe máy có sidecar là một box `motorcycle` gồm cả sidecar. Xích lô đạp gán `three_wheeler` và tick `can_review`. Các ca khác không chắc: chọn lớp gần nhất và tick `can_review`.
- **R-13 — Vật bị chia mảnh.** Cột điện, cành cây chắn ngang một vật thì vẽ một box bao cả hai mảnh.
- **R-14 — Ảnh không có vật nào đủ điều kiện** vẫn phải có mặt trong COCO với danh sách box rỗng.

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe chở xe, 7–8 ô tô trên khung | R-06 | 1 `truck` + `car` riêng cho từng xe nhận ra | Mỗi xe là một phương tiện nhìn thấy được |
| img_024 | Xe nhỏ màu vàng đặt trên nóc xe wagon đen | R-04 | Gán `car` cho xe đen; xe vàng trên nóc không gán, tick `can_review` | Vật trưng bày được chở trên nóc |
| img_005 | Nhiều người đi xe máy che nhau | R-02, R-05 | `person` từng người thấy rõ, `motorcycle` bị che nhiều thì `occluded_level` 2 hoặc 3 + `can_review`; `van` trắng gán | Che chồng chéo |
| img_006, img_008, img_010 | Xe máy chở 1–2 người, xe đỗ san sát | R-05, R-09 | Mỗi người và mỗi xe một box; hàng xe dày ở xa dùng `ignore_region` | Tách rõ thì gán |
| img_004, img_016, img_035 | Tuk-tuk, xe ba bánh | R-05, R-07 | `three_wheeler`; `person` ngồi trong nếu rõ; box không ôm bao hàng (img_035) | Có lớp riêng; hàng không thuộc xe |
| img_023 | Xích lô đạp, ô tô đen, xe máy xa | R-12 | `three_wheeler` + `can_review`; `person` đạp; `car` đen | Chưa có lớp xích lô riêng |
| img_009, img_037 | Xe buýt | R-12 | `bus`; img_037 thêm `truck` cho bán tải mui kín | Có lớp riêng |
| img_012 | Bán tải đầy hàng, cắt mép phải và dưới | R-03, R-07 | `truck` (`truncated=yes`) ôm cả hàng trong thùng; `car` bạc phía sau | Hàng trong thùng thuộc xe |
| img_019 | Xe diễu hành vàng có chuột bông, đám đông sau rào | R-04, R-09 | `car` vàng, người lái `person`, chuột bông không gán, đám đông `ignore_region` | Xe thật có đồ trang trí |
| img_021, img_030 | Đường đông xe máy; bãi xe kín | R-09 | Xe tách rõ ở tiền cảnh thì gán; cụm dày một `ignore_region` | Không tách được |
| img_014 | Người đi bộ nét, xe máy phía sau nhoè | R-11 | `person` ×2; xe nhoè gán nếu nhận ra + `can_review` | Mờ chuyển động |
| img_015 | Ô tô đỏ, bạc cắt mép; phản chiếu trên đường | R-03, R-04 | `car` ×2 `truncated=yes`; phản chiếu không gán | Phản chiếu là vật giả |
| img_011, img_017, img_027, img_033, img_038 | Tượng khủng long, tượng ngựa, bia mộ xe bán tải, ma-nơ-canh, xe đồ chơi | R-04 | Không gán | Vật giả |
| img_020 | Gương chiếu hậu có hình xe phản chiếu | R-04, R-10 | Không gán | Phản chiếu |
| img_028 | Bán tải cổ có tượng ngựa trên thùng | R-04 | `truck` cho xe; không gán ngựa | Xe thật, ngựa là tượng |
| img_029 | Người đạp xe và người lái xe máy vác ống dài | R-05, R-07 | `person` ×2, `motorcycle`, `bicycle`; box không ôm ống | Hàng mang theo không tính |
| img_040 | Mũ bảo hiểm tiền cảnh bị cắt, taxi cắt mép, tuk-tuk và SUV phía sau | R-03, R-05 | `person` và `car` taxi `truncated=yes`; `motorcycle` chỉ thấy gương + `can_review`; `three_wheeler`, SUV gán | Cắt mép, che nhiều |
Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản
Ngưỡng QC áp dụng, ai quyết ca mơ hồ, lịch sử thay đổi:
 Ngưỡng QC (lấy từ `taxonomy.json`, không đổi): chiều cao box tối thiểu 20 px; IoU ghép cặp 0.5; IAA IoU 0.85 (vật từ 50 px) và 0.7 (vật nhỏ); đồng thuận lớp 0.95; tỉ lệ ảnh lỗi theo ảnh tối đa 2%.
 Ca mơ hồ: tick `can_review`; nhóm chốt ở Pha 5 và ghi vào `calibration_decisions.csv`.
 Mọi thay đổi làm đổi cách chọn vật, lớp, box hoặc thuộc tính thì tăng phiên bản guideline.
 Mã quy tắc R-01…R-14 của bản này thay thế các mã trong guideline cá nhân (ví dụ `R-PER-MOTO-01` của 064 tương ứng R-05 mới). Các bản cá nhân được giữ nguyên để đối chiếu.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1 | 5 bản cá nhân (049, 051, 061, 064, 217) | Bổ sung lớp, ngưỡng, quy tắc, ca khó |
| v1.0 | Hợp nhất 5 bản: bộ 9 lớp, ngưỡng che 10/35/70, R-01…R-14 | img_002, img_005, img_023, img_024, img_037 |
