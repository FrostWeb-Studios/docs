---
title: Installation - FWSkillSystem
---

# Installation

This page covers how to install and enable FWSkillSystem in your Unreal Engine 5 project.

---

## Prerequisites

| Requirement | Minimum Version |
|---|---|
| Unreal Engine | 5.3 or later |
| Project Type | C++ (Blueprint-only projects are not supported) |
| Dependencies | GameplayAbilities |

!!! warning "GameplayAbilities Required"
    FWSkillSystem depends on the Gameplay Ability System. You must enable the `GameplayAbilities` plugin before enabling FWSkillSystem. If GameplayAbilities is not enabled, the plugin will fail to compile.

---

## Step 1 -- Enable GameplayAbilities

If your project does not already use the Gameplay Ability System, add it to your `.uproject` file:

```json
{
    "Plugins": [
        {
            "Name": "GameplayAbilities",
            "Enabled": true
        }
    ]
}
```

And add the module dependency in your game module's `Build.cs`:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "GameplayAbilities",
    "GameplayTags",
    "GameplayTasks"
});
```

---

## Step 2 -- Copy the Plugin

Copy the `FWSkillSystem/` folder into your project's `Plugins/` directory:

```
YourProject/
    Plugins/
        FWSkillSystem/
            FWSkillSystem.uplugin
            Source/
            Content/
```

---

## Step 3 -- Enable the Plugin

Add the plugin to your `.uproject` file:

```json
{
    "Plugins": [
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

Alternatively, enable it through **Edit > Plugins** in the Unreal Editor and restart.

---

## Step 4 -- Regenerate Project Files

Right-click your `.uproject` file and select **Generate Visual Studio project files**. This ensures your build system picks up the new module.

---

## Step 5 -- Add Module Dependency

In your game module's `Build.cs`, add the module dependency:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "GameplayAbilities",
    "GameplayTags",
    "GameplayTasks",
    "FWSkillSystem"
});
```

---

## Step 6 -- Configure Project Settings

After enabling the plugin and restarting the editor, navigate to **Edit > Project Settings > Plugins > FW Skill System**. This is where you configure global skill system behavior through `UFWSkillSystemSettings`.

At minimum, you should set:

- **Skill Database** -- Reference to your `UFWSkillDatabase` asset (created in the [Configuration](configuration.md) guide)
- **Default XP Curve** -- Reference to a `UFWSkillXPCurve` asset that defines the default XP-per-level progression

!!! info "Project Settings Are Optional at Install Time"
    You can complete installation without configuring project settings immediately. The plugin will load without errors, but skill operations will not function until a `UFWSkillDatabase` is assigned. See [Configuration](configuration.md) for full setup.

---

## Step 7 -- Verify Installation

Build your project and check the Output Log for:

```
LogModuleManager: Loading module FWSkillSystem
```

!!! tip "Troubleshooting"
    If the module fails to load:

    - Verify that `GameplayAbilities` is enabled and compiles successfully before enabling FWSkillSystem.
    - Verify the plugin folder name matches `FWSkillSystem` exactly.
    - Regenerate project files and perform a full rebuild.
    - Check the Output Log for unresolved symbol errors, which typically indicate a missing `Build.cs` dependency.

---

## Include Paths

After installation, you can include plugin headers in your C++ code:

```cpp
#include "FWSkillProgressionComponent.h"
#include "FWSkillMilestoneComponent.h"
#include "Data/FWSkillDefinition.h"
#include "Data/FWSkillDatabase.h"
#include "Data/FWSkillXPCurve.h"
#include "Data/FWSkillMilestoneDefinition.h"
#include "Requirements/FWSkillRequirement.h"
#include "Settings/FWSkillSystemSettings.h"
#include "Utils/FWSkillTypeLibrary.h"
#include "Types/FWSkillTypes.h"
```

---

## Next Steps

- Follow the [Quick Start](quick-start.md) to get skill progression running in 10 minutes.
- Read the [Configuration](configuration.md) guide to set up your skill database, XP curves, and milestones.
