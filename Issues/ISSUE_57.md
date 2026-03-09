## **1. Tổng quan & Bối cảnh (Overview & Context)**

### **Vấn đề (Problem)**

Nhà hàng cần quản lý kho nguyên liệu và định lượng theo món để:

- Thiết lập **định lượng nguyên liệu (Recipe/BOM)** cho từng món ăn/đồ uống.

- **Tự động trừ kho** theo định lượng khi món được bán (order/hoá đơn).

- **Cảnh báo** nguyên liệu sắp hết hoặc sắp hết hạn để tránh “cháy nguyên liệu”, giảm lãng phí và gián đoạn vận hành.

### **Pain Points hiện tại**

- Quản lý kho thủ công → lệch tồn kho, khó biết còn đủ nguyên liệu để bán.

- Không có định lượng chuẩn → khó kiểm soát cost, dễ thất thoát.

- Không có cảnh báo hạn dùng → dễ hư hỏng, lãng phí, rủi ro an toàn thực phẩm.

- Khi hết nguyên liệu đột ngột → bếp/bar phải REJECTED món/đơn gây trải nghiệm xấu.

### **Giá trị Nghiệp vụ (Business Value)**

- **Tồn kho chính xác hơn** nhờ auto trừ theo bán hàng.

- **Giảm gián đoạn**: cảnh báo sớm để nhập thêm hoặc điều chỉnh menu.

- **Tối ưu chi phí**: định lượng chuẩn giúp kiểm soát cost và thất thoát.

- **Nâng chất lượng vận hành**: giảm tình trạng hết nguyên liệu đột ngột khi đang COOKING.

## **2. Đối tượng (Actor)**

### **Primary Actors**

- **Manager**: cấu hình kho, tạo/sửa nguyên liệu, set recipe, xem báo cáo tồn kho/cảnh báo.

- **Chef/Bar** _(tùy quyền)_: có thể xem tồn kho/cảnh báo để chủ động chuẩn bị.

### **Secondary Actors**

- **Cashier/Waiter**: không thao tác kho; hệ thống tự trừ kho dựa trên bán hàng.

## **3. User Story Statement**

Là Manager, tôi muốn thiết lập định lượng nguyên liệu theo từng món (recipe) và hệ thống tự động trừ kho khi bán, đồng thời cảnh báo sắp hết/hết hạn, để đảm bảo vận hành liên tục, giảm lãng phí và kiểm soát chi phí tốt hơn.

## **4. Luồng Người dùng (User Flow)**

### **4.1. Quản lý Nguyên liệu (Ingredient Master)**

1. Manager vào màn **Inventory/Ingredients** (/inventory/ingredients).

2. Danh sách nguyên liệu hiển thị:
   - Tên nguyên liệu

   - Đơn vị (g, kg, ml, l, cái…)

   - Tồn hiện tại (on-hand)

   - Ngưỡng cảnh báo sắp hết (low stock threshold)

   - Trạng thái (Active/Inactive)

3. Manager có thể: **tạo/sửa/xem/ẩn (inactive)** nguyên liệu.

**Form tạo/sửa nguyên liệu** gồm:

- Tên nguyên liệu (required, unique)

- Đơn vị (required)

- Ngưỡng cảnh báo sắp hết (required)

- (Optional) Mô tả/nhà cung cấp… **\[TBD\]**

### **4.2. Nhập kho theo lô (Stock In / Batch)**

1. Manager vào **Stock In** (/inventory/stock-in) → bấm “+ Nhập kho”.

2. Chọn nguyên liệu + nhập thông tin:
   - Số lượng nhập (required, &gt;0)

   - Giá nhập (optional)

   - Ngày nhập (default today)

   - Hạn sử dụng (expiry date) (optional/required tùy nguyên liệu)

   - Số lô (batch code) (optional)

3. Lưu → tăng tồn kho, lưu theo **batch** để theo dõi hạn dùng.

### **4.3. Thiết lập Recipe định lượng theo món (Recipe/BOM)**

1. Manager vào màn **Menu Item Recipe** (/menu/{itemId}/recipe hoặc /inventory/recipes).

2. Chọn món → danh sách nguyên liệu cấu thành (recipe lines).

3. Bấm “+ Thêm nguyên liệu”:
   - Ingredient (required)

   - Quantity per serving (required, &gt;0)

   - Unit (auto theo nguyên liệu; nếu cho phép quy đổi thì

4. Rule:
   - Một nguyên liệu chỉ xuất hiện 1 lần trong recipe (trùng → cộng dồn hoặc báo lỗi).

5. Lưu recipe → áp dụng cho auto trừ kho.

### **4.4. Auto trừ kho khi bán (Auto Deduct)**

**Trigger đề xuất:** khi order **COMPLETED** (đã phục vụ).

Trừ kho khi **order COMPLETED** (phù hợp nếu “bán = phục vụ”)

Flow:

1. Hệ thống nhận event “món được bán” (Completed/Paid).

2. Với mỗi món trong phiếu:
   - Lấy recipe → tính tổng nguyên liệu cần trừ = quantity món \* quantity recipe

3. Trừ tồn kho:
   - Nếu có batch: trừ theo nguyên tắc **FEFO** (hết hạn trước trừ trước)

   - Nếu không batch: trừ vào tồn tổng

4. Ghi log kho (inventory ledger):
   - time, item/orderCode, ingredient, qty deducted, actor/system, reason=SALE.

### **4.5. Cảnh báo sắp hết / hết hạn**

1. Manager vào màn **Inventory Alerts** (/inventory/alerts) hoặc thấy badge trên sidebar.

2. Hệ thống hiển thị 2 nhóm cảnh báo:
   - **Low stock**: tồn &lt;= threshold

   - **Expiring soon / Expired**: lô sắp hết hạn hoặc đã hết hạn

3. Rule cảnh báo:
   - Low stock: khi on-hand &lt;= threshold

   - Expiring soon: hết hạn trong X ngày **\[TBD: ví dụ 3/7/14 ngày\]**

4. Action:
   - Xem chi tiết nguyên liệu/lô

   - (Optional) tạo “đề nghị nhập” **\[TBD\]**

   - (Optional) đánh dấu “đã xử lý” **\[TBD\]**

## **5. Business Rules**

- **BR-INV-01:** Nguyên liệu có đơn vị chuẩn; recipe dùng đúng đơn vị hoặc có quy đổi.

- **BR-INV-02:** Recipe gắn theo từng món; một nguyên liệu không được trùng dòng trong cùng recipe.

- **BR-INV-03:** Auto trừ kho theo sự kiện bán (COMPLETED).

- **BR-INV-04:** Khi trừ kho, hệ thống ghi inventory log (ledger) để audit.

- **BR-INV-05:** Cảnh báo low stock khi tồn &lt;= threshold.

- **BR-INV-06:** Cảnh báo hết hạn theo batch expiry; ngưỡng “sắp hết hạn” là X ngày.

- **BR-INV-07:** Quyền: chỉ Manager được CRUD nguyên liệu/recipe/stock-in; role khác view-only hoặc không truy cập.

- **BR-INV-08 (TBD):** Nếu tồn không đủ khi bán thì xử lý thế nào?
  - (A) Cho trừ âm (negative stock) và cảnh báo

  - (B) Chặn bán món (liên quan module Menu/Out-of-stock)

  - (C) Cho bán nhưng flag “mismatch” để Manager xử lý

---

## **6. Tiêu chí Chấp nhận (Acceptance Criteria)**

- **AC-INV-01: Tạo nguyên liệu**
  - Given: Manager đăng nhập

  - When: tạo ingredient với tên unique + unit + threshold hợp lệ

  - Then: nguyên liệu được tạo và hiển thị trong danh sách.

- **AC-INV-02: Nhập kho theo lô**
  - Given: đã có nguyên liệu

  - When: nhập qty &gt; 0 và lưu (kèm expiry nếu có)

  - Then: tồn kho tăng đúng, lưu log stock-in.

- **AC-INV-03: Thiết lập recipe cho món**
  - Given: món tồn tại trong Menu

  - When: thêm các ingredient + qty hợp lệ và lưu

  - Then: recipe được lưu, không có ingredient trùng.

- **AC-INV-04: Auto trừ kho khi bán**
  - Given: món có recipe, tồn kho đủ

  - When: order/bill đạt trạng thái trigger (COMPLETED/PAID)

  - Then: hệ thống trừ đúng số lượng nguyên liệu, ghi log.

- **AC-INV-05: Cảnh báo sắp hết**
  - Given: tồn &lt;= threshold

  - When: Manager xem Alerts

  - Then: nguyên liệu xuất hiện trong danh sách low stock.

- **AC-INV-06: Cảnh báo sắp hết hạn**
  - Given: batch có expiry trong X ngày

  - When: Manager xem Alerts

  - Then: batch xuất hiện trong danh sách expiring soon/expired.

- **AC-INV-07: Phân quyền**
  - Given: user không phải Manager

  - When: truy cập màn Inventory/Recipe CRUD hoặc gọi API

  - Then: bị chặn 403 (hoặc chỉ read-only theo rule).
