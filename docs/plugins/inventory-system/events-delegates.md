---
title: Inventory Events and Delegates
description: Complete reference for all multicast delegates, event handling patterns, and event flow in FWInventorySystem.
---

# Inventory Events and Delegates

All delegates in FWInventorySystem are `DECLARE_DYNAMIC_MULTICAST_DELEGATE` variants and are marked `BlueprintAssignable`, making them fully available for both C++ and Blueprint binding.

---

## Inventory Component Delegates

### OnInventoryChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnInventoryChanged);
```

Fires whenever any modification occurs to the inventory contents. This is the most general-purpose delegate -- use it to trigger full UI refreshes.

!!! tip "Debouncing"
    When loading inventory from serialized data, `OnInventoryChanged` may fire once for each item added. Consider deferring UI refreshes until `OnInventoryLoaded` fires, which is a single event after the full load completes.

---

### OnItemAdded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnItemAdded,
    const FFWItemInstance&, Item,
    int32, SlotIndex);
```

Fires when a new item is placed in the inventory. `SlotIndex` indicates which slot the item occupies. For stackable items being added to an existing stack, `OnItemStackChanged` fires instead.

---

### OnItemRemoved

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnItemRemoved,
    const FFWItemInstance&, Item,
    int32, SlotIndex);
```

Fires when an item is completely removed from a slot (stack depleted or single item removed).

---

### OnItemMoved

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnItemMoved,
    const FGuid&, InstanceId,
    int32, FromSlot,
    int32, ToSlot);
```

Fires when an item is moved between slots within the same inventory.

---

### OnItemStackChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnItemStackChanged,
    const FGuid&, InstanceId,
    int32, OldQuantity,
    int32, NewQuantity);
```

Fires when a stackable item's quantity changes (partial add or partial remove). Does not fire for the initial add or final remove (those use `OnItemAdded`/`OnItemRemoved`).

---

### OnInventoryLoaded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnInventoryLoaded);
```

Fires once after inventory data is deserialized and fully loaded. Use this instead of `OnInventoryChanged` for initial UI population.

---

### OnInventoryFull

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnInventoryFull,
    const FFWItemInstance&, RejectedItem);
```

Fires when `AddItem` fails because all inventory slots are occupied. The `RejectedItem` is the item that could not be added.

---

### Bank Delegates

#### OnBankCollectionChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnBankCollectionChanged);
```

Fires when any item in the bank is added, removed, or moved.

#### OnBankCustomTabChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnBankCustomTabChanged,
    int32, TabIndex);
```

Fires when a bank tab's configuration is modified (renamed, icon changed).

#### OnBankLoaded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnBankLoaded);
```

Fires once after bank data is deserialized and fully loaded.

---

## Equipment Component Delegates

### OnItemEquipped

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnItemEquipped,
    EFWEquipmentType, Slot,
    const FFWItemInstance&, Item);
```

Fires when an item is equipped to a slot. Use this to update character visuals (attach mesh, apply materials).

---

### OnItemUnequipped

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnItemUnequipped,
    EFWEquipmentType, Slot,
    const FFWItemInstance&, Item);
```

Fires when an item is removed from an equipment slot.

---

### OnEquipmentStatsChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnEquipmentStatsChanged);
```

Fires whenever the total equipment stats change (equip, unequip, augmentation, set bonus change). Bind this to refresh stat display panels.

---

### OnSetBonusActivated

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnSetBonusActivated,
    const UFWItemSetDefinition*, SetDefinition,
    int32, PieceCount);
```

Fires when equipping an item causes a set bonus threshold to be reached.

---

### OnSetBonusDeactivated

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnSetBonusDeactivated,
    const UFWItemSetDefinition*, SetDefinition);
```

Fires when unequipping an item causes a set bonus to drop below its minimum threshold.

---

## Crafting Component Delegates

### OnCraftingStarted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnCraftingStarted,
    const UFWRecipeDefinition*, Recipe);
```

Fires when crafting begins. Use for progress bar animations or cast-bar UI.

---

### OnCraftingCompleted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnCraftingCompleted,
    const UFWRecipeDefinition*, Recipe,
    const FFWItemInstance&, CraftedItem);
```

Fires when crafting succeeds. `CraftedItem` is the newly created item with any rolled affixes.

---

### OnCraftingFailed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnCraftingFailed,
    const UFWRecipeDefinition*, Recipe,
    const FString&, Reason);
```

Fires when crafting fails validation or material checks.

---

### OnRecipeLearned

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnRecipeLearned,
    const UFWRecipeDefinition*, Recipe);
```

Fires when a new recipe is added to the player's known recipes.

---

## Vendor Component Delegates

### OnItemPurchased

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnItemPurchased,
    const FFWItemInstance&, Item,
    int32, Price,
    int32, Quantity);
```

Fires after a successful purchase from the vendor.

---

### OnItemSold

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnItemSold,
    const FFWItemInstance&, Item,
    int32, Price,
    int32, Quantity);
```

Fires after successfully selling an item to the vendor.

---

### OnTransactionFailed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnTransactionFailed,
    const FString&, Reason);
```

Fires when a buy or sell transaction is rejected. Common reasons: insufficient currency, inventory full, item is bound.

---

### OnVendorInventoryRefreshed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnVendorInventoryRefreshed);
```

Fires when the vendor's stock is restocked.

---

## Binding Patterns

### C++ -- Centralized Event Handler

```cpp
void AMyCharacter::SetupInventoryBindings()
{
    // Inventory events
    Inventory->OnItemAdded.AddDynamic(this, &AMyCharacter::OnItemAdded);
    Inventory->OnItemRemoved.AddDynamic(this, &AMyCharacter::OnItemRemoved);
    Inventory->OnInventoryFull.AddDynamic(this, &AMyCharacter::OnBagsFull);
    Inventory->OnInventoryLoaded.AddDynamic(this, &AMyCharacter::OnInventoryReady);

    // Equipment events
    Equipment->OnItemEquipped.AddDynamic(this, &AMyCharacter::OnEquipped);
    Equipment->OnItemUnequipped.AddDynamic(this, &AMyCharacter::OnUnequipped);
    Equipment->OnEquipmentStatsChanged.AddDynamic(
        this, &AMyCharacter::RefreshStatUI);
    Equipment->OnSetBonusActivated.AddDynamic(
        this, &AMyCharacter::ShowSetBonusNotification);
}
```

### C++ -- Widget-Level Binding

```cpp
void UInventoryWidget::NativeConstruct()
{
    Super::NativeConstruct();

    auto* Inv = GetOwningPlayerPawn()
        ->FindComponentByClass<UFWInventoryComponent>();

    if (Inv)
    {
        Inv->OnInventoryChanged.AddDynamic(
            this, &UInventoryWidget::RefreshGrid);
    }
}

void UInventoryWidget::NativeDestruct()
{
    auto* Inv = GetOwningPlayerPawn()
        ->FindComponentByClass<UFWInventoryComponent>();

    if (Inv)
    {
        Inv->OnInventoryChanged.RemoveDynamic(
            this, &UInventoryWidget::RefreshGrid);
    }

    Super::NativeDestruct();
}
```

### Blueprint

1. In your widget or actor Blueprint, get a reference to the inventory component.
2. Right-click the component reference, select **Assign** for the desired event.
3. The event node is created automatically with the correct signature.
4. Connect your UI refresh logic to the event node.

---

## Event Flow Summary

```
AddItem()
    |
    +--> [Stack exists?]
    |       |
    |       +--> Yes: OnItemStackChanged
    |       |
    |       +--> No: OnItemAdded
    |
    +--> OnInventoryChanged

RemoveItem()
    |
    +--> [Stack depleted?]
    |       |
    |       +--> Yes: OnItemRemoved
    |       |
    |       +--> No: OnItemStackChanged
    |
    +--> OnInventoryChanged

EquipItem()
    |
    +--> [Slot occupied?]
    |       |
    |       +--> Yes: OnItemUnequipped (old item)
    |       |       |
    |       |       +--> OnItemAdded (old item to inventory)
    |       |
    |       +--> OnItemRemoved (new item from inventory)
    |
    +--> OnItemEquipped (new item)
    +--> OnEquipmentStatsChanged
    +--> [Set threshold?] --> OnSetBonusActivated
    +--> OnInventoryChanged
```
