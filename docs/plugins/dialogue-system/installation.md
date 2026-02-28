---
title: Installation - FWDialogueSystem
---

# Installation

This page covers how to install and enable FWDialogueSystem in your Unreal Engine 5 project.

---

## Prerequisites

| Requirement | Minimum Version |
|---|---|
| Unreal Engine | 5.4 or later |
| Project Type | C++ (Blueprint-only projects are not supported) |
| Dependencies | None |

!!! note "No External Dependencies"
    FWDialogueSystem is fully self-contained. It does not depend on any other FrostWeb plugin or third-party module. The built-in conditions and actions reference game concepts (items, quests, skills) through abstract interfaces that you implement in your game code.

---

## Step 1 -- Copy the Plugin

Copy the `FWDialogueSystem/` folder into your project's `Plugins/` directory:

```
YourProject/
    Plugins/
        FWDialogueSystem/
            FWDialogueSystem.uplugin
            Source/
            Content/
```

---

## Step 2 -- Enable the Plugin

Add the plugin to your `.uproject` file:

```json
{
    "Plugins": [
        {
            "Name": "FWDialogueSystem",
            "Enabled": true
        }
    ]
}
```

Alternatively, enable it through **Edit > Plugins** in the Unreal Editor and restart.

---

## Step 3 -- Regenerate Project Files

Right-click your `.uproject` file and select **Generate Visual Studio project files**. This ensures your build system picks up the new module.

---

## Step 4 -- Add Module Dependency

In your game module's `Build.cs`, add the module dependency:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "FWDialogueSystem"
});
```

---

## Step 5 -- Verify Installation

Build your project and check the Output Log for:

```
LogModuleManager: Loading module FWDialogueSystem
```

!!! tip "Troubleshooting"
    If the module fails to load:

    - Verify the plugin folder name matches `FWDialogueSystem` exactly.
    - Regenerate project files and perform a full rebuild.
    - Check the Output Log for unresolved symbol errors.

---

## Include Paths

After installation, you can include plugin headers in your C++ code:

```cpp
#include "FWDialogueComponent.h"
#include "Data/FWDialogueTree.h"
#include "Conditions/FWDialogueConditionBase.h"
#include "Actions/FWDialogueActionBase.h"
#include "Types/FWDialogueTypes.h"
```

---

## Next Steps

- Follow the [Quick Start](quick-start.md) to get a working NPC conversation in under 10 minutes.
- Read the [Configuration](configuration.md) guide to learn how to build dialogue trees with conditions and actions.
