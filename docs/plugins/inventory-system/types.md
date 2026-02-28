---
title: Inventory Types Reference
description: Complete reference for all enums, structs, and type definitions in FWInventoryTypes.h.
---

# Inventory Types Reference

**Header:** `FWInventoryTypes.h`

All type definitions used by the FWInventorySystem plugin. Every struct is marked `BlueprintType` for full Blueprint compatibility.

---

## Enums

### EFWItemType

Categorizes items by their fundamental behavior.

```cpp
UENUM(BlueprintType)
enum class EFWItemType : uint8
{
    Equipment,    // Equippable gear (weapons, armor, accessories)
    Consumable,   // Single-use items (potions, food, scrolls)
    Material,     // Crafting materials
    Quest,        // Quest-related items (non-tradeable)
    Currency,     // Currency tokens
    Augmentor,    // Items used to augment other items
    Gizmo,        // Perks/gizmos slotted into equipment
    Tool,         // Tools (fishing rod, pickaxe, etc.)
    Container     // Containers that hold other items
};
```

---

### EFWEquipmentType

Defines the equipment slot an item occupies.

```cpp
UENUM(BlueprintType)
enum class EFWEquipmentType : uint8
{
    MeleeWeapon,
    RangedWeapon,
    Shield,
    Helmet,
    Chest,
    Legs,
    Boots,
    Gloves,
    Ring,
    Amulet,
    Cloak,
    Belt,
    Bag
};
```

| Slot | Description | Typical Stats |
|---|---|---|
| `MeleeWeapon` | Main-hand melee weapon | Damage, Strength, CritChance |
| `RangedWeapon` | Main-hand ranged weapon | Damage, Dexterity, Range |
| `Shield` | Off-hand defensive | Defense, Block, Vitality |
| `Helmet` | Head armor | Defense, Intelligence, MagicResist |
| `Chest` | Body armor | Defense, Vitality, MaxHP |
| `Legs` | Leg armor | Defense, Vitality |
| `Boots` | Foot armor | Defense, MoveSpeed |
| `Gloves` | Hand armor | Defense, AttackSpeed, CritChance |
| `Ring` | Finger accessory | Various stats |
| `Amulet` | Neck accessory | Various stats |
| `Cloak` | Back slot | Defense, MagicResist, Stealth |
| `Belt` | Waist slot | Vitality, PotionSlots |
| `Bag` | Inventory expansion | InventorySlots |

---

### EFWItemQuality

Item rarity/quality tier, affecting visuals, stats, and vendor pricing.

```cpp
UENUM(BlueprintType)
enum class EFWItemQuality : uint8
{
    Poor,       // Grey
    Common,     // White
    Uncommon,   // Green
    Rare,       // Blue
    Epic,       // Purple
    Legendary,  // Orange
    Mythic,     // Red
    Artifact    // Gold
};
```

| Quality | Color | Max Affixes | Vendor Multiplier |
|---|---|---|---|
| Poor | Grey | 0 | 0.5x |
| Common | White | 0 | 1.0x |
| Uncommon | Green | 1 | 1.5x |
| Rare | Blue | 2 | 2.5x |
| Epic | Purple | 3 | 5.0x |
| Legendary | Orange | 4 | 10.0x |
| Mythic | Red | 5 | 25.0x |
| Artifact | Gold | 6 | 50.0x |

---

### EFWItemBinding

Determines when and how an item becomes bound to a player.

```cpp
UENUM(BlueprintType)
enum class EFWItemBinding : uint8
{
    None,           // Freely tradeable
    BindOnEquip,    // Binds when equipped
    BindOnPickup,   // Binds when looted/received
    BindOnUse,      // Binds when consumed
    AccountBound    // Bound to the account, tradeable between characters
};
```

---

### EFWAugmentState

Tracks the augmentation progress of an item instance.

```cpp
UENUM(BlueprintType)
enum class EFWAugmentState : uint8
{
    None,           // Not augmented
    Augmented_1,    // First augmentation tier
    Augmented_2,    // Second augmentation tier
    Augmented_3,    // Third augmentation tier
    Augmented_4,    // Fourth augmentation tier
    Augmented_5,    // Fifth (maximum) augmentation tier
    Failed          // Augmentation failed (item may be degraded)
};
```

!!! info "Augmentation Mechanics"
    Each augmentation tier increases the item's stat values by a percentage. Higher tiers have lower success rates. A failed augmentation may reduce the item's augment level by one tier or reset it entirely, depending on configuration.

---

### EFWInventoryType

Identifies the type of inventory container.

```cpp
UENUM(BlueprintType)
enum class EFWInventoryType : uint8
{
    Backpack,     // Player's main inventory
    Equipment,    // Currently equipped items
    Bank,         // Personal bank storage
    GuildBank,    // Shared guild bank
    Trade,        // Trade window
    Mail,         // Mail attachments
    Loot,         // Loot window from kills/chests
    Vendor,       // Vendor shop inventory
    Crafting      // Crafting material slots
};
```

---

## Structs

### FFWItemInstance

Runtime representation of an individual item. Contains the item's unique identity, current state, and generated properties.

```cpp
USTRUCT(BlueprintType)
struct FFWItemInstance
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    FGuid InstanceId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    TObjectPtr<UFWItemDefinition> Definition;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    int32 Quantity = 1;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    float Durability = -1.0f;  // -1 = indestructible

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    TArray<FFWAffixInstance> Affixes;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    EFWAugmentState AugmentState = EFWAugmentState::None;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    int32 AugmentLevel = 0;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    FString BoundToPlayerId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    FDateTime AcquiredAt;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item")
    int32 ItemLevel = 1;

    // Helpers
    bool IsValid() const { return InstanceId.IsValid() && Definition != nullptr; }
    bool IsBound() const { return !BoundToPlayerId.IsEmpty(); }
    bool HasDurability() const { return Durability >= 0.0f; }
    float GetDurabilityPercent() const;
};
```

| Field | Type | Description |
|---|---|---|
| `InstanceId` | `FGuid` | Globally unique identifier for this specific item instance. |
| `Definition` | `UFWItemDefinition*` | Reference to the item's data asset definition. |
| `Quantity` | `int32` | Stack count. Always 1 for non-stackable items. |
| `Durability` | `float` | Current durability. `-1` means indestructible. |
| `Affixes` | `TArray<FFWAffixInstance>` | Generated affixes with rolled stat values. |
| `AugmentState` | `EFWAugmentState` | Current augmentation tier. |
| `AugmentLevel` | `int32` | Numeric augment level (0-5). |
| `BoundToPlayerId` | `FString` | Player ID this item is bound to. Empty if unbound. |
| `AcquiredAt` | `FDateTime` | Timestamp when the item was first obtained. |
| `ItemLevel` | `int32` | Effective item level (affects affix scaling). |

---

### FFWAffixRule

Defines a single affix that can roll on an item, including stat, value range, weight, and constraints.

```cpp
USTRUCT(BlueprintType)
struct FFWAffixRule
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Affix")
    FGameplayTag StatTag;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Affix")
    float MinValue = 0.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Affix")
    float MaxValue = 0.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Affix")
    bool bIsPercentage = false;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Affix")
    float Weight = 1.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Affix")
    int32 RequiredItemLevel = 0;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Affix")
    FGameplayTagContainer ExclusionTags;
};
```

| Field | Type | Description |
|---|---|---|
| `StatTag` | `FGameplayTag` | The stat this affix modifies (e.g., `Stat.Strength`). |
| `MinValue` | `float` | Minimum rolled value (inclusive). |
| `MaxValue` | `float` | Maximum rolled value (inclusive). |
| `bIsPercentage` | `bool` | Whether the value is a percentage modifier. |
| `Weight` | `float` | Relative weight in the affix pool. Higher = more likely. |
| `RequiredItemLevel` | `int32` | Minimum item level for this affix to be eligible. |
| `ExclusionTags` | `FGameplayTagContainer` | Tags of other affixes that cannot appear alongside this one. |

!!! example "Affix Rule Example"
    ```cpp
    FFWAffixRule StrengthAffix;
    StrengthAffix.StatTag = FGameplayTag::RequestGameplayTag(TEXT("Stat.Strength"));
    StrengthAffix.MinValue = 5.0f;
    StrengthAffix.MaxValue = 25.0f;
    StrengthAffix.bIsPercentage = false;
    StrengthAffix.Weight = 10.0f;
    StrengthAffix.RequiredItemLevel = 1;
    // No exclusion tags
    ```

---

### FFWAffixInstance

A rolled affix instance on an item, with the concrete value determined by the affix rule's range.

```cpp
USTRUCT(BlueprintType)
struct FFWAffixInstance
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Affix")
    FGameplayTag StatTag;

    UPROPERTY(BlueprintReadOnly, Category = "Affix")
    float Value;

    UPROPERTY(BlueprintReadOnly, Category = "Affix")
    bool bIsPercentage;

    UPROPERTY(BlueprintReadOnly, Category = "Affix")
    FGameplayTag AffixId;
};
```

---

### FFWRecipeIngredient

A single material requirement for a crafting recipe.

```cpp
USTRUCT(BlueprintType)
struct FFWRecipeIngredient
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Crafting")
    TObjectPtr<UFWItemDefinition> ItemDefinition;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Crafting")
    int32 Quantity = 1;
};
```

---

### FFWEquipmentSlotConfig

Configuration for a single equipment slot, allowing customization of slot behavior.

```cpp
USTRUCT(BlueprintType)
struct FFWEquipmentSlotConfig
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Equipment")
    EFWEquipmentType SlotType;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Equipment")
    bool bIsEnabled = true;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Equipment")
    int32 SlotCount = 1;  // >1 for slots like Ring (2 ring slots)
};
```

---

## Type Relationships

```
UFWItemDefinition (Data Asset)
    |
    +-- EFWItemType (categorization)
    +-- EFWEquipmentType (slot mapping)
    +-- EFWItemQuality (rarity)
    +-- EFWItemBinding (trade restrictions)
    +-- TArray<FFWAffixRule> (affix pool)
    |
    v
FFWItemInstance (Runtime)
    |
    +-- FGuid InstanceId (unique)
    +-- int32 Quantity (stack)
    +-- float Durability (wear)
    +-- TArray<FFWAffixInstance> (rolled affixes)
    +-- EFWAugmentState (upgrade tier)
    +-- FString BoundToPlayerId (binding)
```
