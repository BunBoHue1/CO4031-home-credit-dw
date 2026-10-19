# TASKS — theo dõi và bàn giao

Nguồn tiến độ chính. Trạng thái: TODO, IN_PROGRESS, BLOCKED, DONE. BLOCKED ghi nguyên nhân. DONE phải có bằng chứng; không tự đánh dấu tính năng chưa kiểm chứng. Owner có thể giao agent thực hiện nhưng vẫn chịu trách nhiệm review.

Đọc README → dataset_guide → schema_doc → CONTRIBUTING/AGENTS và hướng dẫn phần liên quan. Dưới đây là owner, phụ thuộc, đầu ra và tiêu chí ban đầu; điền lệnh thực tế và bằng chứng trong task khi triển khai.

## Roadmap

Tài khoản thành viên: [Kiên (@BunBoHue1)](https://github.com/BunBoHue1), [Phúc (@nhpucpuc)](https://github.com/nhpucpuc). Tên owner trong các task bên dưới tương ứng các tài khoản này.

| Mốc nội bộ | Kết quả |
|---|---|
| 19–21/10/2026 | Repo, onboarding, nghiệp vụ/KPI, phân công |
| 22–28/10 | DDL, ETL, CLI, kiểm chứng |
| 29/10–04/11 | Reporting, API, dashboard, chốt DSS |
| 05–10/11 | DSS, tích hợp, đánh giá |
| 11–14/11 | Báo cáo, slide, demo/video, đóng gói |
| 15/11 | Kiểm tra/nộp trước 23:59 giờ Việt Nam |

## Khởi tạo và môi trường

### INIT-01 — Tài liệu và placeholder

- Owner: Kiên. Trạng thái: DONE. Phụ thuộc: không.
- Đọc: kế hoạch, đề bài, schema, AGENTS.
- Đầu ra: README, CONTRIBUTING, TASKS, env example, folder README, AGENTS từng phần, PR template, ignore.
- Verify: UTF-8, link nội bộ, đủ thư mục, không secret/raw CSV; không tạo dependency hoặc code ứng dụng.
- Bằng chứng: kiểm tra UTF-8/link trên 22 Markdown file, README của 9 placeholder và git check-ignore đều PASS. Chưa có ứng dụng để chạy test. Xuất bản GitHub theo GIT-01.

### GIT-01 — Xuất bản GitHub

- Owner: Kiên. Trạng thái: IN_PROGRESS. Phụ thuộc: INIT-01.
- Đọc: CONTRIBUTING, README.
- Repository: [BunBoHue1/CO4031-home-credit-dw](https://github.com/BunBoHue1/CO4031-home-credit-dw); owner: [Kiên (@BunBoHue1)](https://github.com/BunBoHue1).
- Đã có: remote origin đúng URL, main theo dõi origin/main; trạng thái local ghi nhận hai nhánh cùng commit 69c53b6 (Update account name).
- Visibility: public, theo xác nhận của Kiên.
- Collaborator: [Phúc (@nhpucpuc)](https://github.com/nhpucpuc) đã nhận được lời mời, theo xác nhận của Kiên; chưa xác nhận đã chấp nhận lời mời.
- Còn lại: Phúc chấp nhận lời mời và kiểm tra quyền push branch/tạo PR; kiểm tra clone sạch, cấu hình review PR/main khi tài khoản hỗ trợ. Repo public cho phép clone nhưng không tự cấp quyền push.
- Đầu ra: repository GitHub, origin/main upstream, Phúc có quyền đóng góp và quy trình PR hoạt động.
- Verify: remote đúng, không secrets/dataset; clone sạch đọc đủ tài liệu, Phúc có quyền contribute.
- Bằng chứng: Kiên cung cấp link repo, xác nhận repo public và Phúc đã nhận được lời mời; git remote -v và git branch -vv xác nhận cấu hình remote/upstream local trong lần kiểm tra trước. Chưa kiểm tra quyền collaborator qua mạng; chỉ DONE sau khi xác nhận các mục còn lại.

### ENV-01 — Python ETL và PostgreSQL local

- Owner: Phúc. Trạng thái: TODO. Phụ thuộc: INIT-01.
- Đọc: src/etl/README, .env.example, data/README.
- Đầu ra: phiên bản Python/PostgreSQL, .venv, dependency khóa phiên bản, root .env loader, kết nối Python bằng role project.
- Verify: clone sạch setup theo README; SELECT current_database/current_user qua Python, DATA_DIR đúng từ root và src/etl; log không password.
- Bằng chứng/PR: chưa có; điền lệnh cài/chạy và kết quả khi bàn giao.

### DATA-01 — Nguồn Kaggle và sample

- Owner: Phúc; Kiên cung cấp command Kaggle. Trạng thái: TODO. Phụ thuộc: INIT-01.
- Đọc: data/README, dataset_guide, schema.
- Đầu ra: command tải kiểm chứng, bố trí nguồn, checksum/số dòng, sample nhất quán quan hệ và cách tái tạo.
- Verify: đủ file/đúng đường dẫn; sample ID/lịch sử liên kết đúng, raw CSV/token không staged; ghi giới hạn sample.
- Bằng chứng/PR: Kiên đã cung cấp kagglehub.dataset_download("megancrenshaw/home-credit-default-risk"); data/README có cài đặt, tải cache, copy đúng layout và kiểm tra. Chưa tải mạng/đối soát phiên bản nguồn; sample, checksum và kiểm chứng DATA-01 còn TODO.

## Nghiệp vụ đến OLAP — Phúc

### BIZ-01 — Câu hỏi và KPI

- Owner: Phúc; Kiên review. Trạng thái: TODO. Phụ thuộc: INIT-01.
- Đọc: đề bài, docs/business/README, schema, dataset_guide.
- Đầu ra: người dùng, vấn đề quyết định, câu hỏi, KPI (tử/mẫu, grain, population, cutoff, unit, NULL), filter/drill-down.
- Verify: mỗi KPI ánh xạ schema, ví dụ SQL hoặc tính tay có mẫu số đúng; hai người thống nhất.
- Bằng chứng/PR: chưa có.

### DB-01 — Schema vật lý

- Owner: Phúc. Trạng thái: TODO. Phụ thuộc: BIZ-01, ENV-01.
- Đọc: schema_doc, src/db/AGENTS.
- Đầu ra: DDL staging/warehouse/reporting, ERD, PK/FK/unique/check/index, member mặc định; bảng vận hành tách khỏi mô hình phân tích.
- Verify: database rỗng tái tạo được 5 dim/4 fact đúng mapping; constraint chặn key sai/trùng, NULL đúng nghĩa; ghi lệnh SQL.
- Bằng chứng/PR: chưa có.

### ETL-01 — Staging và profiling

- Owner: Phúc. Trạng thái: TODO. Phụ thuộc: DB-01, DATA-01.
- Đọc: schema, dataset guide, src/etl/AGENTS.
- Đầu ra: chunk/bulk load, schema nguồn, profiling, log và cách ly bản ghi lỗi; thống kê source/accepted/rejected.
- Verify: đếm/đối soát nguồn-staging, encoding và NULL, nguồn không bị sửa, không đọc toàn bộ CSV lớn vào RAM.
- Bằng chứng/PR: chưa có.

### ETL-02 — Manifest và warehouse

- Owner: Phúc. Trạng thái: TODO. Phụ thuộc: ETL-01.
- Đọc: schema, docs/demo, AGENTS.
- Đầu ra: manifest hai batch cố định, dimension lookup/upsert, fact loads, cutoff và transaction/publish strategy ghi rõ trước code.
- Verify: manifest disjoint/union train; lịch sử liên quan đủ, key ổn định; installment partial payments/completion/conflicts; monthly UNION ALL; không lộ batch dở dang.
- Bằng chứng/PR: chưa có.

### CLI-01 — Điều khiển demo

- Owner: Phúc. Trạng thái: TODO. Phụ thuộc: ETL-02, OLAP-01.
- Đọc: docs/demo, ETL README.
- Đầu ra: clear-data, load-data-1, load-data-2, status qua python -m etl; cập nhật hướng dẫn lệnh thật.
- Verify: batch 2 trước batch 1 bị từ chối; reset giới hạn project/giữ schema và raw CSV; member mặc định được tạo lại; lỗi exit khác 0/log; nạp lại không trùng; retry sau lỗi an toàn.
- Bằng chứng/PR: chưa có.

### QA-01 — Kiểm chứng warehouse

- Owner: Phúc. Trạng thái: TODO. Phụ thuộc: ETL-02.
- Đọc: schema, tests/README, AGENTS invariants.
- Đầu ra: checks tái lập cho grain/unique/FK/reconciliation/idempotency/cutoff và báo cáo số dòng cách ly.
- Verify: có trường hợp partial payment, future payment, xung đột kỳ, Unknown/Not applicable/NULL, orphan, account trùng ID khác source; chạy lại cùng nguồn không đổi dữ liệu.
- Bằng chứng/PR: chưa có; ghi lệnh và kết quả, không chỉ checklist tự tick.

### CONTRACT-01 — Reporting → backend

- Owner: Phúc và Kiên. Trạng thái: TODO. Phụ thuộc: BIZ-01, DB-01.
- Đọc: docs/reporting-contract.
- Đầu ra: tên view/query, grain, cột/kiểu, KPI/filter/NULL, aggregation, batch publication; ví dụ output; cách serialize decimal/bigint.
- Verify: hai người review, ví dụ có expected output; endpoint contract do Kiên bổ sung trước FE.
- Bằng chứng/PR: chưa có.

### OLAP-01 — Reporting và drill-down

- Owner: Phúc. Trạng thái: TODO. Phụ thuộc: CONTRACT-01, ETL-02.
- Đọc: KPI, schema, reporting-contract.
- Đầu ra: query/view, refresh reporting sau batch, filter/aggregation/drill-down và SQL đối soát.
- Verify: không fact multiplication, không sum snapshot qua tháng; mẫu số đúng, chỉ batch published; kết quả khớp QA-01 trước bàn giao.
- Bằng chứng/PR: chưa có. Bàn giao cuối cần QA-01 pass.

## Backend đến bàn giao — Kiên

### BE-01 — Express API

- Owner: Kiên. Trạng thái: TODO. Phụ thuộc: CONTRACT-01; tích hợp thật cần OLAP-01.
- Đọc: src/be/AGENTS, reporting-contract.
- Đầu ra: Node version, manifest/lockfile, env loader, API contract, filter/drill-down, validation/error handling; chỉ đọc reporting.
- Verify: SQL/API khớp batch 1/2, tham số sai, empty data, NULL/decimal/bigint, credentials không lộ; README có lệnh cài/chạy/verify.
- Bằng chứng/PR: chưa có; nếu dùng mock phải ghi rõ chưa tích hợp thật.

### FE-01 — Dashboard React

- Owner: Kiên. Trạng thái: TODO. Phụ thuộc: BE-01.
- Đọc: src/fe/AGENTS, KPI/API contract.
- Đầu ra: Node version, manifest/lockfile, biểu đồ/bảng/filter/drill-down, loading/empty/error, refresh.
- Verify: số liệu khớp API/SQL, filter/drill-down có kết quả đúng; reset không giữ cache cũ; không credentials DB trong client.
- Bằng chứng/PR: chưa có.

### DSS-01 — Chốt bài toán hỗ trợ quyết định

- Owner: Kiên; Phúc review. Trạng thái: TODO. Phụ thuộc: BIZ-01, CONTRACT-01.
- Đọc: đề bài, KPI, schema.
- Đầu ra: mục tiêu quyết định, input warehouse, logic, output giải thích, baseline và cách đánh giá. Simulator chỉ là ứng viên.
- Verify: làm được với schema; TARGET chỉ đánh giá, không input screening/model; ghi cutoff và hạn chế.
- Bằng chứng/PR: chưa có.

### DSS-02 — Hiện thực DSS

- Owner: Kiên. Trạng thái: TODO. Phụ thuộc: DSS-01, BE-01, FE-01, OLAP-01.
- Đọc: DSS spec và API contract.
- Đầu ra: backend/frontend DSS dùng warehouse, giải thích kết quả và so sánh baseline.
- Verify: kịch bản thường/biên/input sai/thiếu dữ liệu; tái lập output, không leakage; có kết quả đánh giá và hạn chế.
- Bằng chứng/PR: chưa có.

### INT-01 — Tích hợp, demo và đánh giá

- Owner: Kiên; Phúc hỗ trợ dữ liệu. Trạng thái: TODO. Phụ thuộc: QA-01, CLI-01, FE-01, DSS-02.
- Đọc: tests/README, docs/demo, hợp đồng.
- Đầu ra: kịch bản end-to-end, SQL/API/UI evidence, thời gian ETL/query/API trên cấu hình ghi nhận, hạn chế và cải tiến.
- Verify: reset → batch 1 → batch 2 → rerun → reset/repeat; lỗi giữa batch/retry; clone sạch làm theo setup; KPI/DSS khớp warehouse.
- Bằng chứng/PR: chưa có.

### DOC-01 — Báo cáo và slide

- Owner: Phúc và Kiên. Trạng thái: TODO. Phụ thuộc: cập nhật song song; chốt sau INT-01.
- Đọc: đề bài, docs/report/README, docs/slides/README.
- Đầu ra: nguồn và PDF nghiệp vụ/DW/ETL/OLAP/API/UI/DSS/đánh giá, phân công đóng góp.
- Verify: hình/số liệu khớp phiên bản cuối; mọi yêu cầu đề bài có bằng chứng; cả hai có phần trình bày.
- Bằng chứng/PR: chưa có.

### SUBMIT-01 — Đóng gói và nộp

- Owner: Kiên; Phúc cùng trình bày. Trạng thái: TODO. Phụ thuộc: DOC-01, INT-01.
- Đọc: đề bài và README.
- Đầu ra: ZIP báo cáo PDF, slide PDF, src/, sample data/, hướng dẫn chạy; video link trên bìa nếu áp dụng.
- Verify: clone/sample setup sạch, ZIP đầy đủ, không secrets/raw dataset; video 10–15 phút cả hai xuất hiện/trình bày và link xem được, hoặc diễn tập trực tiếp; nộp LMS trước 15/11/2026 23:59 Việt Nam.
- Bằng chứng/PR: chưa có; lưu xác nhận nộp, không tự nhận đã nộp.

## Mẫu cập nhật khi bàn giao

```text
Task ID / Owner / Trạng thái:
Thay đổi và đầu ra:
Phụ thuộc / tài liệu đã đọc:
Lệnh verify thực sự chạy:
Kết quả / expected / evidence hoặc PR:
Giới hạn / chưa verify / blocker / việc tiếp theo:
```
