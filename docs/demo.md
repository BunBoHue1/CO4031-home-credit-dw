# Demo hai batch

Thiết kế, **chưa có CLI**. Phúc triển khai; Kiên nghiệm thu API/dashboard.

## Manifest

Sắp xếp SK_ID_CURR train duy nhất, hợp lệ tăng dần. Batch 1 lấy floor(N/2) ID đầu, batch 2 lấy phần còn lại. Lưu manifest/checksum nguồn. Hai tập không giao nhau và hợp bằng population train. Mỗi batch giữ toàn bộ lịch sử; Bureau balance lọc qua SK_ID_BUREAU. Không chia raw rows độc lập hoặc dùng train/test.

## CLI dự kiến

Sau triển khai và setup, tại src/etl:

```powershell
python -m etl clear-data
python -m etl load-data-1
python -m etl status
python -m etl load-data-2
python -m etl load-data-2
```

- clear-data: xóa dữ liệu staging/warehouse/reporting và trạng thái batch thuộc project; giữ schema, khởi tạo member mặc định. Không xóa database/raw CSV.
- load-data-1/2: nạp batch, verify và cập nhật reporting; batch 2 cần batch 1 thành công.
- status: database đích không có password, trạng thái/lần chạy batch, số dòng bảng.
- Chạy lại không trùng, key cũ ổn định. Lỗi exit code khác 0, có log, không đánh dấu thành công; retry an toàn.
- Reporting chỉ thấy batch hoàn tất; Phúc ghi cơ chế transaction/publish trước code. Không có API/nút reset.

## Nghiệm thu

Reset → empty dashboard → batch 1 → ghi KPI → batch 2 → refresh/đối soát SQL → nạp lại từng batch không đổi → reset và tái lập. Member mặc định không tính là dữ liệu nghiệp vụ; không yêu cầu surrogate key giống sau reset.

Giả lập lỗi giữa batch: không lộ dữ liệu dở dang, retry không nhân bản. Reconciliation giải thích số dòng lọc/cách ly. Ghi SQL, ảnh KPI, thời gian và phiên bản dữ liệu. Số khách hàng tăng rõ; tỷ lệ có thể chỉ đổi ít. Đây là nạp bổ sung khách hàng, không mô phỏng thời gian.
