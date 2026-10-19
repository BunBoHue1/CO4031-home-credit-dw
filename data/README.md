# Nguồn và sample

Raw CSV nằm trong data_set/, được ignore. data/ dành cho sample nhỏ nhất quán quan hệ của bài nộp.

## Kaggle

**Chờ Kiên cung cấp command tải đã kiểm chứng.** Chưa có script tải. Chuẩn bị Kaggle CLI, xác thực và chấp nhận điều khoản competition nếu cần. Không commit token/kaggle.json.

Sắp xếp file giải nén từ root:

```text
data_set/
  previous_application.csv
  installments_payments.csv
  bureau.csv
  bureau_balance.csv
  POS_CASH_balance.csv
  credit_card_balance.csv
  HomeCredit_columns_description.csv
  home-credit-default-risk/
    application_train.csv
    application_test.csv
    sample_submission.csv
    HomeCredit_columns_description.csv
```

Kiểm tra bằng Get-ChildItem -Recurse data_set -Filter *.csv. Không giả định cấu trúc giải nén tự đúng. Train và sáu nguồn lịch sử là đầu vào; không nạp test/submission. Ý nghĩa và encoding xem [dataset guide](../dataset_guide.md).

## Sample

Chưa có sample. Phúc tạo danh sách khách hàng train cố định, giữ lịch sử và Bureau balance liên quan. Không lấy số dòng đầu độc lập từng CSV. Ghi lệnh tái tạo, checksum, số dòng, giới hạn và cách chạy. CSV data/ ignore mặc định; chỉ thêm sample có chủ đích sau review.
