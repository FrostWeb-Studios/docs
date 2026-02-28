---
title: Inventory System API Reference
description: Complete API reference for all BlueprintCallable functions, delegates, data assets, and serialization utilities in FWInventorySystem.
---

# Inventory System API Reference

Complete function and delegate reference for the FWInventorySystem plugin, organized by component and class.

---

## UFWInventoryComponent

### BlueprintCallable Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `AddItem` | `Item: FFWItemInstance` | `bool` | Adds item to inventory (auto-stacks). |
| `RemoveItem` | `InstanceId: FGuid`, `Quantity: int32 = 1` | `bool` | Removes quantity from stack. |
| `MoveItem` | `InstanceId: FGuid`, `TargetSlot: int32` | `bool` | Moves/swaps item to target slot. |
| `EquipItem` | `InstanceId: FGuid` | `bool` | Equips item to matching slot. |
| `UnequipItem` | `Slot: EFWEquipmentType` | `bool` | Returns equipped item to inventory. |

### BlueprintPure Functions

| Function | Returns | Description |
|---|---|---|
| `GetItemAt` | `const FFWItemInstance*` | Item at slot index (nullptr if empty). |
| `FindItemByDefinition` | `const FFWItemInstance*` | First matching item instance. |
| `GetItemCount` | `int32` | Total quantity of matching definition. |
| `HasItem` | `bool` | Whether inventory has required quantity. |
| `GetCapacity` | `int32` | Total slot count. |
| `GetUsedSlots` | `int32` | Occupied slot count. |
| `GetFreeSlots` | `int32` | Empty slot count. |
| `IsFull` | `bool` | Whether all slots are occupied. |
| `GetAllItems` | `const TArray<FFWItemInstance>&` | All items in the inventory. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnInventoryChanged` | `()` | Any inventory modification. |
| `OnItemAdded` | `(const FFWItemInstance& Item, int32 SlotIndex)` | Item added to inventory. |
| `OnItemRemoved` | `(const FFWItemInstance& Item, int32 SlotIndex)` | Item removed from inventory. |
| `OnItemMoved` | `(const FGuid& InstanceId, int32 FromSlot, int32 ToSlot)` | Item moved between slots. |
| `OnItemStackChanged` | `(const FGuid& InstanceId, int32 OldQuantity, int32 NewQuantity)` | Stack count changed. |
| `OnInventoryLoaded` | `()` | Inventory loaded from serialized data. |
| `OnInventoryFull` | `(const FFWItemInstance& RejectedItem)` | Item rejected due to full inventory. |
| `OnBankCollectionChanged` | `()` | Bank contents modified. |
| `OnBankCustomTabChanged` | `(int32 TabIndex)` | Bank tab configuration changed. |
| `OnBankLoaded` | `()` | Bank data loaded from serialized data. |

---

## UFWEquipmentComponent

### BlueprintCallable Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `Equip` | `Item: FFWItemInstance` | `bool` | Equips item to slot by type. |
| `Unequip` | `Slot: EFWEquipmentType` | `FFWItemInstance` | Removes and returns equipped item. |
| `RecalculateStats` | -- | `void` | Forces stat recalculation. |

### BlueprintPure Functions

| Function | Returns | Description |
|---|---|---|
| `GetEquippedItem` | `const FFWItemInstance*` | Item in specified slot. |
| `IsSlotOccupied` | `bool` | Whether slot has an item. |
| `GetAllSlotTypes` | `TArray<EFWEquipmentType>` | All available slot types. |
| `GetAllEquippedItems` | `TMap<EFWEquipmentType, FFWItemInstance>` | All equipped items by slot. |
| `GetTotalStatValue` | `float` | Sum of stat across all equipment. |
| `GetActiveSetBonuses` | `TArray<UFWItemSetDefinition*>` | Currently active set bonuses. |
| `GetSetPieceCount` | `int32` | Equipped pieces from a set. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnItemEquipped` | `(EFWEquipmentType Slot, const FFWItemInstance& Item)` | Item equipped. |
| `OnItemUnequipped` | `(EFWEquipmentType Slot, const FFWItemInstance& Item)` | Item unequipped. |
| `OnEquipmentStatsChanged` | `()` | Equipment stat totals changed. |
| `OnSetBonusActivated` | `(const UFWItemSetDefinition* Set, int32 PieceCount)` | Set bonus threshold reached. |
| `OnSetBonusDeactivated` | `(const UFWItemSetDefinition* Set)` | Set bonus no longer active. |

---

## UFWCraftingComponent

### BlueprintCallable Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `CraftItem` | `Recipe: UFWRecipeDefinition*`, `Quantity: int32 = 1` | `bool` | Crafts item, consuming materials. |
| `LearnRecipe` | `Recipe: UFWRecipeDefinition*` | `void` | Adds recipe to known list. |
| `UnlearnRecipe` | `Recipe: UFWRecipeDefinition*` | `void` | Removes recipe from known list. |
| `SetRecipeDatabase` | `Database: UFWRecipeDatabase*` | `void` | Sets the recipe database. |

### BlueprintPure Functions

| Function | Returns | Description |
|---|---|---|
| `CanCraft` | `bool` | Whether recipe can be crafted. |
| `GetAvailableRecipes` | `TArray<UFWRecipeDefinition*>` | Craftable recipes with current materials. |
| `GetLearnedRecipes` | `TArray<UFWRecipeDefinition*>` | All known recipes. |
| `KnowsRecipe` | `bool` | Whether player knows the recipe. |
| `GetRecipeDatabase` | `UFWRecipeDatabase*` | Current recipe database asset. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnCraftingStarted` | `(const UFWRecipeDefinition* Recipe)` | Crafting initiated. |
| `OnCraftingCompleted` | `(const UFWRecipeDefinition* Recipe, const FFWItemInstance& CraftedItem)` | Crafting succeeded. |
| `OnCraftingFailed` | `(const UFWRecipeDefinition* Recipe, const FString& Reason)` | Crafting failed. |
| `OnRecipeLearned` | `(const UFWRecipeDefinition* Recipe)` | New recipe learned. |

---

## UFWVendorComponent

### BlueprintCallable Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `BuyItem` | `BuyerInventory`, `VendorSlotIndex`, `Quantity = 1` | `bool` | Purchase from vendor. |
| `SellItem` | `SellerInventory`, `ItemInstanceId`, `Quantity = 1` | `bool` | Sell to vendor. |
| `BuybackItem` | `BuyerInventory`, `BuybackSlotIndex` | `bool` | Repurchase from buyback. |
| `RefreshVendorInventory` | -- | `void` | Force restock. |

### BlueprintPure Functions

| Function | Returns | Description |
|---|---|---|
| `GetBuyPrice` | `int32` | Buy price for vendor item. |
| `GetSellPrice` | `int32` | Sell price for player item. |
| `GetVendorInventory` | `const TArray<FFWItemInstance>&` | Vendor's stock. |
| `GetBuybackInventory` | `const TArray<FFWItemInstance>&` | Buyback buffer contents. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnItemPurchased` | `(const FFWItemInstance& Item, int32 Price, int32 Quantity)` | Item bought from vendor. |
| `OnItemSold` | `(const FFWItemInstance& Item, int32 Price, int32 Quantity)` | Item sold to vendor. |
| `OnTransactionFailed` | `(const FString& Reason)` | Transaction rejected. |
| `OnVendorInventoryRefreshed` | `()` | Vendor restocked. |

---

## UFWInventorySerializationLibrary (Static)

### BlueprintCallable Static Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `SerializeInventory` | `Component: UFWInventoryComponent*` | `FString` | Serializes inventory to JSON. |
| `DeserializeInventory` | `Component: UFWInventoryComponent*`, `JsonString: FString` | `bool` | Loads inventory from JSON. |
| `SerializeEquipment` | `Component: UFWEquipmentComponent*` | `FString` | Serializes equipment to JSON. |
| `DeserializeEquipment` | `Component: UFWEquipmentComponent*`, `JsonString: FString` | `bool` | Loads equipment from JSON. |
| `SerializeItemInstance` | `Item: FFWItemInstance` | `FString` | Serializes single item to JSON. |
| `DeserializeItemInstance` | `JsonString: FString` | `FFWItemInstance` | Deserializes single item from JSON. |
| `ValidateSerializedData` | `JsonString: FString` | `bool` | Validates JSON structure. |

---

## Data Flow Diagram

```
Player Action (UI / Gameplay)
    |
    v
UFWInventoryComponent
    |
    +--> AddItem / RemoveItem / MoveItem
    |       |
    |       +--> FFWItemArray (FFastArraySerializer)
    |       |       |
    |       |       +--> Delta replication to clients
    |       |
    |       +--> Delegates fire (OnItemAdded, etc.)
    |
    +--> EquipItem
            |
            +--> UFWEquipmentComponent::Equip()
                    |
                    +--> Apply stats via GAS
                    +--> Check set bonuses
                    +--> OnItemEquipped fires

UFWInventorySerializationLibrary
    |
    +--> SerializeInventory() --> JSON string --> Database
    +--> DeserializeInventory() <-- JSON string <-- Database
```
