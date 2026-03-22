# ISSUE 82 - FE Screens And Actions Needed

## Muc tieu

Tai lieu nay liet ke cac man hinh, nut, form, bang, badge, va action ma FE can co de hien thi day du nhung gi BE da implement cho ISSUE 82.

Scope duoc viet theo nhung BE endpoint/flow da co:

- Recalculate COGS theo ky
- Inventory alerts + badge
- Lot dispose
- Lot-aware stock in / stock out / reverse

## 1. Sidebar / Navigation

FE can bo sung cac muc dieu huong sau trong khu vuc kho:

- `Bao cao ton kho`
  - link toi man hinh bao cao ton va so cai ton kho
- `Tinh gia xuat kho`
  - route de xai recalculate COGS
- `Canh bao kho`
  - route de xem low stock / out of stock / near expiry / expired
- `Lo ton kho`
  - route de xem va thao tac voi lot, toi thieu la vao dispose lot

FE can co `badge tong canh bao` tren icon / menu kho.

Nguon du lieu:

- `GET /api/v{version}/inventory/alerts/badge`

Hien thi:

- badge an khi = 0
- badge hien tong so canh bao khi > 0

Refresh / invalidate:

- sau stock in
- sau stock out
- sau reverse stock in
- sau reverse stock out
- sau dispose lot
- sau recalculate COGS thi khong bat buoc doi badge, nhung co the refetch inventory pages

## 2. Man hinh Tinh Gia Xuat Kho

De xuat route:

- `/inventory/cogs`

Nguon BE:

- `POST /api/v{version}/inventory/cogs/recalculate`

### 2.1. Form can co

- `Tu ngay`
- `Den ngay`
- `Nguyen lieu`
  - dropdown
  - co option `Tat ca`
- nut `Tinh gia`
- nut `Dat lai`

### 2.2. Validation FE can co

- `Den ngay >= Tu ngay`
- khoang ngay toi da `31 ngay`
- disable submit khi form invalid
- show inline validation truoc khi goi API

### 2.3. Ket qua can hien thi sau khi thanh cong

- `Tu ngay`
- `Den ngay`
- `So nguyen lieu duoc xu ly`
- `So phieu xuat duoc cap nhat`
- `So dong xuat duoc cap nhat`
- `Tong gia tri dieu chinh`
- message thanh cong

### 2.4. UX can co

- loading state tren nut `Tinh gia`
- confirm dialog neu user bam tinh gia
  - text de xuat: "He thong se tinh lai gia von xuat kho trong ky da chon. Ban co muon tiep tuc?"
- toast thanh cong
- toast loi khi API fail

### 2.5. Empty state

Neu ket qua tra ve `UpdatedItems = 0`:

- van hien message thanh cong
- hien them dong mo ta: `Khong co phieu xuat nao trong ky can cap nhat`

## 3. Man hinh Canh Bao Kho

De xuat route:

- `/inventory/alerts`

Nguon BE:

- `GET /api/v{version}/inventory/alerts`

### 3.1. Layout tong

Can co 2 tab chinh:

- `Sap het / Het hang`
- `Sap het han / Da het han`

Va 1 khu summary nho o dau trang:

- tong badge count
- so nguyen lieu het hang
- so nguyen lieu sap het
- so lo da het han
- so lo sap het han

### 3.2. Tab Sap het / Het hang

Can co bang / list hien:

- `Ma NVL`
- `Ten NVL`
- `Ton hien tai`
- `Nguong canh bao`
- `Don vi`
- optional: nut `Xem chi tiet`

Can sap xep:

- `Out of stock` len tren
- sau do `Low stock`
- trong low stock thi sap xep theo `CurrentStock` tang dan

Hien thi visual:

- badge `Het hang`
- badge `Sap het`

### 3.3. Tab Sap het han / Da het han

Can co bang / list hien:

- `Ma NVL`
- `Ten NVL`
- `Ma lo`
- `Han su dung`
- `So ngay con lai`
- `So luong con lai`
- `Don vi`
- `Trang thai`
- action `Huy lo`

Can sap xep:

- `Expired` len tren
- sau do `Near expiry`
- trong moi nhom, sap theo `ExpiryDate` som nhat truoc

Hien thi visual:

- badge `Da het han`
- badge `Sap het han`

### 3.4. Action tu tab lot expiry

FE can co nut / menu action:

- `Huy lo`

Khi bam:

- mo modal nhap:
  - `So luong huy`
  - `Ly do`
- submit len endpoint dispose lot

## 4. Modal / Drawer Huy Lo

Nguon BE:

- `POST /api/v{version}/inventory/lots/{id}/dispose`

### 4.1. Field can co

- read-only:
  - `Ten NVL`
  - `Ma lo`
  - `Han su dung`
  - `So luong con lai`
- editable:
  - `So luong huy`
  - `Ly do`

### 4.2. Validation FE can co

- `So luong huy > 0`
- `So luong huy <= so luong con lai`
- `Ly do` required
- `Ly do <= 500 ky tu`

### 4.3. Sau khi thanh cong

- dong modal
- toast thanh cong
- refetch:
  - alerts list
  - badge
  - neu dang o trang lot detail thi refetch lot detail

## 5. Man hinh Lo Ton Kho

BE hien tai moi co dispose endpoint, chua co list/detail endpoint rieng.

Vi vay FE nen chuan bi man hinh nay theo 2 muc:

### 5.1. Muc can co ngay

- link tu tab expiry alert vao modal `Huy lo`
- hien thong tin lot ngay tren man hinh alerts

### 5.2. Muc nen co de san sang cho phase tiep theo

De xuat route:

- `/inventory/lots`
- `/inventory/lots/:id`

UI de san:

- filter theo ingredient
- filter theo status
- filter theo expiry range
- list cot:
  - `Ten NVL`
  - `Ma lo`
  - `Ngay nhap`
  - `Han su dung`
  - `So luong goc`
  - `So luong con lai`
  - `Trang thai`
- detail panel:
  - movement history
  - nut `Huy lo`

## 6. Man hinh Nhap Kho

Route hien co FE dang co tuong ung voi stock-in.

Voi nhung gi BE da them, FE can dam bao form nhap kho co cac field nay o tung dong:

- `Nguyen lieu`
- `So luong`
- `Don vi`
- `Don gia nhap`
- `Han su dung`
- `Ma lo`

### 6.1. Validation / UX

- `Ma lo` optional, neu trong thi BE se tu sinh
- `Han su dung` optional
- `Don gia nhap >= 0`
- co hint:
  - "Neu khong nhap ma lo, he thong se tu tao ma lo"

### 6.2. Sau khi tao phieu nhap

FE nen:

- refresh inventory report / stock screens
- refresh alerts
- refresh badge

## 7. Man hinh Xuat Kho

Route hien co FE dang co tuong ung voi stock-out.

BE da xu ly FEFO o backend, nen FE khong can cho user chon lot o phase nay.

### 7.1. Form toi thieu

- `Ngay xuat`
- `Ly do`
- danh sach dong:
  - `Nguyen lieu`
  - `So luong`
  - optional `Don gia`

### 7.2. UX can co

- thong bao ro:
  - "He thong se tu dong chon lo theo FEFO"
- khi loi vi lot het han hoac khong du ton kha dung:
  - hien error API ro rang gan form

### 7.3. Sau khi tao phieu xuat

FE nen:

- refresh stock screens
- refresh alerts
- refresh badge

## 8. Reverse Phieu Nhap / Phieu Xuat

Voi logic lot moi, FE can tiep tuc giu action reverse cho:

- phieu nhap
- phieu xuat

Nhung can them canh bao UI:

- reverse co the fail neu da co bien dong tiep theo tren lot / inventory

De xuat confirm text:

- reverse phieu nhap:
  - "Phieu nhap chi co the dao khi lot chua duoc xuat. Ban co chac chan?"
- reverse phieu xuat:
  - "So luong se duoc hoan ve dung cac lo da tru truoc do. Ban co chac chan?"

Sau khi reverse thanh cong:

- refresh badge
- refresh alerts
- refresh danh sach phieu

## 9. Bao cao Ton Kho / So Cai Ton Kho

Man hinh bao cao ton kho va so cai ton kho hien co nen duoc cap nhat nhe o FE:

### 9.1. Bao cao ton kho

- tiep tuc hien:
  - `AverageUnitCost`
  - `ClosingStockValue`
- can co link / CTA sang man hinh `Tinh gia xuat kho`

### 9.2. So cai ton kho

- nen hien ro transaction type
- neu sau nay mo rong lot detail, co the them link tu dong xuat kho sang lot allocations

## 10. Quyen va route guard

FE can route-guard theo quyen:

- xem alerts / badge / cogs screen:
  - can `Permissions.Inventory.View`
- recalculate COGS:
  - can `Permissions.Inventory.Update`
- dispose lot:
  - can `Permissions.Inventory.Update`

Neu user khong du quyen:

- an nut action
- block route neu can
- xu ly 403 tu API va hien thong bao phu hop

## 11. Danh sach nut / action FE can co

Danh sach ngan de dev FE checklist nhanh:

- Sidebar badge kho
- Menu `Tinh gia xuat kho`
- Menu `Canh bao kho`
- Nut `Tinh gia`
- Nut `Dat lai` trong form COGS
- Tab `Sap het / Het hang`
- Tab `Sap het han / Da het han`
- Nut `Huy lo`
- Modal `Xac nhan huy lo`
- Nut reverse phieu nhap
- Nut reverse phieu xuat

## 12. Danh sach man hinh FE nen co

- `/inventory/cogs`
- `/inventory/alerts`
- man hinh / modal dispose lot
- man hinh stock-in co field `batchCode`, `expiryDate`
- man hinh stock-out co thong diep FEFO

Man hinh nen chuan bi them cho phase tiep:

- `/inventory/lots`
- `/inventory/lots/:id`

## 13. Thu tu uu tien FE de khop voi BE hien tai

Uu tien 1:

- sidebar badge
- man hinh alerts
- modal dispose lot

Uu tien 2:

- man hinh recalculate COGS

Uu tien 3:

- cap nhat stock-in form de nhap `batchCode`, `expiryDate`
- cap nhat stock-out form de hien thong diep FEFO va xu ly loi lot

Uu tien 4:

- lot list / lot detail page day du

## 14. Ghi chu quan trong

- BE da tu dong chon lot theo FEFO, FE khong can tu tinh.
- BE da tu dong tao lot khi stock-in, FE chi can gui dung field.
- BE da co badge endpoint rieng, FE nen goi endpoint nay cho sidebar thay vi tai full alerts.
- BE da co migration schema lot/cogs, nen FE co the xem day la phan backend da mo du du lieu cho lot-based UX.
