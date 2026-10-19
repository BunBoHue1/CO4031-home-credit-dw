# ETL — Phúc

Placeholder Python pipeline và CLI, chưa có code/dependency. Đọc TASKS, schema, data README và docs/demo.md.

## Setup local

Cài Git, Python, PostgreSQL local; không Docker. Phúc chốt phiên bản đã kiểm chứng trong ENV-01. Từ root:

```powershell
python --version
psql --version
python -m venv .venv
.\.venv\Scripts\Activate.ps1
Copy-Item .env.example .env
```

Nếu activate bị chặn, dùng trực tiếp .venv/Scripts/python.exe, không cần đổi policy toàn máy. Nếu psql không trong PATH, dùng binary PostgreSQL hoặc pgAdmin.

Dùng admin local tạo role LOGIN home_credit_app với password riêng và database home_credit_dw do role đó sở hữu. Điền .env; không chạy pipeline bằng superuser. Kiểm tra:

```powershell
psql -h localhost -p 5432 -U home_credit_app -d home_credit_dw -c "SELECT current_database(), current_user;"
```

Password nhập khi được hỏi, không đưa vào command. ETL/backend phải đọc .env root tường minh; DATA_DIR luôn tính từ root.

Chưa có lệnh cài dependency. ENV-01 bổ sung manifest khóa phiên bản, lệnh cài trong .venv và kiểm tra kết nối Python. Dự kiến psycopg/COPY, đọc chunk và SQL transformations; không giữ toàn CSV lớn trong RAM.

## Bàn giao

Setup → schema → nguồn → manifest → batch → verify → publish. Verify cutoff, installment partial payments/conflicts, UNION ALL, unique/FK/reconciliation, idempotency, lỗi/retry và reset. Lệnh python -m etl trong demo chưa chạy được.
