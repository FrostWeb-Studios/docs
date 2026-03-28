---
title: FWGuildSystem Plugin
description: Full-featured guild system with ranks, permissions, invitations, and backend integration for Unreal Engine 5.
tags:
  - plugin
  - guild
  - social
  - multiplayer
---

# FWGuildSystem Plugin

**Module:** `FWGuildSystem` | **Type:** Runtime

A comprehensive guild system plugin for Unreal Engine 5 that provides guild creation, rank hierarchies with granular permissions, member management, invitation workflows, audit logging, and backend persistence.

---

## Feature Overview

| Feature | Description |
|---|---|
| Guild Lifecycle | Create, disband, search, and manage guilds |
| Rank System | Configurable rank hierarchies with bitmask permissions |
| Member Management | Invite, kick, promote, and demote guild members |
| Invitation Workflow | Send, accept, decline, and expire invitations |
| Audit Logging | Track all guild operations with timestamps and actor info |
| Backend Integration | Full backend persistence via the Guild Manager Component |
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

## Components

| Component | Purpose |
|---|---|
| **UFWGuildManagerComponent** | Primary Blueprint-facing API for all guild operations: configuration, guild lifecycle, member management, invitations, rank management, and audit log retrieval |
| **UFWGuildStateComponent** | Read-only local cache of guild state. Provides guild queries, member queries, permission checks, rank queries, and state-change events |
| **UFWGuildChatIntegrationComponent** | Optional bridge between the guild system and FWChatSystem for real-time guild chat events |

---

## Quick Start

### 1. Enable the Plugin

In your `.uproject` file or via the Plugin Manager, enable `FWGuildSystem`.

### 2. Add Components to Your Player Controller

```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Guild")
TObjectPtr<UFWGuildManagerComponent> GuildManager;

UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Guild")
TObjectPtr<UFWGuildStateComponent> GuildState;
```

### 3. Configure the Manager

```cpp
GuildManager->SetApiBaseUrl(TEXT("https://your-api.example.com"));
GuildManager->SetAuthToken(AuthToken);
GuildManager->SetLocalPlayerId(PlayerId);
GuildManager->SetGuildStateComponent(GuildState);
```

### 4. Create a Guild

=== "C++"

    ```cpp
    GuildManager->CreateGuild(TEXT("My Guild"), TEXT("A guild for adventurers."));
    GuildManager->OnGuildCreated.AddDynamic(this, &AMyPlayerController::HandleGuildCreated);
    ```

=== "Blueprint"

    Call **Create Guild** on the Guild Manager Component, then bind the **On Guild Created** event to handle the result.

---

## Permissions System

FWGuildSystem uses a bitmask-based permission model. Each rank holds a `uint8` permissions field where individual bits map to specific abilities.

| Permission | Value | Description |
|---|---|---|
| None | 0 | No permissions |
| Invite | 1 | Can send guild invitations |
| Kick | 2 | Can remove members |
| Promote | 4 | Can promote members |
| Demote | 8 | Can demote members |
| EditInfo | 16 | Can edit guild name and description |
| EditRanks | 32 | Can modify rank hierarchy |
| Disband | 64 | Can disband the guild |
| ViewAuditLog | 128 | Can view the guild audit log |

Combine permissions with bitwise OR. For example, an Officer rank with Invite, Kick, and ViewAuditLog would have a permissions value of `1 | 2 | 128 = 131`.

---

## Core Types

| Type | Kind | Description |
|---|---|---|
| `FFWGuildInfo` | Struct | Basic guild summary (ID, name, member count) |
| `FFWGuildRank` | Struct | Rank definition with name, priority, and permissions bitmask |
| `FFWGuildMember` | Struct | Member data including player ID, display name, rank, and join date |
| `FFWGuildInvitation` | Struct | Invitation record with sender, target, status, and expiry |
| `FFWGuildDetails` | Struct | Full guild data including info, ranks, members, and metadata |
| `EFWGuildOperationResult` | Enum | Result codes for all guild operations (Success, NotInGuild, InsufficientPermission, etc.) |
| `EFWGuildUpdateType` | Enum | Identifies which aspect of guild state changed |

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
    Guild operations that communicate with the backend should only execute on the server or in a client-authoritative context with proper validation. The `UFWGuildManagerComponent` handles authority checks internally when used on replicated actors.
