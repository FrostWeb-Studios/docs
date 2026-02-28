---
title: Installation - FWPartySystem
description: Install and configure the FWPartySystem plugin in your Unreal Engine 5 project.
---

# Installation

This page covers how to install and enable FWPartySystem in your Unreal Engine 5 project.

---

## Prerequisites

| Requirement | Minimum Version |
|---|---|
| Unreal Engine | 5.3 or later |
| Project Type | C++ (Blueprint-only projects are not supported) |
| OnlineSubsystem | Any configured OSS (Null, Steam, EOS, etc.) |
| SocketIOClient | Plugin must be installed and enabled |

!!! note "OnlineSubsystem Configuration"
    FWPartySystem requires an active OnlineSubsystem for unique player identity resolution. For local development and testing, `OnlineSubsystemNull` is sufficient. For production, configure your target platform's OSS (Steam, EOS, etc.).

---

## Step 1 -- Copy the Plugin

Copy the `FWPartySystem/` folder into your project's `Plugins/` directory:

```
YourProject/
    Plugins/
        FWPartySystem/
            FWPartySystem.uplugin
            Source/
            Content/
```

---

## Step 2 -- Enable the Plugin

Add the plugin and its dependencies to your `.uproject` file:

```json
{
    "Plugins": [
        {
            "Name": "FWPartySystem",
            "Enabled": true
        },
        {
            "Name": "OnlineSubsystem",
            "Enabled": true
        },
        {
            "Name": "OnlineSubsystemUtils",
            "Enabled": true
        },
        {
            "Name": "SocketIOClient",
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
    "FWPartySystem",
    "OnlineSubsystem",
    "OnlineSubsystemUtils"
});
```

---

## Step 5 -- Configure the OnlineSubsystem

Ensure your `DefaultEngine.ini` has an OnlineSubsystem configured. For development:

```ini
[OnlineSubsystem]
DefaultPlatformService=Null

[OnlineSubsystemNull]
bEnabled=true
```

For production with Steam:

```ini
[OnlineSubsystem]
DefaultPlatformService=Steam

[OnlineSubsystemSteam]
bEnabled=true
SteamDevAppId=480
```

---

## Step 6 -- Verify Installation

Build your project and check the Output Log for:

```
LogModuleManager: Loading module FWPartySystem
```

!!! tip "Troubleshooting"
    If the module fails to load:

    - Verify the plugin folder name matches `FWPartySystem` exactly.
    - Confirm `OnlineSubsystem`, `OnlineSubsystemUtils`, and `SocketIOClient` are all enabled.
    - Regenerate project files and perform a full rebuild.
    - Check the Output Log for unresolved symbol errors, which typically indicate a missing `Build.cs` dependency.

---

## Optional: Enable Chat Integration

If you want automatic party chat channels, install and enable `FWChatSystem`:

```json
{
    "Plugins": [
        {
            "Name": "FWChatSystem",
            "Enabled": true
        }
    ]
}
```

!!! info "Runtime Detection"
    FWPartySystem detects FWChatSystem at runtime. No additional configuration is needed -- party chat channels are created automatically when a party is formed and torn down when it disbands.

---

## Include Paths

After installation, you can include plugin headers in your C++ code:

```cpp
#include "FWPartyManagerComponent.h"
#include "Beacons/FWPartyBeaconHost.h"
#include "Beacons/FWPartyBeaconClient.h"
#include "Beacons/FWPartyHostObject.h"
#include "Types/FWPartyTypes.h"
```

---

## Next Steps

- Follow the [Quick Start](quick-start.md) to get a party system running in under 10 minutes.
- Read the [Configuration](configuration.md) guide to customize party defaults and beacon settings.
