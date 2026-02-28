---
title: Installation - FWCustomizationSystem
---

# Installation

This page covers how to install and enable FWCustomizationSystem in your Unreal Engine 5 project.

---

## Prerequisites

| Requirement | Minimum Version |
|---|---|
| Unreal Engine | 5.4 or later |
| Project Type | C++ (Blueprint-only projects are not supported) |
| Dependencies | None |

!!! note "No External Dependencies"
    FWCustomizationSystem is fully self-contained. It does not depend on any other FrostWeb plugin or third-party module.

---

## Step 1 -- Copy the Plugin

Copy the `FWCustomizationSystem/` folder into your project's `Plugins/` directory:

```
YourProject/
    Plugins/
        FWCustomizationSystem/
            FWCustomizationSystem.uplugin
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
            "Name": "FWCustomizationSystem",
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
    "FWCustomizationSystem"
});
```

---

## Step 5 -- Verify Installation

Build your project and check the Output Log for:

```
LogModuleManager: Loading module FWCustomizationSystem
```

!!! tip "Troubleshooting"
    If the module fails to load:

    - Verify the plugin folder name matches `FWCustomizationSystem` exactly.
    - Regenerate project files and perform a full rebuild.
    - Check the Output Log for unresolved symbol errors, which typically indicate a missing `Build.cs` dependency.

---

## Include Paths

After installation, you can include plugin headers in your C++ code:

```cpp
#include "FWCustomizationComponent.h"
#include "Data/FWCustomizationDatabase.h"
#include "Data/FWRaceConfig.h"
#include "Types/FWCustomizationTypes.h"
```

---

## Next Steps

- Follow the [Quick Start](quick-start.md) to get a working customization component in under 10 minutes.
- Read the [Configuration](configuration.md) guide to set up your customization database and race configs.
