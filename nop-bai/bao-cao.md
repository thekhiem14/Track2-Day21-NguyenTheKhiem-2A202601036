# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

<!--
HƯỚNG DẪN - đọc rồi XÓA TOÀN BỘ các khối chú thích này sau khi điền xong:

  - Giới hạn: KHÔNG QUÁ 1 TRANG A4, tương đương khoảng 450 - 550 từ nội dung.
  - Chỉ điền vào các chỗ ___ và các ô trong bảng. Không thêm mục mới.
  - Viết bằng câu hoàn chỉnh, không gạch đầu dòng cụt lủn.
  - Kiểm tra độ dài sau khi đã xóa hết chú thích:
        wc -w nop-bai/bao-cao.md
    và xem trước bản in bằng cách mở file trên GitHub rồi Ctrl+P / Cmd+P.
-->

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

**Lý do:** Bộ này đạt f1_score cao nhất (0,7149) trong bốn lần chạy, dù accuracy (0,874) không phải
cao nhất. Lần chạy có accuracy cao nhất (100/0.1/3, accuracy=0,878) chỉ đạt f1_score=0,7109, thấp
hơn bộ được chọn — chứng minh accuracy không phải chỉ số đáng tin cậy để so sánh mô hình trên dữ
liệu mất cân bằng, vì nó có thể tăng nhờ đoán đúng lớp đa số trong khi bỏ sót lớp thiểu số quan
trọng hơn. Về đánh đổi giữa hai tham số: bộ learning_rate thấp (0,05) kết hợp n_estimators nhỏ (50)
cho f1 thấp nhất (0,6051, dưới ngưỡng 0,65) vì mô hình chưa học đủ; tăng n_estimators lên 200 để bù
cho learning_rate vừa phải (0,1) giúp f1 cải thiện rõ rệt, đúng với quan hệ đánh đổi được mô tả
trong tasks/buoc-1.md.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult có phân bố lớp mất cân bằng: chỉ 24,8% số mẫu thuộc lớp thu nhập cao (>50K). Một
mô hình vô dụng, luôn trả lời "thu nhập thấp" cho mọi mẫu, vẫn đạt accuracy 0,752 — con số nghe có
vẻ cao nhưng gây hiểu nhầm nghiêm trọng, vì mô hình đó không học được gì và không bao giờ nhận diện
đúng một người thu nhập cao nào. F1-score của lớp dương đo đồng thời precision và recall trên đúng
lớp thiểu số quan trọng (thu nhập cao), nên phản ánh đúng khả năng thực sự của mô hình — điều mà
accuracy hoàn toàn không đo được trên dữ liệu lệch lớp. Vì vậy lab đặt ngưỡng chất lượng trên
`f1_score >= 0.65` thay vì accuracy. Khi gọi `f1_score()`, không dùng `average="weighted"` hay
`average="macro"`, vì hai cách tính này bị lớp đa số kéo giá trị lên cao, làm mất hoàn toàn ý nghĩa
cảnh báo của ngưỡng chất lượng.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Gõ `$env:VAR = "..."` bị báo lỗi cú pháp | Đang gõ lệnh PowerShell trong cửa sổ Command Prompt (cmd.exe), hai shell không tương thích cú pháp biến môi trường | Chuyển hẳn sang PowerShell, kích hoạt lại venv bằng `Activate.ps1` |
| `train.py` báo lỗi `Invalid parameter name: 'ï»¿n_estimators'` | `Set-Content -Encoding utf8` trên Windows PowerShell 5.1 tự thêm BOM vào đầu `params.yaml`, làm hỏng tên tham số đầu tiên khi PyYAML đọc file | Đổi sang `-Encoding ascii` khi ghi `params.yaml` bằng PowerShell |
| Không tạo được `sa-key.json` cho service account như README yêu cầu ở Bước 2 | GCP Organization của VinUni khóa `constraints/iam.disableServiceAccountKeyCreation` ở cấp tổ chức, không có quyền project-owner nào gỡ được | Chuyển sang xác thực không cần key: Application Default Credentials cho máy cá nhân, Workload Identity Federation cho GitHub Actions, service account gắn thẳng vào VM lúc tạo |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

<!-- Lấy số liệu từ bảng ở mục 3.6 của tasks/buoc-3.md. Điền sau khi hoàn thành Bước 2 và Bước 3. -->

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | ___ | ___ |
| Bước 3 (thêm `train_batch2`) | ___ | ___ |

**Nhận xét:** ___

<!--
Một câu trả lời trung thực kiểu "f1 giảm 0,01 vì dữ liệu mới cùng phân phối, không mang
thêm thông tin mới" được đánh giá cao hơn kết luận sai rằng thêm dữ liệu luôn tốt hơn.
-->

---

## 5. Phần Bonus Đã Thực Hiện (nếu có)

<!-- Xóa cả mục 5 nếu không làm bonus. Mỗi bonus tối đa 1 dòng. -->

- [ ] Bonus 1 - Tracking MLflow từ xa với DagsHub: ___
- [ ] Bonus 2 - Điều chỉnh ngưỡng quyết định: ___
- [ ] Bonus 3 - Báo cáo precision / recall tự động: ___
- [ ] Bonus 4 - Hoàn trả về phiên bản trước: ___
- [ ] Bonus 5 - Cảnh báo lệch lạc dữ liệu: ___
