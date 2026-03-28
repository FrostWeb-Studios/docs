---
title: Installation - FWChatSystem
---

# Installation

This guide covers enabling FWChatSystem in your Unreal Engine 5 project.

---

## Prerequisites

- Unreal Engine 5.4 or later
- C++ project (Blueprint-only projects are not supported for plugin development)
- A Socket.IO chat server to connect to

---

## Step 1: Enable FWChatSystem

FWChatSystem includes a built-in lightweight Socket.IO v4 client using UE5's native WebSockets. No external Socket.IO plugin is required.

Copy the `FWChatSystem` folder into your project's `Plugins/` directory:

```
YourProject/
  Plugins/
    FWChatSystem/
      FWChatSystem.uplugin
      Source/
        FWChatSystem/
          ...
```

Open your project in the Unreal Editor. Navigate to **Edit > Plugins**, search for "FW Chat System", and verify it is enabled. Restart the editor if prompted.

---

## Step 2: Add Module Dependencies

Add `FWChatSystem` to your game module's `Build.cs` file:

=== "Public Dependency"

    ```csharp title="YourGame.Build.cs"
    PublicDependencyModuleNames.AddRange(new string[]
    {
        "Core",
        "CoreUObject",
        "Engine",
        "FWChatSystem"  // Add this
    });
    ```

=== "Private Dependency"

    ```csharp title="YourGame.Build.cs"
    PrivateDependencyModuleNames.AddRange(new string[]
    {
        "FWChatSystem"  // Add this if only using in .cpp files
    });
    ```

---

## Step 3: Include Headers

After adding the module dependency, include the headers you need:

```cpp
#include "FWChatTypes.h"                                    // Enums, structs
#include "Components/FWChatStateComponent.h"                // State management
#include "Components/FWChatRouterComponent.h"               // Input routing
#include "Components/FWSocketIOChatTransportComponent.h"    // Network transport
#include "IFWChatGuildProvider.h"                           // Guild integration interface
#include "IFWChatUIController.h"                            // UI controller interface
```


---

## Verification

To verify the plugin is correctly installed:

1. Open the Unreal Editor.
2. Open the **Output Log** and search for `FWChatSystem` -- you should see the module loading without errors.
3. In a Blueprint, search for "Chat" in the component list -- you should see:
    - `Chat State Component`
    - `Chat Router`
    - `Socket.IO Chat Transport`

!!! tip "No External Socket.IO Plugin Needed"
    FWChatSystem ships with a built-in Socket.IO v4 client. You do not need to install any third-party Socket.IO or WebSocket plugins.
