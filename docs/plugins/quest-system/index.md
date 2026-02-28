---
title: FWQuestSystem Plugin
description: Modular, Blueprint-extensible quest system with DataAsset-based definitions, instanced tasks, conditions, rewards, party support, and daily/weekly resets for Unreal Engine 5.
tags:
  - plugin
  - quest
  - gameplay
  - rpg
---

# FWQuestSystem Plugin

**Version:** 3.0 | **Module:** `FWQuestSystem` | **Type:** Runtime

A modular, Blueprint-extensible quest system for Unreal Engine 5 built on DataAsset-based definitions. FWQuestSystem provides instanced task tracking, composable conditions, configurable rewards, party synchronization modes, daily and weekly reset cycles, and full JSON serialization for persistence.

---

## Feature Overview

| Feature | Description |
|---|---|
| DataAsset Definitions | Define quests, tasks, conditions, and rewards as `UPrimaryDataAsset` entries |
| Instanced Tasks | 7 built-in task types (Slay, Travel, Interact, TalkTo, Collect, Timer, Custom) with runtime progress |
| Composable Conditions | 9 condition types with boolean combinators (And, Or) for complex prerequisite chains |
| Configurable Rewards | 7 reward types (XP, Items, Currency, Reputation, Skill XP, Unlocks, Custom) |
| Quest Categories | 10 categories: MainStory, Side, Daily, Weekly, World, Dungeon, Guild, Event, Profession, Challenge |
| Repeat Types | None, Cooldown, Daily, Weekly, Unlimited -- with automatic reset scheduling |
| Party Support | 4 party modes: None, ShareOnly, SyncProgress, RequireAll |
| Quest Priorities | Low, Normal, High, Urgent -- for UI sorting and notification filtering |
| JSON Serialization | Full quest state serialization for save/load and server persistence |
| World Subsystem | `UFWQuestStateSubsystem` manages quest state at the world level |
| Blueprint Extensible | All task, condition, and reward types can be subclassed in Blueprints |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| `GameplayAbilities` | **Required** | Used for reward application and condition evaluation via Gameplay Effects and Tags |

!!! info "GameplayAbilities Usage"
    FWQuestSystem uses Gameplay Effects for reward delivery (XP grants, stat modifications) and Gameplay Tags for condition matching and quest event filtering. Your project must have the GameplayAbilities plugin enabled.

---

## Plugin Architecture

```
FWQuestSystem
+-- Components/
|   +-- UFWQuestManagerComponent         // Blueprint-facing quest operations
+-- Data/
|   +-- UFWQuestDefinition               // Quest definition (PrimaryDataAsset)
|   +-- UFWQuestDatabase                 // Collection of quest definitions
+-- Subsystems/
|   +-- UFWQuestStateSubsystem           // World subsystem for state persistence
+-- Tasks/
|   +-- UFWQuestTaskBase                 // Abstract task base class
|   +-- UFWTask_Slay                     // Kill X enemies
|   +-- UFWTask_Travel                   // Reach a location
|   +-- UFWTask_Interact                 // Interact with an object
|   +-- UFWTask_TalkTo                   // Talk to an NPC
|   +-- UFWTask_Collect                  // Collect X items
|   +-- UFWTask_Timer                    // Complete within time limit
|   +-- UFWTask_Custom                   // Blueprint-defined task logic
+-- Conditions/
|   +-- UFWQuestConditionBase            // Abstract condition base class
|   +-- UFWCondition_QuestComplete       // Requires a quest to be completed
|   +-- UFWCondition_Level               // Requires minimum player level
|   +-- UFWCondition_HasItem             // Requires item in inventory
|   +-- UFWCondition_Reputation          // Requires faction reputation threshold
|   +-- UFWCondition_Time                // Requires specific time window
|   +-- UFWCondition_And                 // All child conditions must pass
|   +-- UFWCondition_Or                  // Any child condition must pass
|   +-- UFWCondition_Custom              // Blueprint-defined condition logic
+-- Rewards/
|   +-- UFWQuestRewardBase               // Abstract reward base class
|   +-- UFWReward_Experience             // Grants XP
|   +-- UFWReward_Item                   // Grants items
|   +-- UFWReward_SkillXP               // Grants skill-specific XP
|   +-- UFWReward_Reputation             // Grants faction reputation
|   +-- UFWReward_Currency               // Grants currency
|   +-- UFWReward_Unlock                 // Unlocks content (recipes, areas, etc.)
|   +-- UFWReward_Custom                 // Blueprint-defined reward logic
+-- Types/
    +-- FWQuestTypes.h                   // Enums, structs, delegates
```

---

## Quick Start

### 1. Enable the Plugin

In your `.uproject` file or via the Plugin Manager, enable `FWQuestSystem` and `GameplayAbilities`.

### 2. Create a Quest Database

1. In the Content Browser, create a Data Asset of type `FWQuestDatabase`.
2. Name it `DA_QuestDatabase`.

### 3. Create a Quest Definition

1. Create a Data Asset of type `FWQuestDefinition`.
2. Add tasks, conditions, and rewards in the Details panel.
3. Add the definition to your `DA_QuestDatabase`.

### 4. Add the Quest Manager Component

=== "C++"

    ```cpp
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Quest")
    TObjectPtr<UFWQuestManagerComponent> QuestManager;

    // In constructor
    QuestManager = CreateDefaultSubobject<UFWQuestManagerComponent>(TEXT("QuestManager"));
    QuestManager->SetQuestDatabase(QuestDB);
    ```

=== "Blueprint"

    Add **FW Quest Manager Component** to your Player Controller or Pawn and set the **Quest Database** property.

### 5. Accept and Track a Quest

```cpp
QuestManager->AcceptQuest(QuestDefinition);
QuestManager->OnQuestAccepted.AddDynamic(this, &AMyController::HandleQuestAccepted);
QuestManager->OnTaskProgressChanged.AddDynamic(this, &AMyController::HandleTaskProgress);
```

---

## File Reference

| Page | Description |
|---|---|
| [Installation](installation.md) | Add the plugin to your project and configure dependencies |
| [Quick Start](quick-start.md) | Get a quest system running in under 10 minutes |
| [Quest Manager Component](quest-manager-component.md) | Primary API for quest operations |
| [Data Definitions](data-definitions.md) | Quest definitions, databases, tasks, conditions, and rewards |
| [Types Reference](types.md) | Enums, structs, and delegate definitions |
| [API Reference](api-reference.md) | Complete function and delegate reference |
| [Configuration](configuration.md) | Plugin settings, reset schedules, and serialization options |
| [Events and Delegates](events-delegates.md) | All multicast delegates and event flow diagrams |
| [Tutorial: Building a Quest Chain](tutorial.md) | Step-by-step quest chain implementation |
| [Changelog](changelog.md) | Version history and release notes |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |

!!! warning "Server Authority"
    Quest state mutations (accept, abandon, turn in, fail) should be validated server-side. The `UFWQuestManagerComponent` supports server RPCs for all mutating operations. Client-only usage is supported for single-player games but is not recommended for multiplayer titles.
