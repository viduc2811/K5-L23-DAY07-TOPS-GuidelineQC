Ghi chú cá nhân — <064>
Bản nháp được xây dựng từ danh sách ảnh phân công trong 064.txt và ảnh tương ứng trong images.zip. Chỉ sử dụng ảnh được phân công cho cá nhân này. Các nội dung chưa có dữ liệu đối chiếu được đánh dấu [CẦN BỔ SUNG], không tự suy đoán.

1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh
Ảnh	v0 không trả lời được gì	Quyết định ban đầu của tôi
img_007.jpg	[CẦN BỔ SUNG: kết quả chạy v0. Quan sát ảnh: không gian trong nhà/nhà kho tối, nhiều xe và vật thể bị che khuất; cần đối chiếu đầu ra v0 để xác định chính xác phần không trả lời được.]	Đánh dấu đây là ca cần kiểm tra kỹ các đối tượng bị che khuất và ranh giới box. Chưa chốt nhãn/box nếu chưa có guideline và đầu ra v0.
img_014.jpg	[CẦN BỔ SUNG: kết quả chạy v0. Quan sát ảnh: người đi bộ và phương tiện chuyển động, có nhòe chuyển động; ranh giới đối tượng có thể khó xác định.]	Ưu tiên kiểm tra vùng nhòe và xác định phạm vi đối tượng theo đúng quy tắc guideline; không tự suy diễn phần bị nhòe.
img_033.jpg	[CẦN BỔ SUNG: kết quả chạy v0. Quan sát ảnh: ảnh trưng bày trong cửa hàng, mannequin và quần áo/đồ vật ở gần nhau; có thể dễ nhầm giữa người thật, mannequin và vật thể.]	Cần áp dụng đúng định nghĩa lớp đối tượng trong guideline trước khi gán. Nếu guideline chưa nói rõ mannequin có thuộc lớp cần gán hay không thì đưa vào nhóm cần hỏi lại.


2. Pha 2: chỗ tôi bất đồng với nhóm
- Bản của tôi khác bản của bạn ở điểm nào: [CẦN BỔ SUNG — cần bản gán/ý kiến của tôi và của thành viên đối chiếu.]
- Cuối cùng nhóm chốt gì: [CẦN BỔ SUNG — cần biên bản trao đổi hoặc quyết định cuối của nhóm.]
- Tôi đồng ý / chưa đồng ý vì sao: [CẦN BỔ SUNG — ghi lý do thực tế của cá nhân, không suy đoán.]
3. Pha 3: ảnh tôi đã gán
- Số ảnh được giao: 25 ảnh.
- Danh sách ảnh được giao: img_001, img_002, img_005, img_007, img_009, img_010, img_011, img_012, img_014, img_016, img_018, img_019, img_021, img_024, img_025, img_026, img_028, img_029, img_032, img_033, img_034, img_036, img_037, img_040.
- Số box đã vẽ: [CẦN BỔ SUNG — cần file annotation/kết quả gán box.]
- Số lần tôi tick can_review: [CẦN BỔ SUNG — cần log hoặc dữ liệu review.]
- Ca tôi lưỡng lự nhất khi gán: tạm đề xuất img_033.jpg vì ảnh có mannequin trong bối cảnh trưng bày quần áo, dễ phát sinh cách hiểu khác nhau về đối tượng nào được tính là người. Đây mới là nhận xét từ ảnh; cần đối chiếu guideline và thao tác gán thực tế để xác nhận đây có đúng là ca tôi lưỡng lự nhất hay không.
4. Pha 4–5: tôi lệch ở đâu
- Các ca có tên tôi trong calibration_candidates.csv: [CẦN BỔ SUNG — chưa có tệp calibration_candidates.csv để kiểm tra.]
- Tôi lệch vì guideline mơ hồ hay vì chưa nắm quy tắc: [CẦN BỔ SUNG — cần danh sách ca lệch, nhãn chuẩn và phần giải thích calibration để phân loại nguyên nhân.]
- Quyết định calibration tôi thấy quan trọng nhất: [CẦN BỔ SUNG — cần nội dung các quyết định calibration của nhóm.]
5. Dùng AI
- Công cụ: ChatGPT.
- Việc sử dụng: đọc danh sách ảnh được phân công, xem các ảnh tương ứng trong bộ ảnh và hỗ trợ soạn cấu trúc ghi chú cá nhân; nhận diện sơ bộ những ảnh có tình huống trực quan dễ gây nhầm lẫn để đưa vào danh sách cần kiểm tra.
- Cách kiểm tra: đối chiếu tên ảnh với 064.txt, chỉ xem ảnh có tên trong danh sách được giao và kiểm tra nhận xét trực quan trên ảnh gốc trong images.zip. Các nhận xét về nhãn, box, kết quả v0 và calibration chưa được xem là kết luận nếu chưa có dữ liệu gốc đối chiếu.
6. Bài học
Một điều tôi sẽ làm khác khi viết guideline lần sau: mô tả rõ các trường hợp biên bằng ví dụ cụ thể, đặc biệt là đối tượng bị che khuất/nhòe và trường hợp mannequin hoặc vật thể có hình dáng giống người. Với mỗi trường hợp, guideline nên nêu rõ điều kiện được gán, không được gán và cách xử lý khi không đủ căn cứ để quyết định.