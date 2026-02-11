# **USER STORY — Quy trình Gọi món (Ordering System) (Sprint I)**

## **1. Tổng quan & Bối cảnh (Overview & Context)**

### **Vấn đề (Problem)**

Nhà hàng cần số hóa quy trình gọi món để:

- Waiter/Manager tạo **phiếu order** cho **Dine-in** hoặc **Takeaway** trên thiết bị di động/tablet.

- Phiếu order được tạo rỗng (**DRAFT**) trước, sau đó mới thêm món và ghi chú.

- Khi bấm **“Gửi bếp” (Submit)**, hệ thống **chốt phiếu** và chuyển trạng thái sang **PREPARING** để chuyển sang giai đoạn bếp xử lý (sprint sau).

- Waiter/Manager có thể theo dõi danh sách order theo loại đơn/trạng thái để vận hành.

**Lưu ý scope:** Sprint II chỉ quản lý vòng đời phiếu order ở FOH. Các xử lý bếp/KDS sẽ tách user story riêng (Sprint III).

### **Pain Points hiện tại**

- Ghi order thủ công dễ sai món/sai số lượng.

- Dễ trùng đơn khi mạng chập chờn hoặc bấm “Gửi bếp” nhiều lần.

- Rule sửa/hủy chưa rõ → tranh cãi giữa nhân viên.

- Thiếu audit log → khó truy vết ai sửa/hủy và lý do.

### **Giá trị Nghiệp vụ (Business Value)**

- Nhanh hơn: tạo phiếu rỗng, thêm món tại bàn nhanh.

- Ít lỗi hơn: rule sửa/hủy rõ ràng + chống trùng submit.

- Dễ vận hành: có màn hình quản lý order + filter dine-in/takeaway.

- Truy vết tốt: log đầy đủ thao tác quan trọng.

## **2. Đối tượng (Actor)**

### **Primary Actors**

- **Waiter:** tạo order, thêm/sửa/hủy món khi DRAFT/PREPARING, submit “Gửi bếp”.

- **Manager:** toàn quyền quản lý order (xem log, can thiệp/hủy theo rule).

### **Secondary Actors**

- **Cashier:** *Iteration sau* xem order để thanh toán/in bill (read-only), **không thêm món**.

## **3. User Story Statement**

Là Waiter/Manager, tôi muốn tạo và quản lý phiếu order (tạo DRAFT rỗng, thêm/sửa/hủy món theo rule, submit “Gửi bếp”, quản lý danh sách theo trạng thái/loại đơn, xem audit log) để phục vụ khách nhanh, giảm sai sót, tránh trùng đơn và truy vết thao tác.

## **4. Trạng thái Order (Order Status)**

Flow chính của order:

- **DRAFT → PREPARING → COOKING → READY → COMPLETED**

Ngoại lệ:

- **CANCELLED:** hủy đơn khi DRAFT hoặc PREPARING.

- **REJECTED:** bếp/bar từ chối trong lúc COOKING do thiếu nguyên liệu/sự cố (bắt buộc lý do) *(trạng thái này do KDS cập nhật, FOH chỉ hiển thị và có nút “xử lý” cho Manager).*

*\*\*\*Flow chính của sprint này cần làm (FOH only):*

- ***DRAFT → PREPARING → COMPLETED***

*Ngoại lệ:*

- ***CANCELLED:*** *hủy phiếu theo rule.*

***Out of scope Sprint I:*** *COOKING / READY / REJECTED (đây là status do KDS/bếp xử lý ở sprint sau).*

## **5. Luồng Người dùng (User Flow)**

### **5.1 Tạo Order rỗng (DRAFT)**

1. Waiter/Manager bấm **“Tạo order”**.

2. Hệ thống tạo order mới với:

   - Status = **DRAFT**

   - OrderCode = **ORD-yyyymmdd-0001** (unique theo ngày)

   - createdBy = user đang đăng nhập

   - OrderItems = rỗng

3. Người dùng nhập thông tin:

   - Loại đơn: **Dine-in / Takeaway** (required)

   - Nếu Dine-in: **Chọn bàn** (required)

   - Ghi chú chung (optional)

### **5.2 Thêm món vào Order (DRAFT)**

- Item tối thiểu:

  - Món (required)

  - Số lượng (required, &gt;0)

  - Note món (optional)

- **Rule chọn món trùng:** chọn lại cùng món → **tăng qty** (không tạo dòng mới)

### **5.3 Submit “Gửi bếp” (DRAFT → PREPARING)**

1. Người dùng bấm **“Gửi bếp”**.

2. Hệ thống validate:

   - Có ít nhất 1 item

   - Nếu Dine-in phải có bàn

3. Nếu hợp lệ:

   - Status chuyển **PREPARING**

   - Hiển thị toast “Đã gửi bếp”

4. Nếu user bấm nhiều lần / mạng lỗi:

   - **Có cơ chế chống lặp submit** để **không tạo trùng phiếu** (chi tiết kỹ thuật dev tự chọn)

**Note:** PREPARING ở Sprint I chỉ mang nghĩa “đã submit/chờ bếp xử lý”, chưa có hàng đợi hay KDS.

### **5.4 Sửa/Hủy trong DRAFT**

- Waiter/Manager được:

  - Thêm/sửa qty, sửa note, xóa món

  - Hủy phiếu (CANCELLED) *(khuyến nghị: hủy DRAFT có thể không cần lý do)*

### **5.5 Sửa/Hủy khi PREPARING (có lý do)**

- Khi order **PREPARING**, Waiter/Manager có thể:

  - Đổi qty

  - Hủy món

  - Hủy phiếu (CANCELLED)

- **Bắt buộc nhập lý do** cho:

  - Đổi qty

  - Hủy món

  - Hủy phiếu trong PREPARING

**Rule khóa thay đổi:**

- Sprint I **chưa có COOKING** nên chưa khóa theo COOKING; việc “khóa khi COOKING” sẽ thuộc sprint KDS.

### **5.6 COMPLETED (đã phục vụ)**

- Waiter/Manager bấm **COMPLETED** khi đã phục vụ xong (FOH xác nhận).

**Note:** Nếu bạn muốn COMPLETED chỉ xảy ra sau khi bếp READY (KDS), thì COMPLETED cũng nên để Sprint III. Nhưng theo bạn nói “COMPLETED là đã phục vụ”, Sprint II vẫn làm được.

### **5.7 Màn hình Quản lý Order (Order Board)**

Cần màn tổng hợp giống ảnh bạn gửi (để vận hành):

- Filters:

  - **Dine-in / Takeaway**

- Lọc theo trạng thái:

  - DRAFT / PREPARING / COMPLETED / CANCELLED

- Search:

  - theo mã order, mã bàn (nếu có)

- Mỗi card/dòng hiển thị tối thiểu:

  - OrderCode

  - Bàn (nếu dine-in)

  - Loại đơn (dine-in/takeaway)

  - Thời gian tạo

  - Trạng thái

  - Tag **VIP/Đặt trước** (nếu có)

- Action:

  - Waiter: xem chi tiết, sửa/hủy theo rule

  - Manager: toàn quyền + xem audit log

## **6. Business Rules (Sprint II)**

- **BR-ORDER-01:** Order tạo mới luôn ở **DRAFT rỗng**.

- **BR-ORDER-02:** Loại đơn: **DINE-IN / TAKEAWAY**.

- **BR-ORDER-03:** DINE-IN bắt buộc chọn **bàn** trước khi “Gửi bếp”.

- **BR-ORDER-04:** OrderCode format **ORD-yyyymmdd-0001**, unique theo ngày.

- **BR-ORDER-05:** createdBy auto theo tài khoản đăng nhập.

- **BR-ORDER-06:** Order có **note chung**; mỗi item có **note riêng**.

- **BR-ORDER-07:** Add món trùng → tăng qty.

- **BR-ORDER-08:** Đổi qty **bắt buộc lý do** (cả tăng/giảm).

- **BR-ORDER-09:** Hủy món **bắt buộc lý do** (áp dụng khi PREPARING, DRAFT thì k cần).

- **BR-ORDER-10:** “Gửi bếp” chuyển trạng thái **DRAFT → PREPARING**.

- **BR-ORDER-11:** Anti-duplicate: bấm gửi nhiều lần không tạo trùng phiếu (dev triển khai).

- **BR-ORDER-12 (Audit):** log các action: tạo, thêm món, đổi qty (kèm reason), hủy món (kèm reason), gửi bếp, cancel, complete.

## **7. Tiêu chí Chấp nhận (Acceptance Criteria)**

**AC-01: Tạo order DRAFT rỗng**\
Given user đã login\
When bấm “Tạo order”\
Then tạo order DRAFT rỗng + sinh OrderCode + createdBy đúng

**AC-02: Không cho gửi bếp khi chưa có món**\
Given order DRAFT items rỗng\
When bấm “Gửi bếp”\
Then báo lỗi + không chuyển PREPARING

**AC-03: Món trùng tăng qty**\
Given có món A qty=1\
When add lại món A\
Then qty tăng + không tạo dòng mới

**AC-04: Gửi bếp thành công**\
Given order DRAFT có ít nhất 1 item và đủ field required\
When bấm “Gửi bếp”\
Then order chuyển PREPARING + toast thành công

**AC-05: Chống submit trùng**\
Given user bấm “Gửi bếp” nhiều lần / retry do mạng\
When server nhận request lặp\
Then chỉ ghi nhận 1 lần hợp lệ, không tạo trùng

**AC-06: Sửa/hủy trong DRAFT**\
Given order DRAFT\
When sửa item/note/hủy phiếu\
Then lưu thành công theo rule

**AC-07: Sửa/hủy trong PREPARING bắt buộc lý do**\
Given order PREPARING\
When đổi qty/hủy món/hủy phiếu\
Then yêu cầu reason; thiếu reason → không cho lưu

**AC-08: Order Board lọc dine-in/takeaway + trạng thái**\
Given có nhiều order\
When lọc loại đơn/trạng thái/search\
Then danh sách hiển thị đúng, không lag rõ rệt

**AC-09: Audit log**\
Given có thao tác add/sửa/cancel/submit\
When mở lịch sử\
Then hiển thị actor/time/action/reason

## **8. Out of scope Sprint I**

- KDS screens (bếp nóng/lạnh/bar), station routing hiển thị trên KDS

- Queue ưu tiên + WIP limit (4 đơn)

- Status COOKING / READY / REJECTED và các rule liên quan

- Cashier thanh toán/in bill
ORDERS
Lưu đơn hàng (phiếu order) – “khung” của một order.

ID
Column
Key
Type
Description
1
order_id
PK
UUID
Khóa chính định danh đơn hàng
2
order_code
UQ
VARCHAR(30)
Mã đơn (ORD-YYYYMMDD-XXXX), duy nhất
3
order_type


INT (DINE_IN, TAKEAWAY)
Loại đơn: DINE_IN, TAKEAWAY
4
status


INT (DRAFT, PREPARING, COMPLETED, CANCELLED)
Trạng thái: DRAFT → PREPARING → COOKING → READY → COMPLETED
5
table_id
FK
UUID
Định danh bàn (Bắt buộc nếu DINE_IN)
6
note


TEXT


7
total_amount


DECIMAL(15,2)
Tổng giá trị đơn hàng
8
created_by
FK
UUID)
ID nhân viên tạo đơn (employee_id)
9
created_at


TIMESTAMP
Thời điểm tạo phiếu rỗng
10
updated_at


TIMESTAMP
Thời điểm cập nhật trạng thái cuối cùng
11
submitted_at


TIMESTAMP
Khi “Gửi bếp”
12
completed_at


TIMESTAMP
Khi hoàn tất
13
cancelled_at


TIMESTAMP
Khi hủy
14
transaction_id


UUID
Chống trùng đơn


ORDER_ITEMS
Lưu các dòng món ăn trong một đơn (mỗi món gọi = 1 dòng, có số lượng).
ID
Column
Key
Type
Description
1
order_item_id
PK
UUID
Khóa chính định danh dòng món ăn
2
order_id
FK
UUID
Liên kết tới bảng orders
3
menu_item_id
FK
UUID
Định danh món ăn từ menu
4
item_code_snapshot


VARCHAR(50)
Mã món tại thời điểm order (Snapshot để các attribute ko bị thay đổi khi menu đổi)
5
item_name_snapshot


VARCHAR(150)
Tên món tại thời điểm order
6
station_snapshot


VARCHAR(20)
Station tại thời điểm order
7
quantity


INT
Số lượng món (phải > 0)
8
unit_price_snapshot


DECIMAL(15,2)
Giá món tại thời điểm gọi đơn
9
item_note


VARCHAR(255)
Ghi chú riêng cho món (ví dụ: ít cay)
10
created_at


TIMESTAMP
Thời điểm thêm món vào đơn
11
updated_at


TIMESTAMP
Thời điểm cập nhật số lượng/ghi chú


ORDER_ITEM_OPTION
Lưu tùy chọn (option) mà khách chọn cho từng dòng món: topping, đường đá, độ cay, ghi chú tự do…
ID
Column
Key
Type
Description
1
order_item_option_id
PK
VARCHAR(100)
Khóa chính option của dòng order
2
order_item_id
FK
VARCHAR(100)
Thuộc dòng order_item nào
3
option_group_name_snapshot


VARCHAR(100)
Tên group (snapshot)
4
option_item_label_snapshot


VARCHAR(100)
Nhãn option (snapshot)
5
option_value_snapshot


VARCHAR(50)
Giá trị option (snapshot)
6
extra_price_snapshot


NUMERIC(12,2)
Giá cộng thêm (snapshot)
7
free_text_value


VARCHAR(500)
Ghi chú tự do (nếu type FREE_TEXT)
8
created_at


TIMESTAMP
Thời điểm tạo


ORDER_AUDIT_LOGS
Lưu log thao tác trên order (phục vụ kiểm soát & truy vết).
ID
Column
Key
Type
Description
1
log_id
PK
UUID
Khóa chính định danh bản ghi log
2
order_id
FK
UUID
Đơn hàng bị tác động
3
employee_id
FK
UUID
Người thực hiện thao tác
4
action


VARCHAR(50)
Hành động: CREATE, ADD_ITEM, UPDATE_QTY, SUBMIT, CANCEL
5
old_value


JSONB
Giá trị trước khi sửa (JSON format)
6
new_value


JSONB
Giá trị sau khi sửa (JSON format)
7
change_reason


TEXT
Lý do thay đổi (Bắt buộc khi PREPARING)
8
created_at


TIMESTAMP
Thời điểm phát sinh thao tác




CATEGORY
Lưu danh mục món để phân loại menu và filter hiển thị (Khai vị/Món chính/Đồ uống…).
ID
Column
Key
Type
Description
1
category_id
PK
VARCHAR(100)
Khóa chính danh mục
2
name
UQ
VARCHAR(100)
Tên danh mục (Khai vị, Món chính, Đồ uống, Tráng miệng, …)
3
category_type


VARCHAR(30)
Loại danh mục: NORMAL / SPECIAL_GROUP (Combo, Set Morning, Set Lunch)
4
sort_order


INT
Thứ tự hiển thị
5
is_active


BOOLEAN
Trạng thái danh mục
6
created_at


TIMESTAMP
Thời điểm tạo
7
updated_at


TIMESTAMP
Thời điểm cập nhật gần nhất
8
deleted_at


TIMESTAMP
Thời điểm xóa mềm

