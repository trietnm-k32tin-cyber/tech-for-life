TECH FOR LIFE
Dự án: hệ thống dự đoán và tư vấn nguy cơ nghiện sử dụng điện thoại của học sinh THPT 
CÁC HƯỚNG CẢI TIẾN:
1. tập dữ liệu mẫu:
- sau khi nhận được hướng cải tiến của Ban giám khảo, nhóm thực hiện khảo sát thực tế dựa trên mô hình đã được huấn luyện trước đó. tổng kết được 345 bài khảo sát được thực hiện. Tổng số dữ liệu đang có là 507 mẫu. Mô hình có tính năng đánh giá sau khi có kết quả dự đoán, tức là người dùng cảm thấy dự đoán của mô hình chính xác so với thực tế thì họ sẽ đánh giá 4-5 sao. từ đó nhóm chỉ lấy những mẫu có đánh giá 4-5 sao để huấn luyện mô hình được chính xác hơn.
2. Xem xét chất lượng mô hình FNN so với các mô hình khác khi số lượng đặc trưng còn hạn chế:
- Sau khi có số lượng mẫu là 507, thì nhóm huấn luyện lại mô hình FNN, so sánh sự quan trọng của 7 đặc trưng và thấy rằng, việc lựa chọn ra 7 đặc trung còn nhiều hạn chế và hạn chế lớn nhất là đặc trung tổng. Đặc trưng tổng có độ quan trọng rất cao so với các đặc trưng còn lại, vì vậy nó khiến cho mô hình bị rò rỉ dữ liệu, tức là mô hình sẽ dựa trên tổng để dự đoán mức độ theo ngưỡng cắt cố định mà không xem xét các đặc trưng khác.
<img width="1189" height="690" alt="image" src="https://github.com/user-attachments/assets/8e11b06a-8bd8-476d-bf79-39fd44977cb0" />

- Vì vậy, nhóm tiến hiến lên ý tưởng cải tiến mô hình FNN với kĩ thuật K-fold Ensemble:
  + Kỹ thuật Tạo đặc trưng mới: Tạo thêm các chỉ số thống kê từ 10 câu hỏi như Q_mean (điểm trung bình), Q_std (độ lệch chuẩn giữa các câu), Q_max. Việc này khôi phục thông tin tổng quát mà không vi phạm quy tắc rò rỉ dữ liệu Tong.
  + Tăng lên 3 tầng ẩn.
  + Cải tiến Kiến trúc FNN: Bổ sung BatchNormalization để ổn định luồng dữ liệu, đổi hàm kích hoạt sang mish (hoặc swish) cho độ mượt cao hơn relu, và thêm Dropout(0.15) để chống Overfitting.
  + Bộ điều chỉnh Học phí: Cho phép tăng epochs lên 300 – 400 an toàn; thuật toán sẽ tự động giảm Learning Rate khi gặp vùng tối ưu và dừng lại ở Epoch có kết quả tốt nhất.
  + Tăng từ 5-Fold lên 10-Fold Ensemble: Tăng số lượng mô hình tham gia bình chọn (Ensemble), giúp giảm thiểu tối đa các dự đoán sai lệch cá biệt.
- sau đó, nhóm tiến hành so sánh với các mô hình khác là XGboost, Logistic Repression và SVM. khởi tạo kiến trúc của các mô hình với bộ dữ liệu, chuẩn hóa SMOTE ,IQR giống như huấn luyện mô hình FNN. Chỉ khác là kiến trúc của mỗi mô hình là khác nhau. Tạo 2 kiến trúc mạng bao gồm 7 đặc trưng có tổng và 11 đặc trưng không tổng. So sánh cho thấy:
+ đối với 7 đặc trung có tổng thì mô hình FNN và Xgboost có dộ chính xác ngang nhau và hơn hẳn các mô hình khác.
+ đối với 11 đặc trung cùng 3 đặc trưng thêm thì mô hình FNN hơn hẳn Xgboost và các mô hình còn lại.
<img width="1183" height="583" alt="image" src="https://github.com/user-attachments/assets/5e873645-8cd5-4255-bd32-6f876cc2bc8d" />

3. Việc cải tiến tính năng tư vấn để có thể chứng minh được tính cá nhân hóa và đúng khoa học:
  Giải pháp đưa ra:
+ định hướng RAG bằng SHAP/XAP từ mô hình dự đoán: kết quả phân tích từng đặc trưng truyền cho RAG để phân tích.
+ tìm độ tương đồng giữa các đặc trưng của người dùng và các lời khuyên chuẩn y khoa ( đã được chuyển đổi trong vector DB)  xác định được đoạn văn y khoa liên quan đến đặc trung đó.
+ phối hợp các bước trên: kết quả dự đoán + nguyên nhân từ SHAP + tài liệu y khoa vừa trích xuất thành một promt để gửi cho gemini.
+ gửi promt sang Gemini để đưa ra lời khuyên, tư vấn. 
+ giao diện trực quan cho người dùng: đồ thị thể hiện mức độ nguy cơ từ mô hình, đoạn văn lời khuyên, trích dẫn nguồn (cơ sở khoa học).

4. Bổ sung thêm ghi nhớ theo phiên và đánh giá kết quả:
- giao diện đầu tiên là có thêm phần nhập mã cho từng người dùng (nếu người chưa có mã thì là đăng kí mã, còn đã có thì đăng nhập bằng mã người dùng) với cơ chế sinh mã duy nhất. Sau đó AI sẽ nắm rõ rằng mã đó đã từng thực hiện khảo sát chưa, nếu có rồi thì người dùng sau khi thực hiện khảo sát lần đó thì mô hình sẽ lấy những thông tin trước đó (thông tin về các câu trả lời) và thông tin lần làm khảo sát hiện tại, cộng dồn lại để đưa ra lời tư vấn chuẩn hơn. sử dụng supabase để lưu trữ mã từng người dùng cùng với các đặc trưng câu trả lời. có thêm phần đánh giá ở cuối để có thể cho Gemini API học sau này.
5. link sản phẩm: https://smartphoneapi.vercel.app/
