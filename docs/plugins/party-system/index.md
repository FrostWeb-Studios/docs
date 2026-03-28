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

**Module:** `FWPartySystem` | **Type:** Runtime

A production-ready, beacon-based party system for Unreal Engine 5 that persists across level transfers. FWPartySystem supports direct join via codes, invitation workflows, configurable party sizes, leader management, and integrates with FWChatSystem for automatic party chat channels.

---

## Feature Overview

| Feature | Description |
|---|---|
| Party Lifecycle | Create, join, leave, and disband parties with full state management |
| Join Codes | Share short alphanumeric codes for direct party join without friend lists |
| Invitation System | Send, accept, decline, and auto-expire party invitations |
| Beacon Architecture | Server-authoritative state via Online Beacon Host / Client |
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
| `FWChatSystem` | **Optional** | Enables automatic party chat channel creation and teardown |

!!! info "Optional Chat Integration"
    When FWChatSystem is present, creating a party automatically provisions a private chat channel scoped to party members. When absent, all chat-related functionality is silently skipped with no errors.

---

## Components and Classes

| Class | Purpose |
|---|---|
| **UFWPartyManagerComponent** | Primary Blueprint-facing API for party creation, joining, leaving, leader actions, queries, chat integration, and events |
| **UFWPartyManagerSubsystem** | Game Instance subsystem that manages beacon lifecycle, global party queries, and configuration |
| **AFWPartyBeaconHost** | Server-side beacon that holds authoritative party state |
| **AFWPartyBeaconClient** | Client-side beacon for communicating with the host |
| **AFWPartyHostObject** | Per-party state object managed by the beacon host |

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

Once the party is created, retrieve the join code from the party info and display it in your UI for other players.

### 5. Join a Party by Code

```cpp
PartyManager->JoinPartyByCode(TEXT("ABC123"));
PartyManager->OnJoinPartyResult.AddDynamic(this, &AMyController::HandleJoinResult);
```

---

## Core Types

| Type | Kind | Description |
|---|---|---|
| `FFWPartyInfo` | Struct | Complete party state including members, join code, privacy, and settings |
| `FFWPartyMemberInfo` | Struct | Member data including player ID, display name, role, and connection state |
| `FFWPartyInvitation` | Struct | Invitation record with sender, target, party ID, and expiry timestamp |
| `FFWPartySettings` | Struct | Party configuration including max members, privacy mode, and defaults |
| `EFWPartyRole` | Enum | Member roles: Leader, Member |
| `EFWPartyPrivacy` | Enum | Privacy modes: Open, InviteOnly |
| `EFWPartyJoinResult` | Enum | Join result codes: Success, PartyFull, InvalidCode, InviteOnly, AlreadyInParty, etc. |

---

## Page Reference

| Page | Description |
|---|---|
| [API Reference](api-reference.md) | Complete function, property, and delegate reference |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |

!!! warning "Dedicated Server"
    All party mutations are server-authoritative. The beacon host must run on the server or a dedicated beacon host. Client-side calls to `UFWPartyManagerComponent` dispatch requests through the beacon client and do not modify local state until the server confirms the operation.
