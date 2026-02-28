---
title: FWPartySystem Plugin
description: Production-ready beacon-based party system with invitations, join codes, and seamless level transfer persistence for Unreal Engine 5.
tags:
  - plugin
  - party
  - multiplayer
  - social
---

# FWPartySystem Plugin

**Version:** 2.0 | **Module:** `FWPartySystem` | **Type:** Runtime

A production-ready, beacon-based party system for Unreal Engine 5 that persists across level transfers. FWPartySystem supports direct join via codes, invitation workflows, configurable party sizes, leader management, and integrates with FWChatSystem for automatic party chat channels.

---

## Feature Overview

| Feature | Description |
|---|---|
| Party Lifecycle | Create, join, leave, and disband parties with full state management |
| Join Codes | Share short alphanumeric codes for direct party join without friend lists |
| Invitation System | Send, accept, decline, and auto-expire party invitations |
| Beacon Architecture | Server-authoritative state via `AOnlineBeaconHost` / `AOnlineBeaconClient` |
| Level Transfer Persistence | Party state survives seamless travel and hard map loads |
| Leader Management | Automatic leader migration on disconnect, manual leader transfer |
| Configurable Party Size | Per-party max member limits with Blueprint-configurable defaults |
| Privacy Modes | Open parties (anyone with code can join) or invite-only parties |
| Kick System | Leaders can remove members with automatic state cleanup |
| Chat Integration | Optional sync with FWChatSystem for automatic party chat channels |
| Blueprint Support | All operations, queries, and events exposed to Blueprints |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| `OnlineSubsystem` | **Required** | Provides unique net ID resolution for party members |
| `OnlineSubsystemUtils` | **Required** | Utility functions for online identity handling |
| `SocketIOClient` | **Required** | Used for real-time party event synchronization |
| `FWChatSystem` | **Optional** | Enables automatic party chat channel creation and teardown |

!!! info "Optional Chat Integration"
    When FWChatSystem is present, creating a party automatically provisions a private chat channel scoped to party members. When absent, all chat-related functionality is silently skipped with no errors.

---

## Plugin Architecture

```
FWPartySystem
+-- Components/
|   +-- UFWPartyManagerComponent       // Blueprint-facing party operations
+-- Beacons/
|   +-- AFWPartyBeaconHost             // Server-side party state authority
|   +-- AFWPartyBeaconClient           // Client-side beacon communication
|   +-- AFWPartyHostObject             // Party host management object
+-- Types/
|   +-- FWPartyTypes.h                 // Enums, structs, delegates
```

---

## Quick Start

### 1. Enable the Plugin

In your `.uproject` file or via the Plugin Manager, enable `FWPartySystem`.

### 2. Add the Component to Your Player Controller

```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Party")
TObjectPtr<UFWPartyManagerComponent> PartyManager;
```

### 3. Create a Party

=== "C++"

    ```cpp
    PartyManager->CreateParty();
    PartyManager->OnPartyCreated.AddDynamic(this, &AMyController::HandlePartyCreated);
    ```

=== "Blueprint"

    Call **Create Party** on the Party Manager Component, then bind the **On Party Created** event to handle the result.

### 4. Share the Join Code

```cpp
void AMyController::HandlePartyCreated(const FFWPartyInfo& PartyInfo)
{
    // Display PartyInfo.JoinCode in your UI for other players to use
    UE_LOG(LogParty, Log, TEXT("Party created with code: %s"), *PartyInfo.JoinCode);
}
```

### 5. Join a Party by Code

```cpp
PartyManager->JoinPartyByCode(TEXT("ABC123"));
PartyManager->OnJoinPartyResult.AddDynamic(this, &AMyController::HandleJoinResult);
```

---

## File Reference

| Page | Description |
|---|---|
| [Installation](installation.md) | Add the plugin to your project and configure dependencies |
| [Quick Start](quick-start.md) | Get a party system running in under 10 minutes |
| [Party Manager Component](party-manager-component.md) | Primary API for all party operations |
| [Beacon Architecture](beacon-architecture.md) | Server-side beacon host, client, and host object internals |
| [Types Reference](types.md) | Enums, structs, and delegate definitions |
| [API Reference](api-reference.md) | Complete function and delegate reference |
| [Configuration](configuration.md) | Plugin settings, party defaults, and project configuration |
| [Events and Delegates](events-delegates.md) | All multicast delegates and event flow diagrams |
| [Tutorial: Implementing Multiplayer Parties](tutorial.md) | Step-by-step implementation guide |
| [Changelog](changelog.md) | Version history and release notes |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |

!!! warning "Dedicated Server"
    All party mutations are server-authoritative. The `AFWPartyBeaconHost` must run on the server or a dedicated beacon host. Client-side calls to `UFWPartyManagerComponent` dispatch requests through the beacon client and do not modify local state until the server confirms the operation.
