# Nguồn và sample

Raw CSV nằm trong data_set/, được ignore. data/ dành cho sample nhỏ nhất quán quan hệ của bài nộp.

## Kaggle

Nguồn Kiên cung cấp: [megancrenshaw/home-credit-default-risk](https://www.kaggle.com/datasets/megancrenshaw/home-credit-default-risk). Dùng thư viện Python `kagglehub`, không cần cài Kaggle CLI riêng. Các lệnh dưới chạy trong **PowerShell tại root repo**.

### 1. Chuẩn bị Python

Nếu chưa có `.venv`, tạo một lần; nếu đã có thì bỏ qua lệnh đầu. Dùng Python của môi trường trực tiếp, không cần activate:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install kagglehub
.\.venv\Scripts\python.exe -m pip show kagglehub
```

Đây là dependency hỗ trợ tải nguồn, chưa phải cài pipeline ETL. Ghi lại phiên bản đã cài khi kiểm chứng DATA-01; không commit `.venv/`.

### 2. Tải về cache

Đoạn Kiên cung cấp là Python, không dán trực tiếp từng dòng vào PowerShell. Chạy cả block sau:

```powershell
@'
import kagglehub

path = kagglehub.dataset_download("megancrenshaw/home-credit-default-risk")
print("Path to dataset files:", path)
'@ | .\.venv\Scripts\python.exe -
```

Mặc định tải phiên bản mới nhất vào **cache của KaggleHub**, chưa đặt vào `data_set/`. Ghi lại đường dẫn được in; lần chạy lại có thể dùng cache. Không dùng `force_download=True` khi chỉ cần dùng lại dữ liệu.

Nếu bị yêu cầu xác thực, chạy `.\.venv\Scripts\python.exe -c "import kagglehub; kagglehub.login()"` và nhập token theo lời nhắc. Kiểm tra quyền truy cập/điều khoản trên trang dataset nếu gặp 401/403. Không ghi token vào code, repo hoặc log. Xem [hướng dẫn chính thức KaggleHub](https://github.com/Kaggle/kagglehub) về cache, xác thực và version handle.

### 3. Đặt CSV vào data_set/

Đoạn dưới tải lại qua cache nếu có, tìm CSV trong các thư mục con và copy theo cấu trúc repo. Không giả định bản tải có sẵn layout giống máy Kiên. Chạy tại root, sau bước cài thư viện:

```powershell
@'
from pathlib import Path
import filecmp
import shutil
import kagglehub

root = Path.cwd()
if not (root / "schema_doc.md").is_file():
    raise SystemExit("Run this block from the repository root.")

source = Path(kagglehub.dataset_download("megancrenshaw/home-credit-default-risk"))
print("Downloaded dataset:", source)
destination = root / "data_set"
history = [
    "previous_application.csv", "installments_payments.csv", "bureau.csv",
    "bureau_balance.csv", "POS_CASH_balance.csv", "credit_card_balance.csv",
]
applications = ["application_train.csv", "application_test.csv", "sample_submission.csv"]
mapping = {name: [destination / name] for name in history}
mapping.update({name: [destination / "home-credit-default-risk" / name] for name in applications})
mapping["HomeCredit_columns_description.csv"] = [
    destination / "HomeCredit_columns_description.csv",
    destination / "home-credit-default-risk" / "HomeCredit_columns_description.csv",
]

# Validate all files before copying; never overwrite different local data.
copies = []
for name, targets in mapping.items():
    matches = sorted(source.rglob(name))
    matches = [path for path in matches if path.is_file()]
    if not matches:
        raise SystemExit(f"Missing CSV in downloaded dataset: {name}")
    original = matches[0]
    if any(not filecmp.cmp(original, other, shallow=False) for other in matches[1:]):
        raise SystemExit(f"Conflicting source copies: {name}")
    for target in targets:
        if target.exists():
            if not target.is_file() or not filecmp.cmp(original, target, shallow=False):
                raise SystemExit(f"Local file differs; review before replacing: {target}")
        else:
            copies.append((original, target))

for original, target in copies:
    target.parent.mkdir(parents=True, exist_ok=True)
    shutil.copy2(original, target)
    print("Copied:", target.relative_to(root))
print("Ready:", destination)
'@ | .\.venv\Scripts\python.exe -
```

Nếu thiếu CSV hoặc có nhiều bản khác nội dung, dừng và kiểm tra layout nguồn; không tự chọn bản khác. File local giống nội dung được giữ nguyên; file khác bị từ chối ghi đè. Không cần xóa dữ liệu hiện có để chạy hướng dẫn. Cache vẫn tồn tại nên cần dung lượng cho cả cache và bản copy. Nếu nguồn chỉ chứa archive chưa giải nén, giải nén vào thư mục tải riêng rồi kiểm tra trước khi copy.

Hai bản từ điển cột được copy từ cùng một file để phù hợp layout đã có. Dataset upload lại trên Kaggle có thể khác dữ liệu local: DATA-01 vẫn cần đối soát cột, số dòng và checksum, không xem tải thành công là đủ.

### 4. Kiểm tra và tái lập

```powershell
Get-ChildItem -Recurse data_set -Filter *.csv
Get-ChildItem -Recurse data_set -Filter *.csv | Get-FileHash -Algorithm SHA256
git check-ignore data_set/bureau.csv
git status --short
```

Giữ `DATA_DIR=./data_set` trong `.env`. Không có CSV thô trong danh sách staged. Sau lần tải đã kiểm chứng, ghi phiên bản Kaggle, phiên bản kagglehub và SHA256 vào bằng chứng DATA-01; dùng handle `megancrenshaw/home-credit-default-risk/versions/<N>` thay cho latest ở **cả hai đoạn** để nhóm tái tạo đúng phiên bản (thay `<N>` bằng số thực tế, không đoán).

Hướng dẫn đã đối chiếu API chính thức; chưa thực hiện tải mạng trong lần cập nhật tài liệu này. Repo chưa có script tải độc lập; code trên là hướng dẫn thao tác.

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
