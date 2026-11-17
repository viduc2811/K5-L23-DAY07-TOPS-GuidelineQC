# Guideline gán nhãn — v0.1 (cá nhân) — 051

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v0.1 (bản cá nhân)
- Ngày: 16/11/2026
- Người viết / duyệt: Vi Thanh Đức - 051

## 1. Mục tiêu
Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện trên ảnh đường phố 2D (camera giao thông, xe tự lái). Mô hình chỉ thấy ảnh, nên nhãn chỉ ghi điều nhìn thấy được.

Hai kiểu sai nguy hiểm nhất:
- Bỏ sót người hoặc xe hai bánh thật (hướng gây hại cho an toàn).
- Gán nhãn cho vật giả (tượng, đồ chơi, hình in, phản chiếu), làm mô hình học nhầm và báo động giả.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô thân liền một khối, không có thùng hàng riêng: sedan, hatchback, SUV, xe cổ. | img_023 (sedan đen), img_012 (Mazda bạc), img_024 (xe wagon đen) | Có thùng hàng hở phía sau là `truck`; thân hộp 7–16 chỗ là `van`. |
| `truck` | Xe có cabin tách riêng khỏi thùng hàng: xe tải, bán tải, xe chở xe. | img_001, img_007, img_036, img_037, img_002 | Thân hộp một khối là `van`. |
| `van` | Xe thân hộp kín một khối, cỡ nhỏ, tối đa khoảng 16 chỗ: minivan, xe giao hàng nhỏ. | img_005 (xe trắng 51LD), img_016 (van sau xe máy) | Xe khách lớn, xe buýt là `bus`. |
| `bus` | Xe khách hoặc xe buýt cỡ lớn, từ khoảng 17 chỗ trở lên. | img_009, img_037 (xe buýt xa) | Xe 7–16 chỗ là `van`. |
| `motorcycle` | Xe hai bánh có động cơ, kể cả xe tay ga và xe có thùng bên (sidecar). | img_006, img_008, img_010, img_022 | Xe đạp là `bicycle`; xe ba bánh là `three_wheeler`. |
| `three_wheeler` | Xe ba bánh chở người hoặc hàng (tuk-tuk, xe ba bánh có cabin, xích lô). | img_004, img_016, img_035 | Xe máy có sidecar vẫn là `motorcycle`. |
| `bicycle` | Xe đạp hai bánh chạy bằng sức người, kể cả khi đang đỗ. | img_003, img_029 | Xe đạp điện, xe máy là `motorcycle`. |
| `person` | Một con người nhìn thấy trên ảnh: đi bộ, đứng, ngồi, hoặc ngồi / đạp trên xe. | img_034, img_014, img_003 | Tượng, ma-nơ-canh, hình in, người trong phản chiếu không phải `person`. |
| `ignore_region` | Vùng không gán từng vật; bỏ sót trong vùng này không tính là lỗi. | Đám đông dày, bãi xe dày đặc | Không dùng để né vật rõ ràng. |

Vật **không thuộc lớp nào** thì làm gì (không gán? vùng ignore? lớp mới?).
không vẽ box, không thêm lớp mới.

### Thuộc tính
Mỗi thuộc tính: tên, **các mức cố định**, nghĩa từng mức, **ngưỡng cụ thể** (ví dụ che bao nhiêu % thì mức nào).
| Thuộc tính | Mức | Nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Thấy trọn vật, bị che dưới 10% diện tích. |
| | `1` | Bị che từ 10% đến dưới 40%. |
| | `2` | Bị che từ 40% đến dưới 80%, vẫn nhận ra lớp. |
| | `3` | Bị che từ 80% trở lên, hoặc chỉ nhận ra lớp nhờ ngữ cảnh; chỉ gán khi chắc về lớp, nếu không thì tick `can_review`. |
| `truncated` | `no` | Vật nằm trọn trong ảnh. |
| | `yes` | Vật chạm hoặc vượt mép ảnh nên mất một phần. Bị vật khác che thì không tính. |
| `can_review` | tick / không | Tick khi không chắc lớp, mức che hay có nên gán hay không. Tick xong vẫn gán nhãn tốt nhất có thể. |

Khi ước lượng % che sát ngưỡng, chọn mức cao hơn và tick `can_review`.

## 3. Quy tắc vẽ box
Box ôm sát phần nhìn thấy; vật bị che; vật bị cắt mép ảnh; phản chiếu / hình in / tượng / đồ chơi; xe chủ (ego); vật quá nhỏ; đám đông; một vật có nhiều phần (người và xe)…
- **R-01:** gán box cho mọi vật thuộc 8 lớp (trừ `ignore_region`) nhìn thấy trên ảnh, trừ ca bị loại bởi R-05 đến R-08.
- **R-02:** box ôm sát phần nhìn thấy, lề tối đa 2 px mỗi cạnh, không cắt vào vật. Vật bị che thì chỉ ôm phần thấy được. Vật cắt mép ảnh thì box dừng ở mép.
- **R-03:** người đi xe máy gán hai box riêng: một box `motorcycle` ôm thân xe (kể cả sidecar), và một box `person` cho mỗi người ngồi (kể cả người ngồi sau, trẻ em). Hai box được chồng nhau, không gộp.
- **R-04:** người đạp xe đạp, người ngồi trong tuk-tuk hay xích lô: vẫn gán `person` riêng, và vật chở gán lớp tương ứng (`bicycle`, `three_wheeler`). Xích lô đạp tính là `three_wheeler`, tick `can_review` để nhóm thống nhất.
- **R-05:** vật cao dưới 20 px không gán, không tính là lỗi. Cụm từ 5 vật trở lên chồng lấn đến mức không phân ranh được (đám đông, bãi xe kín): vẽ một `ignore_region` ôm cả cụm, không gán từng vật bên trong. Vật tách rõ ngoài cụm vẫn gán.
- **R-06:** chỉ gán khi nhận ra được lớp. Không chắc thì `occluded_level=3` cùng `can_review`, hoặc bỏ nếu dưới 20 px.
- **R-07:** vật giả không gán: tượng, ma-nơ-canh, đồ chơi hay mô hình xe, hình người in trên biển quảng cáo, phản chiếu (gương, vũng nước, kính). Xe thật có gắn đồ trang trí thì gán box cho xe, không gán đồ trang trí.
- **R-08:** xe chở xe (img_002): một box `truck` cho đầu kéo và khung chở, mỗi ô tô thấy rõ trên khung có thêm một box `car` riêng. Tick `can_review` ở box `truck`.
- **R-09:** hàng hóa và vật mang theo (gánh hàng, ống dài, túi hàng cao) không nằm trong box người hay xe, trừ hàng chất trong thùng xe tải hay bán tải thì nằm trong box xe.
- **R-10:** vật chỉ lộ một phần ở mép ảnh (mũi xe, gương) thì gán nếu còn nhận ra lớp và cao từ 20 px, đặt `truncated=yes`. Chỉ thấy gương hoặc kính mà không nhận ra xe (img_020) thì không gán.
- **R-11:** ảnh mờ do chuyển động hoặc thiếu sáng: nhận ra lớp thì vẫn gán, tick `can_review` nếu rìa nhòe; không nhận ra thì không gán.
- **R-12:** ảnh không có vật nào đủ điều kiện vẫn phải có mặt trong COCO với danh sách box rỗng.

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe chở xe: đầu kéo và khung chở 7–8 ô tô, nhiều xe che nhau | R-08 | `truck` cho đầu kéo + khung; `car` riêng cho từng ô tô thấy rõ, `occluded_level` theo phần bị che; tick `can_review` | Ô tô trên khung là phương tiện nhìn thấy được, nhưng box `truck` bao luôn chúng nên hai box chồng lên nhau, dễ bị hiểu khác |
| img_024 | Xe wagon đen có một xe nhỏ màu vàng đặt trên nóc, nhiều xe cổ trong sảnh | R-06, R-07, R-08 | `car` cho xe đen; xe vàng trên nóc: `car` + `can_review`; xe cổ phía sau gán nếu nhận ra lớp | Chưa rõ xe vàng là xe thật hay mô hình |
| img_005 | Cận cảnh nhiều người đi xe máy, đầu, vai và mũ che nhau | R-03, R-06 | `person` cho từng người thấy rõ; `motorcycle` bị che nhiều: `occluded_level` 2 hoặc 3 + `can_review`; `van` trắng phía sau vẫn gán | Che chồng chéo, ranh giới người và xe máy mơ hồ |
| img_023 | Người đạp xích lô quay lưng, ô tô đen, xe máy xa | R-04 | `three_wheeler` cho xích lô (tick `can_review`); `person` đạp xích lô; `car`; xe máy xa gán nếu từ 20 px | Xích lô chưa có lớp riêng, có thể bị gán `bicycle` |
| img_009 | Xe buýt nhỏ giữa phố | R-01 | `bus` | Ranh giới `van` và `bus`: lấy 16 chỗ làm mốc, tick `can_review` nếu chưa chắc |
| img_012 | Bán tải ở tiền cảnh, thùng chất đầy hàng, bị cắt mép phải và dưới | R-02, R-09, R-10 | `truck` (`truncated=yes`) ôm cả thùng và hàng trong thùng; `car` bạc phía sau | Hàng nằm trong thùng thì thuộc box xe, khác với hàng mang trên người (R-09) |
| img_019 | Xe diễu hành vàng có chuột bông trên xe, đám đông sau hàng rào | R-05, R-07 | `car` cho xe vàng; chuột bông không gán; người lái là `person`; đám đông sau rào dùng `ignore_region` | Xe thật có đồ trang trí; đám đông dày không tách được |
| img_021, img_030 | Đường đông xe máy dày đặc (img_021); bãi xe máy kín (img_030) | R-05 | Xe tách rõ ở tiền cảnh thì gán; cụm dày phía sau thì một `ignore_region` | Không phân ranh từng vật được; cần thống nhất ngưỡng "5 vật chồng lấn" |
| img_014 | Hai người đi bộ nét, xe máy phía sau nhòe do chuyển động | R-11 | `person` ×2; xe máy nhòe: gán nếu nhận ra lớp + `can_review`, nếu không thì bỏ | Cùng một vật có thể bị người này gán, người kia bỏ |
| img_015 | Ô tô đỏ và ô tô bạc bị cắt mép ảnh, nhiều phản chiếu trên mặt đường ướt | R-07, R-10 | `car` ×2 với `truncated=yes`; xe nhỏ ở xa gán nếu từ 20 px; phản chiếu không gán | Phản chiếu dễ bị gán nhầm thành xe thật |
| img_011, img_028 | Mô hình xe máy kim loại (img_011); tượng ngựa trên thùng xe bán tải cổ (img_028) | R-07 | img_011: không gán mô hình xe máy; img_028: gán `truck` cho xe, không gán tượng ngựa | Vật giả cạnh vật thật, dễ gán nhầm |
| img_040 | Người đội mũ bảo hiểm ở tiền cảnh bị cắt mép, taxi xanh đỏ bị cắt, tuk-tuk và SUV phía sau | R-03, R-04, R-10 | `person` (`truncated=yes`); `car` taxi (`truncated=yes`); `motorcycle` chỉ thấy gương, tick `can_review`; `three_wheeler` phía sau; SUV phía sau `car` | Nhiều vật cắt mép và chồng nhau, xe máy gần như không thấy |

Ca cần hỏi reviewer / họp nhóm (chưa chốt):
1. Người trên xe máy: tách hai box (R-03) hay gộp một box.
2. Ô tô trên khung xe chở xe, xe nhỏ trên nóc xe (img_002, img_024).
3. Xích lô đạp thuộc `three_wheeler` hay `bicycle` (img_023).
4. Ngưỡng 5 vật chồng lấn thì chuyển sang `ignore_region` (R-05).
5. Ranh giới `van` và `bus` (tạm lấy 16 chỗ).

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản
Ngưỡng QC áp dụng, ai quyết ca mơ hồ, lịch sử thay đổi:
- Ngưỡng áp dụng (lấy từ `taxonomy.json`, không đổi): chiều cao tối thiểu 20 px; IoU ghép cặp 0.5; IAA IoU 0.85 (vật từ 50 px) và 0.7 (vật nhỏ); đồng thuận lớp 0.95; tỉ lệ lỗi theo ảnh tối đa 2%.
- Người quyết ca mơ hồ: 
- Quy tắc nào làm đổi cách chọn vật, lớp, box hoặc thuộc tính thì phải tăng phiên bản guideline và ghi vào bảng dưới.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên (`guideline_v0_starter.md`) | — |
| v0.1 (051) | Bộ 9 lớp (thêm `three_wheeler`, `bicycle`, `bus`), ngưỡng che, 12 quy tắc R-01…R-12, bảng ca khó | v0 thiếu quy tắc người đi xe máy, vật ngoài taxonomy, vật giả, đám đông, vật cắt mép |
