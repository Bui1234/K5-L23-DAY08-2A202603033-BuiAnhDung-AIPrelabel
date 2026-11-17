# Báo cáo Day 08 — 2A202603033 - Bùi Anh Dũng

## 1. Model và recipe (Part 2–3)

| Lớp | Model và cỡ | Ngưỡng | Cơ sở lựa chọn |
|---|---|---|---|
| person, bicycle, motorbike, bus, truck | YOLO26n | 0.25 mặc định khi merge | `r02`: 148 box/8 ảnh Pilot, 0.90 giây/ảnh; quét Gold chưa tìm được ngưỡng theo lớp đạt mục tiêu recall 90%, hoặc số mẫu Gold dưới 5 |
| car | YOLO26n | 0.15 được đề xuất từ quét Gold; batch đã chạy ở 0.25 | `threshold_sweep.md`: recall 9/9 = 100%, precision khoảng 47% trên 4 ảnh Gold; chưa áp dụng lại ngưỡng 0.15 vào bản Batch đã review |
| three_wheeler | YOLO26n kết hợp kiểm tra/gán tay | 0.25 mặc định khi merge; không có ngưỡng riêng đáng tin cậy | Quét Gold: recall 0/7; bảng review có 20 đối tượng three_wheeler được thêm thủ công; model chưa đủ tin cậy cho lớp này |

Đối chứng `r01`: YOLO11n tạo 163 box/8 ảnh Pilot, 1.23 giây/ảnh. `r03`: YOLO26n tại confidence 0.50 còn 81 box/8 ảnh, 0.83 giây/ảnh, nguy cơ bỏ sót tăng. Vì vậy lựa chọn YOLO26n confidence 0.25 (`r02`) cho Batch, thay vì 0.50. Bài thực nghiệm không có kết quả chạy YOLOE-26; YOLO11n được sử dụng làm đối chứng.

Lệnh trên gộp hai tập dự đoán YOLO26n, **không phải phép ensemble hai model khác nhau**. Kết quả gộp 422 box trên Batch và Gold. Ngưỡng `car=0.15` được đề xuất sau đó từ quét Gold nhưng chưa được áp dụng lại vào lô đã duyệt.

SAM chọn: **SAM 2.1 Tiny**, checkpoint `models/sam2.1_t.pt`, chạy CPU. Ở `r04`, YOLO26n + SAM 2.1 Tiny tạo 8 mask từ 8 box trên 2 ảnh (`img_004.jpg`, `img_013.jpg`), trung bình 6.63 giây/ảnh. SAM phụ thuộc box/điểm gợi ý nên không tự khắc phục trường hợp YOLO bỏ sót tuk-tuk; cần thêm điểm hoặc box thủ công.

Sai lớp `three_wheeler` và vật thể bị bỏ sót không giải quyết triệt để chỉ bằng tăng/giảm confidence. Cách xử lý là đối chiếu ảnh gốc, sửa lớp hoặc thêm đối tượng thủ công, sau đó kiểm tra lại box. Các trường hợp confidence cao nhưng sai hoặc thấp nhưng đúng chưa được lập bảng định danh theo từng box, vì vậy không khẳng định một ID cụ thể. Việc chọn ngưỡng và đánh giá trên cùng 4 ảnh Gold khiến số đo có nguy cơ lạc quan và độ tin cậy hạn chế.

## 2. AI error profile

Nguồn: `submission/qc/review_stats.md`. Số liệu loại 20 đối tượng nằm quanh bẫy nhân tạo.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 224 | 224/366 = 61.2% | 366 box máy |
| edited | 88 | 88/366 = 24.0% | 366 box máy |
| deleted | 53 | 53/366 = 14.5% | 366 box máy |
| added | 202 | 202/514 = 39.3% | 514 đối tượng sau duyệt |

Có 9 trường hợp đổi lớp trong nhóm xóa/thêm; ước lượng số vật bị máy bỏ sót thực sự khoảng 193. Lớp yếu nhất theo bằng chứng là `three_wheeler`: 0 accepted và 20 added trong bảng review, đồng thời recall trên Gold là 0/7. Ví dụ liên quan xe ba bánh: `img_004.jpg`, `img_013.jpg`. Trên 4 ảnh Gold có 55 đối tượng: TP=28, FP=18, FN=27, precision 28/(28+18)=60.9%, recall 28/55=50.9%. Đây là số đo trên mẫu nhỏ.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Đã bắt được **12/12 = 100%** bẫy: `loose` 3/3, `wrong_class` 3/3, `missing` 3/3, `extra` 3/3; không có loại bẫy nào lọt trong 12 bẫy đã kiểm. Khoảng tin cậy 95% được script báo là 76–100%, vì vậy không thể suy ra mọi bẫy ngoài mẫu đều sẽ được phát hiện. Accepted 224/366 = 61.2% không thay thế cho kiểm tra độc lập: phải quét toàn ảnh để tìm vật bỏ sót, xem nhãn và kiểm tra box dù máy tự tin.

## 4. Máy có đáng dùng không?

Nguồn: `submission/time_log.csv` (thời gian ước tính do người thực hiện xác nhận). Gold gán tay 4 ảnh trong 15 phút = **3.75 phút/ảnh**. Batch duyệt pre-label 24 ảnh trong 60 phút = **2.50 phút/ảnh**. Chênh lệch quan sát là 1.25 phút/ảnh, tương đương khoảng 33.3% so với thời gian gán tay; đây không phải phép so sánh thực nghiệm ngẫu nhiên vì hai tập ảnh khác nhau, và người gán có thể đã quen cảnh.

Kết luận: AI hữu ích để tạo gợi ý ban đầu cho một số lớp phổ biến, nhưng chưa đủ để tự động xuất nhãn không qua người duyệt, đặc biệt với `three_wheeler` và vật nhỏ/che khuất.

Ba quy tắc giảm thiên kiến: (1) quét toàn ảnh trước khi chỉ sửa các box có sẵn, ghi lại đối tượng added; (2) kiểm tra lớp và độ khít của từng box, ghi edited/deleted theo thống kê; (3) chèn bẫy và kiểm mẫu QC độc lập, chỉ quyết định Release khi chỉ số đạt ngưỡng.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Số click | Người lái / hàng hóa | Lỗi quan sát / giới hạn | File trong `submission/masks/` |
|---|---|---|---|---|---|
| img_004.jpg | SAM 2.1 Tiny | Chưa xác minh tổng số click; có điểm dương, điểm âm và box | Cần loại khỏi mask xe nếu không thuộc thân xe | Xe ba bánh xanh, vật thể bị che khuất, cần kiểm tra biên | `sam_img004_green_tuktuk.jpg` |
| img_004.jpg | SAM 2.1 Tiny | Chưa xác minh tổng số click; có điểm dương, điểm âm và box | Cần loại khỏi mask xe nếu không thuộc thân xe | Xe ba bánh xanh dương, biên mask cần kiểm tra thủ công | `sam_img004_blue_tuktuk.jpg` |
| img_013.jpg | SAM 2.1 Tiny | Chưa xác minh tổng số click; có điểm dương, điểm âm và box | Người và hàng hóa che một phần xe, không tự động tính vào mask xe | Biên mask chưa hoàn hảo ở vùng chồng lấp | `sam_img013_03.jpg` |

Có thêm hai file minh họa `sam_img013_01.jpg` và `sam_img013_02.jpg`. Quy tắc phân biệt thân xe với người/hàng hóa giúp đánh giá mask; tình huống che khuất và biên kính/cửa sổ cần tiếp tục hiệu chỉnh theo guideline. Không có bằng chứng đã triển khai SAM Interactor trực tiếp trong CVAT; các kết quả trên là thực nghiệm offline.

## 6. Calibration (Part 6)

Có **3 quyết định** trong `submission/qc/calibration_decisions.csv`: CAL-01 (`loi_may`) xác định xe ba bánh có mui là `three_wheeler`; CAL-02 (`cau_hoi_mo`, câu hỏi số 3 v1.0) không tự động tính hàng hóa nhô ra vào box xe; CAL-03 bổ sung kiểm tra độ khít box sau pre-label. Các quy tắc và ảnh ví dụ `img_003.jpg`, `img_004.jpg`, `img_006.jpg`, `img_013.jpg`, `img_031.jpg` đã ghi trong `submission/guidelines/guideline_v1.1_calibrated.md`. Nguyên nhân gồm hạn chế nhận diện của máy, ranh giới đối tượng chưa rõ và sai lệch vị trí/kích thước được phát hiện khi QC; chưa xác định chắc CAL-03 bắt nguồn từ model hay thao tác duyệt.

## 7. QC và quyết định (Part 6)

Nguồn: `submission/qc/qc_sample.csv`, `qc_review.csv`, `qc_report_auto.md`. Mẫu **5/24 ảnh Batch** (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42), gồm **89 đối tượng**. Người kiểm: Bùi Anh Dũng; hạn chế là người kiểm cũng tham gia duyệt nhãn.

Lỗi theo đối tượng: **5/89 = 5.6%**. Ảnh có lỗi: **5/5 = 100%**, vượt ngưỡng Release ≤2%. Loại lỗi ghi nhận là `sai_vi_tri_kich_thuoc`, 5 trường hợp; ảnh ví dụ `img_003.jpg` (ID 18), `img_006.jpg` (ID 60), `img_014.jpg` (ID 122), `img_031.jpg` (ID 405), `img_035.jpg` (ID 454). Không kết luận các loại lỗi khác bằng 0 trên toàn bộ dữ liệu chỉ từ mẫu này.

Cận trên Wilson 95% của tỉ lệ ảnh lỗi được script báo là **100%**. Với chỉ 5 ảnh, không thể chứng minh ngưỡng 2%. Script kiểm được cấu trúc, số lượng và thống kê theo phiếu; không tự chứng minh box ôm sát vật thật, phát hiện vật mà cả máy và người cùng bỏ sót, hoặc xác minh người kiểm có tuân thủ checklist.

**Quyết định: REWORK — 2026-11-17.** Lý do: 5/5 ảnh QC có lỗi, cao hơn ngưỡng 2%, và lỗi vị trí/kích thước lặp lại. Đây là quyết định trên bản Release Candidate, **không phải xác nhận đã sửa xong**. Việc tiếp theo: sửa 5 box đã xác nhận trong CVAT, xuất `submission/labels/qc_fixed.json` chỉ gồm các ảnh sửa, chạy lại `build_release.py`, lấy mẫu/kiểm QC vòng tiếp theo và ghi lịch sử thay đổi trước khi cân nhắc Release.

## 8. Ghi chú cá nhân

- Trong quá trình làm bài, em có sử dụng ChatGPT để hỗ trợ đọc hướng dẫn, phân tích lỗi và viết báo cáo. Các bước chạy mô hình, gán nhãn và kiểm tra QC đều do tôi thực hiện.
- Em sử dụng máy Windows, CPU Ryzen 5 5500U, không có GPU CUDA nên chạy SAM khá chậm. Khó khăn lớn nhất là mô hình bỏ sót xe ba bánh và một số bounding box chưa chính xác.
- Qua bài này, tôi nhận thấy AI giúp tiết kiệm thời gian nhưng vẫn cần kiểm tra thủ công. Lần sau, em sẽ chú ý hơn đến các lớp khó và thực hiện QC sớm để hạn chế lỗi trước khi nộp.
