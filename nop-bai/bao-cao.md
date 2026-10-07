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

Lý do: Cấu hình này đạt f1_score (0.7222) và accuracy (0.8800) cao nhất trên tập holdout, vượt ngưỡng f1 >= 0.65. So với lần 2, số cây và độ sâu đủ lớn để nhận diện tốt lớp thiểu số. So với lần 3, độ sâu 3 tránh overfitting khi độ sâu 5 làm giảm accuracy từ 0.8800 xuống 0.8740. Tỷ lệ học 0.1 kết hợp 150 cây giúp mô hình hội tụ cân bằng và ổn định.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult mất cân bằng khi lớp thu nhập cao chỉ chiếm khoảng 24.8%. Nếu mô hình dự đoán toàn bộ là thu nhập thấp, accuracy vẫn đạt 75.2% dù mô hình hoàn toàn vô dụng với lớp mục tiêu.

F1 lớp dương là trung bình điều hòa giữa Precision và Recall, phản ánh chính xác khả năng phát hiện đúng người thu nhập cao mà không đoán bừa. Bắt buộc dùng f1_score lớp dương, không dùng trung bình weighted hay macro vì sẽ bị lớp đa số kéo điểm lên cao giả tạo, làm mất ý nghĩa kiểm soát của quality gate.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi thiếu pkg_resources | Setuptools mới bỏ pkg_resources | Cố định setuptools<70 trong requirements.txt |
| Xung đột thư viện SQLAlchemy | SQLAlchemy 2.1 không tương thích MLflow 2.13 | Giới hạn sqlalchemy<2.1.0 trong requirements.txt |
| Lỗi unpickle model trên VM | Khác biệt phiên bản scikit-learn (1.7 vs 1.4.2) | Cài đúng scikit-learn==1.4.2 trên máy ảo |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7222 | 0.8800 |
| Bước 3 (thêm `train_batch2`) | 0.7306 | 0.8820 |

Nhận xét: Thêm 22.361 mẫu từ batch 2 giúp f1_score tăng từ 0.7222 lên 0.7306 và accuracy tăng từ 0.8800 lên 0.8820 nhờ đường biên phân lớp chuẩn xác hơn. Hệ thống CI/CD đã tự động kích hoạt continuous training, kiểm tra chất lượng và triển khai lại thành công lên máy ảo mà không cần thao tác thủ công.
