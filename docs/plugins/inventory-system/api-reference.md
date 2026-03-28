---
title: Inventory System API Reference
description: Complete API reference for all functions, delegates, data assets, and types in FWInventorySystem.
---

# Inventory System API Reference

Complete function and delegate reference for the FWInventorySystem plugin, organized by component and class.

---

## UFWInventoryComponent

**Parent:** `UActorComponent`

Core inventory management with auto-stacking, slot organization, bank storage, and delta replication.

### Core Operations

| Function | Parameters | Return | Description |
|---|---|---|---|
| `AddItem` | `Item` | `bool` | Adds item to inventory, auto-stacking if possible |
| `RemoveItem` | `InstanceId, Quantity` | `bool` | Removes quantity from an item stack |
| `MoveItem` | `InstanceId, TargetSlot` | `bool` | Moves or swaps an item to a target slot |
| `SplitStack` | `InstanceId, Quantity, TargetSlot` | `bool` | Splits a stack into a new slot |
| `MergeStacks` | `SourceId, TargetId` | `bool` | Merges two compatible stacks |
| `SortInventory` | -- | `void` | Sorts inventory by type and quality |
| `EquipItem` | `InstanceId` | `bool` | Equips item to the matching equipment slot |
| `UnequipItem` | `Slot` | `bool` | Returns equipped item to inventory |

### Queries

| Function | Return | Description |
|---|---|---|
| `GetItemAt` | `ItemInstance` | Item at slot index (null if empty) |
| `FindItemByDefinition` | `ItemInstance` | First matching item instance |
| `GetItemCount` | `int32` | Total quantity of a matching item definition |
| `HasItem` | `bool` | Whether inventory has required quantity of an item |
| `GetCapacity` | `int32` | Total slot count |
| `GetUsedSlots` | `int32` | Occupied slot count |
| `GetFreeSlots` | `int32` | Empty slot count |
| `IsFull` | `bool` | Whether all slots are occupied |
| `GetAllItems` | `Array of ItemInstances` | All items in the inventory |

### Organization

| Function | Parameters | Return | Description |
|---|---|---|---|
| `SortByType` | -- | `void` | Sorts items by item type |
| `SortByQuality` | -- | `void` | Sorts items by quality tier |
| `SortByName` | -- | `void` | Sorts items alphabetically |

### Bank Operations

| Function | Parameters | Return | Description |
|---|---|---|---|
| `DepositToBank` | `InstanceId, TabIndex` | `bool` | Moves an item from inventory to a bank tab |
| `WithdrawFromBank` | `InstanceId, TabIndex` | `bool` | Moves an item from a bank tab to inventory |
| `GetBankTab` | `TabIndex` | `Array of ItemInstances` | Returns contents of a bank tab |
| `GetCollectionTabs` | -- | `Array of BankTabs` | Returns all 6 collection tabs |
| `GetCustomTabs` | -- | `Array of BankTabs` | Returns all 4 custom tabs |
| `RenameCustomTab` | `TabIndex, NewName` | `void` | Renames a custom bank tab |

### Events

| Delegate | Signature | Description |
|---|---|---|
| `OnInventoryChanged` | `()` | Any inventory modification |
| `OnItemAdded` | `(Item, SlotIndex)` | Item added to inventory |
| `OnItemRemoved` | `(Item, SlotIndex)` | Item removed from inventory |
| `OnItemMoved` | `(InstanceId, FromSlot, ToSlot)` | Item moved between slots |
| `OnItemStackChanged` | `(InstanceId, OldQuantity, NewQuantity)` | Stack count changed |
| `OnInventoryLoaded` | `()` | Inventory loaded from serialized data |
| `OnInventoryFull` | `(RejectedItem)` | Item rejected due to full inventory |
| `OnBankCollectionChanged` | `()` | Bank contents modified |
| `OnBankCustomTabChanged` | `(TabIndex)` | Bank tab configuration changed |
| `OnBankLoaded` | `()` | Bank data loaded from serialized data |

---

## UFWEquipmentComponent

**Parent:** `UActorComponent`

Equipment slot management with automatic stat application via GAS and set bonus tracking.

### Equipment Operations

| Function | Parameters | Return | Description |
|---|---|---|---|
| `Equip` | `Item` | `bool` | Equips item to the appropriate slot by type |
| `Unequip` | `Slot` | `ItemInstance` | Removes and returns the equipped item |
| `SwapEquipment` | `Slot, NewItem` | `ItemInstance` | Swaps equipped item and returns the old one |
| `RecalculateStats` | -- | `void` | Forces a full stat recalculation from all equipment |

### Queries

| Function | Return | Description |
|---|---|---|
| `GetEquippedItem` | `ItemInstance` | Item in specified slot |
| `IsSlotOccupied` | `bool` | Whether slot has an item |
| `GetAllSlotTypes` | `Array of EquipmentTypes` | All available slot types |
| `GetAllEquippedItems` | `Map of Slot to ItemInstance` | All equipped items by slot |
| `GetTotalStatValue` | `float` | Sum of a stat across all equipment |
| `GetActiveSetBonuses` | `Array of ItemSetDefinitions` | Currently active set bonuses |
| `GetSetPieceCount` | `int32` | Number of equipped pieces from a set |

### Standard Equipment Slots

| Slot | Description |
|---|---|
| `Head` | Helmets, hats, hoods |
| `Chest` | Chest armor, robes |
| `Legs` | Leg armor, pants |
| `Feet` | Boots, shoes |
| `Hands` | Gloves, gauntlets |
| `Shoulders` | Shoulder armor, pauldrons |
| `Back` | Cloaks, capes, backpacks |
| `MainHand` | Primary weapon |
| `OffHand` | Shield, secondary weapon, focus |
| `Ring1` | First ring slot |
| `Ring2` | Second ring slot |
| `Amulet` | Necklace, amulet |
| `Belt` | Belt, sash |

### Events

| Delegate | Signature | Description |
|---|---|---|
| `OnItemEquipped` | `(Slot, Item)` | Item equipped to a slot |
| `OnItemUnequipped` | `(Slot, Item)` | Item unequipped from a slot |
| `OnEquipmentStatsChanged` | `()` | Equipment stat totals recalculated |
| `OnSetBonusActivated` | `(SetDefinition, PieceCount)` | Set bonus threshold reached |
| `OnSetBonusDeactivated` | `(SetDefinition)` | Set bonus no longer active |

---

## UFWCraftingComponent

**Parent:** `UActorComponent`

Recipe-based crafting with material consumption, skill integration, and recipe learning.

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `CraftItem` | `Recipe, Quantity` | `bool` | Crafts item, consuming required materials |
| `LearnRecipe` | `Recipe` | `void` | Adds recipe to the player's known recipes |
| `UnlearnRecipe` | `Recipe` | `void` | Removes recipe from the player's known recipes |
| `SetRecipeDatabase` | `Database` | `void` | Sets the active recipe database |
| `CanCraft` | `Recipe` | `bool` | Whether the recipe can be crafted with current materials |
| `GetAvailableRecipes` | -- | `Array of RecipeDefs` | Recipes craftable with current materials |
| `GetLearnedRecipes` | -- | `Array of RecipeDefs` | All known recipes |
| `KnowsRecipe` | `Recipe` | `bool` | Whether player knows the recipe |
| `GetRecipeDatabase` | -- | `RecipeDatabase` | Current recipe database asset |

### Events

| Delegate | Signature | Description |
|---|---|---|
| `OnCraftingStarted` | `(Recipe)` | Crafting initiated |
| `OnCraftingCompleted` | `(Recipe, CraftedItem)` | Crafting succeeded |
| `OnCraftingFailed` | `(Recipe, Reason)` | Crafting failed |
| `OnRecipeLearned` | `(Recipe)` | New recipe learned |

---

## UFWVendorComponent

**Parent:** `UActorComponent`

NPC vendor with buy, sell, and buyback functionality. Attach to NPC actors.

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `BuyItem` | `BuyerInventory, SlotIndex, Quantity` | `bool` | Purchase an item from the vendor |
| `SellItem` | `SellerInventory, InstanceId, Quantity` | `bool` | Sell an item to the vendor |
| `BuybackItem` | `BuyerInventory, BuybackIndex` | `bool` | Repurchase a recently sold item |
| `RefreshVendorInventory` | -- | `void` | Force a vendor restock |
| `GetBuyPrice` | `SlotIndex` | `int32` | Buy price for a vendor item |
| `GetSellPrice` | `InstanceId` | `int32` | Sell price for a player item |
| `GetVendorInventory` | -- | `Array of ItemInstances` | Vendor's current stock |
| `GetBuybackInventory` | -- | `Array of ItemInstances` | Buyback buffer contents |

### Events

| Delegate | Signature | Description |
|---|---|---|
| `OnItemPurchased` | `(Item, Price, Quantity)` | Item bought from vendor |
| `OnItemSold` | `(Item, Price, Quantity)` | Item sold to vendor |
| `OnTransactionFailed` | `(Reason)` | Transaction rejected |
| `OnVendorInventoryRefreshed` | `()` | Vendor restocked |

---

## UFWInventorySubsystem

**Parent:** `UWorldSubsystem`

World-level subsystem for item management, minting, and global inventory operations.

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `MintItem` | `Definition, Seed` | `ItemInstance` | Creates an item instance from a definition using a deterministic seed |
| `ValidateItem` | `ItemInstance` | `bool` | Validates an item instance against its definition |
| `GetItemDefinition` | `ItemTag` | `ItemDefinition` | Looks up an item definition by gameplay tag |

---

## Data Assets

### UFWItemDefinition

A data asset representing a single item type with all its properties.

| Property | Type | Description |
|---|---|---|
| `ItemTag` | `FGameplayTag` | Unique identifier tag |
| `ItemName` | `FText` | Localized display name |
| `Description` | `FText` | Item description |
| `Icon` | `Texture2D` | Inventory icon |
| `Mesh` | `StaticMesh` | World mesh (for dropped items) |
| `ItemType` | `EFWItemType` | Item classification |
| `EquipmentSlot` | `EFWEquipmentType` | Which slot this item equips to (equipment only) |
| `Quality` | `EFWItemQuality` | Base quality tier |
| `MaxStackSize` | `int32` | Maximum stack size (1 = non-stackable) |
| `BaseStats` | `Map of Tag to Float` | Base stat values |
| `AffixPool` | `Array of AffixDefinitions` | Possible random affixes |
| `RequiredLevel` | `int32` | Minimum level to use |
| `SellValue` | `int32` | Base vendor sell value |
| `Weight` | `float` | Item weight (if using weight system) |

### UFWRecipeDefinition

A data asset representing a crafting recipe.

| Property | Type | Description |
|---|---|---|
| `RecipeTag` | `FGameplayTag` | Unique identifier |
| `ResultItem` | `ItemDefinition` | Item produced |
| `ResultQuantity` | `int32` | Quantity produced |
| `Materials` | `Array of (ItemDef, Quantity)` | Required materials |
| `RequiredSkillTag` | `FGameplayTag` | Skill required to craft |
| `RequiredSkillLevel` | `int32` | Minimum skill level |
| `CraftTime` | `float` | Time to craft in seconds |

### UFWPerkDefinition

A data asset for passive perk bonuses tied to items.

| Property | Type | Description |
|---|---|---|
| `PerkTag` | `FGameplayTag` | Unique identifier |
| `PerkName` | `FText` | Display name |
| `Description` | `FText` | Perk description |
| `StatModifiers` | `Array of StatModifiers` | Stat changes applied when active |

### UFWItemSetDefinition

A data asset defining item set bonuses with tiered thresholds.

| Property | Type | Description |
|---|---|---|
| `SetTag` | `FGameplayTag` | Unique identifier |
| `SetName` | `FText` | Display name |
| `SetPieces` | `Array of ItemDefinitions` | Items that belong to this set |
| `Bonuses` | `Array of (PieceCount, Effects)` | Tiered bonuses by piece count |

### UFWRecipeDatabase

A collection asset for crafting recipes.

| Property | Type | Description |
|---|---|---|
| `Recipes` | `Array of RecipeDefinitions` | All recipes in this database |

---

## Core Types

### Item Quality Tiers (EFWItemQuality)

| Value | Description |
|---|---|
| `Common` | Basic quality, white |
| `Uncommon` | Slightly better, green |
| `Rare` | Good quality, blue |
| `Epic` | Excellent quality, purple |
| `Legendary` | Best quality, orange |

### Item Types (EFWItemType)

| Value | Description |
|---|---|
| `Weapon` | Melee and ranged weapons |
| `Armor` | Defensive equipment |
| `Accessory` | Rings, amulets, belts |
| `Consumable` | Potions, food, scrolls |
| `Material` | Crafting materials |
| `Quest` | Quest-related items |
| `Currency` | Currency items |
| `Miscellaneous` | Everything else |

### FFWItemInstance

A runtime item instance with unique identity and generated properties.

| Field | Type | Description |
|---|---|---|
| `InstanceId` | `FGuid` | Unique instance identifier |
| `Definition` | `ItemDefinition` | Reference to the item definition |
| `Quantity` | `int32` | Stack count |
| `Quality` | `EFWItemQuality` | Rolled quality tier |
| `Seed` | `int32` | Deterministic generation seed |
| `Affixes` | `Array of AffixInstances` | Generated random affixes |
| `Durability` | `float` | Current durability (if applicable) |
| `MaxDurability` | `float` | Maximum durability |

---

## Data Flow

```
Player Action (UI / Gameplay)
    |
    v
UFWInventoryComponent
    |
    +--> AddItem / RemoveItem / MoveItem
    |       |
    |       +--> Delta replication to clients
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
```
