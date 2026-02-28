---
title: UFWEquipmentComponent
description: Equipment slot management component with typed slots, stat application via Gameplay Effects, and set bonus tracking.
---

# UFWEquipmentComponent

**Header:** `FWEquipmentComponent.h` | **Parent:** `UActorComponent`

Manages equipment slots, applies item stats via Gameplay Effects, and tracks item set bonuses. Works in conjunction with `UFWInventoryComponent` for the equip/unequip flow.

---

## Class Declaration

```cpp
UCLASS(ClassGroup = (FrostWeb), meta = (BlueprintSpawnableComponent))
class FWINVENTORYSYSTEM_API UFWEquipmentComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFWEquipmentComponent();

    // --- Slot Management ---
    UFUNCTION(BlueprintCallable, Category = "Equipment")
    bool Equip(const FFWItemInstance& Item);

    UFUNCTION(BlueprintCallable, Category = "Equipment")
    FFWItemInstance Unequip(EFWEquipmentType Slot);

    UFUNCTION(BlueprintPure, Category = "Equipment")
    const FFWItemInstance* GetEquippedItem(EFWEquipmentType Slot) const;

    UFUNCTION(BlueprintPure, Category = "Equipment")
    bool IsSlotOccupied(EFWEquipmentType Slot) const;

    UFUNCTION(BlueprintPure, Category = "Equipment")
    TArray<EFWEquipmentType> GetAllSlotTypes() const;

    UFUNCTION(BlueprintPure, Category = "Equipment")
    TMap<EFWEquipmentType, FFWItemInstance> GetAllEquippedItems() const;

    // --- Stats ---
    UFUNCTION(BlueprintPure, Category = "Equipment")
    float GetTotalStatValue(FGameplayTag StatTag) const;

    UFUNCTION(BlueprintCallable, Category = "Equipment")
    void RecalculateStats();

    // --- Set Bonuses ---
    UFUNCTION(BlueprintPure, Category = "Equipment")
    TArray<UFWItemSetDefinition*> GetActiveSetBonuses() const;

    UFUNCTION(BlueprintPure, Category = "Equipment")
    int32 GetSetPieceCount(const UFWItemSetDefinition* SetDefinition) const;

    // --- Delegates ---
    UPROPERTY(BlueprintAssignable, Category = "Equipment|Events")
    FOnItemEquipped OnItemEquipped;

    UPROPERTY(BlueprintAssignable, Category = "Equipment|Events")
    FOnItemUnequipped OnItemUnequipped;

    UPROPERTY(BlueprintAssignable, Category = "Equipment|Events")
    FOnEquipmentStatsChanged OnEquipmentStatsChanged;

    UPROPERTY(BlueprintAssignable, Category = "Equipment|Events")
    FOnSetBonusActivated OnSetBonusActivated;

    UPROPERTY(BlueprintAssignable, Category = "Equipment|Events")
    FOnSetBonusDeactivated OnSetBonusDeactivated;
};
```

---

## Equipment Slots

The equipment component supports 13 typed slots:

| Slot | Enum Value | Typical Item |
|---|---|---|
| Melee Weapon | `EFWEquipmentType::MeleeWeapon` | Swords, axes, maces |
| Ranged Weapon | `EFWEquipmentType::RangedWeapon` | Bows, crossbows, wands |
| Shield | `EFWEquipmentType::Shield` | Shields, off-hand focuses |
| Helmet | `EFWEquipmentType::Helmet` | Helms, hoods, crowns |
| Chest | `EFWEquipmentType::Chest` | Breastplates, robes |
| Legs | `EFWEquipmentType::Legs` | Greaves, leggings |
| Boots | `EFWEquipmentType::Boots` | Boots, shoes, sabatons |
| Gloves | `EFWEquipmentType::Gloves` | Gauntlets, gloves |
| Ring | `EFWEquipmentType::Ring` | Rings |
| Amulet | `EFWEquipmentType::Amulet` | Necklaces, amulets |
| Cloak | `EFWEquipmentType::Cloak` | Cloaks, capes |
| Belt | `EFWEquipmentType::Belt` | Belts, sashes |
| Bag | `EFWEquipmentType::Bag` | Bags (increases inventory capacity) |

!!! tip "Multiple Ring Slots"
    By default the system supports one ring slot. To support multiple rings (e.g., two ring slots), configure additional slots in the equipment component settings. See [Configuration](configuration.md).

---

## Functions

### Equip

```cpp
UFUNCTION(BlueprintCallable, Category = "Equipment")
bool Equip(const FFWItemInstance& Item);
```

Equips an item to its matching slot based on `Item.Definition->EquipmentType`. If the slot is occupied, the currently equipped item is returned via the `OnItemUnequipped` delegate before the new item is equipped.

| Parameter | Type | Description |
|---|---|---|
| `Item` | `FFWItemInstance` | The item instance to equip. Must have `EFWItemType::Equipment`. |

**Returns:** `true` if equipped successfully.

**Side effects:**

- Applies the item's base stats and affix stats as Gameplay Effects.
- Updates set bonus tracking.
- Broadcasts `OnItemEquipped` and `OnEquipmentStatsChanged`.

---

### Unequip

```cpp
UFUNCTION(BlueprintCallable, Category = "Equipment")
FFWItemInstance Unequip(EFWEquipmentType Slot);
```

Removes the item from the specified slot and returns it.

| Parameter | Type | Description |
|---|---|---|
| `Slot` | `EFWEquipmentType` | The slot to clear. |

**Returns:** The unequipped item instance. Returns a default-constructed instance if the slot was empty.

**Side effects:**

- Removes the Gameplay Effect for the item's stats.
- Updates set bonus tracking (may deactivate a set bonus).
- Broadcasts `OnItemUnequipped` and `OnEquipmentStatsChanged`.

---

### GetEquippedItem

```cpp
UFUNCTION(BlueprintPure, Category = "Equipment")
const FFWItemInstance* GetEquippedItem(EFWEquipmentType Slot) const;
```

Returns the item currently in the specified slot, or `nullptr` if empty.

---

### GetTotalStatValue

```cpp
UFUNCTION(BlueprintPure, Category = "Equipment")
float GetTotalStatValue(FGameplayTag StatTag) const;
```

Returns the total value of a stat across all equipped items, including base stats, affix stats, and active set bonuses.

| Parameter | Type | Description |
|---|---|---|
| `StatTag` | `FGameplayTag` | The gameplay tag identifying the stat (e.g., `Stat.Strength`). |

---

### RecalculateStats

```cpp
UFUNCTION(BlueprintCallable, Category = "Equipment")
void RecalculateStats();
```

Forces a full recalculation of all equipment stats and set bonuses. This is called automatically when items are equipped or unequipped, but can be called manually if item properties change (e.g., after augmentation).

---

## Stat Application

Equipment stats are applied to the character's `AbilitySystemComponent` as infinite-duration Gameplay Effects.

### Application Flow

```
Item Equipped
    |
    v
Read base stats from UFWItemDefinition
    |
    v
Read affix stats from FFWItemInstance::Affixes
    |
    v
Combine into a dynamic Gameplay Effect
    |
    v
Apply via AbilitySystemComponent->ApplyGameplayEffectToSelf()
    |
    v
Store effect handle for removal on unequip
```

### Stat Tags

Stats are identified by `FGameplayTag` values. Common examples:

| Tag | Description |
|---|---|
| `Stat.Strength` | Physical damage bonus |
| `Stat.Intelligence` | Magic damage bonus |
| `Stat.Vitality` | Health bonus |
| `Stat.Defense` | Physical damage reduction |
| `Stat.CriticalChance` | Critical hit probability |
| `Stat.CriticalDamage` | Critical hit damage multiplier |
| `Stat.AttackSpeed` | Attack speed modifier |
| `Stat.MoveSpeed` | Movement speed modifier |

---

## Set Bonuses

The equipment component automatically tracks item set pieces and activates/deactivates set bonuses.

```cpp
// Check active sets
TArray<UFWItemSetDefinition*> ActiveSets = Equipment->GetActiveSetBonuses();
for (UFWItemSetDefinition* Set : ActiveSets)
{
    int32 PieceCount = Equipment->GetSetPieceCount(Set);
    UE_LOG(LogInventory, Log, TEXT("Set '%s': %d pieces equipped"),
        *Set->SetName.ToString(), PieceCount);
}
```

See [Data Assets](data-assets.md) for `UFWItemSetDefinition` details.

---

## Usage Example

=== "C++"

    ```cpp
    void AMyCharacter::BeginPlay()
    {
        Super::BeginPlay();

        Equipment->OnItemEquipped.AddDynamic(
            this, &AMyCharacter::HandleItemEquipped);
        Equipment->OnSetBonusActivated.AddDynamic(
            this, &AMyCharacter::HandleSetBonusActivated);
    }

    void AMyCharacter::HandleItemEquipped(
        EFWEquipmentType Slot, const FFWItemInstance& Item)
    {
        UE_LOG(LogInventory, Log, TEXT("Equipped '%s' in slot %s"),
            *Item.Definition->DisplayName.ToString(),
            *UEnum::GetValueAsString(Slot));

        // Update character mesh/visuals
        UpdateEquipmentVisuals(Slot, Item);
    }

    void AMyCharacter::HandleSetBonusActivated(
        const UFWItemSetDefinition* SetDef, int32 PieceCount)
    {
        UE_LOG(LogInventory, Log, TEXT("Set bonus '%s' activated at %d pieces"),
            *SetDef->SetName.ToString(), PieceCount);
    }
    ```

=== "Blueprint"

    1. Add **FW Equipment Component** to your character.
    2. Bind **On Item Equipped** to update your character's visual mesh.
    3. Bind **On Equipment Stats Changed** to refresh the stat display UI.
    4. Use **Get Total Stat Value** with a gameplay tag to read current equipment stats.
