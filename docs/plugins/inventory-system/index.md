---
title: FWInventorySystem Plugin
description: Comprehensive inventory and equipment system with instanced items, equipment slots, crafting, vendors, bank storage, and serialization for Unreal Engine 5.
tags:
  - plugin
  - inventory
  - equipment
  - loot
  - crafting
---

# FWInventorySystem Plugin

**Version:** 1.0 | **Module:** `FWInventorySystem` | **Type:** Runtime

A comprehensive inventory and equipment system for Unreal Engine 5 featuring item definitions as data assets, instanced items with deterministic seed-based generation, equipment slot management, crafting, vendor NPCs, bank storage with collection and custom tabs, and full JSON serialization for persistence. Built on `FFastArraySerializer` for bandwidth-efficient delta replication.

---

## Feature Overview

| Feature | Description |
|---|---|
| Inventory Management | Add, remove, move, and stack items with delta-replicated arrays |
| Equipment System | 13 standard equipment slots with automatic stat application |
| Item Definitions | DataAsset-based definitions with full property support |
| Item Instances | Runtime instances with unique IDs, quality tiers, affixes, and augments |
| Deterministic Generation | Seed-based item generation using the Mint system for consistent results |
| Crafting | Recipe-based crafting with material requirements and skill integration |
| Vendor System | NPC vendor component with buy, sell, and buyback functionality |
| Bank Storage | 6 Collection Tabs and 4 Custom Tabs for organized storage |
| Item Sets | Set bonus definitions with tiered thresholds |
| Perks | Passive perk system tied to item properties |
| Serialization | JSON save/load for database persistence |
| Blueprint Extensible | All systems accessible from Blueprints |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| `GameplayAbilities` | **Required** | Used for stat application via Gameplay Effects and Attribute Sets |
| `FWSkillSystem` | **Optional** | Enables skill-gated crafting recipes and item level requirements |

!!! info "GameplayAbilities Setup"
    FWInventorySystem uses GAS for applying equipment stats. Ensure your project has `GameplayAbilities` enabled and your characters have an `AbilitySystemComponent`. Equipment stats are applied as infinite-duration Gameplay Effects.

---

## Core Concepts

### Item Instance System

Items exist as lightweight **definitions** (data assets) and runtime **instances**. When an item enters the game world, an instance is created from the definition using the **Mint** system, which generates affixes, quality, and stats from a deterministic seed. The same seed always produces the same item.

### Equipment Slots

The system provides 13 standard equipment slots covering head, chest, legs, feet, hands, shoulders, back, main hand, off hand, two rings, an amulet, and a belt. Equipment automatically applies and removes stat effects via GAS.

### Banking System

Players have access to **6 Collection Tabs** (organized by item type) and **4 Custom Tabs** (player-named, for personal organization). Bank operations are fully replicated and serialized.

---

## Quick Start

### 1. Enable the Plugin

Enable `FWInventorySystem` and `GameplayAbilities` in your `.uproject` file or Plugin Manager.

### 2. Create an Item Definition

1. Right-click in the Content Browser.
2. Select **Miscellaneous > Data Asset**.
3. Choose `FWItemDefinition` as the class.
4. Configure item properties (type, quality, stats, affix pool).

### 3. Add Inventory and Equipment Components

=== "C++"

    ```cpp
    // Add to your Character or PlayerState
    Inventory = CreateDefaultSubobject<UFWInventoryComponent>(TEXT("Inventory"));
    Equipment = CreateDefaultSubobject<UFWEquipmentComponent>(TEXT("Equipment"));
    ```

=== "Blueprint"

    Add **FW Inventory Component** and **FW Equipment Component** to your Character or Player State.

### 4. Add an Item

```cpp
FFWItemInstance NewSword = UFWInventorySubsystem::MintItem(SwordDefinition, Seed);
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
    The inventory uses `FFastArraySerializer` for delta replication. This requires the owning actor to be replicated. Ensure your character or player state has `bReplicates = true` and that the inventory component is registered as a replicated subobject.
