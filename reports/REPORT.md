# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Ngô Văn Cao

Công cụ gán nhãn đã dùng: CVAT


## 1. Dữ liệu và cách chia tập

Pool và test set được chia theo thời gian và có vùng đệm để tránh các frame gần nhau, gần như giống nhau, xuất hiện ở cả hai tập.

Nếu chia ngẫu nhiên, test set dễ chứa các frame rất giống dữ liệu train/pool, làm kết quả đánh giá **cao hơn thực tế** do rò rỉ dữ liệu và không phản ánh đúng khả năng tổng quát hóa của mô hình.

## 2. Mô hình khởi đầu lạnh (cold start)

Vòng 0: yolov8n cold start (COCO car+bus+truck), 0 ảnh train, 0 box train, AP50 = 0.771, P = 0.925, R = 0.489, F1 = 0.640.
Mô hình cold start chủ yếu bỏ sót xe nhỏ, xe ở xa, xe tối hoặc bị che khuất. Recall theo kích thước cho thấy xe nhỏ khó phát hiện nhất: R small = 0.182, thấp hơn nhiều so với medium 0.547 và large 0.561.
Trước khi kết luận model sai, nên rà lại các box tham chiếu rất nhỏ hoặc khó nhìn ở xa, vì có thể chính nhãn tham chiếu chưa chính xác hoặc chưa nhất quán.

## 3. Chiến lược chọn mẫu

score = W_U·U + W_A·A + W_D·D nghĩa là điểm chọn ảnh được tổng hợp từ độ bất định của model (U), mức độ khó/ambiguous (A) và độ đa dạng (D); các W là trọng số quyết định yếu tố nào quan trọng hơn. MIN_GAP_S đặt khoảng cách thời gian tối thiểu giữa các frame được chọn để tránh lấy nhiều ảnh gần như trùng nhau.
## 4. Các vòng học chủ động (active learning)

0	yolov8n cold start	0	0	0.771	—	0.925	0.489	0.640	0.182	0.547	0.561
1	yolov8n fine-tune vòng 1	12	296	0.311	-0.460	1.000	0.005	0.010	0.000	0.003	0.024


Vòng 1: từ 169 box pre-label, tôi giữ nguyên 140, chỉnh 14, xoá 15 box sai và thêm 142 box bị bỏ sót.   round1_diff AP50 giảm 0.460 so với cold start và cũng giảm 0.460 so với vòng trước. Recall của cả xe small, medium và large đều giảm mạnh.
Trong compare_round1.jpg, frame_0050 xấu đi từ TP11, FP2, FN7 ở cold start thành TP0, FP0, FN18 sau fine-tune, cho thấy model vòng 1 gần như bỏ sót toàn bộ xe.
BLIND_SCAN.md là quan sát độc lập trước khi xem pre-label: frame_0099 có 22 xe, trong đó 2 xe truncated ở góc dưới phải dễ bị bỏ sót.   BLIND_SCAN Sau đó round1_diff.md cho thấy frame này phải thêm 10 box và chỉnh 5 box, tức pre-label thực sự có lỗi.   round1_diff
Một ca khó theo guideline là xe tối gần mép trái trong REVIEW_LOG.csv: dù khó nhìn, vẫn được thêm nhãn vì còn thấy thân xe và đủ ranh giới.

## 5. Kết luận và giới hạn

Kết quả vòng 1 xấu hơn cold start: AP50 giảm từ 0.771 xuống 0.311 (−0.460), recall ở cả small/medium/large đều giảm mạnh. Vì vậy nên dừng train thêm ngay, kiểm tra lỗi pipeline trước.
Hai ca nên ưu tiên cho vòng sau:
- frame_0099: nhiều xe bị bỏ sót, đặc biệt xe truncated; chi phí rà nhãn cao vì phải thêm/chỉnh nhiều box.  
- frame_0369: phải thêm 19 box, cho thấy model còn yếu rõ rệt; nhưng cần tránh chọn thêm các frame quá gần thời gian để giảm near-duplicate. 
Giới hạn đánh giá: test chỉ có 20 ảnh, xe quá nhỏ bị bỏ qua và ground truth chưa được rà thủ công, nên AP50 có thể chưa đại diện tốt cho hiệu năng thực tế và vẫn có nguy cơ sai do nhãn tham chiếu.
Nếu AP50 giảm, trước khi train tiếp tôi sẽ kiểm tra mapping class, format YOLO, chất lượng label, tập train quá nhỏ/mất cân bằng và cấu hình fine-tune.
