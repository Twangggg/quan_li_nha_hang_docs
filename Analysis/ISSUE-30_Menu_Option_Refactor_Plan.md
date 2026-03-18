# ISSUE 30 - Menu and Option Refactor Plan

> Purpose: make menu options reusable across multiple menu items without breaking order snapshot behavior.
> Scope: backend refactor plan for domain, application, infrastructure, migration, and tests.
> Date: 2026-03-19

---

## 1. Goal

The current model makes `OptionGroup` a direct child of `MenuItem`.

That is simple, but it is too rigid when the same option group must be shared by multiple menu items, or when a single option group needs different rules per menu item.

The refactor should:

- keep `OrderItem` as the snapshot of what was actually sold
- make option definitions reusable
- move menu-item-specific option rules into a mapping entity
- preserve backward compatibility during the transition

---

## 2. Current Situation

### 2.1 Domain coupling today

Current relationship:

- `MenuItem` owns `OptionGroups`
- `OptionGroup` belongs to exactly one `MenuItem`
- `OptionGroup` owns `OptionItems`
- `OrderItem` stores snapshot data for selected options

This is visible in:

- [`MenuItem.cs`](../../FoodHub_BE/FoodHub.Domain/Entities/MenuItem.cs)
- [`OptionGroup.cs`](../../FoodHub_BE/FoodHub.Domain/Entities/OptionGroup.cs)
- [`OptionItem.cs`](../../FoodHub_BE/FoodHub.Domain/Entities/OptionItem.cs)
- [`Order.cs`](../../FoodHub_BE/FoodHub.Domain/Entities/Order.cs)
- [`OrderItem.cs`](../../FoodHub_BE/FoodHub.Domain/Entities/OrderItem.cs)

### 2.2 Application coupling today

The application layer also assumes direct ownership by menu item:

- `CreateOptionGroupCommand` requires `MenuItemId`
- `CreateOptionGroupHandler` resolves `MenuItem` before creating the group
- `GetOptionGroupsByMenuItemQuery` reads by `MenuItemId`
- `CreateOptionItem` assumes the group already belongs to one menu item context
- validators and response DTOs still expose `MenuItemId`

Files involved:

- [`CreateOptionGroupCommand.cs`](../../FoodHub_BE/FoodHub.Application/Features/Options/Commands/CreateOptionGroup/CreateOptionGroupCommand.cs)
- [`CreateOptionGroupHandler.cs`](../../FoodHub_BE/FoodHub.Application/Features/Options/Commands/CreateOptionGroup/CreateOptionGroupHandler.cs)
- [`CreateOptionGroupValidator.cs`](../../FoodHub_BE/FoodHub.Application/Features/Options/Commands/CreateOptionGroup/CreateOptionGroupValidator.cs)
- [`GetOptionGroupsByMenuItemHandler.cs`](../../FoodHub_BE/FoodHub.Application/Features/Options/Queries/GetOptionGroupsByMenuItem/GetOptionGroupsByMenuItemHandler.cs)
- [`UpdateOptionGroupHandler.cs`](../../FoodHub_BE/FoodHub.Application/Features/Options/Commands/UpdateOptionGroup/UpdateOptionGroupHandler.cs)
- [`CreateOptionItemHandler.cs`](../../FoodHub_BE/FoodHub.Application/Features/Options/Commands/CreateOptionItem/CreateOptionItemHandler.cs)

### 2.3 What is already correct

The order flow is already using snapshot behavior:

- `OrderItem` stores `ItemCodeSnapshot`, `ItemNameSnapshot`, `StationSnapshot`, and `UnitPriceSnapshot`
- `OrderItemOptionGroup` and `OrderItemOptionValue` store selected option snapshots

So the refactor should not introduce a separate `OrderItemSnapshot` entity.
`OrderItem` is already the snapshot entity.

---

## 3. Target Design

### 3.1 Desired ownership model

Recommended structure:

- `MenuItem` = sellable item
- `OptionGroup` = reusable option definition
- `OptionItem` = values inside an option group
- `MenuItemOptionGroup` = mapping and per-item rules
- `OrderItem` = snapshot of a purchased line item

### 3.2 Why this is better

This solves:

- option reuse across many menu items
- menu-item-specific required rules
- different min/max selection limits per menu item
- different display order per menu item
- reusable option master data without duplication

### 3.3 What stays as snapshot

The following should still be copied at order time:

- option group name
- option group type
- whether the option group was required at that moment
- option item label
- option item extra price

That keeps historical orders stable even if menu configuration changes later.

---

## 4. Proposed Data Model

### 4.1 New entity: `MenuItemOptionGroup`

This is the key new table.

Suggested fields:

- `MenuItemOptionGroupId`
- `MenuItemId`
- `OptionGroupId`
- `IsRequired`
- `MinSelect`
- `MaxSelect`
- `SortOrder`
- `IsVisible`
- optional future fields such as `DefaultSelectedCount`

### 4.2 Entity responsibilities

#### `MenuItem`

Only owns the menu item itself.

#### `OptionGroup`

Reusable option metadata:

- `Name`
- `OptionType`
- shared flags

#### `OptionItem`

Reusable option values:

- `Label`
- `ExtraPrice`

#### `MenuItemOptionGroup`

Per-menu-item configuration:

- whether the group appears for this item
- whether it is required
- how many selections are allowed
- how it is ordered on UI

#### `OrderItem`

Snapshot of the sold item and its selected options.

---

## 5. Refactor Strategy

The safest path is a staged migration with dual support for a short period.

### Stage 0 - Inventory and freeze scope

Before changing code, list every endpoint and handler that touches options:

- create option group
- update option group
- delete option group
- create option item
- update option item
- delete option item
- get option groups for menu item
- order item add/update
- order submit to kitchen
- menu item detail query if it surfaces options

This keeps the refactor bounded and prevents surprise regressions.

### Stage 1 - Add new model without removing old behavior

Implement the new mapping entity first.

1. Add `MenuItemOptionGroup` entity.
2. Add EF configuration and `DbSet<MenuItemOptionGroup>` in `AppDbContext`.
3. Add navigation properties:
   - `MenuItem -> MenuItemOptionGroups`
   - `OptionGroup -> MenuItemOptionGroups`
4. Keep `OptionGroup.MenuItemId` temporarily if needed for migration compatibility.
5. Do not remove existing queries yet.

Why this order:

- it lets the database accept the new structure before old code is removed
- it avoids breaking read paths during deployment

### Stage 2 - Add new assignment flow

Instead of creating an option group “for a menu item”, introduce explicit assignment.

Suggested commands:

- `CreateOptionGroupCommand` creates a reusable option group only
- `AssignOptionGroupToMenuItemCommand` links a group to one or more menu items
- `UpdateMenuItemOptionGroupCommand` updates per-item rules

This is the point where the current `MenuItemId` requirement should be removed from group creation.

Files likely to change:

- `CreateOptionGroupCommand.cs`
- `CreateOptionGroupHandler.cs`
- `CreateOptionGroupValidator.cs`
- new assignment command/handler/validator
- `UpdateOptionGroupHandler.cs` if response shape changes

### Stage 3 - Update reads

Update option reads to use the mapping table.

Recommended sequence:

1. Add a new query that returns menu item options through `MenuItemOptionGroup`.
2. Keep the old query temporarily for compatibility.
3. Switch consumers to the new query.
4. Remove the old query only after everything is migrated.

This avoids a big-bang change.

### Stage 4 - Update order validation

Once reads are correct, tighten order validation.

Validation should ensure:

- selected option groups are assigned to the chosen menu item
- selected option items belong to the selected option group
- quantity rules are respected
- required groups are satisfied
- min/max selection limits are respected

Files likely to change:

- `AddOrderItemHandler.cs`
- `SubmitOrderToKitchenHandler.cs`
- `AddOrderItemValidator.cs`
- `SubmitOrderToKitchenValidator.cs`
- any order DTOs that carry selected options

### Stage 5 - Migrate data

Backfill existing data into the new structure.

Migration approach:

1. Create `MenuItemOptionGroup` rows from each existing `OptionGroup.MenuItemId`.
2. Preserve `IsRequired` and any other existing per-group rules.
3. Keep legacy `OptionGroup.MenuItemId` in place for one release if needed.
4. Verify read paths against the new table.

Important:

- do not delete or rewrite old rows until the new read path is proven stable
- this makes rollback much easier

### Stage 6 - Remove old coupling

After the new flow is stable:

1. Remove `MenuItemId` from `OptionGroup` if it is no longer needed.
2. Remove direct `MenuItem -> OptionGroups` ownership.
3. Rename APIs and DTOs so they reflect reusable option groups, not menu-item-owned groups.
4. Clean up caches and tests.

---

## 6. Detailed Implementation Order

This is the recommended order to avoid blocking yourself later.

### Step 1

Create new domain entity and EF mapping:

- `MenuItemOptionGroup`
- `MenuItemOptionGroupConfiguration`

### Step 2

Wire `AppDbContext`:

- add `DbSet<MenuItemOptionGroup>`
- ensure migrations pick it up

### Step 3

Add assignment command and handler:

- create reusable option group
- map it to a menu item
- store item-specific rules

### Step 4

Introduce read-through-mapping query:

- return assigned option groups for a menu item
- include item-level config in the response

### Step 5

Update menu screens and API contract consumers to use the new query.

### Step 6

Update order validation to use the mapping data.

### Step 7

Create migration and backfill data.

### Step 8

Remove legacy coupling after verifying production behavior.

---

## 7. File Impact Map

### Domain

- `MenuItem.cs`
- `OptionGroup.cs`
- `OptionItem.cs`
- `Order.cs`
- `OrderItem.cs`
- new `MenuItemOptionGroup.cs`

### Application

- `CreateOptionGroupCommand.cs`
- `CreateOptionGroupHandler.cs`
- `CreateOptionGroupValidator.cs`
- `UpdateOptionGroupCommand.cs`
- `UpdateOptionGroupHandler.cs`
- `UpdateOptionGroupValidator.cs`
- `GetOptionGroupsByMenuItemQuery.cs`
- `GetOptionGroupsByMenuItemHandler.cs`
- `CreateOptionItemCommand.cs`
- `CreateOptionItemHandler.cs`
- `CreateOptionItemValidator.cs`
- `AddOrderItemHandler.cs`
- `SubmitOrderToKitchenHandler.cs`
- `AddOrderItemValidator.cs`
- `SubmitOrderToKitchenValidator.cs`

### Infrastructure

- `AppDbContext.cs`
- `OptionGroupConfiguration.cs`
- `OptionItemConfiguration.cs`
- new `MenuItemOptionGroupConfiguration.cs`
- migrations
- seed data

### Tests

- option creation tests
- option assignment tests
- option query tests
- order option validation tests
- order snapshot tests
- migration/backfill tests if available

---

## 8. Validation Rules To Enforce

These rules should be explicit in the new design.

### 8.1 Group assignment rules

- An `OptionGroup` may be assigned to multiple menu items.
- A menu item may have multiple option groups.
- The same group can have different `IsRequired`, `MinSelect`, and `MaxSelect` values per menu item.

### 8.2 Order selection rules

- optional groups may be skipped
- required groups must have valid selections
- selected items must belong to the selected group
- max selection cannot be exceeded
- min selection must be satisfied

### 8.3 Snapshot rules

- order snapshots must not depend on live option definitions after save
- order history must remain readable even if option definitions are updated or deleted later

---

## 9. Backward Compatibility Strategy

The safest rollout is a dual-read, then cutover approach.

### Release 1

- introduce new mapping table
- keep old `MenuItemId` on `OptionGroup`
- keep current APIs working
- add new APIs if needed
- backfill mapping data

### Release 2

- move consumers to the new APIs
- stop writing new data through the old ownership model
- remove obsolete fields and handlers

This reduces risk because the system does not change all at once.

---

## 10. Data Migration Details

### Migration input

Existing `OptionGroup` rows with `MenuItemId`.

### Migration output

One `MenuItemOptionGroup` row per existing assignment.

### Backfill logic

For each `OptionGroup`:

- create a mapping row for its `MenuItemId`
- copy any current required/display flags
- preserve ordering if there is already a sort convention

### Verification checks

- every old option group still appears for the same menu item
- no mapping row is missing
- order submission still resolves valid option groups

---

## 11. Test Plan

### Unit tests

- creating an option group without `MenuItemId`
- assigning an option group to a menu item
- rejecting invalid menu-item/option-group pairs
- rejecting option items not in the selected group
- enforcing required/min/max selection rules

### Integration tests

- query menu item options through the new mapping
- create order item with option snapshot
- update order item and preserve option snapshot behavior
- submit order to kitchen with selected options

### Regression tests

- old menus still load correctly after backfill
- order history remains unchanged after option definitions are modified
- delete/update operations do not break snapshot orders

---

## 12. Risks And Mitigations

### Risk: breaking current API contracts

Mitigation:

- keep old endpoints temporarily
- add new endpoints instead of changing everything at once

### Risk: migration gaps

Mitigation:

- backfill first
- compare counts before and after migration
- verify a sample menu item end-to-end

### Risk: order validation becomes inconsistent

Mitigation:

- centralize validation in one place
- do not duplicate rule enforcement across handlers

### Risk: too much change at once

Mitigation:

- split the work into the staged rollout above
- deploy in two releases if needed

---

## 13. Recommended Final Shape

The end state I recommend is:

- `OptionGroup` is reusable master data
- `OptionItem` stays under `OptionGroup`
- `MenuItemOptionGroup` owns per-item rules
- `OrderItem` remains the order snapshot
- option validation happens before snapshot creation

That keeps the system flexible without losing the simplicity of snapshot-based orders.

