---
title: UFWInventoryComponent
description: Core inventory management component with delta-replicated item arrays, stacking, and multi-inventory support.
---

# UFWInventoryComponent

**Header:** `FWInventoryComponent.h` | **Parent:** `UActorComponent` | **Specifier:** `BlueprintSpawnableComponent`

The primary inventory management component. Uses `FFWItemArray` (backed by `FFastArraySerializer`) for bandwidth-efficient delta replication. Supports adding, removing, moving, equipping, and stacking items with full Blueprint exposure.

---

## Class Declaration

```cpp
UCLASS(ClassGroup = (FrostWeb), meta = (BlueprintSpawnableComponent))
class FWINVENTORYSYSTEM_API UFWInventoryComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFWInventoryComponent();

    // --- Item Operations ---
    UFUNCTION(BlueprintCallable, Category = "Inventory")
    bool AddItem(const FFWItemInstance& Item);

    UFUNCTION(BlueprintCallable, Category = "Inventory")
    bool RemoveItem(const FGuid& InstanceId, int32 Quantity = 1);

    UFUNCTION(BlueprintCallable, Category = "Inventory")
    bool MoveItem(const FGuid& InstanceId, int32 TargetSlot);

    UFUNCTION(BlueprintCallable, Category = "Inventory")
    bool EquipItem(const FGuid& InstanceId);

    UFUNCTION(BlueprintCallable, Category = "Inventory")
    bool UnequipItem(EFWEquipmentType Slot);

    // --- Queries ---
    UFUNCTION(BlueprintPure, Category = "Inventory")
    const FFWItemInstance* GetItemAt(int32 SlotIndex) const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    const FFWItemInstance* FindItemByDefinition(
        const UFWItemDefinition* Definition) const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    int32 GetItemCount(const UFWItemDefinition* Definition) const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    bool HasItem(const UFWItemDefinition* Definition,
        int32 RequiredQuantity = 1) const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    int32 GetCapacity() const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    int32 GetUsedSlots() const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    int32 GetFreeSlots() const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    bool IsFull() const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    const TArray<FFWItemInstance>& GetAllItems() const;
};
```

---

## Replication

### FFWItemArray

The inventory's item storage uses `FFWItemArray`, a custom `FFastArraySerializer` implementation that provides delta replication. Only changed items are replicated each network update, significantly reducing bandwidth compared to full array replication.

```cpp
USTRUCT()
struct FFWItemArray : public FFastArraySerializer
{
    GENERATED_BODY()

    UPROPERTY()
    TArray<FFWItemArrayEntry> Items;

    bool NetDeltaSerialize(FNetDeltaSerializeInfo& DeltaParms);
};
```

!!! note "Replication Requirements"
    For delta replication to function:

    1. The owning actor must have `bReplicates = true`.
    2. The inventory component must be added to the replicated subobject list.
    3. The `GetLifetimeReplicatedProps` function must include the item array.

    ```cpp
    void UFWInventoryComponent::GetLifetimeReplicatedProps(
        TArray<FLifetimeProperty>& OutLifetimeProps) const
    {
        Super::GetLifetimeReplicatedProps(OutLifetimeProps);
        DOREPLIFETIME(UFWInventoryComponent, ItemArray);
    }
    ```

---

## Functions

### Item Operations

#### AddItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Inventory")
bool AddItem(const FFWItemInstance& Item);
```

Adds an item to the inventory. If the item is stackable and a matching stack exists with available capacity, the quantity is added to the existing stack. Otherwise, a new slot is used.

| Parameter | Type | Description |
|---|---|---|
| `Item` | `FFWItemInstance` | The item instance to add. |

**Returns:** `true` if the item was successfully added, `false` if the inventory is full.

**Behavior:**

1. If the item definition has `bIsStackable = true`, search for an existing stack of the same definition.
2. If found and the stack has room (current + new <= `MaxStackSize`), add to existing stack.
3. If no room in existing stacks or not stackable, place in the first empty slot.
4. If no empty slots, broadcast `OnInventoryFull` and return `false`.

---

#### RemoveItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Inventory")
bool RemoveItem(const FGuid& InstanceId, int32 Quantity = 1);
```

Removes a quantity of items from the inventory by instance ID.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `InstanceId` | `FGuid` | | The unique instance identifier. |
| `Quantity` | `int32` | `1` | Number to remove. If this equals the stack size, the entire slot is cleared. |

**Returns:** `true` if the requested quantity was removed.

---

#### MoveItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Inventory")
bool MoveItem(const FGuid& InstanceId, int32 TargetSlot);
```

Moves an item to a different inventory slot. If the target slot is occupied by a compatible stackable item, the items are merged. If occupied by an incompatible item, the items swap positions.

| Parameter | Type | Description |
|---|---|---|
| `InstanceId` | `FGuid` | The instance to move. |
| `TargetSlot` | `int32` | Zero-based target slot index. |

---

#### EquipItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Inventory")
bool EquipItem(const FGuid& InstanceId);
```

Equips an item from the inventory to the appropriate equipment slot. The slot is determined by the item's `EFWEquipmentType`. If the slot is already occupied, the equipped item is returned to the inventory.

| Parameter | Type | Description |
|---|---|---|
| `InstanceId` | `FGuid` | The instance to equip. |

!!! info "Equipment Component Required"
    This function requires a `UFWEquipmentComponent` on the same actor. If not present, it logs a warning and returns `false`.

---

#### UnequipItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Inventory")
bool UnequipItem(EFWEquipmentType Slot);
```

Unequips the item in the specified equipment slot and returns it to the inventory.

| Parameter | Type | Description |
|---|---|---|
| `Slot` | `EFWEquipmentType` | The equipment slot to clear. |

---

### Queries

#### GetItemAt

```cpp
UFUNCTION(BlueprintPure, Category = "Inventory")
const FFWItemInstance* GetItemAt(int32 SlotIndex) const;
```

Returns the item at the specified slot index, or `nullptr` if the slot is empty.

---

#### FindItemByDefinition

```cpp
UFUNCTION(BlueprintPure, Category = "Inventory")
const FFWItemInstance* FindItemByDefinition(
    const UFWItemDefinition* Definition) const;
```

Finds the first item instance matching the given definition. Returns `nullptr` if not found.

---

#### GetItemCount

```cpp
UFUNCTION(BlueprintPure, Category = "Inventory")
int32 GetItemCount(const UFWItemDefinition* Definition) const;
```

Returns the total quantity of items matching the given definition across all stacks.

---

## Delegates

```cpp
UPROPERTY(BlueprintAssignable, Category = "Inventory|Events")
FOnInventoryChanged OnInventoryChanged;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Events")
FOnItemAdded OnItemAdded;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Events")
FOnItemRemoved OnItemRemoved;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Events")
FOnItemMoved OnItemMoved;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Events")
FOnItemStackChanged OnItemStackChanged;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Events")
FOnInventoryLoaded OnInventoryLoaded;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Events")
FOnInventoryFull OnInventoryFull;

// Bank delegates
UPROPERTY(BlueprintAssignable, Category = "Inventory|Bank")
FOnBankCollectionChanged OnBankCollectionChanged;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Bank")
FOnBankCustomTabChanged OnBankCustomTabChanged;

UPROPERTY(BlueprintAssignable, Category = "Inventory|Bank")
FOnBankLoaded OnBankLoaded;
```

See [Events and Delegates](events-delegates.md) for full delegate signatures.

---

## Usage Example

=== "C++"

    ```cpp
    void AMyCharacter::PickupItem(AActor* ItemActor)
    {
        AWorldItem* WorldItem = Cast<AWorldItem>(ItemActor);
        if (!WorldItem) return;

        FFWItemInstance NewItem;
        NewItem.InstanceId = FGuid::NewGuid();
        NewItem.Definition = WorldItem->GetItemDefinition();
        NewItem.Quantity = WorldItem->GetQuantity();

        if (Inventory->AddItem(NewItem))
        {
            WorldItem->Destroy();
        }
        else
        {
            // Inventory full notification is handled by OnInventoryFull delegate
        }
    }
    ```

=== "Blueprint"

    1. Add **FW Inventory Component** to your character.
    2. Create an `FFWItemInstance` struct with the item definition and quantity.
    3. Call **Add Item** on the inventory component.
    4. Bind **On Inventory Full** to show a "bags full" notification.
