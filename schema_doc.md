# Home Credit — Thiết kế Data Warehouse 
---

## 1. Phạm vi dữ liệu

DW dùng các nguồn chính sau:

- `application_train.csv`: hồ sơ vay hiện tại và nhãn `TARGET`.
- `previous_application.csv`: các đơn vay Home Credit trước đây.
- `installments_payments.csv`: lịch và các lần thanh toán trả góp thực tế.
- `bureau.csv`: thông tin khoản tín dụng bên ngoài.
- `bureau_balance.csv`: trạng thái bureau theo tháng.
- `POS_CASH_balance.csv`: snapshot POS/Cash theo tháng.
- `credit_card_balance.csv`: snapshot thẻ tín dụng theo tháng.

**Quy tắc phạm vi:** chỉ giữ lịch sử có `SK_ID_CURR` thuộc `application_train.csv`. Vì vậy các bảng lịch sử như `previous_application`, `installments_payments`, `bureau`, `POS_CASH_balance` và `credit_card_balance` chỉ được nạp nếu cuối cùng ánh xạ được về một `SK_ID_CURR` có trong `dim_customer`. Tức là lấy `SK_ID_CURR` trong `application_train.csv` làm chuẩn.

---

## 2. Tổng quan mô hình

Mô hình gồm đúng **5 dimension + 4 fact** và là một **fact constellation** vì nhiều fact dùng chung dimension.

### 5 Dimension

1. `dim_customer`
2. `dim_loan_type`
3. `dim_organization`
4. `dim_time`
5. `dim_status`

### 4 Fact

1. `fact_loan_application`
2. `fact_previous_application`
3. `fact_installment_payment`
4. `fact_monthly_balance`

### Ý nghĩa ngắn gọn của từng table

| Phần dim | --- |
|---|---|
| `dim_customer` | Người/hồ sơ hiện tại này là ai và có đặc điểm gì? |
| `dim_loan_type` | Khoản vay/tín dụng này thuộc loại nào? |
| `dim_organization` | Người nộp hồ sơ làm ở loại tổ chức nào? |
| `dim_time` | Sự kiện xảy ra ở ngày/tháng tương đối nào hoặc slot thứ/giờ nào? |
| `dim_status` | Trạng thái này thuộc domain nào và nghĩa là gì? |
|Phần fact|---|
| `fact_loan_application` | Hồ sơ hiện tại xin vay bao nhiêu, có đặc điểm gì, TARGET là gì? |
| `fact_previous_application` | Các đơn vay cũ đã xin gì và được xử lý ra sao? |
| `fact_installment_payment` | Mỗi kỳ phải trả bao nhiêu, đã trả bao nhiêu, có thiếu/trễ không? |
| `fact_monthly_balance` | Tài khoản nào, nguồn nào, ở tháng nào và trạng thái/chỉ số lúc đó ra sao? |

---



# 3. DIMENSIONS

## 3.1. `dim_customer`

**Một dòng:** một hồ sơ hiện tại, nhận diện bằng `SK_ID_CURR` trong `application_train`.

**Dùng để:** phân tích các fact theo đặc điểm người nộp hồ sơ như tuổi, học vấn, nghề nghiệp, loại thu nhập và tình trạng nhà ở.

**PK:** `customer_key`  
**UNIQUE:** `sk_id_curr`

> Lưu ý: Trong project này, `SK_ID_CURR` được dùng để nhận biết từng khách hàng trong bộ dữ liệu và liên kết thông tin của họ giữa các bảng. Tuy nhiên, bộ dữ liệu không cho biết đây có phải là mã khách hàng được Home Credit sử dụng lâu dài hay không. Nói thêm thôi chứ ở dưới cùng có=]]

### `customer_key`

**Nguồn**

Không có trực tiếp trong CSV. Đây là surrogate key do DB/ETL sinh.

**Logic ETL + ví dụ**

ETL tra `dim_customer` bằng `sk_id_curr`. Nếu đã có thì giữ nguyên `customer_key`; nếu chưa có thì INSERT và để DB sinh key mới.

Ví dụ:

```text
application_train.SK_ID_CURR = 100001

Nếu dim_customer đã có:
customer_key = 42, sk_id_curr = 100001
→ dùng lại customer_key = 42
```

Chạy lại ETL với cùng dữ liệu không được tạo thêm một customer khác cho `100001`.

### `sk_id_curr`

**Nguồn**

```text
application_train.SK_ID_CURR
```

**Logic ETL + ví dụ**

Giữ nguyên mã nguồn sau khi kiểm tra không thiếu và không trùng bất thường.

```text
SK_ID_CURR = 100001
→ sk_id_curr = 100001
```

Đây là khóa nguồn dùng để lookup `customer_key` từ các fact.

### `gender`

**Nguồn**

```text
application_train.CODE_GENDER
```

**Logic ETL + ví dụ**

Lấy giá trị nguồn sau bước làm sạch chuỗi. Giá trị thiếu hoặc không hợp lệ được giữ/chuẩn hóa theo quy tắc chất lượng dữ liệu đã thống nhất, không tự suy đoán giới tính.

```text
CODE_GENDER = 'F'
→ gender = 'F'
```

### `age_years`

**Nguồn**

```text
application_train.DAYS_BIRTH
```

**Logic ETL + ví dụ**

`DAYS_BIRTH` là độ lệch ngày âm so với mốc hồ sơ hiện tại. Tính tuổi xấp xỉ:

```text
age_years = FLOOR(-DAYS_BIRTH / 365.25)
```

Ví dụ:

```text
DAYS_BIRTH = -10958
→ -(-10958) / 365.25 ≈ 30
→ age_years = 30
```

Nếu thiếu, không phải số hoặc không âm thì để `NULL` và log lỗi chất lượng thay vì tự lấy trị tuyệt đối.

### `age_band`

**Nguồn**

Không có trực tiếp trong CSV. Tính từ `age_years`.

**Logic ETL + ví dụ**

Nhóm tuổi:

```text
<25
25–34
35–44
45–54
55+
Unknown
```

Ví dụ:

```text
age_years = 30
→ age_band = '25–34'
```

### `education_type`

**Nguồn**

```text
application_train.NAME_EDUCATION_TYPE
```

**Logic ETL + ví dụ**

Lấy và làm sạch chuỗi nguồn; nếu thiếu/rỗng thì dùng `Unknown`.

```text
NAME_EDUCATION_TYPE = 'Higher education'
→ education_type = 'Higher education'
```

### `family_status`

**Nguồn**

```text
application_train.NAME_FAMILY_STATUS
```

**Logic ETL + ví dụ**

Lấy trạng thái gia đình từ nguồn, chuẩn hóa khoảng trắng; thiếu/rỗng → `Unknown`.

```text
NAME_FAMILY_STATUS = 'Married'
→ family_status = 'Married'
```

### `income_type`

**Nguồn**

```text
application_train.NAME_INCOME_TYPE
```

**Logic ETL + ví dụ**

Giữ loại nguồn thu nhập, không nhầm với số tiền thu nhập.

```text
NAME_INCOME_TYPE = 'Working'
→ income_type = 'Working'
```

### `occupation_type`

**Nguồn**

```text
application_train.OCCUPATION_TYPE
```

**Logic ETL + ví dụ**

Lấy nghề nghiệp từ nguồn; thiếu/rỗng → `Unknown`.

```text
OCCUPATION_TYPE = 'Laborers'
→ occupation_type = 'Laborers'
```

### `housing_type`

**Nguồn**

```text
application_train.NAME_HOUSING_TYPE
```

**Logic ETL + ví dụ**

Giữ loại hình nhà ở theo nguồn.

```text
NAME_HOUSING_TYPE = 'House / apartment'
→ housing_type = 'House / apartment'
```

### `own_car_flag`

**Nguồn**

```text
application_train.FLAG_OWN_CAR
```

**Logic ETL + ví dụ**

Giữ cờ sở hữu ô tô.

```text
FLAG_OWN_CAR = 'Y'
→ own_car_flag = 'Y'
```

### `own_realty_flag`

**Nguồn**

```text
application_train.FLAG_OWN_REALTY
```

**Logic ETL + ví dụ**

Giữ cờ sở hữu bất động sản. Không suy ra rằng bất động sản đó là tài sản thế chấp.

```text
FLAG_OWN_REALTY = 'Y'
→ own_realty_flag = 'Y'
```

### `children_count`

**Nguồn**

```text
application_train.CNT_CHILDREN
```

**Logic ETL + ví dụ**

Lấy số con của người nộp hồ sơ.

```text
CNT_CHILDREN = 2
→ children_count = 2
```

### `family_member_count`

**Nguồn**

```text
application_train.CNT_FAM_MEMBERS
```

**Logic ETL + ví dụ**

Lấy số thành viên gia đình.

```text
CNT_FAM_MEMBERS = 4
→ family_member_count = 4
```

---

## 3.2. `dim_loan_type`

**Một dòng:** một tổ hợp phân loại khoản vay.

**Dùng để:** cho biết fact đang nói tới loại khoản vay/tín dụng nào, đồng thời phân biệt Home Credit với Bureau.

**PK:** `loan_type_key`  
**UNIQUE:** `loan_domain + contract_type + credit_type + portfolio + product_type + goods_category`

Bộ khóa logic K:

```text
K = (
  loan_domain,
  contract_type,
  credit_type,
  portfolio,
  product_type,
  goods_category
)
```

### `loan_type_key`

**Nguồn**

Không có trong source. DB/ETL sinh sau khi chuẩn hóa bộ K.

**Logic ETL + ví dụ**

ETL tìm đúng tổ hợp K trong `dim_loan_type`. Nếu đã tồn tại thì dùng lại PK; nếu chưa có thì INSERT rồi lấy PK mới.

```text
K = (HOME_CREDIT, Consumer loans, Not applicable, POS, x-sell, Mobile)
→ loan_type_key = 7
```

### `loan_domain`

**Nguồn**

Do ETL gán theo nguồn:

```text
application_train / previous_application → HOME_CREDIT
bureau → BUREAU
thiếu cha/không xác định được → UNKNOWN
```

**Logic ETL + ví dụ**

Nếu đang tạo loan type cho đơn hiện tại hoặc đơn cũ Home Credit:

```text
loan_domain = HOME_CREDIT
```

Nếu lấy `CREDIT_TYPE` từ `bureau`:

```text
loan_domain = BUREAU
```

### `contract_type`

**Nguồn**

Home Credit:

```text
application_train.NAME_CONTRACT_TYPE
previous_application.NAME_CONTRACT_TYPE
```

Bureau không có khái niệm contract type Home Credit tương ứng.

**Logic ETL + ví dụ**

Home Credit: `TRIM`, rỗng/NULL → `Unknown`.

```text
NAME_CONTRACT_TYPE = 'Cash loans'
→ contract_type = 'Cash loans'
```

Bureau:

```text
contract_type = 'Not applicable'
```

### `credit_type`

**Nguồn**

```text
bureau.CREDIT_TYPE
```

**Logic ETL + ví dụ**

Bureau giữ loại tín dụng nguồn:

```text
CREDIT_TYPE = 'Consumer credit'
→ credit_type = 'Consumer credit'
```

Home Credit không dùng phân loại bureau này:

```text
credit_type = 'Not applicable'
```

### `portfolio`

**Nguồn**

```text
previous_application.NAME_PORTFOLIO
```

**Logic ETL + ví dụ**

Đơn cũ/POS/thẻ/trả góp lấy giá trị từ `previous_application` qua `SK_ID_PREV`.

```text
NAME_PORTFOLIO = 'POS'
→ portfolio = 'POS'
```

Hồ sơ hiện tại `application_train` không có trường này nên:

```text
portfolio = 'Unknown'
```

Bureau:

```text
portfolio = 'Not applicable'
```

### `product_type`

**Nguồn**

```text
previous_application.NAME_PRODUCT_TYPE
```

**Logic ETL + ví dụ**

Đơn cũ/POS/thẻ/trả góp kế thừa qua `SK_ID_PREV`.

```text
NAME_PRODUCT_TYPE = 'x-sell'
→ product_type = 'x-sell'
```

Hồ sơ hiện tại:

```text
product_type = 'Unknown'
```

Bureau:

```text
product_type = 'Not applicable'
```

### `goods_category`

**Nguồn**

```text
previous_application.NAME_GOODS_CATEGORY
```

**Logic ETL + ví dụ**

Đơn cũ/POS/thẻ/trả góp lấy qua `SK_ID_PREV`.

```text
NAME_GOODS_CATEGORY = 'Mobile'
→ goods_category = 'Mobile'
```

Hồ sơ hiện tại không có trường này:

```text
goods_category = 'Unknown'
```

Bureau:

```text
goods_category = 'Not applicable'
```

Không tự suy `goods_category` từ `AMT_GOODS_PRICE`.

---

## 3.3. `dim_organization`

**Một dòng:** một loại tổ chức nơi người nộp hồ sơ làm việc.

**Dùng để:** phân tích hồ sơ hiện tại theo loại nơi làm việc.

**PK:** `organization_key`  
**UNIQUE:** `organization_type`

### `organization_key`

**Nguồn**

Không có trong CSV. DB/ETL sinh.

**Logic ETL + ví dụ**

Sau khi chuẩn hóa `organization_type`, ETL lookup dimension. Nếu đã có thì dùng lại key; nếu chưa có thì INSERT.

```text
organization_type = 'School'
→ organization_key = 7
```

### `organization_type`

**Nguồn**

```text
application_train.ORGANIZATION_TYPE
```

**Logic ETL + ví dụ**

`TRIM` giá trị; NULL/rỗng → `Unknown`.

```text
ORGANIZATION_TYPE = 'School'
→ organization_type = 'School'
```

Đây là **loại nơi người nộp hồ sơ làm việc**, không phải tổ chức cho vay.

---

## 3.4. `dim_time`

**Một dòng:** một giá trị thời gian thuộc một trong bốn miền: `DAY`, `MONTH`, `SLOT`, `UNKNOWN`.

**Dùng để:** dùng chung một dimension cho ngày tương đối, tháng tương đối và thứ/giờ nộp hồ sơ.

**PK:** `time_key`  
**UNIQUE:** `time_code`

Ba dạng chính:

```text
DAY   → D:-30
MONTH → M:-3
SLOT  → S:MONDAY:14
```

`D:-3` và `M:-3` là hai giá trị khác nhau: ba ngày trước khác ba tháng trước.

### `time_key`

**Nguồn**

Không có trong source. DB/ETL sinh.

**Logic ETL + ví dụ**

ETL tra bằng `time_code`. Có rồi thì giữ PK, chưa có thì INSERT.

```text
time_code = 'D:-30'
→ time_key = 15
```

### `time_code`

**Nguồn**

Do ETL tạo từ các thuộc tính thời gian đã chuẩn hóa.

**Logic ETL + ví dụ**

```text
DAY   → 'D:' + day_offset
MONTH → 'M:' + month_offset
SLOT  → 'S:' + weekday_name + ':' + hour_of_day
UNKNOWN → 'UNKNOWN'
```

Ví dụ:

```text
day_offset = -30       → D:-30
month_offset = -3      → M:-3
MONDAY + 14            → S:MONDAY:14
```

Dùng biểu diễn số nhất quán, không tạo đồng thời `D:-3` và `D:-3.0`.

### `time_type`

**Nguồn**

Do ETL gán dựa trên cột nguồn:

```text
DAYS_DECISION / DAYS_INSTALMENT / completion_day → DAY
MONTHS_BALANCE → MONTH
WEEKDAY_APPR_PROCESS_START + HOUR_APPR_PROCESS_START → SLOT
không xác định → UNKNOWN
```

**Logic ETL + ví dụ**

```text
previous_application.DAYS_DECISION = -30
→ time_type = DAY
```

```text
credit_card_balance.MONTHS_BALANCE = -3
→ time_type = MONTH
```

### `day_offset`

**Nguồn**

Có thể đến từ:

```text
previous_application.DAYS_DECISION
installments_payments.DAYS_INSTALMENT
completion_day tính từ installments_payments
```

**Logic ETL + ví dụ**

Giữ số nguyên tương đối so với mốc hồ sơ hiện tại.

```text
DAYS_DECISION = -30
→ day_offset = -30
→ nghĩa là 30 ngày trước mốc hồ sơ hiện tại
```

Các dòng `MONTH`, `SLOT`, `UNKNOWN` để `day_offset = NULL`.

### `month_offset`

**Nguồn**

```text
bureau_balance.MONTHS_BALANCE
POS_CASH_balance.MONTHS_BALANCE
credit_card_balance.MONTHS_BALANCE
```

**Logic ETL + ví dụ**

Giữ nguyên độ lệch tháng, không chia `DAYS_*` cho 30 để tạo tháng.

```text
MONTHS_BALANCE = -3
→ month_offset = -3
→ 3 tháng trước mốc hồ sơ hiện tại
```

### `weekday_name`

**Nguồn**

```text
application_train.WEEKDAY_APPR_PROCESS_START
```

**Logic ETL + ví dụ**

`TRIM` + `UPPER`, chỉ nhận `MONDAY` đến `SUNDAY`. Phải đi cùng giờ hợp lệ để tạo SLOT.

```text
WEEKDAY_APPR_PROCESS_START = 'Monday'
→ weekday_name = 'MONDAY'
```

Các loại thời gian khác để `NULL`.

### `hour_of_day`

**Nguồn**

```text
application_train.HOUR_APPR_PROCESS_START
```

**Logic ETL + ví dụ**

Chỉ nhận số nguyên từ `0` đến `23`.

```text
HOUR_APPR_PROCESS_START = 14
→ hour_of_day = 14
```

`14` chỉ biểu diễn giờ trong ngày, không chứa phút/giây.

### `period_group`

**Nguồn**

Không có trong CSV. Tính từ `time_type` và offset/giờ.

**Logic ETL + ví dụ**

Đề xuất grouping:

```text
DAY:
0–30 ngày trước   → DAY:0-30_AGO
31–90             → DAY:31-90_AGO
91–365            → DAY:91-365_AGO
>365              → DAY:GT365_AGO
offset > 0        → DAY:FUTURE

MONTH:
0–3 tháng trước   → MONTH:0-3_AGO
4–6               → MONTH:4-6_AGO
7–12              → MONTH:7-12_AGO
>12               → MONTH:GT12_AGO
offset > 0        → MONTH:FUTURE

SLOT:
0–5   → SLOT:NIGHT
6–11  → SLOT:MORNING
12–17 → SLOT:AFTERNOON
18–23 → SLOT:EVENING
```

Ví dụ:

```text
hour_of_day = 14
→ period_group = SLOT:AFTERNOON
```

Các nhóm trên là quy tắc thiết kế của project, không phải nhãn gốc Kaggle.

---

## 3.5. `dim_status`

**Một dòng:** một cặp `(status_domain, status_code)`.

**Dùng để:** phân biệt trạng thái của đơn cũ, kỳ trả góp, bureau tháng, POS tháng và thẻ tháng.

**PK:** `status_key`  
**UNIQUE:** `status_domain + status_code`

Ví dụ:

```text
PREVIOUS_APPLICATION / Approved
INSTALLMENT / PAID_LATE
BUREAU_MONTHLY / C
POS_MONTHLY / Active
CREDIT_CARD_MONTHLY / Active
```

### `status_key`

**Nguồn**

Không có trực tiếp trong source. DB/ETL sinh.

**Logic ETL + ví dụ**

Lookup theo cả `status_domain + status_code`. Có rồi thì dùng lại PK; chưa có thì INSERT.

```text
(PREVIOUS_APPLICATION, Approved)
→ status_key = 5
```

### `status_domain`

**Nguồn**

ETL gán theo nguồn/quy trình:

```text
previous_application → PREVIOUS_APPLICATION
kỳ trả góp đã tổng hợp → INSTALLMENT
bureau_balance → BUREAU_MONTHLY
POS_CASH_balance → POS_MONTHLY
credit_card_balance → CREDIT_CARD_MONTHLY
không xác định → UNKNOWN
```

**Logic ETL + ví dụ**

```text
Nguồn: bureau_balance
→ status_domain = BUREAU_MONTHLY
```

Nếu mã trạng thái thiếu nhưng biết nguồn, vẫn giữ đúng domain nguồn và dùng `status_code = UNKNOWN`.

### `status_code`

**Nguồn**

Đơn cũ:

```text
previous_application.NAME_CONTRACT_STATUS
```

POS:

```text
POS_CASH_balance.NAME_CONTRACT_STATUS
```

Credit Card:

```text
credit_card_balance.NAME_CONTRACT_STATUS
```

Bureau:

```text
bureau_balance.STATUS
```

Installment: do ETL tính.

**Logic ETL + ví dụ**

Đơn cũ/POS/thẻ: `TRIM` code nguồn, rỗng/NULL → `UNKNOWN` trong đúng domain.

```text
previous_application.NAME_CONTRACT_STATUS = 'Approved'
→ PREVIOUS_APPLICATION / Approved
```

Bureau giữ các code nguồn như `0..5`, `C`, `X`:

```text
STATUS = 'C'
→ BUREAU_MONTHLY / C
```

Installment:

```text
đã trả đủ và completion_day <= due_day → PAID_ON_TIME
đã trả đủ và completion_day > due_day  → PAID_LATE
chưa đủ và due_day <= 0                → INCOMPLETE_DUE
chưa đủ và due_day > 0                 → NOT_DUE
thiếu/xung đột dữ liệu                 → UNKNOWN
```

Ví dụ:

```text
due_day = -10
completion_day = -8
→ status_code = PAID_LATE
```

### `status_description`

**Nguồn**

Không có trực tiếp trong CSV. Lấy từ bảng mapping cấu hình ETL theo `(status_domain, status_code)`.

**Logic ETL + ví dụ**

```text
(PREVIOUS_APPLICATION, Approved)
→ 'Đơn vay được duyệt'

(BUREAU_MONTHLY, C)
→ 'Khoản bureau đã đóng'

(INSTALLMENT, PAID_LATE)
→ 'Kỳ đã hoàn tất trễ'
```

Nếu code mới chưa có mô tả thì vẫn giữ code, gán mô tả tạm kiểu `Chưa ánh xạ: <domain>/<code>` và log để bổ sung mapping.

---

# 4. FACT TABLES

## 4.1. `fact_loan_application`

**Một dòng:** một hồ sơ vay hiện tại trong `application_train`, tức một `SK_ID_CURR`.

**Dùng để:** phân tích khoản vay hiện tại theo khách hàng, loại hợp đồng, nơi làm việc, thứ/giờ nộp hồ sơ, thu nhập, số tiền tín dụng, external scores và `TARGET`.

**PK:** `loan_application_key`  
**FK:** `customer_key`, `loan_type_key`, `organization_key`, `application_time_key`  
**UNIQUE:** `customer_key`

### `loan_application_key`

**Nguồn**

Không có trong CSV. DB/ETL sinh.

**Logic ETL + ví dụ**

Lookup fact theo `customer_key`. Nếu hồ sơ đã có thì giữ PK; nếu chưa có thì INSERT.

```text
customer_key = 42
→ loan_application_key = 100
```

Chạy lại ETL không được tạo fact thứ hai cho cùng `customer_key`.

### `customer_key`

**Nguồn**

```text
application_train.SK_ID_CURR
```

Sau đó lookup:

```text
dim_customer.sk_id_curr
```

**Logic ETL + ví dụ**

```text
application_train.SK_ID_CURR = 100001

dim_customer:
customer_key = 42, sk_id_curr = 100001

→ fact.customer_key = 42
```

Nếu không lookup được customer thì cách ly/đối soát, không tự tạo customer giả.

### `loan_type_key`

**Nguồn**

Từ `application_train.NAME_CONTRACT_TYPE`; các thuộc tính còn lại do ETL gán vì hồ sơ hiện tại không có `portfolio`, `product_type`, `goods_category`.

**Logic ETL + ví dụ**

Dựng K:

```text
(
  HOME_CREDIT,
  NAME_CONTRACT_TYPE,
  Not applicable,
  Unknown,
  Unknown,
  Unknown
)
```

Ví dụ:

```text
NAME_CONTRACT_TYPE = 'Cash loans'
→ K = (HOME_CREDIT, Cash loans, Not applicable, Unknown, Unknown, Unknown)
→ lookup dim_loan_type
→ loan_type_key = 5
```

### `organization_key`

**Nguồn**

```text
application_train.ORGANIZATION_TYPE
```

**Logic ETL + ví dụ**

Chuẩn hóa `ORGANIZATION_TYPE`, rỗng/NULL → `Unknown`, rồi lookup `dim_organization`.

```text
ORGANIZATION_TYPE = 'School'
→ organization_key = 7
```

### `application_time_key`

**Nguồn**

```text
application_train.WEEKDAY_APPR_PROCESS_START
application_train.HOUR_APPR_PROCESS_START
```

**Logic ETL + ví dụ**

Ví dụ:

```text
WEEKDAY_APPR_PROCESS_START = MONDAY
HOUR_APPR_PROCESS_START = 14
→ time_code = S:MONDAY:14
→ lookup dim_time
→ application_time_key = 25
```

Đây là thứ/giờ nộp hồ sơ, không phải một ngày lịch đầy đủ.

### `amt_income_total`

**Nguồn**

```text
application_train.AMT_INCOME_TOTAL
```

**Logic ETL + ví dụ**

Lấy số tiền từ source sau kiểm tra kiểu dữ liệu.

```text
AMT_INCOME_TOTAL = 180000
→ amt_income_total = 180000
```

### `amt_credit`

**Nguồn**

```text
application_train.AMT_CREDIT
```

**Logic ETL + ví dụ**

```text
AMT_CREDIT = 500000
→ amt_credit = 500000
```

### `amt_annuity`

**Nguồn**

```text
application_train.AMT_ANNUITY
```

**Logic ETL + ví dụ**

```text
AMT_ANNUITY = 25000
→ amt_annuity = 25000
```

Đây là khoản niên kim/thanh toán định kỳ theo định nghĩa nguồn.

### `amt_goods_price`

**Nguồn**

```text
application_train.AMT_GOODS_PRICE
```

**Logic ETL + ví dụ**

```text
AMT_GOODS_PRICE = 450000
→ amt_goods_price = 450000
```

### `ext_source_1`

**Nguồn**

```text
application_train.EXT_SOURCE_1
```

**Logic ETL + ví dụ**

Giữ điểm chuẩn hóa từ nguồn bên ngoài số 1 như một tín hiệu riêng.

```text
EXT_SOURCE_1 = 0.62
→ ext_source_1 = 0.62
```

Không hiểu `0.62` là xác suất 62% và không tự cộng score này với hai score còn lại.

### `ext_source_2`

**Nguồn**

```text
application_train.EXT_SOURCE_2
```

**Logic ETL + ví dụ**

```text
EXT_SOURCE_2 = 0.71
→ ext_source_2 = 0.71
```

Giữ riêng như một feature/tín hiệu đánh giá rủi ro.

### `ext_source_3`

**Nguồn**

```text
application_train.EXT_SOURCE_3
```

**Logic ETL + ví dụ**

```text
EXT_SOURCE_3 = 0.55
→ ext_source_3 = 0.55
```

Giữ riêng, không tạo một “tổng external score” nếu chưa có quy tắc được chứng minh.

### `target_difficulty_flag`

**Nguồn**

```text
application_train.TARGET
```

**Logic ETL + ví dụ**

Giữ nhãn nguồn:

```text
TARGET = 1
→ target_difficulty_flag = 1
```

```text
TARGET = 0
→ target_difficulty_flag = 0
```

Đây là **nhãn có sẵn trong dữ liệu train**, không phải dự đoán do DSS/model của nhóm tạo ra.

---

## 4.2. `fact_previous_application`

**Một dòng:** một đơn vay Home Credit trước đây, định danh bằng `SK_ID_PREV`.

**Dùng để:** phân tích lịch sử các đơn cũ: loại sản phẩm, trạng thái duyệt/từ chối, số tiền đề nghị, số tiền tín dụng, số kỳ, tiền trả trước.

**PK:** `previous_application_key`  
**FK:** `customer_key`, `loan_type_key`, `status_key`, `decision_time_key`  
**UNIQUE:** `sk_id_prev`

> Chỉ giữ đơn cũ có `SK_ID_CURR` thuộc `application_train`. Tức là `previous_application.csv` có thể có lịch sử đơn vay của nhiều `SK_ID_CURR`, nhưng btl hiện tại t chỉ phân tích những khách hàng có mặt trong `application_train.csv`.

### `previous_application_key`

**Nguồn**

Không có trong CSV. DB/ETL sinh.

**Logic ETL + ví dụ**

Lookup fact bằng `sk_id_prev`. Có rồi thì giữ PK; chưa có thì INSERT.

```text
SK_ID_PREV = 5001
→ previous_application_key = 20
```

### `customer_key`

**Nguồn**

```text
previous_application.SK_ID_CURR
```

**Logic ETL + ví dụ**

Chỉ giữ row nếu `SK_ID_CURR` lookup được trong `dim_customer`.

```text
previous_application.SK_ID_CURR = 100001
→ dim_customer.customer_key = 42
→ customer_key = 42
```

Một `SK_ID_PREV` phải thuộc nhất quán về một `SK_ID_CURR`.

### `loan_type_key`

**Nguồn**

```text
previous_application.NAME_CONTRACT_TYPE
previous_application.NAME_PORTFOLIO
previous_application.NAME_PRODUCT_TYPE
previous_application.NAME_GOODS_CATEGORY
```

**Logic ETL + ví dụ**

Dựng:

```text
K = (
  HOME_CREDIT,
  NAME_CONTRACT_TYPE,
  Not applicable,
  NAME_PORTFOLIO,
  NAME_PRODUCT_TYPE,
  NAME_GOODS_CATEGORY
)
```

Ví dụ:

```text
Consumer loans / POS / x-sell / Mobile
→ K = (HOME_CREDIT, Consumer loans, Not applicable, POS, x-sell, Mobile)
→ loan_type_key = 7
```

### `status_key`

**Nguồn**

```text
previous_application.NAME_CONTRACT_STATUS
```

**Logic ETL + ví dụ**

```text
NAME_CONTRACT_STATUS = 'Approved'
→ (PREVIOUS_APPLICATION, Approved)
→ lookup dim_status
→ status_key = 5
```

NULL/rỗng → `PREVIOUS_APPLICATION / UNKNOWN`.

### `decision_time_key`

**Nguồn**

```text
previous_application.DAYS_DECISION
```

**Logic ETL + ví dụ**

```text
DAYS_DECISION = -30
→ time_code = D:-30
→ lookup dim_time
→ decision_time_key = 15
```

`-30` nghĩa là quyết định của đơn cũ diễn ra 30 ngày trước mốc hồ sơ hiện tại.

### `sk_id_prev`

**Nguồn**

```text
previous_application.SK_ID_PREV
```

**Logic ETL + ví dụ**

Giữ ID nguồn để truy vết đơn cũ.

```text
SK_ID_PREV = 5001
→ sk_id_prev = 5001
```

### `amt_application`

**Nguồn**

```text
previous_application.AMT_APPLICATION
```

**Logic ETL + ví dụ**

Đây là số tiền khách đề nghị trong đơn cũ.

```text
AMT_APPLICATION = 100000
→ amt_application = 100000
```

### `amt_credit`

**Nguồn**

```text
previous_application.AMT_CREDIT
```

**Logic ETL + ví dụ**

Giữ số tiền tín dụng cuối cùng ghi trong đơn trước.

```text
AMT_APPLICATION = 100000
AMT_CREDIT = 80000
→ amt_application = 100000
→ amt_credit = 80000
```

Không tự khẳng định `AMT_CREDIT` là số tiền thực tế đã giải ngân.

### `amt_annuity`

**Nguồn**

```text
previous_application.AMT_ANNUITY
```

**Logic ETL + ví dụ**

```text
AMT_ANNUITY = 8500
→ amt_annuity = 8500
```

### `amt_down_payment`

**Nguồn**

```text
previous_application.AMT_DOWN_PAYMENT
```

**Logic ETL + ví dụ**

```text
AMT_DOWN_PAYMENT = 5000
→ amt_down_payment = 5000
```

Đây là tiền trả trước theo dữ liệu nguồn.

### `amt_goods_price`

**Nguồn**

```text
previous_application.AMT_GOODS_PRICE
```

**Logic ETL + ví dụ**

```text
AMT_GOODS_PRICE = 20000
→ amt_goods_price = 20000
```

### `cnt_payment`

**Nguồn**

```text
previous_application.CNT_PAYMENT
```

**Logic ETL + ví dụ**

```text
CNT_PAYMENT = 12
→ cnt_payment = 12
```

Hiểu là số kỳ thanh toán được ghi cho đơn trước.

---

## 4.3. `fact_installment_payment`

**Một dòng:** một kỳ trả góp cụ thể của một khoản vay cũ, sau khi tổng hợp tất cả các lần thanh toán thuộc kỳ đó.

**Dùng để:** biết một kỳ phải trả bao nhiêu, đã trả bao nhiêu, còn thiếu bao nhiêu, hoàn tất khi nào và có trễ hay không.

**PK:** `installment_payment_key`  
**FK:** `customer_key`, `loan_type_key`, `status_key`, `due_time_key`, `completion_time_key`  
**UNIQUE:** `sk_id_prev + num_instalment_version + num_instalment_number`

Khóa nhóm G:

```text
G = (SK_ID_PREV, NUM_INSTALMENT_VERSION, NUM_INSTALMENT_NUMBER)
```

Ý nghĩa:

```text
SK_ID_PREV               → đơn cũ nào
NUM_INSTALMENT_VERSION   → phiên bản lịch trả góp nào
NUM_INSTALMENT_NUMBER    → kỳ số mấy trong phiên bản đó
```

Một kỳ có thể có nhiều raw rows vì khách có thể thanh toán từng phần nhiều lần.

### `installment_payment_key`

**Nguồn**

Không có trong CSV. DB/ETL sinh.

**Logic ETL + ví dụ**

Lookup theo G. Có rồi thì giữ PK; chưa có thì INSERT.

```text
G = (5001, 1, 3)
→ installment_payment_key = 50
```

### `customer_key`

**Nguồn**

```text
installments_payments.SK_ID_CURR
```

**Logic ETL + ví dụ**

Trong cùng G, tất cả row phải có cùng `SK_ID_CURR`. Sau đó lookup `dim_customer`.

```text
G = (5001,1,3)
SK_ID_CURR = 100001 ở mọi row
→ customer_key = 42
```

Nếu cùng kỳ xuất hiện nhiều `SK_ID_CURR` khác nhau thì cách ly kỳ, không tự chọn một customer.

### `loan_type_key`

**Nguồn**

`installments_payments` không có đủ thông tin loại sản phẩm, nên dùng:

```text
installments_payments.SK_ID_PREV
→ JOIN previous_application.SK_ID_PREV
```

rồi lấy các thuộc tính loan type từ `previous_application`.

**Logic ETL + ví dụ**

```text
SK_ID_PREV = 5001
→ previous_application:
   Consumer loans / POS / x-sell / Mobile
→ loan_type_key = 7
```

### `status_key`

**Nguồn**

Không có trực tiếp. Tính từ:

```text
scheduled_amount
paid_amount
due_day
completion_day
```

**Logic ETL + ví dụ**

```text
completion_day <= due_day → PAID_ON_TIME
completion_day > due_day  → PAID_LATE
chưa đủ tiền + due_day <= 0 → INCOMPLETE_DUE
chưa đủ tiền + due_day > 0  → NOT_DUE
thiếu/xung đột               → UNKNOWN
```

Ví dụ:

```text
due_day = -10
completion_day = -8
→ status = INSTALLMENT / PAID_LATE
→ lookup dim_status
→ status_key = 12
```

### `due_time_key`

**Nguồn**

```text
installments_payments.DAYS_INSTALMENT
```

**Logic ETL + ví dụ**

Tất cả raw rows trong cùng G phải có `DAYS_INSTALMENT` nhất quán.

```text
DAYS_INSTALMENT = -10
→ due_day = -10
→ time_code = D:-10
→ due_time_key = 21
```

Nếu các rows cùng kỳ có hạn khác nhau thì cách ly thay vì chọn tùy tiện.

### `completion_time_key`

**Nguồn**

Tính từ:

```text
installments_payments.DAYS_ENTRY_PAYMENT
installments_payments.AMT_PAYMENT
```

**Logic ETL + ví dụ**

Tìm **ngày đầu tiên tổng tiền thanh toán lũy kế đạt `scheduled_amount`**.

Ví dụ:

```text
scheduled_amount = 1000

Ngày -12: trả 400 → lũy kế 400
Ngày -8 : trả 600 → lũy kế 1000

→ completion_day = -8
→ time_code = D:-8
→ completion_time_key = 35
```

Nếu chưa bao giờ đạt đủ tiền thì `completion_time_key = NULL`.

Không dùng đơn giản `MAX(DAYS_ENTRY_PAYMENT)` vì có thể có khoản trả thêm sau khi khách đã hoàn tất nghĩa vụ.

### `sk_id_prev`

**Nguồn**

```text
installments_payments.SK_ID_PREV
```

**Logic ETL + ví dụ**

Giữ ID nguồn của đơn cũ.

```text
SK_ID_PREV = 5001
→ sk_id_prev = 5001
```

Kết hợp với version và number để nhận diện một kỳ.

### `num_instalment_version`

**Nguồn**

```text
installments_payments.NUM_INSTALMENT_VERSION
```

**Logic ETL + ví dụ**

Giữ phiên bản lịch trả góp.

```text
NUM_INSTALMENT_VERSION = 1
→ num_instalment_version = 1
```

Không gộp `(version=1, number=3)` với `(version=2, number=3)`.

### `num_instalment_number`

**Nguồn**

```text
installments_payments.NUM_INSTALMENT_NUMBER
```

**Logic ETL + ví dụ**

Giữ số thứ tự kỳ trong một version.

```text
NUM_INSTALMENT_VERSION = 1
NUM_INSTALMENT_NUMBER = 3
→ kỳ số 3 của lịch version 1
```

### `scheduled_amount`

**Nguồn**

```text
installments_payments.AMT_INSTALMENT
```

**Logic ETL + ví dụ**

Trong cùng G, `AMT_INSTALMENT` có thể bị lặp ở nhiều payment rows. Kiểm tra các giá trị phải nhất quán và chỉ lấy **một lần**, không SUM.

Ví dụ:

```text
Row 1: AMT_INSTALMENT = 1000, AMT_PAYMENT = 400
Row 2: AMT_INSTALMENT = 1000, AMT_PAYMENT = 600

scheduled_amount = 1000
KHÔNG phải 2000
```

Nếu cùng kỳ có `AMT_INSTALMENT` xung đột thì cách ly.

### `paid_amount`

**Nguồn**

```text
installments_payments.AMT_PAYMENT
installments_payments.DAYS_ENTRY_PAYMENT
```

**Logic ETL + ví dụ**

Cộng các payment events quan sát được tới mốc ngày 0 (`DAYS_ENTRY_PAYMENT <= 0`).

```text
Ngày -12: 400
Ngày -8 : 600
→ paid_amount = 1000
```

Nếu có payment ngày `+5`, khoản đó chưa được tính tại mốc quan sát ngày 0.

Không tự biến dữ liệu thiếu thành `0`.

### `payment_shortfall`

**Nguồn**

Không có trực tiếp. Tính từ `scheduled_amount`, `paid_amount`, `due_day`.

**Logic ETL + ví dụ**

Chỉ tính khi kỳ đã đến hạn (`due_day <= 0`) và số liệu đủ:

```text
payment_shortfall = MAX(scheduled_amount - paid_amount, 0)
```

Ví dụ:

```text
scheduled_amount = 1000
paid_amount = 600
due_day = -10
→ payment_shortfall = 400
```

Nếu kỳ chưa tới hạn hoặc dữ liệu chưa xác định thì để `NULL`.

### `days_late`

**Nguồn**

Không có trực tiếp. Tính từ `completion_day` và `due_day`.

**Logic ETL + ví dụ**

```text
days_late = MAX(completion_day - due_day, 0)
```

Ví dụ:

```text
due_day = -10
completion_day = -8
→ days_late = 2
```

Nếu chưa trả đủ thì `completion_day = NULL` và `days_late = NULL`, không gán `0`.

### Ví dụ tổng hợp một kỳ

Raw:

```text
SK_ID_PREV = 5001
SK_ID_CURR = 100001
VERSION = 1
NUMBER = 3

Row 1:
DAYS_INSTALMENT = -10
AMT_INSTALMENT = 1000
DAYS_ENTRY_PAYMENT = -12
AMT_PAYMENT = 400

Row 2:
DAYS_INSTALMENT = -10
AMT_INSTALMENT = 1000
DAYS_ENTRY_PAYMENT = -8
AMT_PAYMENT = 600
```

Sau ETL:

```text
scheduled_amount = 1000
paid_amount = 1000
completion_day = -8
due_day = -10
payment_shortfall = 0
days_late = 2
status = PAID_LATE
```

---

## 4.4. `fact_monthly_balance`

**Một dòng:** một tài khoản của một nguồn tại một tháng tương đối.

**Dùng để:** gộp các periodic snapshots từ `bureau_balance`, `POS_CASH_balance`, `credit_card_balance` vào cùng một fact.

**PK:** `monthly_balance_key`  
**FK:** `customer_key`, `loan_type_key`, `status_key`, `balance_time_key`  
**UNIQUE:** `balance_source + source_account_id + balance_time_key`

Ba nguồn được chuẩn hóa về cùng schema rồi **`UNION ALL`**, không JOIN ba bảng tháng vào nhau.

### `monthly_balance_key`

**Nguồn**

Không có trong CSV. DB/ETL sinh.

**Logic ETL + ví dụ**

Lookup theo:

```text
(balance_source, source_account_id, balance_time_key)
```

Ví dụ:

```text
(CREDIT_CARD, 5001, M:-3)
→ monthly_balance_key = 100
```

### `customer_key`

**Nguồn**

POS/Credit Card:

```text
POS_CASH_balance.SK_ID_CURR
credit_card_balance.SK_ID_CURR
```

Bureau:

```text
bureau_balance.SK_ID_BUREAU
→ JOIN bureau.SK_ID_BUREAU
→ bureau.SK_ID_CURR
```

**Logic ETL + ví dụ**

Credit Card:

```text
SK_ID_CURR = 100001
→ dim_customer.customer_key = 42
```

Bureau:

```text
bureau_balance.SK_ID_BUREAU = 7001
bureau: 7001 → SK_ID_CURR = 100001
→ dim_customer.customer_key = 42
```

Chỉ giữ lịch sử nếu customer thuộc `application_train`.

### `loan_type_key`

**Nguồn**

Bureau:

```text
bureau_balance.SK_ID_BUREAU
→ JOIN bureau
→ bureau.CREDIT_TYPE
```

POS/Credit Card:

```text
SK_ID_PREV
→ JOIN previous_application
→ NAME_CONTRACT_TYPE, NAME_PORTFOLIO, NAME_PRODUCT_TYPE, NAME_GOODS_CATEGORY
```

**Logic ETL + ví dụ**

Bureau:

```text
CREDIT_TYPE = 'Consumer credit'
→ K = (BUREAU, Not applicable, Consumer credit,
       Not applicable, Not applicable, Not applicable)
→ loan_type_key = 15
```

POS/Card:

```text
SK_ID_PREV = 5001
→ previous_application = Consumer loans / POS / x-sell / Mobile
→ loan_type_key = 7
```

### `status_key`

**Nguồn**

```text
BUREAU      → bureau_balance.STATUS
POS_CASH    → POS_CASH_balance.NAME_CONTRACT_STATUS
CREDIT_CARD → credit_card_balance.NAME_CONTRACT_STATUS
```

**Logic ETL + ví dụ**

Bureau:

```text
STATUS = 'C'
→ (BUREAU_MONTHLY, C)
→ status_key = 20
```

POS:

```text
NAME_CONTRACT_STATUS = 'Active'
→ (POS_MONTHLY, Active)
```

Credit Card:

```text
NAME_CONTRACT_STATUS = 'Active'
→ (CREDIT_CARD_MONTHLY, Active)
```

Hai code `Active` ở hai domain khác nhau vẫn là hai member khác nhau.

### `balance_time_key`

**Nguồn**

```text
bureau_balance.MONTHS_BALANCE
POS_CASH_balance.MONTHS_BALANCE
credit_card_balance.MONTHS_BALANCE
```

**Logic ETL + ví dụ**

```text
MONTHS_BALANCE = -3
→ time_code = M:-3
→ lookup dim_time
→ balance_time_key = 30
```

`M:-3` nghĩa là ba tháng trước mốc hồ sơ hiện tại.

### `balance_source`

**Nguồn**

Do ETL gán theo file:

```text
bureau_balance.csv      → BUREAU
POS_CASH_balance.csv    → POS_CASH
credit_card_balance.csv → CREDIT_CARD
```

**Logic ETL + ví dụ**

Nếu đang load một row từ `credit_card_balance.csv`:

```text
balance_source = CREDIT_CARD
```

Không suy nguồn dựa vào việc cột nào NULL hay khác NULL.

### `source_account_id`

**Nguồn**

```text
BUREAU      → bureau_balance.SK_ID_BUREAU
POS_CASH    → POS_CASH_balance.SK_ID_PREV
CREDIT_CARD → credit_card_balance.SK_ID_PREV
```

**Logic ETL + ví dụ**

```text
BUREAU + SK_ID_BUREAU=7001
→ source_account_id = 7001
```

```text
POS_CASH + SK_ID_PREV=5001
→ source_account_id = 5001
```

Cùng số `5001` ở POS và Credit Card không bị gộp vì `balance_source` khác nhau.

### `cnt_instalment`

**Nguồn**

```text
POS_CASH_balance.CNT_INSTALMENT
```

**Logic ETL + ví dụ**

```text
CNT_INSTALMENT = 12
→ cnt_instalment = 12
```

Chỉ áp dụng cho POS_CASH. Bureau/Credit Card → `NULL`.

### `cnt_instalment_future`

**Nguồn**

```text
POS_CASH_balance.CNT_INSTALMENT_FUTURE
```

**Logic ETL + ví dụ**

```text
CNT_INSTALMENT_FUTURE = 5
→ cnt_instalment_future = 5
```

Chỉ áp dụng cho POS_CASH.

### `amt_balance`

**Nguồn**

```text
credit_card_balance.AMT_BALANCE
```

**Logic ETL + ví dụ**

```text
AMT_BALANCE = 25000
→ amt_balance = 25000
```

POS/Bureau → `NULL`.

### `amt_credit_limit`

**Nguồn**

```text
credit_card_balance.AMT_CREDIT_LIMIT_ACTUAL
```

**Logic ETL + ví dụ**

```text
AMT_CREDIT_LIMIT_ACTUAL = 50000
→ amt_credit_limit = 50000
```

### `amt_drawings`

**Nguồn**

```text
credit_card_balance.AMT_DRAWINGS_CURRENT
```

**Logic ETL + ví dụ**

```text
AMT_DRAWINGS_CURRENT = 7000
→ amt_drawings = 7000
```

Đây là tiền sử dụng/rút trong tháng theo định nghĩa nguồn.

### `amt_payment`

**Nguồn**

```text
credit_card_balance.AMT_PAYMENT_TOTAL_CURRENT
```

**Logic ETL + ví dụ**

```text
AMT_PAYMENT_TOTAL_CURRENT = 5000
→ amt_payment = 5000
```

Không cộng thêm `AMT_PAYMENT_CURRENT` vào measure này.

### `amt_receivable`

**Nguồn**

```text
credit_card_balance.AMT_TOTAL_RECEIVABLE
```

**Logic ETL + ví dụ**

```text
AMT_TOTAL_RECEIVABLE = 23000
→ amt_receivable = 23000
```

POS/Bureau → `NULL`.

### `days_past_due`

**Nguồn**

```text
POS_CASH_balance.SK_DPD
credit_card_balance.SK_DPD
```

**Logic ETL + ví dụ**

```text
SK_DPD = 5
→ days_past_due = 5
```

Bureau → `NULL` vì `bureau_balance.STATUS` là nhóm trạng thái, không phải số ngày DPD chính xác.

### `days_past_due_def`

**Nguồn**

```text
POS_CASH_balance.SK_DPD_DEF
credit_card_balance.SK_DPD_DEF
```

**Logic ETL + ví dụ**

```text
SK_DPD = 8
SK_DPD_DEF = 5
→ days_past_due = 8
→ days_past_due_def = 5
```

`SK_DPD_DEF` có xét định nghĩa/ngưỡng dung sai của nguồn nên không tự coi hoàn toàn giống `SK_DPD`.

### Ví dụ một snapshot Credit Card hoàn chỉnh

Source:

```text
SK_ID_PREV = 5001
SK_ID_CURR = 100001
MONTHS_BALANCE = -3
NAME_CONTRACT_STATUS = Active
AMT_BALANCE = 25000
AMT_CREDIT_LIMIT_ACTUAL = 50000
AMT_DRAWINGS_CURRENT = 7000
AMT_PAYMENT_TOTAL_CURRENT = 5000
AMT_TOTAL_RECEIVABLE = 23000
SK_DPD = 4
SK_DPD_DEF = 2
```

Sau ETL:

```text
balance_source = CREDIT_CARD
source_account_id = 5001
customer_key = 42
loan_type_key = 7
status_key = <CREDIT_CARD_MONTHLY / Active>
balance_time_key = <M:-3>

amt_balance = 25000
amt_credit_limit = 50000
amt_drawings = 7000
amt_payment = 5000
amt_receivable = 23000
days_past_due = 4
days_past_due_def = 2

cnt_instalment = NULL
cnt_instalment_future = NULL
```

---

# 5. Quy tắc ETL chung

## 5.1. Chỉ dùng population của `application_train`

`dim_customer được` tạo từ các khách hàng có trong `application_train`. Vì vậy, dữ liệu lịch sử như đơn vay cũ, lịch sử trả góp, Bureau, POS và thẻ tín dụng chỉ được giữ nếu thuộc về những khách hàng này. Giống cái lưu ý ở trên phần dim_customer

```text
application_train
      ↓
dim_customer
      ↓
filter previous / installment / bureau / POS / card history
```

## 5.2. Unknown và Not applicable

```text
Unknown
→ thuộc tính có thể có ý nghĩa trong domain này nhưng nguồn không cung cấp/không xác định được.

Not applicable
→ thuộc tính không áp dụng cho domain đó.
```

Ví dụ:

```text
application_train không có portfolio
→ portfolio = Unknown

bureau không dùng Home Credit portfolio
→ portfolio = Not applicable
```

## 5.3. Quy tắc installment

Một kỳ có thể có nhiều payment rows. Luôn gom theo:

```text
(SK_ID_PREV, NUM_INSTALMENT_VERSION, NUM_INSTALMENT_NUMBER)
```

Sau đó:

```text
scheduled_amount → lấy một AMT_INSTALMENT nhất quán, KHÔNG SUM
paid_amount      → SUM AMT_PAYMENT hợp lệ tới ngày 0
completion_day   → ngày đầu tiên cumulative payment >= scheduled_amount
days_late        → max(completion_day - due_day, 0)
shortfall        → max(scheduled_amount - paid_amount, 0) khi đã đến hạn
```

---

# 6. Tóm tắt luồng nguồn → Data Warehouse

```text
application_train
    ├── dim_customer
    ├── dim_organization
    ├── dim_time (SLOT)
    ├── dim_loan_type (HOME_CREDIT current)
    └── fact_loan_application

previous_application
    ├── dim_loan_type
    ├── dim_status (PREVIOUS_APPLICATION)
    ├── dim_time (DAY)
    └── fact_previous_application

installments_payments
    ├── GROUP BY kỳ trả góp
    ├── dim_status (INSTALLMENT)
    ├── dim_time (due/completion DAY)
    └── fact_installment_payment

bureau + bureau_balance
    ├── dim_loan_type (BUREAU)
    ├── dim_status (BUREAU_MONTHLY)
    ├── dim_time (MONTH)
    └── fact_monthly_balance

POS_CASH_balance + previous_application
    ├── dim_loan_type
    ├── dim_status (POS_MONTHLY)
    ├── dim_time (MONTH)
    └── fact_monthly_balance

credit_card_balance + previous_application
    ├── dim_loan_type
    ├── dim_status (CREDIT_CARD_MONTHLY)
    ├── dim_time (MONTH)
    └── fact_monthly_balance
```

---


