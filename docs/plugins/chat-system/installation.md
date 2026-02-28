---
title: Installation - FWChatSystem
---

# Installation

This guide covers enabling FWChatSystem in your Unreal Engine 5 project and configuring its required SocketIOClient dependency.

---

## Prerequisites

- Unreal Engine 5.4 or later
- C++ project (Blueprint-only projects are not supported for plugin development)
- A chat server compatible with Socket.IO protocol (e.g., FrostWeb ChatServer or any Socket.IO-based server)

---

## Step 1: Install the SocketIOClient Plugin

FWChatSystem requires the SocketIOClient plugin for WebSocket communication.

1. Obtain the SocketIOClient plugin (available on the Unreal Marketplace or from [GitHub](https://github.com/getnamo/SocketIOClient-Unreal)).
2. Place it in your project's `Plugins/` directory:

```
YourProject/
  Plugins/
    SocketIOClient/
      SocketIOClient.uplugin
      ...
```

3. Verify the plugin is enabled in **Edit > Plugins**.

!!! warning "SocketIOClient Required"
    FWChatSystem will not compile without the SocketIOClient plugin. Ensure it is installed and enabled before proceeding.

---

## Step 2: Enable FWChatSystem

Copy the `FWChatSystem` folder into your project's `Plugins/` directory:

```
YourProject/
  Plugins/
    FWChatSystem/
      FWChatSystem.uplugin
      Source/
        FWChatSystem/
          ...
    SocketIOClient/
      ...
```

Open your project in the Unreal Editor. Navigate to **Edit > Plugins**, search for "FW Chat System", and verify it is enabled. Restart the editor if prompted.

---

## Step 3: Add Module Dependencies

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

If you also need to directly use Socket.IO or SIOJson types in your game code, add those as well:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "SocketIOClient",
    "SIOJson"
});
```

!!! tip "When to Add SocketIOClient"
    You only need `SocketIOClient` and `SIOJson` as direct dependencies if your game code creates or manipulates `USIOJsonValue`/`USIOJsonObject` instances. If you only interact with FWChatSystem through its component API, you do not need them.

---

## Step 4: Include Headers

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

## Step 5: Chat Server Setup

FWChatSystem connects to a Socket.IO-based chat server. The server must support the following events:

### Client-to-Server Events

| Event | Payload | Description |
|-------|---------|-------------|
| `chat:send` | `{ channel, body, target? }` | Send a chat message |
| `presence:update` | `{ zoneId, position }` | Update player presence |
| `party:sync` | `{ partyId }` | Join a party chat room |
| `party:leave` | -- | Leave the current party chat |
| `guild:sync` | `{ guildId }` | Join a guild chat room |
| `guild:leave` | -- | Leave the current guild chat |
| `dm:open` | `{ targetPlayerId }` | Open a DM conversation |

### Server-to-Client Events

| Event | Payload | Description |
|-------|---------|-------------|
| `chat:recv` | `FFWChatMessage` JSON | Incoming chat message |
| `system:notice` | `{ code, text }` | System notification |
| `party:update` | `FFWChatPartyInfo` JSON | Party info update |
| `guild:roster` | `{ guildId, roster }` | Guild roster data |
| `guild:update` | `{ updateData }` | Guild change event |
| `error` | `{ code, message }` | Error from the server |

### Authentication

The server must accept a JWT token during the Socket.IO handshake. The transport component passes the token as a query parameter:

```
wss://chat.example.com?token=<JWT>
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

!!! warning "Missing SocketIOClient"
    If you see errors about unresolved symbols referencing `SIO` or `SocketIO`, the SocketIOClient plugin is not installed or enabled. Install it first and regenerate project files.

---

## Full Module Dependencies

For reference, FWChatSystem depends on the following modules:

| Module | Type | Purpose |
|--------|------|---------|
| Core | Public | Core UE types |
| CoreUObject | Public | UObject system |
| Engine | Public | Engine framework |
| SocketIOClient | Public | Socket.IO WebSocket client |
| SIOJson | Public | JSON serialization for Socket.IO |
| NetCore | Private | Network core types |
