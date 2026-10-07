# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Trần Đức Lộc |
| MSSV | 2A202602431 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/ducloc24/K4-L3L4-Track2-Day21-TranDucLoc-2A202602431-CI-CD-for-AI-Systems |
| Ngày nộp | 7/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.874 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

Bộ này có F1 cao nhất là 0.7149 và vượt ngưỡng 0.65. Accuracy cao nhất thuộc lần 1, nhưng F1 cao nhất thuộc lần 3, cho thấy accuracy không phản ánh đầy đủ khả năng nhận diện lớp thu nhập cao. Bộ có learning rate thấp, ít cây và cây nông cho F1 thấp hơn.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Dữ liệu Adult bị mất cân bằng: nhóm thu nhập trên 50K chiếm khoảng 24,8%, còn nhóm thu nhập thấp chiếm khoảng 75,2%. Mô hình luôn đoán “thu nhập thấp” vẫn đạt accuracy khoảng 0.752, nhưng bỏ sót toàn bộ lớp thu nhập cao nên F1 bằng 0.

F1 của lớp dương kết hợp precision và recall, phản ánh việc dự đoán đúng và không bỏ sót quá nhiều trường hợp thu nhập cao. Bài dùng `f1_score(y_eval, preds)` cho `target=1`, không dùng weighted hoặc macro vì lớp đa số có thể kéo điểm lên.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow thiếu `pkg_resources` | MLflow 2.13.0 còn dùng API cũ | Hạ setuptools xuống `80.9.0`. |
| MLflow lỗi với SQLAlchemy | SQLAlchemy mới không còn tương thích với MLflow 2.13.0 | Ghim `SQLAlchemy==2.0.30` trong môi trường và requirements. |
| Chuyển pipeline từ GCP sang AWS | Workflow mẫu dùng GCP | Dùng DVC S3, boto3, AWS credentials và `aws s3 cp`. |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.874 |
| Bước 3 (thêm `train_batch2`) | 0.7014 | 0.874 |

**Nhận xét:** Sau khi thêm 22.361 mẫu, F1 giảm nhẹ còn 0.7014, trong khi accuracy giữ nguyên. Hai batch có phân phối tương tự nên dữ liệu mới không bổ sung nhiều thông tin khác biệt. Commit file DVC đã tự động kích hoạt pipeline từ pull dữ liệu đến triển khai API.
