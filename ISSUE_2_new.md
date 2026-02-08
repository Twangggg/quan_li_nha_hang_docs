# USER STORY — Quy trình Gọi món (Ordering System) (Sprint I)

## 1. Tổng quan & Bối cảnh (Overview & Context)

### Vấn đề (Problem)

Nhà hàng cần số hóa quy trình gọi món tại FOH để:

- **Waiter/Manager** gọi món trực tiếp tại bàn hoặc cho đơn Takeaway trên thiết bị di động/tablet.
- Nhân viên chọn bàn → chọn món → gửi bếp theo đúng nghiệp vụ vận hành thực tế.
- Khi bấm **"Gửi bếp"**, hệ thống tạo Order, gán trạng thái phục vụ và gửi món sang bếp.
- Waiter/Manager theo dõi, chỉnh sửa, hủy đơn/món theo rule rõ ràng.
- Có **Audit Log** để truy vết thao tác.

**Scope Sprint I:**

- Chỉ xử lý vòng đời Order ở FOH.
- Chưa triển khai KDS/bếp (`COOKING` / `READY` / `REJECTED`).

### Pain Points hiện tại

- Ghi order thủ công dễ sai món/sai số lượng.
- Dễ trùng đơn khi mạng chập chờn hoặc bấm "Gửi bếp" nhiều lần.
- Rule sửa/hủy chưa rõ → tranh cãi giữa nhân viên.
- Thiếu audit log → khó truy vết ai sửa/hủy và lý do.

### Giá trị Nghiệp vụ (Business Value)

- **Nhanh hơn:** Tạo phiếu rỗng, thêm món tại bàn nhanh.
- **Ít lỗi hơn:** Rule sửa/hủy rõ ràng + chống trùng submit.
- **Dễ vận hành:** Có màn hình quản lý order + filter dine-in/takeaway.
- **Truy vết tốt:** Log đầy đủ thao tác quan trọng.

---

## 2. Đối tượng (Actor)

- **Primary Actors:**
  - **Waiter:** Tạo order, thêm/sửa/hủy món khi `DRAFT`/`PREPARING`, submit "Gửi bếp".
  - **Manager:** Toàn quyền quản lý order (xem log, can thiệp/hủy theo rule).
- **Secondary Actors:**
  - **Cashier:** (Iteration sau) Xem order để thanh toán/in bill (read-only), không thêm món.

---

## 3. User Story Statement

> Là Waiter/Manager, tôi muốn tạo và quản lý phiếu order (tạo DRAFT rỗng, thêm/sửa/hủy món theo rule, submit “Gửi bếp”, quản lý danh sách theo trạng thái/loại đơn, xem audit log) để phục vụ khách nhanh, giảm sai sót, tránh trùng đơn và truy vết thao tác.

---

## 4. Trạng thái Order (Order Status)

### Flow chính của toàn hệ thống:

- **Order Level:** `SERVING` → `COMPLETE`
- **Order Item Level:** `PREPARING` → `COOKING` → `READY` → `COMPLETED`

### Ngoại lệ:

- **CANCELLED:** Hủy đơn khi đang `SERVING`, hủy món khi `PREPARING`.
- **REJECTED:** Bếp/bar từ chối trong lúc `COOKING` do thiếu nguyên liệu/sự cố (bắt buộc lý do). _(Trạng thái này do KDS cập nhật, FOH chỉ hiển thị)._

### **_Phạm vi Sprint I (FOH only):_**

- **Flow chính:**
  1. Chuyển trạng thái Order Item sang **PREPARING**.
  2. Chuyển trạng thái Order thành **SERVING**.
- **Ngoại lệ:**
  - **CANCELLED:** Hủy đơn/món theo rule. Nếu hủy theo đơn thì hủy cả đơn (cần lý do), hủy món thì hủy 1 món.
- **Out of scope Sprint I:** `COOKING` / `READY` / `REJECTED`.

---

## 5. Luồng Người dùng (User Flow)

### 5.1 Chọn Bàn & Bắt đầu Gọi Món (Pre-Order State – UI only)

1. Waiter/Manager vào Màn hình Quản lý Bàn.
2. Chọn 1 bàn đang trống / đang phục vụ.
3. Hệ thống mở màn hình gọi món cho bàn đó.
   - **Lưu ý:** Ở bước này **CHƯA** tạo Order trong DB. Mọi thứ chỉ ở UI (Temporary Cart).

### 5.2 Chọn Món từ Thực Đơn (UI Cart)

- Chọn món, số lượng (>0), note món.
- **Rule món trùng:** Chọn lại cùng món → tăng qty, không tạo dòng mới.
- **Lưu ý:** Chưa có OrderCode, chưa có Status, chưa có Audit Log.

### 5.3 Gửi Bếp – Tạo Order & Submit (SERVING)

- Bấm **"Gửi bếp"** → Hệ thống validate (phải có món/có bàn).
- Nếu hợp lệ, thực hiện **Atomic Transaction**:
  1. **Tạo Order:** `Status = SERVING`, `OrderCode = ORD-yyyymmdd-nnnn`.
  2. **Tạo Order Items:** `Status = PREPARING`.
  3. **Mapping:** Order là `SERVING` (FOH đang phục vụ), Item là `PREPARING` (Chờ bếp).
- **Anti-duplicate:** Đảm bảo chỉ tạo 1 Order dù bấm nhiều lần hoặc mạng lỗi.

### 5.4 Hủy đơn khi đã ở trạng thái SERVING

- Tại màn quản lý order: Chọn **Hủy phiếu (CANCELLED)**.
- **Bắt buộc** nhập lý do.

### 5.5 Sửa/Hủy khi món PREPARING

- Có thể đổi `qty` hoặc **Hủy món (CANCELLED)**.
- **Bắt buộc** nhập lý do.

### 5.6 Thêm Món vào Order đang SERVING (Add-on)

1. Mở Order đang `SERVING` → Bấm **"Thêm món"**.
2. Chọn món tại UI Cart phụ.
3. **Rule món trùng:** Khi gửi bếp, hệ thống **Tạo Order Item mới**, KHÔNG gộp vào cũ (vì khác lượt gửi bếp).
4. Bấm **"Gửi bếp"**: Tạo Item mới gán vào `order_id` cũ. Order Level vẫn là `SERVING`.

### 5.7 COMPLETED (Đã phục vụ)

- Waiter/Manager xác nhận khi đã phục vụ xong món cuối cùng.

### 5.8 Màn hình Quản lý Order (Order Board)

- Filters: Dine-in/Takeaway, Status (`SERVING`/`COMPLETED`/`CANCELLED`).
- Search: Mã order, mã bàn.
- Hiển thị: OrderCode, Bàn, Loại đơn, Thời gian, Trạng thái, Tag VIP.

### 5.9 Luồng Gọi món Takeaway

- Tương tự Dine-in nhưng **KHÔNG** chọn bàn.
- `OrderType = TAKEAWAY`, `TableId = NULL`.

---

## 6. Business Rules (BR)

- **BR-01:** Order chỉ được tạo khi bấm "Gửi bếp".
- **BR-02:** Loại đơn: `DINE-IN` / `TAKEAWAY`.
- **BR-03:** `DINE-IN` bắt buộc chọn bàn.
- **BR-04:** `OrderCode` unique theo ngày (reset mỗi ngày).
- **BR-05:** `createdBy` lấy từ user đang đăng nhập.
- **BR-06:** Order có note chung; item có note riêng.
- **BR-07:** Thêm món trùng trong lúc chọn (UI Cart) → tăng qty.
- **BR-08:** Đổi qty item `PREPARING` bắt buộc nhập lý do.
- **BR-09:** Hủy item `PREPARING` bắt buộc nhập lý do.
- **BR-10:** Hủy order `SERVING` bắt buộc nhập lý do.
- **BR-11:** Cơ chế chống submit trùng (Anti-duplicate submit).
- **BR-12:** Audit log cho: Tạo, Gửi bếp, Đổi qty, Hủy món, Hủy đơn, Complete.

---

## 7. Tiêu chí Chấp nhận (Acceptance Criteria)

- **AC-01:** Không tạo Order trước khi nhấn "Gửi bếp" (chỉ lưu UI).
- **AC-02:** Không cho gửi bếp khi giỏ hàng rỗng.
- **AC-03:** Chọn món trùng trong giỏ hàng UI phải cộng dồn số lượng.
- **AC-04:** Gửi bếp thành công: Tạo Order `SERVING`, sinh `OrderCode` đúng format.
- **AC-05:** Order Item mới tạo luôn ở trạng thái `PREPARING`.
- **AC-06:** Chống submit trùng: Gửi nhiều lần chỉ tạo 1 Order duy nhất.
- **AC-07:** Hủy đơn `SERVING` phải chuyển trạng thái sang `CANCELLED` và lưu lý do.
- **AC-08:** Chặn hành động Hủy đơn nếu thiếu lý do.
- **AC-09:** Sửa số lượng món `PREPARING` phải lưu qty mới và lý do.
- **AC-10:** Chặn hành động sửa qty nếu thiếu lý do.
- **AC-11:** Hủy món `PREPARING` chuyển trạng thái món sang `CANCELLED` và lưu lý do.
- **AC-12:** Bấm "COMPLETED" chuyển Order sang trạng thái `COMPLETED`.
- **AC-13:** Order Board hiển thị đúng theo bộ lọc và tìm kiếm.
- **AC-14:** Audit log hiển thị đầy đủ: Actor, Time, Action, Reason.
- **AC-15:** Thêm món vào đơn đang `SERVING` tạo Item mới, không gộp vào Item cũ.
- **AC-16:** Chống submit trùng khi gửi thêm món (Add-on).
- **AC-17:** Đơn Takeaway không yêu cầu bàn và vẫn hoạt động đúng quy trình.

---

## 8. Ngoài phạm vi (Out of Scope Sprint I)

- Màn hình KDS/Trạm bếp/Bar.
- Hàng đợi ưu tiên (Priority Queue) & WIP Limit.
- Trạng thái `COOKING`, `READY`, `REJECTED`.
- Quy trình thanh toán & In bill.
