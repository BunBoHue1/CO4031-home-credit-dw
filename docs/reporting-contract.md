# Hợp đồng reporting → backend

**Chờ CONTRACT-01**, Phúc và Kiên cùng duyệt. Chưa có view hoặc API hiện thực.

Mỗi dataset/KPI ghi tên/mục đích, grain/key, cột/kiểu PostgreSQL/đơn vị/nullable, công thức tử số/mẫu số, population/cutoff, filter/kiểu/mặc định, Unknown/NULL, tổng hợp/drill-down, batch publication và ví dụ output đối soát SQL.

Phúc cung cấp query/view và bằng chứng; Kiên thiết kế endpoint/response. Chốt serialize decimal/bigint và NULL để không mất chính xác. Backend không đọc staging hoặc join fact gây nhân dòng. Thay đổi cần PR và hai người review.
