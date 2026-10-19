# Home Credit Data Warehouse — CO4031

PostgreSQL local, Python ETL, Express JavaScript và React. Không Docker hoặc Power BI. Baseline đã chốt: 5 dimension + 4 fact trong [schema_doc.md](schema_doc.md).

Repository: [BunBoHue1/CO4031-home-credit-dw](https://github.com/BunBoHue1/CO4031-home-credit-dw).

## Bắt đầu

Hiện chỉ có tài liệu và placeholder; chưa có ứng dụng, CLI, DDL hoặc dependency.

1. Clone repository và mở PowerShell tại root theo lệnh bên dưới.
2. Đọc [dataset guide](dataset_guide.md), [schema](schema_doc.md), [TASKS](TASKS.md), [CONTRIBUTING](CONTRIBUTING.md), [AGENTS](AGENTS.md).
3. Nhận task, đọc README/AGENTS phần liên quan, tạo branch và gửi PR.
4. Chuẩn bị dữ liệu theo [data README](data/README.md) và môi trường theo [ETL README](src/etl/README.md).

```powershell
git clone https://github.com/BunBoHue1/CO4031-home-credit-dw.git
cd CO4031-home-credit-dw
```

## Cấu trúc và phân công

| Phần                                                                     | Phụ trách                                                                                            |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| src/db: staging, warehouse, OLAP/reporting; docs/business: nghiệp vụ/KPI | [Phúc (@nhpucpuc)](https://github.com/nhpucpuc)                                                      |
| src/etl: Python pipeline, CLI và kiểm chứng dữ liệu                      | [Phúc (@nhpucpuc)](https://github.com/nhpucpuc)                                                      |
| src/be: Express; src/fe: React, DSS                                      | [Kiên (@BunBoHue1)](https://github.com/BunBoHue1)                                                    |
| tests: SQL/data checks và tích hợp                                       | [Phúc (@nhpucpuc)](https://github.com/nhpucpuc) và [Kiên (@BunBoHue1)](https://github.com/BunBoHue1) |
| docs/report, docs/slides, hợp đồng reporting, review và trình bày        | [Phúc (@nhpucpuc)](https://github.com/nhpucpuc) và [Kiên (@BunBoHue1)](https://github.com/BunBoHue1) |
| Đánh giá, đóng gói, nộp                                                  | [Kiên (@BunBoHue1)](https://github.com/BunBoHue1)                                                    |

## Setup và vận hành

Cài PostgreSQL local, Python, Node.js khi bắt đầu phần liên quan. Phiên bản và dependency khóa khi triển khai; không cài package ở đợt init. Copy .env.example thành .env tại root và điền credentials local. ETL/backend đọc file này; DATA_DIR tương đối root; frontend chỉ nhận địa chỉ API.

Không commit .env, token, node_modules, .venv hoặc raw CSV. Chưa có lệnh build/test. Xem [demo](docs/demo.md), [reporting contract](docs/reporting-contract.md) và TASKS để biết đầu ra và cách verify.

## Bài nộp

Deadline **15/11/2026, 23:59 giờ Việt Nam**, qua LMS. ZIP gồm báo cáo PDF, slide PDF, src/, sample data/; video link trên bìa nếu áp dụng. Mọi thành viên trình bày; video 10–15 phút hoặc trực tiếp 15 phút + 5 phút Q&A. Đối chiếu đề bài PDF và thông báo lớp trước nộp.
