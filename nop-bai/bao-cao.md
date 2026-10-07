# Báo Cáo Lab Day 21 - CI/CD cho AI Systems



| | |
|---|---|
| Họ và tên | Hoàng Văn Tài |
| MSSV | 2A202602400 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/htai2329102003-web/K4-L3-DAY21-HoangVanTai-2A202602400-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 200 | 0.05 | 3 | 0.7014 | 0.8740 |
| 3 | 100 | 0.2 | 5 | 0.7207 | 0.8760 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.2`, `max_depth=5`.

**Lý do:** Trong 3 lần chạy nghiệm, lần 3 cho điểm F1 cao nhất là 0.7207 so với hai bộ còn lại, đáp ứng tốt nhất mục tiêu dự đoán lớp mất cân bằng (thu nhập > 50K). Lần 1 dù có Accuracy cao nhất (0.8780) nhưng F1 lại thấp hơn (0.7109), điều này cho thấy Accuracy không phản ánh chính xác khả năng mô hình nắm bắt lớp thiểu số. Việc tăng learning_rate lên 0.2 kết hợp tăng max_depth=5 đã giúp mô hình học các tương tác phức tạp tốt hơn.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Trong tập dữ liệu Adult, lớp thu nhập cao (>50K) chỉ chiếm 24,8% (phân bố lớp mất cân bằng). Hệ quả là một mô hình vô dụng đoán bừa tất cả là "thu nhập thấp" vẫn sẽ đạt độ chính xác (Accuracy) lên tới 75,2%. Do vậy, Accuracy trong trường hợp này gây ảo giác về hiệu suất. F1 score của lớp dương (lớp thu nhập cao) được dùng thay thế vì nó là trung bình điều hòa của Precision và Recall, giúp đo lường chính xác khả năng nhận diện lớp thiểu số mà không bị lớp đa số làm nhiễu. Ta cũng không dùng average="weighted" hay "macro" vì chúng sẽ bị điểm số cao của lớp đa số kéo lên, làm mất đi ý nghĩa của việc đánh giá khả năng mô hình.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Không tải được model từ S3 trên VM | Thiếu cấu hình credentials của AWS trên EC2 | Đã dùng thư viện `boto3` kết hợp nạp credentials vào EC2 hoặc GitHub Secrets |
| Lỗi lệnh bash trên máy ảo EC2 | Do file script `.sh` tạo trên Windows mang ký tự xuống dòng `\r\n` (CRLF) nên hệ điều hành Ubuntu không hiểu và báo lỗi `command not found`. | Dùng PowerShell hoặc `sed` loại bỏ ký tự `\r` trước khi chạy lệnh SSH trên máy ảo EC2. |
| Pipeline Bước 3 không tự kích hoạt | Commit nhầm file `.csv` thay vì file `.dvc` | Bổ sung `.csv` vào `.gitignore` và chỉ commit `train_batch1.csv.dvc` |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7207 | 0.8760 |
| Bước 3 (thêm `train_batch2`) | 0.7297 | 0.8800 |

**Nhận xét:** F1 score chỉ tăng nhẹ 0.009 (từ 0.7207 lên 0.7297) dù lượng dữ liệu được tăng gấp đôi. Điều này là do dữ liệu mới lấy ngẫu nhiên từ cùng một phân phối gốc, không mang theo nhiều patterns thông tin mới nên mô hình đã gần bão hòa khả năng học.

---

