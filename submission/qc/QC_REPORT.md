# Báo cáo QC — TOPS

Trạng thái bàn giao: **Hold**. Phần đã làm có số liệu thật; phần chưa làm được ghi rõ là chưa làm.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: đặc tả và QC nhãn 2D (người và phương tiện, 9 lớp) cho ảnh đường phố; tập 40 ảnh tuyển chọn để luyện ca biên, chưa đánh giá độ đại diện cho dữ liệu vận hành.
- Nguồn và phép sử dụng dữ liệu: xem `data/ATTRIBUTION.csv` (tác giả, giấy phép, đường dẫn nguồn từng ảnh).
- Manifest và checksum nhãn cuối: chưa có. `batch_manifest.json` chỉ được tạo khi chạy `build_final.py` sau khi có COCO calibration; chưa làm.
- Guideline và taxonomy khi gán / khi bàn giao: gán theo guideline v1.0 và taxonomy v1.0. Đã soạn guideline v1.1 (thêm R-15, R-16, làm rõ R-12) và `guideline_change.json` (`semantic`); chưa kiểm v1.1 trên nhãn thực tế. Taxonomy giữ v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 (5 người, mỗi người 24 ảnh, 40 ảnh, mỗi ảnh 3 người gán).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 708 cặp box ghép (IoU ≥ 0.5, ghép 1-1); IoU TB 0.872 (ngưỡng 0.85); cùng lớp 95.9% (ngưỡng 95%). Chỉ 4/10 cặp người gán đạt; chưa đạt: 049|064, 049|217, 051|061, 051|064, 061|064, 064|217. Cùng `occluded_level` 42.7%; cùng `truncated` 92.8%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: chưa ghép 184/288 (tổng A/B theo cặp); bỏ qua do ignore 52; 232 ca `bat_dong_ignore`.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: ứng viên C477 và C480 (img_011): 049 và 051 cùng gán `van` cho xe trắng bị tượng che, 064 gán `car`; v1.1 đề xuất `car` + `can_review`. Chưa xác nhận bằng duyệt nhóm; ghi là ứng viên, không phải kết luận.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 1456 ca (sai_thuoc_tinh 427, bo_sot_thua 308, co_can_review 262, bat_dong_ignore 232, iou_thap 198, sai_lop 29) có một dòng quyết định mỗi ca trong `submission/qc/calibration_decisions.csv`. Đây là quyết định đề xuất, điền theo nhóm lỗi; **bản sửa nhãn chưa thực hiện và chưa có người kiểm lại**, nên cột `phieu_sua` và `kiem_lai` ghi trạng thái chưa làm.
- Phát hiện từ xem ảnh chồng nhãn img_011 và img_016 (13 ca `sai_lop`): hộp trùng nhau (IoU 0.89–0.96) nhưng lớp của 064 không hợp kích thước vật (hộp cỡ người gán `bus`, hộp xe gán `person`); hai ca C566, C569 là ghép nhầm hai vật chồng nhau (tuk-tuk trước, bán tải sau), dẫn tới R-16. 16 ca `sai_lop` ở các ảnh còn lại chưa phân tích.
- Tác động v1.1 lên nhãn và ảnh: đổi nghĩa, phạm vi `lo`, nghĩa là phải kiểm lại cả 40 ảnh; chưa thực hiện.
- Tham chiếu bằng chứng khi Rework: không áp dụng (chưa có vòng QC).

## 4. Kết quả duyệt

- Vòng QC: chưa thực hiện.
- Người review và ngày: chưa có.
- Đã duyệt / cả lô: 0 / 40 ảnh.
- Ảnh lỗi / ảnh đã duyệt: không có số liệu (chưa duyệt).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: không có số liệu (chưa duyệt).
- Vật bỏ sót, ảnh và vùng: chưa duyệt.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: lỗi lặp quan sát được ở bước IAA, chưa qua vòng duyệt: 064 gán lớp không hợp kích thước (img_011, img_016); `occluded_level` khớp thấp (42.7%).
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: máy chặn: lớp `bicycle` ngoài taxonomy trong bản của 049 trước khi cập nhật taxonomy, và box 16 px ở img_039 (#128) bị cảnh báo; cần người đọc ảnh: hộp cỡ người bị gán `bus` ở img_011, vì `bus` có trong taxonomy nên script không bắt được.

## 5. Quyết định bàn giao

- Quyết định: **Hold**.
- Lý do, người nhận và phạm vi sử dụng: chưa có nhãn cuối sau calibration và chưa có vòng duyệt QC, nên chưa đủ bằng chứng để Release. Dùng để chấm quy trình đến hết IAA và phân tích calibration; không dùng làm nhãn huấn luyện.
- Nếu còn việc: (1) sửa nhãn calibration trong CVAT cho 40 ảnh theo `calibration_decisions.csv` và v1.1, lưu `labels_calibrated_<mã>.json`; (2) người kiểm khác người sửa cập nhật `phieu_sua`, `kiem_lai`; (3) chạy `build_final.py`; (4) `qc_sample.py`, duyệt toàn lô, `qc_report.py`; (5) xác nhận hoặc bác ca cùng sai C477, C480; (6) phân tích 16 ca `sai_lop` còn lại. Người xử lý: 051.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: ví dụ C565 (img_016): `calibration_candidates.csv` → ảnh chồng nhãn `work/viz/img_016.png` (064 gán `person` cho hộp cỡ ô tô, 061 gán `car`) → R-12 → nhãn calibration chưa sửa (Hold).
