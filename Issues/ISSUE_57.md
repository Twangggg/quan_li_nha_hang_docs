# US-INV-00: Thiết lập Hệ thống Kho & Nhập tồn đầu kỳ (8 Points)

---

# 1. Tổng quan & Bối cảnh (Overview & Context)

## Vấn đề (Problem)

Trước khi hệ thống quản lý kho được sử dụng trong vận hành thực tế, cần có bước thiết lập ban đầu để:

- Cấu hình các thông số chung của hệ thống kho như:
  - Số ngày cảnh báo hết hạn

  - Ngưỡng cảnh báo sắp hết

  - Cơ chế tự động trừ kho khi bán

  - Phương pháp tính giá vốn

- Nhập số dư tồn kho ban đầu cho các nguyên vật liệu (NVL) đã có sẵn ngoài thực tế.

Nếu không có bước thiết lập này:

- Hệ thống không biết mức cảnh báo tồn kho

- Không thể tính giá vốn chính xác

- Kho sẽ bắt đầu với số lượng tồn bằng 0, sai lệch với thực tế.

## Pain Points hiện tại

- Hệ thống chưa có màn hình cấu hình kho tập trung.

- Không có nơi nhập tồn kho ban đầu khi bắt đầu sử dụng hệ thống.

- Giá vốn NVL ban đầu không được thiết lập dẫn đến sai lệch khi tính giá vốn bán hàng.

- Không thể bật/tắt cơ chế tự động trừ kho khi hoàn thành đơn hàng.

## Giá trị Nghiệp vụ (Business Value)

### 1. Chuẩn hóa cấu hình kho

Cho phép Manager thiết lập các thông số kho ngay từ đầu để hệ thống hoạt động đúng logic nghiệp vụ.

### 2. Đồng bộ tồn kho thực tế với hệ thống

Nhập số dư tồn kho ban đầu giúp hệ thống phản ánh chính xác số lượng NVL hiện có.

### 3. Đảm bảo tính giá vốn chính xác

Việc nhập đơn giá ban đầu giúp hệ thống tính giá vốn theo Bình quân gia quyền chính xác ngay từ đầu.

### 4. Kiểm soát vận hành kho

Cho phép bật/tắt tự động trừ kho khi bán giúp linh hoạt theo quy trình vận hành của từng cửa hàng.

## Đối tượng (Actor)

### Primary Actor

Manager

- Thiết lập cấu hình hệ thống kho

- Nhập tồn kho ban đầu cho nguyên vật liệu

- Kiểm soát cơ chế tự động trừ kho

### Secondary Actors

System

- Lưu cấu hình kho

- Tạo transaction tồn kho đầu kỳ

- Cập nhật số lượng tồn NVL

User Story Statement

Là Manager, tôi muốn thiết lập các thông số hệ thống kho và nhập tồn kho ban đầu cho nguyên vật liệu, để hệ thống sẵn sàng vận hành kho trước khi bắt đầu sử dụng.

## Tiền điều kiện (Preconditions)

- US-INV-01 đã hoàn thành.

- Danh mục Ingredient (Nguyên vật liệu) đã tồn tại trong hệ thống.

- Tồn kho ban đầu của tất cả NVL = 0.

## Entity liên quan

- InventorySettings

- Ingredient

- InventoryTransaction

# 2. Luồng Người dùng (User Flow)

## 2.1 Luồng: Thiết lập cấu hình kho

1. Manager đăng nhập vào hệ thống

2. Điều hướng đến trang:

Thiết lập hệ thống → Cấu hình Kho

/settings/inventory

3. Hệ thống hiển thị form cấu hình kho gồm:

- Số ngày cảnh báo trước hạn sử dụng
  - ExpiryWarningDays

  - Default: 7

- Ngưỡng cảnh báo sắp hết mặc định
  - DefaultLowStockThreshold

  - Default: 0

  - Áp dụng khi tạo NVL mới

- Tự động trừ kho khi bán
  - AutoDeductOnCompleted

  - Bật / Tắt

  - Default: Bật

- Phương pháp tính giá vốn
  - CostMethod

  - Giá trị: _Bình quân gia quyền_

  - Read-only

- Số ngày tối đa cho phép tính lại giá vốn
  - MaxCostRecalcDays

  - Default: 31

4. Manager chỉnh sửa các giá trị cần thiết

5. Manager bấm "Lưu"

6. Hệ thống thực hiện validation:

- ExpiryWarningDays ≥ 1

- DefaultLowStockThreshold ≥ 0

- MaxCostRecalcDays từ 1 đến 365

7. Nếu hợp lệ:

- Hệ thống lưu cấu hình

- Hiển thị toast:

"Cập nhật cấu hình kho thành công"

---

### Lưu ý

- Bảng InventorySettings chỉ có 1 record duy nhất (singleton).

- Nếu chưa có record → hệ thống tự tạo với giá trị mặc định.

- Thay đổi ExpiryWarningDays sẽ ảnh hưởng ngay đến danh sách cảnh báo hết hạn.

- Nếu AutoDeductOnCompleted bị tắt:
  - Hệ thống không tự trừ kho khi hoàn thành đơn hàng

  - Manager phải tạo Phiếu Xuất kho thủ công.

## 2.2 Luồng: Nhập tồn kho ban đầu (Opening Stock)

1. Manager vào trang:

Thiết lập hệ thống → Nhập số dư ban đầu

/settings/opening-stock

2. Hệ thống hiển thị bảng danh sách NVL (chỉ NVL đang Active).

Mỗi dòng gồm:

- Mã NVL (read-only)

- Tên NVL (read-only)

- Đơn vị (read-only)

- Số lượng tồn ban đầu (input, ≥ 0)

- Đơn giá nhập (input, ≥ 0, optional)

- Thành tiền

Thành tiền = Số lượng × Đơn giá

(read-only)

3. Manager nhập số liệu cho các NVL cần thiết lập tồn kho.

4. Manager bấm "Lưu"

5. Hệ thống validate:

- Số lượng ≥ 0

- Đơn giá ≥ 0 (nếu nhập)

6. Nếu hợp lệ, hệ thống thực hiện:

- Cập nhật

Ingredient.CurrentStock

- Cập nhật

Ingredient.CostPrice

(nếu có đơn giá)

- Ghi InventoryTransaction

Type: OPENING_STOCK

Reference: null

cho mỗi NVL có số lượng &gt; 0.

7. Hệ thống hiển thị toast:

"Nhập số dư ban đầu thành công"

---

### Lưu ý

- Chức năng này dùng khi:
  - Bắt đầu sử dụng hệ thống lần đầu

  - Hoặc cần thiết lập lại tồn kho.

- Nếu NVL đã có tồn &gt; 0:
  - Hệ thống vẫn cho phép sửa

  - Nhưng hiển thị cảnh báo xác nhận.

- Sau khi thiết lập tồn đầu kỳ, mọi biến động kho sẽ được thực hiện qua:

Phiếu Nhập kho (US-INV-02)

Phiếu Xuất kho (US-INV-03)

# 3. Tiêu chí Chấp nhận (Acceptance Criteria)

## AC-01: Hiển thị cấu hình kho

Given

Manager đã đăng nhập

When

Truy cập trang

/settings/inventory

Then

- Hệ thống hiển thị form cấu hình kho

- Hiển thị giá trị hiện tại của cấu hình

- Nếu chưa có cấu hình → hiển thị giá trị mặc định.

---

## AC-02: Lưu cấu hình kho thành công

Given

Manager thay đổi:

ExpiryWarningDays = 14

DefaultLowStockThreshold = 100

When

Manager bấm "Lưu"

Then

- Cấu hình được lưu thành công

- Hiển thị thông báo:

"Cập nhật cấu hình kho thành công"

- Hệ thống sử dụng 14 ngày để tính cảnh báo hết hạn.

---

## AC-03: Validate cấu hình kho

Given

Manager nhập:

ExpiryWarningDays = 0

When

Bấm "Lưu"

Then

Hệ thống hiển thị lỗi:

"Số ngày cảnh báo phải ≥ 1"

---

## AC-04: Nhập tồn kho đầu kỳ

Given

NVL "Thịt bò" đã tồn tại trong hệ thống\
CurrentStock = 0

When

Manager nhập:

Thịt bò = 5000g

Đơn giá = 200.000đ/kg

và bấm "Lưu"

Then

- Ingredient.CurrentStock = 5000g

- Ingredient.CostPrice = 200.000đ/kg

- Hệ thống tạo

InventoryTransaction

Type = OPENING_STOCK

- Hiển thị toast:

"Nhập số dư ban đầu thành công"

---

## AC-05: Nhập tồn khi NVL đã có tồn

Given

NVL Thịt bò hiện có:

CurrentStock = 5000g

When

Manager mở màn Nhập tồn đầu kỳ

Then

- Hiển thị số lượng hiện tại = 5000g

- Nếu Manager chỉnh sửa số lượng:

Hệ thống hiển thị cảnh báo:

"NVL này đã có tồn kho. Bạn có chắc muốn ghi đè?"

---

## AC-06: Tắt tự động trừ kho

Given

Manager tắt cấu hình:

AutoDeductOnCompleted = OFF

When

OrderItem chuyển sang trạng thái:

Completed

Then

- Hệ thống không tự động trừ kho

- Manager phải tạo Phiếu Xuất kho thủ công.
