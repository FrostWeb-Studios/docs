---
title: Installation - FWQuestSystem
description: Install and configure the FWQuestSystem plugin in your Unreal Engine 5 project.
---

# Installation

This page covers how to install and enable FWQuestSystem in your Unreal Engine 5 project.

---

## Prerequisites

| Requirement | Minimum Version |
|---|---|
| Unreal Engine | 5.3 or later |
| Project Type | C++ (Blueprint-only projects are not supported) |
| GameplayAbilities | Must be enabled |

!!! note "GameplayAbilities Dependency"
    FWQuestSystem uses Gameplay Effects for reward delivery and Gameplay Tags for quest event matching. Ensure the GameplayAbilities plugin is enabled in your project before installing FWQuestSystem.

---

## Step 1 -- Copy the Plugin

Copy the `FWQuestSystem/` folder into your project's `Plugins/` directory:

```
YourProject/
    Plugins/
        FWQuestSystem/
            FWQuestSystem.uplugin
            Source/
            Content/
```

---

## Step 2 -- Enable the Plugin

Add the plugin and its dependency to your `.uproject` file:

```json
{
    "Plugins": [
        {
            "Name": "FWQuestSystem",
            "Enabled": true
        },
        {
            "Name": "GameplayAbilities",
            "Enabled": true
        }
    ]
}
```

Alternatively, enable each plugin through **Edit > Plugins** in the Unreal Editor and restart.

---

## Step 3 -- Regenerate Project Files

Right-click your `.uproject` file and select **Generate Visual Studio project files**. This ensures your build system picks up the new modules.

---

## Step 4 -- Add Module Dependencies

In your game module's `Build.cs`, add the required module dependencies:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "FWQuestSystem",
    "GameplayAbilities",
    "GameplayTags",
    "GameplayTasks"
});
```

---

## Step 5 -- Configure Gameplay Tags

FWQuestSystem ships with a default set of Gameplay Tags for quest events. Import them by adding the tag table to your project's `DefaultGameplayTags.ini`:

```ini
[/Script/GameplayTags.GameplayTagsSettings]
+GameplayTagTableList=/FWQuestSystem/Data/DT_FWQuestTags.DT_FWQuestTags
```

Alternatively, define your own tags. The following tags are referenced internally:

| Tag | Purpose |
|---|---|
| `Quest.Event.EnemyKilled` | Dispatched by `OnEnemyKilled` |
| `Quest.Event.LocationReached` | Dispatched by `OnLocationReached` |
| `Quest.Event.NPCInteracted` | Dispatched by `OnNPCInteracted` |
| `Quest.Event.ItemCollected` | Dispatched by `ProcessQuestEvent` with item data |
| `Quest.Event.TimerExpired` | Internal timer task expiration |

---

## Step 6 -- Verify Installation

Build your project and check the Output Log for:

```
LogModuleManager: Loading module FWQuestSystem
```

!!! tip "Troubleshooting"
    If the module fails to load:

    - Verify the plugin folder name matches `FWQuestSystem` exactly.
    - Confirm `GameplayAbilities` is enabled and compiles successfully.
    - Regenerate project files and perform a full rebuild.
    - Check the Output Log for unresolved symbol errors, which typically indicate a missing `Build.cs` dependency.
    - If you see Gameplay Tag errors, verify the tag table is correctly referenced or create the required tags manually.

---

## Include Paths

After installation, you can include plugin headers in your C++ code:

```cpp
#include "FWQuestManagerComponent.h"
#include "Data/FWQuestDefinition.h"
#include "Data/FWQuestDatabase.h"
#include "Subsystems/FWQuestStateSubsystem.h"
#include "Types/FWQuestTypes.h"

// Tasks
#include "Tasks/FWQuestTaskBase.h"
#include "Tasks/FWTask_Slay.h"
#include "Tasks/FWTask_Travel.h"
#include "Tasks/FWTask_Collect.h"

// Conditions
#include "Conditions/FWQuestConditionBase.h"
#include "Conditions/FWCondition_QuestComplete.h"
#include "Conditions/FWCondition_Level.h"

// Rewards
#include "Rewards/FWQuestRewardBase.h"
#include "Rewards/FWReward_Experience.h"
#include "Rewards/FWReward_Item.h"
```

---

## Next Steps

- Follow the [Quick Start](quick-start.md) to get a quest system running in under 10 minutes.
- Read the [Data Definitions](data-definitions.md) guide to understand quest, task, condition, and reward assets.
- See [Configuration](configuration.md) for reset schedules, serialization, and advanced settings.
