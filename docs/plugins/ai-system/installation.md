---
title: Installation - FWAISystem
---

# Installation

This guide covers enabling FWAISystem in your Unreal Engine 5 project and configuring its optional dependencies.

---

## Prerequisites

- Unreal Engine 5.4 or later
- C++ project (Blueprint-only projects are not supported for plugin development)
- AIModule enabled (default in most project templates)

---

## Step 1: Enable the Plugin

Copy the `FWAISystem` folder into your project's `Plugins/` directory:

```
YourProject/
  Plugins/
    FWAISystem/
      FWAISystem.uplugin
      Source/
        FWAISystem/
          ...
```

Open your project in the Unreal Editor. Navigate to **Edit > Plugins**, search for "FW AI System", and verify it is enabled. Restart the editor if prompted.

---

## Step 2: Add Module Dependencies

Add `FWAISystem` to your game module's `Build.cs` file:

=== "Public Dependency"

    ```csharp title="YourGame.Build.cs"
    PublicDependencyModuleNames.AddRange(new string[]
    {
        "Core",
        "CoreUObject",
        "Engine",
        "FWAISystem"  // Add this
    });
    ```

=== "Private Dependency"

    ```csharp title="YourGame.Build.cs"
    PrivateDependencyModuleNames.AddRange(new string[]
    {
        "FWAISystem"  // Add this if only using in .cpp files
    });
    ```

!!! tip "Public vs Private"
    Use a **public** dependency if you expose FWAISystem types in your game module's header files. Use **private** if you only reference FWAISystem in `.cpp` files.

---

## Step 3: Optional -- FWFactionSystem Integration

FWAISystem optionally integrates with FWFactionSystem to provide faction-based aggro gating through the `FWBTDecorator_FactionAttitude` behavior tree decorator.

To enable this integration:

1. Ensure the `FWFactionSystem` plugin is in your `Plugins/` directory.
2. No additional configuration is needed -- the build system auto-detects FWFactionSystem at compile time.

The build system checks for the presence of the `FWFactionSystem` directory adjacent to `FWAISystem` and defines `WITH_FWFACTIONSYSTEM=1` when found.

```csharp title="FWAISystem.Build.cs (auto-detection)"
string FWFactionSystemPath = Path.Combine(PluginDirectory, "..", "FWFactionSystem");
if (Directory.Exists(FWFactionSystemPath))
{
    PublicDependencyModuleNames.Add("FWFactionSystem");
    PublicDefinitions.Add("WITH_FWFACTIONSYSTEM=1");
}
else
{
    PublicDefinitions.Add("WITH_FWFACTIONSYSTEM=0");
}
```

!!! note "Without FWFactionSystem"
    When FWFactionSystem is not present, `FWBTDecorator_FactionAttitude` falls back to Unreal's built-in `IGenericTeamAgentInterface` for attitude checks.

---

## Step 4: Optional -- FWGASSystem Integration

FWAISystem optionally integrates with FWGASSystem (Gameplay Ability System) to provide:

- **FWBTTask_PlayAbility** -- Activate GAS abilities on NPCs via gameplay tags
- **ApplyScalingToNpc** -- Automatically modify GAS attributes (MaxHealth, AttackPower) during scaling

To enable:

1. Ensure the `FWGASSystem` plugin is in your `Plugins/` directory.
2. Auto-detection handles the rest.

```csharp title="FWAISystem.Build.cs (auto-detection)"
string FWGASSystemPath = Path.Combine(PluginDirectory, "..", "FWGASSystem");
if (Directory.Exists(FWGASSystemPath))
{
    PublicDependencyModuleNames.Add("FWGASSystem");
    PublicDefinitions.Add("WITH_FWGASSYSTEM=1");
}
else
{
    PublicDefinitions.Add("WITH_FWGASSYSTEM=0");
}
```

!!! note "Without FWGASSystem"
    When FWGASSystem is not present, `FWBTTask_PlayAbility` returns `EBTNodeResult::Failed` gracefully, and `ApplyScalingToNpc` stores scaling data on the actor for manual application instead of modifying GAS attributes directly.

---

## Step 5: Include Headers

After adding the module dependency, include the headers you need:

```cpp
#include "FWAITypes.h"                         // Enums, structs, delegates
#include "Components/FWThreatComponent.h"       // Threat table
#include "Components/FWLeashComponent.h"        // Leash mechanics
#include "DataAssets/FWNpcDefinition.h"         // NPC data asset
#include "DataAssets/FWSpawnTable.h"            // Spawn table data asset
#include "Scaling/FWInstanceScaling.h"          // Scaling library
#include "Spawning/FWNpcSpawnPoint.h"           // Point spawner
#include "Spawning/FWNpcSpawnVolume.h"          // Volume spawner
```

---

## Verification

To verify the plugin is correctly installed:

1. Open the Unreal Editor.
2. Open the **Output Log** (Window > Developer Tools > Output Log).
3. Search for `FWAISystem` -- you should see the module loading without errors.
4. In the Content Browser, navigate to a Blueprint and search for "FW" in the component list -- you should see `FWThreatComponent` and `FWLeashComponent`.

!!! warning "Compile Errors"
    If you see unresolved symbol errors related to `FWAISYSTEM_API`, ensure `FWAISystem` is listed in your `Build.cs` dependencies and regenerate project files (right-click your `.uproject` > Generate Visual Studio project files).

---

## Full Module Dependencies

For reference, FWAISystem itself depends on the following Unreal modules:

| Module | Type | Purpose |
|--------|------|---------|
| Core | Public | Core UE types |
| CoreUObject | Public | UObject system |
| Engine | Public | Engine framework |
| AIModule | Public | Behavior trees, AI perception |
| GameplayTags | Public | Gameplay tag containers |
| GameplayAbilities | Public | GAS integration |
| GameplayTasks | Public | Async gameplay tasks |
| NavigationSystem | Public | Navmesh queries for patrol and spawning |
| NetCore | Public | Network core types |
| DeveloperSettings | Private | Developer settings classes |
