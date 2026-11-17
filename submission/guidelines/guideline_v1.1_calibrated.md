# Label contract - Phố châu Á - BỔ SUNG v1.1 - 2A202603033 - Bùi Anh Dũng

Chỉ ghi các thay đổi so với guideline v1.0. Các quy tắc không đề cập giữ nguyên theo v1.0.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | case_id |
|---|---|---|---|---|
| 3 | Hàng nhô ra khỏi xe có thuộc bounding box của xe không? | Không tự động tính hàng hóa nhô ra vào box xe; box bao phần xe thuộc đối tượng. | img_003.jpg | CAL-02 |

Các câu hỏi mở khác trong mục 6 của v1.0 chưa được chốt trong đợt calibration này; tiếp tục áp dụng các quy tắc đã rõ và đưa trường hợp mơ hồ vào review.

## 2. Ca khó mới từ lỗi máy và QC

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới |
|---|---|---|---|---|
| img_004.jpg, img_013.jpg | Xe ba bánh có mui | YOLO26n không phát hiện đúng three_wheeler trên 7 mẫu Gold (recall 0/7) | Gán three_wheeler khi xác định được cấu trúc xe ba bánh, không tự động gán car | Mục 2: kiểm tra thủ công xe ba bánh có mui; CAL-01 |
| img_003.jpg | Xe chở hàng có phần hàng nhô ra | Trường hợp cần làm rõ ranh giới box | Không tự động tính hàng hóa nhô ra vào box xe | Mục 3: phân biệt thân xe và hàng hóa; CAL-02 |
| img_006.jpg, img_031.jpg | Bounding box xe đạp chưa sát đối tượng | QC xác nhận box sai vị trí hoặc kích thước; chưa xác định chắc lỗi do model hay thao tác duyệt | Chỉnh box ôm sát phần xe, loại nền và hàng hóa không thuộc xe | Mục 3: kiểm tra vị trí và kích thước box sau prelabel; CAL-03 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản guideline của giảng viên | Quy tắc ban đầu |
| v1.1 | Bổ sung phân loại xe ba bánh có mui, ranh giới box khi chở hàng và kiểm tra độ khít box | CAL-01, CAL-02, CAL-03 |
