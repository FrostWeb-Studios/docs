---
title: Inventory System Configuration
description: Plugin settings, project configuration, equipment slots, and serialization setup for FWInventorySystem.
---

# Inventory System Configuration

This page covers all configuration options for the FWInventorySystem plugin, including inventory capacity, equipment slots, affix generation, vendor defaults, and serialization settings.

---

## Plugin Settings

Inventory system settings are configured in `DefaultGame.ini` or via **Project Settings > Plugins > FW Inventory System**.

```ini
[/Script/FWInventorySystem.FWInventorySystemSettings]
; Default inventory capacity (number of slots)
DefaultInventoryCapacity=40

; Maximum stack size for stackable items (overridden per item definition)
DefaultMaxStackSize=999

; Enable item durability system
bEnableDurability=true

; Durability loss per use/hit (base value, modified per item)
BaseDurabilityLoss=1.0

; Enable the augmentation system
bEnableAugmentation=true

; Maximum augmentation level
MaxAugmentLevel=5

; Augmentation success rate per level (comma-separated percentages)
AugmentSuccessRates=100,80,60,40,20

; Augmentation failure penalty: None, ReduceByOne, ResetToZero
AugmentFailurePenalty=ReduceByOne

; Enable item binding
bEnableItemBinding=true

; Enable set bonuses
bEnableSetBonuses=true

; Serialization format version
SerializationVersion=1

; Bank configuration
DefaultBankTabCount=1
MaxBankTabCount=8
BankTabCapacity=50

; Vendor defaults
DefaultBuyMultiplier=1.0
DefaultSellMultiplier=0.25
DefaultBuybackSlots=12
```

---

## Settings Reference

### Inventory

| Setting | Type | Default | Description |
|---|---|---|---|
| `DefaultInventoryCapacity` | `int32` | `40` | Default number of inventory slots per character. |
| `DefaultMaxStackSize` | `int32` | `999` | Default maximum stack size. Individual items can override. |

### Durability

| Setting | Type | Default | Description |
|---|---|---|---|
| `bEnableDurability` | `bool` | `true` | Master toggle for the durability system. |
| `BaseDurabilityLoss` | `float` | `1.0` | Base durability lost per use/hit. |

### Augmentation

| Setting | Type | Default | Description |
|---|---|---|---|
| `bEnableAugmentation` | `bool` | `true` | Master toggle for item augmentation. |
| `MaxAugmentLevel` | `int32` | `5` | Maximum augmentation tier. |
| `AugmentSuccessRates` | `FString` | `100,80,60,40,20` | Comma-separated success rates per level (%). |
| `AugmentFailurePenalty` | `FString` | `ReduceByOne` | What happens on failure: `None`, `ReduceByOne`, `ResetToZero`. |

!!! tip "Augmentation Balance"
    The default rates create the following progression:

    | Level | Success Rate | Expected Attempts |
    |---|---|---|
    | 0 -> 1 | 100% | 1 |
    | 1 -> 2 | 80% | 1.25 |
    | 2 -> 3 | 60% | 1.67 |
    | 3 -> 4 | 40% | 2.5 |
    | 4 -> 5 | 20% | 5 |

### Bank

| Setting | Type | Default | Description |
|---|---|---|---|
| `DefaultBankTabCount` | `int32` | `1` | Starting bank tabs per character. |
| `MaxBankTabCount` | `int32` | `8` | Maximum purchasable bank tabs. |
| `BankTabCapacity` | `int32` | `50` | Slots per bank tab. |

### Vendor

| Setting | Type | Default | Description |
|---|---|---|---|
| `DefaultBuyMultiplier` | `float` | `1.0` | Default buy price multiplier. |
| `DefaultSellMultiplier` | `float` | `0.25` | Default sell price multiplier (25% return). |
| `DefaultBuybackSlots` | `int32` | `12` | Default buyback buffer size. |

### Serialization

| Setting | Type | Default | Description |
|---|---|---|---|
| `SerializationVersion` | `int32` | `1` | Version tag included in serialized data for migration. |

---

## Enabling the Plugin

### .uproject File

```json
{
    "Plugins": [
        {
            "Name": "FWInventorySystem",
            "Enabled": true
        },
        {
            "Name": "GameplayAbilities",
            "Enabled": true
        },
        {
            "Name": "FWSkillSystem",
            "Enabled": true
        }
    ]
}
```

### Build.cs Module Dependencies

```csharp
public class MyGame : ModuleRules
{
    public MyGame(ReadOnlyTargetRules Target) : base(Target)
    {
        PublicDependencyModuleNames.AddRange(new string[]
        {
            "Core",
            "CoreUObject",
            "Engine",
            "GameplayAbilities",
            "GameplayTags",
            "GameplayTasks",
            "FWInventorySystem"
        });

        // Optional skill system integration
        PrivateDependencyModuleNames.Add("FWSkillSystem");
    }
}
```

---

## Equipment Slot Configuration

Customize available equipment slots and their counts:

```ini
[/Script/FWInventorySystem.FWInventorySystemSettings]
; Equipment slot configuration (JSON array)
EquipmentSlotConfig=[
    {"SlotType": "MeleeWeapon", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "RangedWeapon", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Shield", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Helmet", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Chest", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Legs", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Boots", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Gloves", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Ring", "bIsEnabled": true, "SlotCount": 2},
    {"SlotType": "Amulet", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Cloak", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Belt", "bIsEnabled": true, "SlotCount": 1},
    {"SlotType": "Bag", "bIsEnabled": true, "SlotCount": 1}
]
```

!!! note "Multiple Slots"
    Setting `SlotCount` > 1 (e.g., 2 ring slots) creates indexed sub-slots. The equipment component manages them as `Ring_0` and `Ring_1` internally. The `EquipItem` function fills the first available sub-slot.

---

## Affix Pool Configuration

Affix pools are defined per item definition in the data asset editor. Global affix settings:

```ini
[/Script/FWInventorySystem.FWInventorySystemSettings]
; Maximum affixes per quality tier (comma-separated: Poor,Common,...,Artifact)
MaxAffixesPerQuality=0,0,1,2,3,4,5,6

; Affix value scaling per item level (multiplier)
AffixValueScalePerLevel=0.02

; Minimum affix count for loot-generated items (0 = quality-dependent only)
MinAffixCount=0
```

---

## Item Level Scaling

Item level affects affix value ranges:

```
Effective Value = BaseValue * (1 + (ItemLevel * AffixValueScalePerLevel))
```

Example with `AffixValueScalePerLevel = 0.02`:

| Item Level | Strength Affix (base 5-25) | Effective Range |
|---|---|---|
| 1 | 5 - 25 | 5.1 - 25.5 |
| 10 | 5 - 25 | 6.0 - 30.0 |
| 50 | 5 - 25 | 10.0 - 50.0 |
| 100 | 5 - 25 | 15.0 - 75.0 |

---

## Gameplay Tag Setup

The inventory system uses Gameplay Tags for stat identification. Add these to your project's tag configuration (`DefaultGameplayTags.ini` or the Tag Manager):

```ini
[/Script/GameplayTags.GameplayTagsSettings]
+GameplayTagList=(Tag="Stat.Strength",DevComment="Physical damage bonus")
+GameplayTagList=(Tag="Stat.Intelligence",DevComment="Magic damage bonus")
+GameplayTagList=(Tag="Stat.Dexterity",DevComment="Ranged/finesse bonus")
+GameplayTagList=(Tag="Stat.Vitality",DevComment="Health bonus")
+GameplayTagList=(Tag="Stat.Defense",DevComment="Physical damage reduction")
+GameplayTagList=(Tag="Stat.MagicResist",DevComment="Magic damage reduction")
+GameplayTagList=(Tag="Stat.CriticalChance",DevComment="Crit probability")
+GameplayTagList=(Tag="Stat.CriticalDamage",DevComment="Crit damage multiplier")
+GameplayTagList=(Tag="Stat.AttackSpeed",DevComment="Attack speed modifier")
+GameplayTagList=(Tag="Stat.MoveSpeed",DevComment="Movement speed modifier")
+GameplayTagList=(Tag="Stat.MaxHP",DevComment="Maximum health")
+GameplayTagList=(Tag="Stat.MaxMP",DevComment="Maximum mana")
```

---

## Logging

The plugin uses the `LogInventory` log category:

```cpp
DECLARE_LOG_CATEGORY_EXTERN(LogInventory, Log, All);
```

```ini
[Core.Log]
LogInventory=Verbose
```

| Verbosity | Content |
|---|---|
| `Error` | Serialization failures, invalid item definitions, replication errors |
| `Warning` | Full inventory rejections, invalid operations, missing GAS component |
| `Log` | Item add/remove/equip operations, crafting results |
| `Verbose` | Delta replication details, affix rolls, price calculations |
