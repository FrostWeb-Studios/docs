---
title: Getting Started
---

# Getting Started

This guide walks you through the prerequisites, installation, and initial verification for FrostWeb UE5 plugins.

---

## Prerequisites

Before installing any FrostWeb plugin, ensure your environment meets the following requirements:

| Requirement | Minimum Version |
|---|---|
| Unreal Engine | 5.4 or later |
| Project Type | C++ (Blueprint-only projects are not supported) |
| IDE | Visual Studio 2022 or JetBrains Rider |
| Build Platform | Win64 (other platforms may work but are untested) |

!!! warning "C++ Project Required"
    All FrostWeb plugins contain C++ modules. If your project is Blueprint-only, you must convert it to a C++ project first by adding any C++ class through the Unreal Editor, which will generate the necessary project and solution files.

---

## Installation

### Step 1 -- Copy the Plugin

Copy the plugin folder (e.g., `FWAISystem/`) into your project's `Plugins/` directory:

```
YourProject/
    Plugins/
        FWAISystem/
            FWAISystem.uplugin
            Source/
            Content/
            ...
```

### Step 2 -- Enable the Plugin

Add the plugin to your `.uproject` file under the `Plugins` array:

```json
{
    "Plugins": [
        {
            "Name": "FWAISystem",
            "Enabled": true
        }
    ]
}
```

Alternatively, enable the plugin through **Edit > Plugins** in the Unreal Editor, then restart.

### Step 3 -- Regenerate Project Files

Right-click your `.uproject` file and select **Generate Visual Studio project files** (or use your IDE's equivalent). This ensures the new plugin modules are picked up by your build system.

### Step 4 -- Add Module Dependencies

In your game module's `Build.cs`, add the plugin modules you need:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "FWAISystem",
    // Add other FrostWeb modules as needed
});
```

---

## Plugin Dependencies

Not all plugins are standalone. The chart below shows which plugins depend on other modules. **Solid lines** indicate hard dependencies; **dashed lines** indicate optional integrations.

```
FWAISystem ──────── FWFactionSystem
    :
    : (optional)
    :
FWGASSystem

FWChatSystem ────── SocketIOClient (third-party)

FWGuildSystem - - - FWChatSystem (optional)

FWInventorySystem ── GameplayAbilities
    :
    : (optional)
    :
FWSkillSystem

FWPartySystem ────── OnlineSubsystem
                  ── SocketIOClient (third-party)

FWQuestSystem ────── GameplayAbilities

FWSkillSystem ────── GameplayAbilities

FWGASSystem ──────── GameplayAbilities
                  ── ModularGameplay
                  ── GameFeatures
                  ── EnhancedInput

FWCustomizationSystem  (no external dependencies)
FWDialogueSystem       (no external dependencies)
FWFactionSystem        (no external dependencies)
```

!!! info "Dependency Summary"
    | Plugin | Hard Dependencies | Optional Dependencies |
    |---|---|---|
    | FWAISystem | FWFactionSystem | FWGASSystem |
    | FWChatSystem | SocketIOClient | -- |
    | FWCustomizationSystem | -- | -- |
    | FWDialogueSystem | -- | -- |
    | FWFactionSystem | -- | -- |
    | FWGASSystem | GameplayAbilities, ModularGameplay, GameFeatures, EnhancedInput | -- |
    | FWGuildSystem | -- | FWChatSystem |
    | FWInventorySystem | GameplayAbilities | FWSkillSystem |
    | FWPartySystem | OnlineSubsystem, SocketIOClient | -- |
    | FWQuestSystem | GameplayAbilities | -- |
    | FWSkillSystem | GameplayAbilities | -- |

---

## Quick Verification

After installation, verify that the plugin loaded correctly.

### Step 1 -- Compile

Build your project from your IDE or via Unreal's **Compile** button in the toolbar. A successful build with zero errors confirms module resolution.

### Step 2 -- Check the Output Log

Open the **Output Log** in Unreal Editor (**Window > Developer Tools > Output Log**) and search for the plugin module name. You should see a log entry confirming the module was loaded:

```
LogModuleManager: Loading module FWAISystem
```

!!! tip "Troubleshooting"
    If the module fails to load:

    - Verify the plugin folder name matches the `.uplugin` file name exactly.
    - Confirm all dependencies listed above are also installed and enabled.
    - Regenerate project files and perform a full rebuild.
    - Check the Output Log for specific error messages related to missing modules or unresolved symbols.

---

## Next Steps

- Browse the [Plugins Overview](plugins/index.md) for a summary of all available plugins.
- Read the [Architecture Overview](guides/architecture.md) to understand the shared design patterns.
- Dive into individual plugin documentation for installation details, API references, and tutorials.
