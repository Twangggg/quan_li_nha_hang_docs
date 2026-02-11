# Database Schema Design - Issue 30: Menu Management (MVP Version)

This document defines a simplified database schema focusing on immediate requirements for Issue 30 (Scrum approach).

---

### **1. Core Entities**

#### **CATEGORY**

| ID  | Column      | Key | Type         | Description                              |
| :-- | :---------- | :-- | :----------- | :--------------------------------------- |
| 1   | category_id | PK  | UUID         | Primary Key                              |
| 2   | name        | UQ  | VARCHAR(100) | Category name (e.g., Khai vị, Món chính) |
| 3   | type        |     | INT          | Enum: `FOOD / DRINK / DESSERT / OTHER`   |
| 4   | created_at  |     | TIMESTAMP    | Creation time                            |
| 5   | updated_at  |     | TIMESTAMP    | Last update time                         |

#### **MENU_ITEM**

| ID  | Column          | Key | Type          | Description                              |
| :-- | :-------------- | :-- | :------------ | :--------------------------------------- |
| 1   | menu_item_id    | PK  | UUID          | Primary Key                              |
| 2   | code            | UQ  | VARCHAR(50)   | Item code (Required, Unique)             |
| 3   | name            |     | VARCHAR(150)  | Item name (Required)                     |
| 4   | image_url       |     | VARCHAR(255)  | Image link (Required)                    |
| 5   | description     |     | VARCHAR(500)  | Item description                         |
| 6   | category_id     | FK  | UUID          | Category reference                       |
| 7   | station         |     | INT           | Enum: `HOT_KITCHEN / COLD_KITCHEN / BAR` |
| 8   | expected_time   |     | INT           | Estimated prep time (Minutes)            |
| 9   | price_dine_in   |     | NUMERIC(12,2) | Price for eating in                      |
| 10  | price_take_away |     | NUMERIC(12,2) | Price for take away                      |
| 11  | cost            |     | NUMERIC(12,2) | Internal cost (Manager/Cashier only)     |
| 12  | is_out_of_stock |     | BOOLEAN       | Availability status                      |
| 13  | created_by_id   | FK  | VARCHAR(100)  | Creator ID                               |
| 14  | updated_by_id   | FK  | VARCHAR(100)  | Updater ID                               |
| 15  | created_at      |     | TIMESTAMP     | Creation time                            |
| 16  | updated_at      |     | TIMESTAMP     | Last update time                         |
| 17  | deleted_at      |     | TIMESTAMP     | Soft delete                              |

---

### **2. Options & Customization**

#### **OPTION_GROUP**

| ID  | Column          | Key | Type         | Description                     |
| :-- | :-------------- | :-- | :----------- | :------------------------------ |
| 1   | option_group_id | PK  | UUID         | Primary Key                     |
| 2   | menu_item_id    | FK  | UUID         | Related Menu Item               |
| 3   | name            |     | VARCHAR(100) | Group name (Sugar, Ice, Toping) |
| 4   | type            |     | INT          | `SINGLE / MULTI / SCALE / TEXT` |
| 5   | is_required     |     | BOOLEAN      | Required selection?             |

#### **OPTION_ITEM**

| ID  | Column          | Key | Type          | Description            |
| :-- | :-------------- | :-- | :------------ | :--------------------- |
| 1   | option_item_id  | PK  | UUID          | Primary Key            |
| 2   | option_group_id | FK  | UUID          | Parent Option Group    |
| 3   | label           |     | VARCHAR(100)  | Display label          |
| 4   | extra_price     |     | NUMERIC(12,2) | Additional cost charge |

---

### **3. Combos & Set Menus**

#### **SET_MENU**

| ID  | Column          | Key | Type          | Description                 |
| :-- | :-------------- | :-- | :------------ | :-------------------------- |
| 1   | set_menu_id     | PK  | UUID          | Primary Key                 |
| 2   | code            | UQ  | VARCHAR(50)   | Set code (Required, Unique) |
| 3   | name            |     | VARCHAR(150)  | Set name                    |
| 4   | price           |     | NUMERIC(12,2) | Fixed price for set         |
| 5   | is_out_of_stock |     | BOOLEAN       | Availability status         |
| 6   | created_at      |     | TIMESTAMP     | Creation time               |
| 7   | updated_at      |     | TIMESTAMP     | Last update time            |

#### **SET_MENU_ITEM**

| ID  | Column           | Key | Type | Description       |
| :-- | :--------------- | :-- | :--- | :---------------- |
| 1   | set_menu_item_id | PK  | UUID | Primary Key       |
| 2   | set_menu_id      | FK  | UUID | Parent Set Menu   |
| 3   | menu_item_id     | FK  | UUID | Child Menu Item   |
| 4   | quantity         |     | INT  | Quantity included |
