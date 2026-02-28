---
title: Installation - GAS System
---

# Installation

## Prerequisites

- Unreal Engine 5.4 or later
- C++ project (or a Blueprint project with the plugin pre-compiled)

FWGASSystem requires the following engine plugins to be enabled:

| Plugin | Module | Purpose |
|--------|--------|---------|
| **Gameplay Abilities** | `GameplayAbilities` | Core GAS framework (ASC, abilities, effects, attributes) |
| **Modular Gameplay** | `ModularGameplay` | Component-based game framework initialization |
| **Game Features** | `GameFeatures` | GameFeature plugin/action system |
| **Enhanced Input** | `EnhancedInput` | UE5 input system for ability input binding |

---

## Step 1: Enable Engine Dependencies

Open **Edit > Plugins** and enable all four engine plugins if they are not already active:

- Gameplay Abilities
- Modular Gameplay
- Game Features
- Enhanced Input

---

## Step 2: Add the Plugin

=== "From Source"

    Copy the `FWGASSystem/` folder into your project's `Plugins/` directory:

    ```
    YourProject/
      Plugins/
        FWGASSystem/
          FWGASSystem.uplugin
          Source/
          Content/
    ```

=== "From Marketplace"

    Install FWGASSystem from the Unreal Engine Marketplace. The plugin and its dependencies will be configured automatically.

---

## Step 3: Enable the Plugin

Add it to your `.uproject` file or enable via the Plugins browser:

```json
{
  "Plugins": [
    {
      "Name": "FWGASSystem",
      "Enabled": true
    },
    {
      "Name": "GameplayAbilities",
      "Enabled": true
    },
    {
      "Name": "ModularGameplay",
      "Enabled": true
    },
    {
      "Name": "GameFeatures",
      "Enabled": true
    },
    {
      "Name": "EnhancedInput",
      "Enabled": true
    }
  ]
}
```

Restart the editor after enabling.

---

## Step 4: Add Module Dependencies

Add the module dependency in your game's `Build.cs`:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "GameplayAbilities",
    "GameplayTags",
    "GameplayTasks",
    "FWGASSystem"
});
```

!!! tip "Optional Dependencies"
    If you only use FWGASSystem through Blueprints, you can add it as a private dependency instead. Public dependency is required only when you reference FWGASSystem types in your public headers.

---

## Step 5: Configure the Gameplay Ability System

If this is a fresh project, ensure GAS is properly configured:

### DefaultGame.ini

```ini
[/Script/GameplayAbilities.AbilitySystemGlobals]
+GameplayCueNotifyPaths=/Game/GAS/Cues
bUseDebugTargetFromHud=true
```

### AbilitySystemGlobals Initialization

In your GameInstance or GameMode, call the global init:

```cpp
#include "AbilitySystemGlobals.h"

void UMyGameInstance::Init()
{
    Super::Init();
    UAbilitySystemGlobals::Get().InitGlobalData();
}
```

!!! warning "InitGlobalData"
    Forgetting to call `InitGlobalData()` is the most common cause of GAS-related crashes. Always call it during game initialization, before any abilities are activated.

---

## Step 6: Verify Installation

1. Open the Content Browser and navigate to any Blueprint.
2. Add an **FW Ability System Component** via the Components panel.
3. You should see the component's `GrantedAbilities`, `GrantedAttributes`, `GrantedEffects`, and `GrantedAbilitySets` properties in the Details panel.
4. Search for **FW GAS Core Component** -- this companion component should also be available.

If both components appear, the plugin is installed correctly.

---

## Loading Phase Note

FWGASSystem uses `PreDefault` loading phase. This means it initializes before most gameplay modules, ensuring that GAS types and modular gameplay registrations are available when your game module loads. You do not need to manage loading order in your project.
