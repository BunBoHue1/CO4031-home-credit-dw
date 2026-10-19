# Đóng góp và bàn giao

Đọc README, TASKS, AGENTS và hướng dẫn phần liên quan. Nhận task bằng PR cập nhật owner/trạng thái, tránh làm trùng việc.

- Branch feat/<task-id>-<mo-ta>, fix/... hoặc docs/..., PR vào main, người còn lại review.
- Commit mệnh lệnh ngắn, ví dụ Validate installment grain. Dùng PR template.
- Không tự đổi schema, KPI hoặc hợp đồng; Phúc và Kiên thống nhất và cập nhật tài liệu.
- Bàn giao thay đổi, lệnh thực sự chạy, kết quả/bằng chứng và giới hạn. Chưa kiểm chứng phải ghi rõ lý do.
- DONE chỉ khi đủ tiêu chí; BLOCKED ghi nguyên nhân. Cập nhật TASKS và README khi thêm lệnh.
- Không secrets, raw CSV, dependency hoặc output sinh ra trong PR.

Tài liệu UTF-8, warehouse snake_case, giữ ID nguồn. JavaScript 2 spaces, camelCase, PascalCase React component. Khi triển khai thêm manifest/lockfile và phiên bản đã kiểm chứng; Node dùng npm ci khi có package-lock, Python khóa dependency. Không tạo manifest rỗng.
