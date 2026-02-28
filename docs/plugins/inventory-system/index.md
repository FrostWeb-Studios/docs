---
title: FWInventorySystem Plugin
description: Comprehensive inventory and equipment system with RNG affixes, augmentation, crafting, and database serialization for Unreal Engine 5.
tags:
  - plugin
  - inventory
  - equipment
  - loot
  - crafting
---

# FWInventorySystem Plugin

**Version:** 1.0 | **Module:** `FWInventorySystem` | **Type:** Runtime

A comprehensive inventory and equipment system for Unreal Engine 5 featuring item definitions as data assets, instanced items with RNG affixes, equipment slot management, crafting, vendor NPCs, bank storage, and full JSON serialization for database persistence. Built on `FFastArraySerializer` for bandwidth-efficient delta replication.

---

## Feature Overview

| Feature | Description |
|---|---|
| Inventory Management | Add, remove, move, and stack items with delta-replicated arrays |
| Equipment System | Typed equipment slots with automatic stat application |
| Item Definitions | `PrimaryDataAsset`-based definitions with full property support |
| Item Instances | Runtime instances with unique IDs, durability, affixes, and augments |
| RNG Affix System | Weighted affix pools with min/max ranges, exclusion tags, and item level requirements |
| Augmentation | Multi-stage item augmentation with success/failure states |
| Crafting | Recipe-based crafting with material requirements and skill integration |
| Vendor System | NPC vendor component with buy/sell/buyback functionality |
| Bank Storage | Multi-tab bank with personal and guild bank support |
| Serialization | JSON save/load library for database persistence |
| Item Sets | Set bonus definitions with tiered thresholds |
| Perks | Passive perk system tied to item properties |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| `GameplayAbilities` | **Required** | Used for stat application via Gameplay Effects and Attribute Sets. |
| `FWSkillSystem` | **Optional** | Enables skill-gated crafting recipes and item level requirements. |

!!! info "GameplayAbilities Setup"
    FWInventorySystem uses GAS for applying equipment stats. Ensure your project has `GameplayAbilities` enabled and your characters have an `AbilitySystemComponent`. Equipment stats are applied as infinite-duration Gameplay Effects.

---

## Plugin Architecture

```
FWInventorySystem
├── Components/
│   ├── UFWInventoryComponent         // Core inventory with replication
│   ├── UFWEquipmentComponent         // Equipment slot management
│   ├── UFWCraftingComponent          // Recipe-based crafting
│   └── UFWVendorComponent            // NPC vendor functionality
├── Data/
│   ├── UFWItemDefinition             // PrimaryDataAsset for items
│   ├── UFWItemSetDefinition          // Set bonus definitions
│   ├── UFWPerkDefinition             // Passive perk data
│   ├── UFWRecipeDefinition           // Crafting recipe data
│   └── UFWRecipeDatabase             // Recipe collection asset
├── Types/
│   ├── FWInventoryTypes.h            // Enums and structs
│   └── FFWItemInstance               // Runtime item instance
├── Serialization/
│   └── UFWInventorySerializationLibrary  // JSON save/load
└── Replication/
    └── FFWItemArray                  // FFastArraySerializer wrapper
```

---

## Quick Start

### 1. Enable the Plugin

Enable `FWInventorySystem` and `GameplayAbilities` in your `.uproject` file or Plugin Manager.

### 2. Create an Item Definition

=== "Editor"

    1. Right-click in the Content Browser.
    2. Select **Miscellaneous > Data Asset**.
    3. Choose `FWItemDefinition` as the class.
    4. Configure item properties (type, quality, stats, affix pool).

=== "C++"

    ```cpp
    // Item definitions are typically created as data assets in the editor.
    // Reference them in code via soft object pointers:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Items")
    TSoftObjectPtr<UFWItemDefinition> IronSwordDefinition;
    ```

### 3. Add Inventory and Equipment Components

```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Inventory")
TObjectPtr<UFWInventoryComponent> Inventory;

UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Inventory")
TObjectPtr<UFWEquipmentComponent> Equipment;
```

### 4. Add an Item

```cpp
UFWItemDefinition* SwordDef = LoadObject<UFWItemDefinition>(
    nullptr, TEXT("/Game/Items/Weapons/DA_IronSword"));

FFWItemInstance NewSword;
NewSword.Definition = SwordDef;
NewSword.Quantity = 1;
NewSword.InstanceId = FGuid::NewGuid();

Inventory->AddItem(NewSword);
```

---

## File Reference

| Page | Description |
|---|---|
| [Inventory Component](inventory-component.md) | Core inventory management with replication |
| [Equipment Component](equipment-component.md) | Equipment slot system and stat application |
| [Crafting Component](crafting-component.md) | Recipe-based crafting system |
| [Vendor Component](vendor-component.md) | NPC vendor buy/sell functionality |
| [Types Reference](types.md) | Enums, structs, and item instance details |
| [API Reference](api-reference.md) | Complete function and delegate reference |
| [Configuration](configuration.md) | Plugin settings and project configuration |
| [Events and Delegates](events-delegates.md) | All inventory events and delegate signatures |
| [Data Assets](data-assets.md) | Item definitions, sets, perks, and recipes |
| [Tutorial: Creating a Loot System](tutorial.md) | Step-by-step implementation guide |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |

!!! warning "Replication"
    `FFWItemArray` uses `FFastArraySerializer` for delta replication. This requires the owning actor to be replicated. Ensure your character or player state has `bReplicates = true` and that the inventory component is registered as a replicated subobject.
