# Guideline gán nhãn — v0.1 (bản cá nhân) — 101

- Phiên bản: v0.1 (bản cá nhân, chưa thống nhất với nhóm)
- Ngày: 2026-11-16
- Người viết / duyệt: 061 (051 duyệt)

## 1. Mục tiêu

Nhãn dùng huấn luyện mô hình **phát hiện vật 2D** (bounding box) cho camera giao thông / xe tự lái trên ảnh đường phố: tìm **người và phương tiện thật đang tham gia giao thông**.

Sai ở đâu thì nguy hiểm:
- **Bỏ sót** người hoặc xe thật → mô hình không thấy vật cản (nguy hiểm nhất).
- **Gán nhầm vật giả** (tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu) là người/xe → mô hình báo động giả, phanh nhầm.
- **Lệch lớp** giữa các xe gần giống nhau (xe buýt / xe tải / tuk-tuk) → sai logic tốc độ, kích thước.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con 4 bánh chở người, gồm sedan, hatchback, SUV, bán tải thùng hở. | Ô tô con thường; các xe con nằm trên sàn xe chở ô tô (`img_002`). | Xe thùng kín cỡ nhỏ → `van`; xe mô hình / đồ chơi (`img_038`) và tượng xe bán tải (`img_027`) **không phải** `car`. |
| `truck` | Xe tải có thùng hàng riêng hoặc thùng chở hàng lớn. | Xe tải chở 8 ô tô con (`img_002`). | Bán tải gắn nắp thùng cao (`img_016`, `img_037`) dễ nhầm `van`/`truck`/`car`, xem R-09. |
| `van` | Xe thùng kín cỡ nhỏ (khoang hàng liền thân, không có buồng lái tách rời). | Xe van giao hàng. | Xe buýt (to hơn, nhiều cửa sổ) → `bus`. |
| `bus` | Xe buýt chở khách cỡ lớn, thân dài nhiều cửa sổ. | `img_009`, `img_031`, `img_037`, `img_040`. | Xe van chở khách cỡ nhỏ → `van`. |
| `tuk_tuk` | Xe ba bánh có động cơ chở người hoặc hàng. | `img_004`, `img_016`, `img_035`. | Xích lô / xe đạp không động cơ (xem phần "không thuộc lớp nào"). |
| `motorcycle` | Xe máy, mô tô hai bánh có động cơ, **chỉ phần xe**, không gồm người. | Xe máy đang chạy. | Xe đạp không động cơ → không gán. |
| `person` | Người thật (đi bộ, đi xe, ngồi trên xe). | Người đi bộ; người lái và người ngồi sau xe máy. | Tượng (`img_011`, `img_017`, `img_027`), ma-nơ-canh (`img_033`), chân dung in trên biển quảng cáo (`img_018`), người trong gương chiếu hậu (`img_020`). |
| `ignore_region` | Vùng không gán từng vật: đám đông dày đặc hoặc cụm vật nhỏ cao < 20 px. | Đám đông xa, không tách được từng người. | Vật to, rõ, tách được thì **phải** gán box riêng, không được dùng ignore để né. |

**Vật không thuộc lớp nào** (xe đạp, xích lô không động cơ `img_006`, `img_029`, `img_039`; biển hiệu; đồ vật):
- **Không gán box** cho chính vật đó.
- Người ngồi / đạp trên xe vẫn là người thật → vẫn gán `person`.
- Nếu không chắc, tick `can_review` trên box liên quan hoặc ghi vấn đề cấp ảnh vào `image_issues.csv`. Không tự đặt lớp mới.

### Thuộc tính

| Thuộc tính | Các mức cố định | Ngưỡng cụ thể |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: thấy hết, che 0%. `1`: che 1–30% diện tích. `2`: che 31–70%. `3`: che > 70% nhưng vẫn nhận ra chắc chắn là vật đó. Che > 70% mà không chắc là vật gì thì **không gán**, ghi `can_review` / vấn đề cấp ảnh. Chỉ dùng thuộc tính này, **không** dùng cờ `occluded` (phím `Q`) của CVAT. |
| `truncated` | `no`, `yes` | `yes` khi bất kỳ cạnh nào của vật chạm / bị cắt bởi mép ảnh (vật còn phần nằm ngoài khung). Ngược lại `no`. |
| `can_review` | tick / không | Tick khi bạn **không chắc** về lớp, phạm vi box hoặc mức che. Không tick cho ca đã có quy tắc rõ. |

## 3. Quy tắc vẽ box

- **R-01 — Ôm sát phần nhìn thấy.** Box vừa khít phần nhìn thấy của vật, không chừa viền thừa; không vẽ cả phần bị che. Không bao hàng hóa, bao tải, thùng chất cao vượt thân xe (`img_035`).
- **R-02 — Vật không thật thì không gán.** Tượng, ma-nơ-canh, đồ chơi, xe mô hình, hình in, chân dung trên biển quảng cáo → không gán. Ảnh chỉ có các vật này vẫn giữ nguyên, 0 box (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`).
- **R-03 — Phản chiếu không gán.** Người / xe thấy qua gương chiếu hậu, kính, mặt nước không gán box (`img_020`).
- **R-04 — Xe chủ (ego).** Phần thân xe mà camera gắn trên đó (capô, nội thất xe quay) không gán. Ghi vấn đề cấp ảnh nếu không chắc.
- **R-05 — Xe máy có người.** Mỗi xe máy = **1 box `motorcycle`** (chỉ phần xe) + **1 box `person` cho mỗi người** (chỉ phần người nhìn thấy). Các box được phép chồng lên nhau. Không gộp người và xe vào một box.
- **R-06 — Xe chở xe.** Box `truck` ôm xe tải. Mỗi ô tô con trên sàn nhìn thấy được gán thêm một box `car` riêng (tính `occluded_level` theo phần bị che) (`img_002`: 1 `truck` + 8 `car`).
- **R-07 — Vật bị cắt mép ảnh.** Box ôm phần nhìn thấy, đặt `truncated = yes`; cạnh box nằm sát mép ảnh, không vượt ra ngoài khung.
- **R-08 — Vật bị che một phần.** Box chỉ ôm phần nhìn thấy; chọn `occluded_level` theo ngưỡng ở trên.
- **R-09 — Vật nhỏ, mờ.** Vật cao < 20 px: nếu đứng lẻ, tách được → vẫn vẽ box và tick `can_review`; nếu là cụm / đám đông không tách được → một `ignore_region`. Vật mờ do chuyển động (`img_014`, `img_015`) nhưng nhận ra được → gán bình thường, tick `can_review`.
- **R-10 — Đám đông.** Tách được từng người thì gán từng người. Không tách được → `ignore_region` phủ đám đông, không gán từng box.
- **R-11 — Vỏ xe / vật chở trên nóc.** Thân vỏ xe đặt trên nóc xe khác (`img_024`) không tính là xe riêng; không gán box riêng cho vật đang được chở.
- **R-12 — Bán tải nắp thùng cao.** Nếu có buồng lái riêng + thùng hàng sau → `truck`; không chắc thì chọn `truck` và tick `can_review` (`img_016`, `img_037`).

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe tải chở 8 ô tô con trên hai sàn | R-06 | 1 `truck` + 8 `car` | Mô hình phải thấy mỗi xe con; bỏ sót là lỗi `bo_sot`. |
| img_004, img_035 | Xe ba bánh tuk-tuk | R-01 | `tuk_tuk` | Không phải `car`/`motorcycle`; mô hình cần phân biệt. |
| img_035 | Tuk-tuk chất bao tải cao trên nóc | R-01 | Box ôm thân xe, không bao khối bao tải | Hàng hóa không thuộc xe; tránh box phình to. |
| img_009, img_037 | Xe buýt | — | `bus` | Cỡ lớn, khác `truck`/`van`. |
| img_037, img_016 | Bán tải nắp thùng cao (Master 75) | R-12 | `truck` + `can_review` | Ranh giới `van`/`truck`, đưa ra họp nhóm. |
| img_011, img_017, img_027 | Tượng đồng / đá, tượng xe bán tải | R-02 | Không gán | Không phải người / xe thật. |
| img_033, img_038 | Ma-nơ-canh, đồ chơi trong tủ kính | R-02 | Không gán | Vật giả. |
| img_018 | Chân dung in trên biển quảng cáo | R-02 | Không gán | Hình in. |
| img_020 | Người thấy qua gương chiếu hậu | R-03 | Không gán | Phản chiếu. |
| img_006, img_029, img_039 | Xe đạp, xích lô không động cơ | — | Không gán xe; gán `person` nếu có người | Không thuộc lớp nào; người vẫn là vật thật. |
| img_014, img_015 | Vật mờ do chuyển động | R-09 | Gán + `can_review` | Nhận ra được nhưng biên không rõ. |
| img_024 | Vỏ xe đặt trên nóc xe | R-11 | Không gán box riêng cho vỏ xe | Là vật được chở. |

Ca chưa chốt: đánh dấu `can_review` trên box hoặc ghi vào `image_issues.csv`, dùng mã R-xx ở trên.

## 5. QC và phiên bản

- Ngưỡng QC áp dụng (cố định trong `taxonomy.json`): box cao tối thiểu 20 px; IoU ghép cặp ≥ 0.5; IAA IoU trung bình ≥ 0.85; đồng thuận lớp ≥ 95%; tỉ lệ ảnh lỗi ≤ 2%.
- Ca mơ hồ: tick `can_review`; người điều phối và nhóm chốt ở buổi calibration (Pha 5).
- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1 | Thêm `bus`, `tuk_tuk`; ngưỡng `occluded_level`; quy tắc R-01…R-12; bảng ca khó | `img_002`, `img_004`, `img_009`, `img_035`, `img_037`… |
