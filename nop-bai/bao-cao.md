# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Hoàng Văn Tài |
| MSSV | 2A202602400 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/htai2329102003-web/K4-L3-DAY21-HoangVanTai-2A202602400-CI-CD-for-AI-Systems |
| Ngày nộp | 08/10/2026 |

## 1. Bộ Siêu Tham Số Đã Chọn

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 200 | 0.05 | 3 | 0.7014 | 0.8740 |
| 3 | 100 | 0.2 | 5 | 0.7207 | 0.8760 |

Chọn `n_estimators=100`, `learning_rate=0.2`, `max_depth=5` vì lần chạy 3 có F1 cao nhất. Dù lần chạy 1 có Accuracy nhỉnh hơn, F1 thấp hơn nên khả năng nhận diện lớp thu nhập cao chưa tốt bằng.

## 2. Vì Sao Dùng F1 Thay Vì Accuracy

Lớp thu nhập cao chỉ chiếm khoảng 24,8%. Một mô hình luôn dự đoán “thu nhập thấp” vẫn có thể đạt Accuracy khoảng 75,2% nhưng không nhận diện được lớp dương. Vì vậy, lab dùng F1 của lớp dương — trung bình điều hòa của Precision và Recall — để đánh giá trực tiếp khả năng phát hiện nhóm thu nhập cao. `Weighted F1` có thể bị chi phối bởi lớp đa số; `Macro F1` cho hai lớp trọng số ngang nhau nhưng không tập trung riêng vào lớp dương như mục tiêu của bài.

## 3. Khó Khăn và Cách Giải Quyết

| Khó khăn | Cách giải quyết |
|---|---|
| API trên EC2 không tải được model từ S3 | Cấu hình AWS credentials/region cho user chạy systemd và truyền đúng `ARTIFACT_BUCKET`. |
| Release restart service nhưng health check thất bại | Dùng virtual environment cố định phiên bản thư viện, chờ API theo vòng retry và in journal khi lỗi. |
| `script_stop` làm hỏng heredoc của AWS config và systemd unit | Bỏ `script_stop`, dùng `set -eu`; pipeline sau đó hoàn thành cả bốn job. |
| Cập nhật dữ liệu Bước 3 bằng DVC | Nối `train_batch2` vào `train_batch1`, chạy `dvc add`, `dvc push` và chỉ commit file con trỏ `.dvc`. |

## 4. So Sánh Bước 2 và Bước 3

| Giai đoạn | Dữ liệu huấn luyện | f1_score | accuracy |
|---|---:|---:|---:|
| Bước 2 | 22.361 mẫu | 0.7207 | 0.8760 |
| Bước 3 | 44.722 mẫu | 0.7297 | 0.8800 |

Sau khi bổ sung `train_batch2`, F1 tăng 0.0090 và Accuracy tăng 0.0040. Mức tăng nhỏ vì hai batch được lấy ngẫu nhiên từ cùng một phân phối, nhưng mô hình vẫn vượt quality gate F1 ≥ 0.65 và được triển khai thành công.
