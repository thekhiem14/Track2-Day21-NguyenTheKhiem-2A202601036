# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Thế Khiê, |
| MSSV | 2A202601036 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/thekhiem14/Track2-Day21-NguyenTheKhiem-2A202601036 |
| Ngày nộp | 21/8/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 2 | 150 | 0.2 | 4 | 0.6912 | 0.866 |
| 3 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 4 | 200 | 0.1 | 5 | 0.7149 | 0.874 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ này đạt f1_score cao nhất (0,7149), dù accuracy (0,874) không phải cao nhất — lần chạy
1 có accuracy cao nhất (0,878) nhưng f1 thấp hơn (0,7109), chứng minh accuracy không đáng tin cậy để
chọn mô hình trên dữ liệu mất cân bằng. Về đánh đổi tham số: learning_rate thấp (0,05) kết hợp
n_estimators nhỏ (50) cho f1 thấp nhất (0,6051, dưới ngưỡng), do mô hình chưa học đủ; tăng
n_estimators lên 200 bù cho learning_rate vừa phải giúp f1 cải thiện rõ rệt.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult mất cân bằng lớp: chỉ 24,8% mẫu thuộc lớp thu nhập cao. Một mô hình vô dụng luôn
trả lời "thu nhập thấp" vẫn đạt accuracy 0,752 — cao nhưng gây hiểu nhầm vì mô hình không nhận diện
đúng bất kỳ ai thu nhập cao. F1-score của lớp dương đo cả precision và recall trên đúng lớp thiểu số
quan trọng, phản ánh đúng năng lực mô hình mà accuracy không đo được. Vì vậy lab đặt ngưỡng trên
`f1_score >= 0.65`. Không dùng `average="weighted"`/`"macro"` khi gọi `f1_score()` vì hai cách này bị
lớp đa số kéo giá trị lên cao, làm mất ý nghĩa cảnh báo của ngưỡng.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| `$env:VAR = "..."` báo lỗi cú pháp | Gõ lệnh PowerShell trong cmd.exe | Chuyển sang PowerShell |
| `train.py` lỗi `Invalid parameter name: 'ï»¿n_estimators'` | `Set-Content -Encoding utf8` trên PowerShell 5.1 thêm BOM vào `params.yaml` | Đổi sang `-Encoding ascii` |
| Không tạo được `sa-key.json` như README Bước 2 yêu cầu | Org Policy của VinUni khóa `disableServiceAccountKeyCreation` cấp tổ chức | Xác thực không cần key: ADC cho máy cá nhân, Workload Identity Federation cho GitHub Actions, service account gắn thẳng VM |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`, 22.361 mẫu) | 0,7149 | 0,874 |
| Bước 3 (thêm `train_batch2`, 44.722 mẫu) | 0,7354 | 0,882 |

**Nhận xét:** f1_score tăng nhẹ (+0,0205) khi gấp đôi dữ liệu, dù `train_batch2` lấy mẫu ngẫu nhiên
từ cùng nguồn nên lý thuyết không mang thêm thông tin phân phối mới — với bộ tham số khá phức tạp
(`n_estimators=200, max_depth=5`), thêm dữ liệu vẫn giúp các nhánh cây ước lượng ổn định hơn thay vì
đã bão hòa hoàn toàn. Quan trọng hơn con số: **quy trình tự động chạy đúng** — một commit dữ liệu đã
kích hoạt toàn bộ pipeline từ huấn luyện đến triển khai lại trên VM, không cần can thiệp thủ công.
