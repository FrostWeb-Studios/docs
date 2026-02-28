---
title: Installation - Faction System
---

# Installation

## Prerequisites

- Unreal Engine 5.4 or later
- C++ project (or a Blueprint project with the plugin pre-compiled)

FWFactionSystem has **no external dependencies**. It uses only core engine modules (`Core`, `CoreUObject`, `Engine`, `AIModule`, `GameplayTags`, `NetCore`).

---

## Step 1: Add the Plugin

=== "From Source"

    Copy the `FWFactionSystem/` folder into your project's `Plugins/` directory:

    ```
    YourProject/
      Plugins/
        FWFactionSystem/
          FWFactionSystem.uplugin
          Source/
          Content/
    ```

=== "From Marketplace"

    Install FWFactionSystem from the Unreal Engine Marketplace. The plugin will appear in your Engine's plugin directory automatically.

---

## Step 2: Enable the Plugin

Open **Edit > Plugins** in the Unreal Editor, search for "FW Faction System", and enable it. Restart the editor when prompted.

Alternatively, add it directly to your `.uproject` file:

```json
{
  "Plugins": [
    {
      "Name": "FWFactionSystem",
      "Enabled": true
    }
  ]
}
```

---

## Step 3: Add Module Dependency

If you need to reference FWFactionSystem types from your game module's C++ code, add the module dependency in your `Build.cs`:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "FWFactionSystem"
});
```

---

## Step 4: Register the PrimaryAssetType

FWFactionSystem uses Unreal's `AssetManager` to discover `UFWFactionDefinition` DataAssets at startup. You need to register the asset type so the subsystem can find your faction definitions.

Open **Project Settings > Game > Asset Manager** and add a new entry to **Primary Asset Types to Scan**:

| Field | Value |
|-------|-------|
| Primary Asset Type | `FactionDef` |
| Asset Base Class | `FWFactionDefinition` |
| Directories | `/Game/Data/Factions/` (or wherever you store them) |
| Rules > Apply Recursively | Checked |

!!! warning "Missing Asset Registration"
    If you skip this step, `UFWFactionSubsystem::GetFactionDefinition()` will return `nullptr` for all factions. The subsystem logs a warning at startup if no faction definitions are found.

---

## Step 5: Create Your First Faction

1. Right-click in the Content Browser.
2. Select **Miscellaneous > Data Asset**.
3. Choose `FWFactionDefinition` as the class.
4. Name it (e.g., `DA_Faction_Ironclad`).
5. Set the **FactionId**, **DisplayName**, **TeamId**, and other properties.

See [Configuration](configuration.md) for a detailed walkthrough of each property.

---

## Verification

After restarting the editor with the plugin enabled:

1. Open **Window > Developer Tools > Output Log**.
2. Search for `LogFWFactionSystem`. You should see a log entry confirming the number of faction definitions loaded.
3. Place an actor with a `UFWFactionComponent` in your level, set its `DefaultFactionId`, and press Play. The component should register with the subsystem automatically.

!!! tip
    Use the console command `FWFaction.ListFactions` (non-shipping builds) to verify all definitions loaded correctly.
