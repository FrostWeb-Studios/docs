---
title: Guild System API Reference
description: Complete API reference for all functions, properties, and delegates in FWGuildSystem.
---

# Guild System API Reference

Complete function and delegate reference for the FWGuildSystem plugin, organized by component.

---

## UFWGuildManagerComponent

The primary interface for all guild operations. Attach to your Player Controller or a replicated actor.

### Configuration

| Function | Parameters | Description |
|---|---|---|
| `SetApiBaseUrl` | `Url: FString` | Sets the backend API base URL |
| `SetAuthToken` | `Token: FString` | Sets the authentication token for API requests |
| `SetLocalPlayerId` | `PlayerId: FString` | Sets the local player's unique ID |
| `SetGuildStateComponent` | `StateComponent: UFWGuildStateComponent*` | Links the state component for automatic cache updates |

### Guild Operations

| Function | Parameters | Description |
|---|---|---|
| `CreateGuild` | `GuildName: FString`, `Description: FString` | Creates a new guild with the caller as leader |
| `FetchMyGuild` | -- | Fetches the current player's guild from the backend |
| `DisbandGuild` | -- | Permanently disbands the guild. Requires `Disband` permission |
| `SearchGuilds` | `Query: FString`, `Page: int32`, `PageSize: int32` | Searches guilds by name |
| `EditGuildInfo` | `NewName: FString`, `NewDescription: FString` | Updates guild name/description. Requires `EditInfo` permission |

### Member Management

| Function | Parameters | Description |
|---|---|---|
| `InvitePlayer` | `TargetPlayerId: FString` | Sends a guild invitation. Requires `Invite` permission |
| `KickMember` | `MemberId: FString` | Removes a member. Requires `Kick` permission |
| `PromoteMember` | `MemberId: FString` | Promotes to the next rank. Requires `Promote` permission |
| `DemoteMember` | `MemberId: FString` | Demotes to the next rank. Requires `Demote` permission |

### Invitation Management

| Function | Parameters | Description |
|---|---|---|
| `AcceptInvitation` | `InvitationId: FString` | Accepts a pending guild invitation |
| `DeclineInvitation` | `InvitationId: FString` | Declines a pending guild invitation |

### Rank Management

| Function | Parameters | Description |
|---|---|---|
| `EditGuildRanks` | `NewRanks: TArray<FFWGuildRank>` | Replaces the rank hierarchy. Requires `EditRanks` permission |

### Audit Log

| Function | Parameters | Description |
|---|---|---|
| `ViewAuditLog` | `Page: int32`, `PageSize: int32` | Retrieves a paginated audit log. Requires `ViewAuditLog` permission |

### Events

| Delegate | Signature | Description |
|---|---|---|
| `OnGuildCreated` | `(EFWGuildOperationResult Result, FFWGuildDetails Details)` | Guild creation completed |
| `OnGuildDisbanded` | `(EFWGuildOperationResult Result)` | Guild disband completed |
| `OnGuildFetched` | `(EFWGuildOperationResult Result, FFWGuildDetails Details)` | Guild fetch completed |
| `OnPlayerInvited` | `(EFWGuildOperationResult Result, FString TargetPlayerId)` | Invitation sent |
| `OnMemberKicked` | `(EFWGuildOperationResult Result, FString MemberId)` | Member removal completed |
| `OnMemberPromoted` | `(EFWGuildOperationResult Result, FString MemberId, FFWGuildRank NewRank)` | Promotion completed |
| `OnMemberDemoted` | `(EFWGuildOperationResult Result, FString MemberId, FFWGuildRank NewRank)` | Demotion completed |
| `OnGuildInfoEdited` | `(EFWGuildOperationResult Result)` | Info edit completed |
| `OnGuildRanksEdited` | `(EFWGuildOperationResult Result)` | Rank edit completed |
| `OnGuildSearchCompleted` | `(TArray<FFWGuildInfo> Results, int32 TotalResults, int32 CurrentPage)` | Search results received |
| `OnAuditLogReceived` | `(TArray<FFWGuildAuditLogEntry> Entries, int32 TotalEntries, int32 CurrentPage)` | Audit log page received |
| `OnGuildOperationFailed` | `(EFWGuildOperationResult Result, FString Message)` | Generic operation failure |

---

## UFWGuildStateComponent

Read-only local cache of the current guild state. Automatically updated by `UFWGuildManagerComponent`.

### Guild Queries

| Function | Returns | Description |
|---|---|---|
| `IsInGuild` | `bool` | Whether the owning player belongs to a guild |
| `GetGuildInfo` | `FFWGuildInfo` | Basic guild summary |
| `GetGuildDetails` | `FFWGuildDetails` | Full guild data with ranks and members |

### Member Queries

| Function | Returns | Description |
|---|---|---|
| `GetMembers` | `TArray<FFWGuildMember>` | Full member list |
| `GetMemberCount` | `int32` | Total member count |
| `GetOnlineMemberCount` | `int32` | Online member count |
| `FindMemberById` | `FFWGuildMember` | Looks up a member by player ID |

### Permission Checks

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `HasPermission` | `Permission: EFWGuildPermission` | `bool` | Checks a permission flag against the current player's rank |

### Rank Queries

| Function | Returns | Description |
|---|---|---|
| `GetMyRank` | `FFWGuildRank` | Current player's rank |
| `GetRanks` | `TArray<FFWGuildRank>` | Full rank hierarchy |
| `GetPendingInvitations` | `TArray<FFWGuildInvitation>` | Pending invitations for the player |

### Events

| Delegate | Signature | Description |
|---|---|---|
| `OnGuildStateChanged` | `()` | Fires on any guild state mutation |
| `OnMemberOnlineStatusChanged` | `(FString PlayerId, bool bIsOnline)` | Member online/offline status change |
| `OnInvitationReceived` | `(FFWGuildInvitation Invitation)` | New invitation received |
| `OnInvitationExpired` | `(FString InvitationId)` | Invitation expired |

---

## UFWGuildChatIntegrationComponent

Optional bridge between the guild system and FWChatSystem. Requires FWChatSystem to be loaded; silently disables itself when unavailable.

### Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `SetChatChannelPrefix` | `Prefix: FString` | `void` | Sets the guild channel name prefix |
| `SendGuildMessage` | `Message: FString` | `void` | Sends a player message to guild chat |
| `SendGuildSystemMessage` | `Message: FString` | `void` | Sends a system message to guild chat |
| `GetGuildChannelName` | -- | `FString` | Returns the full guild chat channel name |
| `IsChatSystemAvailable` | -- | `bool` | Whether FWChatSystem is loaded |

### Events

| Delegate | Signature | Description |
|---|---|---|
| `OnGuildChatMessageReceived` | `(FString SenderId, FString Message)` | Chat message received in guild channel |
| `OnGuildChatChannelCreated` | `(FString ChannelName)` | Guild chat channel created |
| `OnGuildChatChannelDestroyed` | `(FString ChannelName)` | Guild chat channel destroyed |

---

## Operation Flow

The following shows the typical flow for a guild operation:

```
Player Action
    |
    v
UFWGuildManagerComponent
    |
    +--> Permission check via UFWGuildStateComponent::HasPermission()
    |       |
    |       +--> Denied: OnGuildOperationFailed delegate fires
    |
    +--> Backend request
            |
            +--> Success: Operation-specific delegate fires
            |               |
            |               +--> UFWGuildStateComponent updates cached state
            |               |
            |               +--> UFWGuildChatIntegrationComponent posts system message (if available)
            |               |
            |               +--> OnGuildStateChanged fires (UI refresh)
            |
            +--> Failure: OnGuildOperationFailed delegate fires
```

---

## Error Handling

All operations follow the same error handling pattern:

1. **Local validation** -- permission checks, state preconditions.
2. **Backend request** -- async call to the backend.
3. **Result delegate** -- operation-specific delegate with `EFWGuildOperationResult`.
4. **Fallback delegate** -- `OnGuildOperationFailed` fires for any non-success result.

Both the specific and generic delegates fire on failure, so you can handle errors at either granularity.

!!! tip "UI Error Display"
    Use `GetResultMessage()` to convert result codes to localized user-facing text for display in UI popups.
