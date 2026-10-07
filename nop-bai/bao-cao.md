# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Quang Hữu |
| MSSV | 2A202602756 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Anreak/K4-L3-DAY21-NguyenQuangHuu-2A202602756-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |
| 4 | 150 | 0.1 | 3 | 0.7222 | 0.8800 |

Bộ siêu tham số đã chọn: n_estimators=150, learning_rate=0.1, max_depth=3.

Lý do: Cấu hình này mang lại f1_score cao nhất (0.7222) và accuracy cao nhất (0.8800) trên tập holdout, vượt xa quality gate f1 >= 0.65. So với lần 2 (n_estimators=50, max_depth=2, F1=0.6051), mô hình có đủ số cây để học tốt lớp thiểu số. So với lần 3 (max_depth=5, 200 cây), việc tăng độ sâu lên 5 khiến mô hình chớm overfitting làm accuracy giảm từ 0.8800 xuống 0.8740. Ta nhận thấy sự đánh đổi: learning_rate vừa phải (0.1) kết hợp 150 cây ở độ sâu 3 giúp gradient boosting hội tụ ổn định nhất.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult có phân bố lớp mất cân bằng nặng khi chỉ 24.8% mẫu thuộc lớp thu nhập cao (target = 1) và 75.2% thuộc lớp thu nhập thấp (target = 0). Nếu dùng một mô hình vô dụng luôn gán nhãn thu nhập thấp cho mọi trường hợp, accuracy vẫn đạt 0.752 (75.2%), tạo ảo tưởng về độ chính xác nhưng hoàn toàn thất bại trong việc nhận diện người thu nhập cao (F1 lớp dương bằng 0). 

F1 của lớp dương là trung bình điều hòa giữa Precision và Recall, phản ánh thực chất cả hai yêu cầu: không bỏ sót người thu nhập cao và không đoán bừa người thu nhập thấp thành cao. Khi tính toán bắt buộc dùng f1_score mặc định cho lớp dương, tuyệt đối không dùng average="weighted" hay average="macro" vì các tùy chọn này sẽ gộp lớp đa số vào làm chỉ số bị kéo lên cao giả tạo, vô hiệu hóa ngưỡng an toàn 0.65.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi thiếu pkg_resources khi chạy pytest và MLflow | Setuptools bản mới đã loại bỏ thư viện pkg_resources | Cố định phiên bản setuptools<70 trong requirements.txt |
| Xung đột thư viện SQLAlchemy với MLflow 2.13.0 | SQLAlchemy 2.1 loại bỏ FallbackAsyncAdaptedQueuePool | Giới hạn sqlalchemy<2.1.0 trong requirements.txt |
| Không tương thích phiên bản scikit-learn khi unpickle model trên VM | Máy ảo cài bản scikit-learn 1.7 mới nhất trong khi model huấn luyện với bản 1.4.2 | Cài đặt đúng scikit-learn==1.4.2 trên máy ảo |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7222 | 0.8800 |
| Bước 3 (thêm `train_batch2`) | 0.7306 | 0.8820 |

Nhận xét: Khi bổ sung thêm 22.361 mẫu từ batch 2, f1_score tăng từ 0.7222 lên 0.7306 và accuracy tăng từ 0.8800 lên 0.8820. Việc có thêm mẫu giúp mô hình phân định biên giới lớp chính xác hơn một phần nhỏ. Quan trọng nhất là toàn bộ quy trình continuous training đã tự động kích hoạt thành công từ commit dữ liệu đến huấn luyện và triển khai lại trên máy ảo mà không cần bất kỳ can thiệp thủ công nào.
