# Guideline gán bounding box — Bản nháp Pha 1 — 217

- Phiên bản: v0.2 cá nhân
- Người viết / duyệt: 217
- Trạng thái: Bản cá nhân theo slide Day 7 và cấu hình 10 lớp của Project thử. Các đề xuất cần đối chiếu với guideline v1.0 trước khi dùng làm nhãn chung.

## 1. Mục tiêu

Huấn luyện mô hình phát hiện người thật và phương tiện thật trong ảnh đường phố 2D. Mỗi box cho biết lớp và vị trí của một đối tượng nhìn thấy. Bỏ sót người đi đường làm thiếu ví dụ quan trọng, còn gán nhãn lên tượng hoặc hình phản chiếu làm mô hình học sai đối tượng.

## 2. Lớp và thuộc tính

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ hoặc ca dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con chở người, gồm sedan, hatchback và SUV. | Sedan đỗ ven đường. | Xe có thùng hàng hở đối chiếu `truck`. |
| `truck` | Xe tải, xe chở xe hoặc bán tải có khoang hàng riêng nhận diện được. | Xe tải sàn chở ô tô. | SUV có khoang hành lý liền thân. |
| `van` | Xe thùng kín cỡ nhỏ có thân liền, nhận diện được là van. | Xe giao hàng thân kín. | Xe tải có cabin và thùng hàng tách rõ. |
| `motorcycle` | Xe máy hoặc mô tô hai bánh có động cơ. | Xe ga có người lái. | Xe đạp và tuk-tuk. |
| `person` | Người thật, gồm người đi bộ, người lái và hành khách đủ điều kiện nhìn thấy. | Người ngồi sau xe máy. | Tượng, ma-nơ-canh hoặc người chỉ xuất hiện trên biển quảng cáo. |
| `three_wheeler` | Xe ba bánh có động cơ, gồm tuk-tuk, xe lam và ba gác máy. | Tuk-tuk có mui. | Xích lô đạp thuộc `cyclo`. |
| `bus` | Xe buýt hoặc xe khách cỡ lớn có thân khoang khách nhận diện được. | Xe buýt tại trạm. | Van chở khách cỡ nhỏ. |
| `bicycle` | Xe đạp hai bánh có bàn đạp, nhận diện được cấu tạo xe đạp. | Xe đạp thường. | Xe máy và xích lô ba bánh. |
| `cyclo` | Xích lô đạp ba bánh có ghế chở khách. | Xích lô du lịch Huế. | Tuk-tuk có động cơ. |
| `ignore_region` | Vùng ảnh chứa vật quá nhỏ hoặc dày đến mức không thể gán từng vật đáng tin cậy. | Cụm người rất xa. | Một người nhìn rõ, cao từ 20 px và vẽ được riêng. |

Bảng trên bám theo 10 label của Project thử. Chín label biểu diễn đối tượng và vùng ignore ghi nơi không thể gán riêng đáng tin cậy. Slide 35 minh họa xe súc vật kéo thuộc `other_vehicle`. Nhãn đó nằm ngoài cấu hình thử. Nếu gặp vật ngoài phạm vi cần nhóm xem xét, ghi tên ảnh, vùng và câu hỏi.

| Thuộc tính | Cách điền |
|---|---|
| `occluded_level` | Ước lượng tỷ lệ diện tích bị vật khác che trong phần đối tượng nằm trong khung ảnh. `0` khi không bị che, `1` khi bị che trên 0% và dưới 50%, `2` khi bị che từ 50% trở lên và vẫn đủ dấu hiệu để nhận diện và vẽ. |
| `occluded_level = 3` | Mức che không xác định theo slide 13. Dùng khi có đủ căn cứ vẽ và nhận diện đối tượng nhưng chưa ước lượng được mức che. Tick `can_review` để reviewer kiểm lại. Nếu chỉ có bóng mờ và không nhận diện được vật, xử lý theo R-10. |
| `truncated` | Chọn `yes` khi mép ảnh thực sự cắt một phần đối tượng. Chọn `no` nếu đối tượng chỉ chạm mép mà vẫn xuất hiện trọn vẹn, hoặc chỉ bị vật khác che. Một box có thể vừa bị che vừa bị cắt mép. |
| `can_review` | Tick khi đã vẽ box và còn nghi vấn về lớp, phạm vi hoặc thuộc tính. Nếu chưa đủ căn cứ vẽ, ghi tên ảnh, vị trí và lý do ở cấp ảnh. |

## 3. Quy tắc vẽ box

- **R-01 — Phần nhìn thấy.** Vẽ hình chữ nhật nhỏ nhất bao hết phần nhìn thấy của một đối tượng. Nếu phần nhìn thấy bị chia làm nhiều mảng và xác định được chúng thuộc cùng một vật, dùng một box bao các mảng đó. Dừng tại mép ảnh và tránh kéo box để ước đoán phần thân xe hoặc cơ thể nằm khuất.
- **R-02 — Người trên xe máy.** Vẽ một box `motorcycle` cho phần xe nhìn thấy và một box `person` cho mỗi người lái hoặc hành khách đủ điều kiện. Ví dụ một xe có bốn người nhìn rõ thì có năm box. Hai lớp box có thể chồng lên nhau. Box xe loại phần nhô ra chỉ thuộc người, và box người loại phần nhô ra chỉ thuộc xe.
- **R-03 — Xe chở xe (đề xuất).** Vẽ box riêng cho xe tải và từng ô tô được chở có thể nhận diện, tách riêng và đạt ngưỡng kích thước. Box xe tải bao phần xe tải nhìn thấy, gồm cabin, khung và sàn thuộc xe tải. Các ô tô trên sàn có box riêng theo R-01.
- **R-04 — Kích thước và đám đông.** Đo chiều cao box phần nhìn thấy theo pixel ảnh gốc. Vật cao dưới 20 px được xử lý bằng vùng `ignore_region` khi nhận ra vị trí vùng. Dùng vùng này cho cụm quá dày không thể tách các vật. Với vật trong cùng khu vực vẫn phân biệt và vẽ được riêng, bám sát ranh giới vùng ignore.
- **R-05 — Hình thay thế và phản chiếu.** Loại tượng, ma-nơ-canh, đồ chơi, hình in và hình phản chiếu. Nếu thấy đối tượng thật trực tiếp và đồng thời thấy hình của nó trong kính, chỉ gán đối tượng thật. Vật thật nhìn xuyên qua kính được xét theo các quy tắc thông thường.
- **R-06 — Xe quay ảnh và ca chưa rõ.** Loại phần nội thất, nắp ca-pô hoặc gương thuộc chính xe mang camera. Khi vẽ xe khác trong cảnh, bỏ phần gương chiếu hậu nhô ra theo ví dụ slide 30. Nếu không xác định được đó là xe mang camera hay một xe độc lập trong cảnh, ghi vùng nghi vấn để reviewer xem.

- **R-07 — Tuk-tuk và xe ba bánh.** Một box `three_wheeler` ôm thân và mui nhìn thấy. Theo ví dụ slide 33, khách ngồi dưới mui không có box `person` riêng. Gán riêng `person` cho người cưỡi bên ngoài thân xe khi đủ điều kiện.
- **R-08 — Hàng gắn liền với xe.** Trên xe máy hoặc xe ba bánh, mở box để bao cả phần hàng nhìn thấy đang được chằng, buộc hoặc chở liền với xe. Lớp giữ theo phương tiện. Với ô tô là đối tượng đang được vận chuyển trên sàn xe tải, áp dụng R-03. Ca hàng trang trí, đạo cụ hoặc xe diễu hành cần ghi vào nhật ký nếu khó xác định phạm vi.
- **R-09 — Ranh giới lớp.** Trên bán tải gắn nắp thùng cao, tìm đường tách cabin và thùng trước khi chọn giữa `truck` và `van`. Với xe máy gắn sidecar, đề xuất dùng `motorcycle` nếu nhận diện rõ mô tô và thùng phụ gắn bên. Với xe buýt, xe đạp và xích lô, đối chiếu cấu tạo thân trước khi chọn lớp. Cách tách người và xe cho xe đạp, xích lô là đề xuất cần nhóm chốt.
- **R-10 — Điều kiện quan sát.** Ảnh mưa, tối và ngược sáng vẫn áp dụng ngưỡng kích thước và quy tắc về phần nhìn thấy. Khi có box đủ căn cứ nhưng còn nghi vấn, tick `can_review`. Nếu không đủ căn cứ xác định vật, ghi vùng ảnh để reviewer xem. Chỉ dùng `ignore_region` khi có căn cứ đó là vùng quá nhỏ, quá dày hoặc khó tách đối tượng.

Trình tự cho mỗi ảnh. Xác định vật thật và lớp → xem có tách được instance và đạt 20 px → vẽ phần thấy → điền thuộc tính → gắn cờ nếu cần.

## 4. Ca khó để đối chiếu ảnh gốc

| Ảnh | Tình huống theo tài liệu lab | Quy tắc | Cách xử lý trong bản nháp | Lý do |
|---|---|---|---|---|
| img_002.jpg | Xe tải chở nhiều ô tô. | R-01, R-03 | Một box `truck` và box `car` riêng cho từng xe chở đủ điều kiện. Xác định số box sau khi xem ảnh. | Xe tải và từng xe con là các đối tượng riêng. |
| img_020.jpg | Phản chiếu trong gương chiếu hậu. | R-05, R-06 | Bỏ hình phản chiếu. Xét riêng đối tượng nhìn trực tiếp trong cảnh nếu có. | Hình trong gương có thể tạo box thừa. |
| img_027.jpg | Tượng hình xe bán tải. | R-05 | Loại tượng theo phạm vi người và phương tiện thật. | Hình dáng giống xe chưa đủ để vào lớp phương tiện. |
| img_004.jpg | Tuk-tuk có mui. | R-07 | Box ôm thân và mui từng xe đủ điều kiện. | Dùng lớp xe ba bánh có động cơ. |
| img_035.jpg | Xe ba bánh chở hàng cồng kềnh, cùng ví dụ nguồn ở slide 34. | R-08 | Box ôm cả thân xe và phần hàng gắn liền. | Hàng làm thay đổi phần không gian xe chiếm. |
| img_008.jpg, img_030.jpg | Cảnh có xe máy để đối chiếu quy tắc tách người và xe. | R-02, R-10 | Đếm các xe và từng người đủ điều kiện sau khi xem ảnh gốc. | Số box phụ thuộc phần nhìn thấy. |
| img_009.jpg, img_039.jpg | Xích lô du lịch Huế. | R-09 | Đối chiếu cấu tạo ba bánh đạp và ghế khách để chọn `cyclo`. | Phân biệt với tuk-tuk. |
| img_032.jpg | Cảnh trạm xe buýt. | R-09 | Kiểm từng phương tiện đủ dấu hiệu của lớp `bus`. | Đề phòng nhầm xe khách lớn với van. |
| img_037.jpg | Bán tải gắn nắp thùng cao. | R-09 | Kiểm dấu vết khoang hàng riêng. Gắn cờ nếu còn mơ hồ. | Nắp thùng có thể làm xe trông giống van. |

Ví dụ lý thuyết theo slide 25 và 38. Một xe và một người đủ điều kiện có hai box. Một xe và bốn người đều đủ điều kiện có năm box. Dùng hai ảnh trong bảng để xác nhận số box thực tế.

## 5. QC và phiên bản

- Đo IAA sau khi đủ lượt gán. Script lab ghép box 1–1 theo IoU từ 0,50, cho phép ghép khác lớp để đo mức trùng lớp. Báo IoU trung bình và tỷ lệ cùng lớp trên các cặp đã ghép, với mục tiêu lần lượt từ 0,85 và 95%. Theo dõi riêng box chưa ghép và ghi N/A nếu không có cặp. Slide 60 minh họa ghép cùng lớp khi tính IoU. Khi báo số liệu của repo TOPS, dùng cách ghép và mẫu số của script lab. Dùng ngưỡng IoU 0,85 khi chiều cao trung bình của cặp từ 50 px, và 0,70 khi thấp hơn.
- Reviewer mở ảnh gốc cùng nhãn để kiểm năm loại lỗi: bỏ sót, thừa, sai lớp, sai vị trí/kích thước và sai thuộc tính. Duyệt toàn bộ lô trước khi bàn giao. Tỷ lệ ảnh lỗi bằng số ảnh có lỗi chia cho số ảnh đã duyệt. Một ảnh lỗi trong 40 ảnh là 2,5%, vượt ngưỡng 2%.
- Hold khi thiếu bằng chứng hoặc quy tắc. Rework khi vượt ngưỡng hoặc lỗi lặp theo cùng quy tắc. Release khi reviewer xác nhận lô đã duyệt đầy đủ và đạt tiêu chí. Ghi ca bất đồng, mã quy tắc, người chốt và phiên bản trong calibration.

| Phiên bản | Thay đổi | Căn cứ |
|---|---|---|
| v0.1 cá nhân | Bổ sung tiêu chí vẽ box, thuộc tính, ca khó và ngưỡng QC. | Tóm tắt slide Day 7, taxonomy starter và hướng dẫn QC của repo TOPS. |
| v0.2 cá nhân | Bổ sung 4 lớp và quy tắc cho tuk-tuk, hàng cồng kềnh, vùng ignore, trường hợp khó quan sát. | Slide Day 7 và taxonomy Project thử. |

Nguồn: [Slide Day 7 — Guideline, Taxonomy & Quy trình QC](https://aithucchien.vinuni.edu.vn/path-player?courseid=ai20k-cohort-5-l2-3&unit=6af9dd5019c87d2816097913Unit), slide 12–16, 21–43, 49–66 và 71–82. Tên ảnh đối chiếu bằng [bảng nguồn ảnh](../../../data/ATTRIBUTION.csv).
