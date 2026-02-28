---
title: FWGuildSystem Plugin
description: Full-featured guild system with ranks, permissions, invitations, and HTTP API integration for Unreal Engine 5.
tags:
  - plugin
  - guild
  - social
  - multiplayer
---

# FWGuildSystem Plugin

**Version:** 1.0 | **Module:** `FWGuildSystem` | **Type:** Runtime

A comprehensive guild system plugin for Unreal Engine 5 that provides guild creation, rank hierarchies with granular permissions, member management, invitation workflows, audit logging, and HTTP API integration for backend persistence.

---

## Feature Overview

| Feature | Description |
|---|---|
| Guild Lifecycle | Create, disband, search, and manage guilds |
| Rank System | Configurable rank hierarchies with bitmask permissions |
| Member Management | Invite, kick, promote, and demote guild members |
| Invitation Workflow | Send, accept, decline, and expire invitations |
| Audit Logging | Track all guild operations with timestamps and actor info |
| HTTP API Integration | Full backend persistence via REST endpoints |
| Chat Integration | Optional sync with FWChatSystem for guild channels |
| Blueprint Support | All operations exposed to Blueprint via callable functions and delegates |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| `FWChatSystem` | **Optional** | Enables guild chat channel synchronization. The plugin functions fully without it. |

!!! info "Optional Dependency Handling"
    FWGuildSystem detects FWChatSystem at runtime. When absent, the `UFWGuildChatIntegrationComponent` gracefully disables itself and all chat-related delegates return without error.

---

## Plugin Architecture

```
FWGuildSystem
├── Components/
│   ├── UFWGuildManagerComponent      // Blueprint-facing guild operations
│   ├── UFWGuildStateComponent        // Local guild state cache
│   └── UFWGuildChatIntegrationComponent  // Optional chat bridge
├── Types/
│   └── FWGuildTypes.h                // Enums, structs, bitmasks
└── Utils/
    └── FFWGuildTypeUtils             // Static helper functions
```

---

## Quick Start

### 1. Enable the Plugin

In your `.uproject` file or via the Plugin Manager, enable `FWGuildSystem`.

### 2. Add Components to Your Player Controller or Pawn

```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Guild")
TObjectPtr<UFWGuildManagerComponent> GuildManager;

UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Guild")
TObjectPtr<UFWGuildStateComponent> GuildState;
```

### 3. Create a Guild

=== "C++"

    ```cpp
    GuildManager->CreateGuild(TEXT("My Guild"), TEXT("A guild for adventurers."));
    GuildManager->OnGuildCreated.AddDynamic(this, &AMyPlayerController::HandleGuildCreated);
    ```

=== "Blueprint"

    Call **Create Guild** on the Guild Manager Component, then bind the **On Guild Created** event to handle the result.

### 4. Configure Rank Permissions

```cpp
FFWGuildRank OfficerRank;
OfficerRank.Name = TEXT("Officer");
OfficerRank.Permissions = static_cast<uint8>(
    EFWGuildPermission::Invite |
    EFWGuildPermission::Kick |
    EFWGuildPermission::ViewAuditLog
);
```

---

## File Reference

| Page | Description |
|---|---|
| [Guild Manager Component](guild-manager-component.md) | Primary API for all guild operations |
| [Guild State Component](guild-state-component.md) | Local state management and caching |
| [Chat Integration Component](chat-integration-component.md) | Optional FWChatSystem bridge |
| [Types Reference](types.md) | Enums, structs, and bitmask definitions |
| [API Reference](api-reference.md) | Complete function and delegate reference |
| [Configuration](configuration.md) | Plugin settings and project configuration |
| [Events and Delegates](events-delegates.md) | All multicast delegates and event types |
| [Permissions Guide](permissions.md) | Rank permission system deep dive |
| [Tutorial: Setting Up a Guild System](tutorial.md) | Step-by-step implementation guide |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |

!!! warning "Dedicated Server"
    Guild operations that communicate with the HTTP API should only execute on the server or in a client-authoritative context with proper validation. The `UFWGuildManagerComponent` handles authority checks internally when used on replicated actors.
