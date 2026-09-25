# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, nếu chỉ có ngân sách rà 5 ảnh, tôi ưu tiên:

1. `frame_0182.jpg` — score **0.9591**, thời điểm **72.8 s**, rank **1**. Đây là frame có điểm cao nhất, đồng thời có **18 box ambiguous**, nên có giá trị cao để kiểm tra các trường hợp model chưa chắc chắn.
2. `frame_0369.jpg` — score **0.9324**, thời điểm **147.6 s**, rank **2**. Frame có độ uncertainty cao (`U = 0.9315`) và **16 box ambiguous**, phù hợp để tìm lỗi dự đoán.
3. `frame_0380.jpg` — score **0.9170**, thời điểm **152.0 s**, rank **3**. Frame nằm ở vùng thời gian khác và có **15 box ambiguous**, giúp tăng độ đa dạng của tập được rà.
4. `frame_0326.jpg` — score **0.9155**, thời điểm **130.4 s**, rank **4**. Frame có uncertainty cao (`U = 0.9310`) và 15 trường hợp ambiguous.
5. `frame_0099.jpg` — score **0.9063**, thời điểm **39.6 s**, rank **8**. Dù điểm thấp hơn một số frame rank 5–7, frame này được chọn để tránh tập trung quá nhiều ảnh ở các thời điểm gần nhau và tăng độ đa dạng theo thời gian.

Một quyết định có xét ảnh gần trùng là **không ưu tiên `frame_0372.jpg` (148.8 s, rank 6, score 0.9101)** khi đã chọn `frame_0369.jpg` ở 147.6 s. Hai frame chỉ cách nhau **1.2 giây**, nên có khả năng chứa nội dung rất giống nhau. Với ngân sách chỉ 5 ảnh, ưu tiên một frame ở thời điểm khác sẽ cung cấp thêm thông tin thay vì rà hai ảnh gần trùng.

Ba frame thuộc lô 12 ảnh model chọn có bằng chứng trực tiếp trong CSV là: `frame_0182.jpg` (rank 1, score 0.9591, `selected=True`), `frame_0369.jpg` (rank 2, score 0.9324, `selected=True`) và `frame_0380.jpg` (rank 3, score 0.9170, `selected=True`). Các frame này cũng xuất hiện trong lô 12 ảnh được chọn để đưa vào contact sheet.

Một frame có điểm cao nhưng không chọn là `frame_0372.jpg`, score **0.9101**, rank **6**. Lý do không ưu tiên là frame này ở **148.8 s**, rất gần `frame_0369.jpg` ở **147.6 s**; do đó có nguy cơ gần trùng về nội dung. Việc bỏ qua nó trong ngân sách 5 ảnh giúp dành chỗ cho một frame ở đoạn video khác.

Phép chọn này **chưa chứng minh chất lượng tổng thể của mô hình**. Điểm selection chỉ giúp xác định những frame đáng được con người kiểm tra, dựa trên uncertainty, ambiguity và diversity; nó không trực tiếp cho biết model dự đoán đúng hay sai trên toàn bộ dữ liệu. Muốn kết luận chất lượng mô hình vẫn cần so sánh prediction với ground truth trên một tập đánh giá độc lập và sử dụng các metric như Precision, Recall, mAP hoặc IoU.