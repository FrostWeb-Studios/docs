---
title: Guild System API Reference
description: Complete API reference for all BlueprintCallable functions, delegates, and static utilities in FWGuildSystem.
---

# Guild System API Reference

Complete function and delegate reference for the FWGuildSystem plugin, organized by component.

---

## UFWGuildManagerComponent

### BlueprintCallable Functions

| Function | Category | Parameters | Returns | Description |
|---|---|---|---|---|
| `CreateGuild` | Lifecycle | `GuildName: FString`, `Description: FString` | `void` | Creates a new guild with the caller as leader. |
| `DisbandGuild` | Lifecycle | -- | `void` | Permanently disbands the guild. Requires `Disband` permission. |
| `InvitePlayer` | Members | `TargetPlayerId: FString` | `void` | Sends a guild invitation. Requires `Invite` permission. |
| `KickMember` | Members | `MemberId: FString` | `void` | Removes a member. Requires `Kick` permission. |
| `PromoteMember` | Members | `MemberId: FString` | `void` | Promotes to next rank. Requires `Promote` permission. |
| `DemoteMember` | Members | `MemberId: FString` | `void` | Demotes to next rank. Requires `Demote` permission. |
| `EditGuildInfo` | Config | `NewName: FString`, `NewDescription: FString` | `void` | Updates guild name/description. Requires `EditInfo` permission. |
| `EditGuildRanks` | Config | `NewRanks: TArray<FFWGuildRank>` | `void` | Replaces rank hierarchy. Requires `EditRanks` permission. |
| `SearchGuilds` | Search | `Query: FString`, `Page: int32 = 0`, `PageSize: int32 = 20` | `void` | Searches guilds by name. |
| `ViewAuditLog` | Audit | `Page: int32 = 0`, `PageSize: int32 = 50` | `void` | Retrieves paginated audit log. Requires `ViewAuditLog` permission. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnGuildCreated` | `(EFWGuildOperationResult Result, const FFWGuildDetails& Details)` | Guild creation completed. |
| `OnGuildDisbanded` | `(EFWGuildOperationResult Result)` | Guild disband completed. |
| `OnPlayerInvited` | `(EFWGuildOperationResult Result, const FString& TargetPlayerId)` | Invitation sent. |
| `OnMemberKicked` | `(EFWGuildOperationResult Result, const FString& MemberId)` | Member removal completed. |
| `OnMemberPromoted` | `(EFWGuildOperationResult Result, const FString& MemberId, const FFWGuildRank& NewRank)` | Promotion completed. |
| `OnMemberDemoted` | `(EFWGuildOperationResult Result, const FString& MemberId, const FFWGuildRank& NewRank)` | Demotion completed. |
| `OnGuildInfoEdited` | `(EFWGuildOperationResult Result)` | Info edit completed. |
| `OnGuildRanksEdited` | `(EFWGuildOperationResult Result)` | Rank edit completed. |
| `OnGuildSearchCompleted` | `(const FFWGuildSearchResult& Result)` | Search results received. |
| `OnAuditLogReceived` | `(const TArray<FFWGuildAuditLogEntry>& Entries, int32 TotalEntries, int32 CurrentPage)` | Audit log page received. |
| `OnGuildOperationFailed` | `(EFWGuildOperationResult Result, const FString& Message)` | Generic operation failure. |

---

## UFWGuildStateComponent

### BlueprintPure Functions

| Function | Returns | Description |
|---|---|---|
| `IsInGuild` | `bool` | Whether the owning player belongs to a guild. |
| `GetGuildInfo` | `const FFWGuildInfo&` | Basic guild summary. |
| `GetGuildDetails` | `const FFWGuildDetails&` | Full guild data with ranks and members. |
| `GetMyRank` | `const FFWGuildRank&` | Current player's rank. |
| `HasPermission` | `bool` | Checks a permission flag against current rank. |
| `GetMembers` | `const TArray<FFWGuildMember>&` | Full member list. |
| `GetRanks` | `const TArray<FFWGuildRank>&` | Rank hierarchy. |
| `GetPendingInvitations` | `const TArray<FFWGuildInvitation>&` | Pending invitations for the player. |
| `GetMemberCount` | `int32` | Total member count. |
| `GetOnlineMemberCount` | `int32` | Online member count. |
| `FindMemberById` | `const FFWGuildMember*` | Looks up member by player ID. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnGuildStateChanged` | `()` | Any guild state mutation. |
| `OnMemberOnlineStatusChanged` | `(const FString& PlayerId, bool bIsOnline)` | Member online/offline change. |
| `OnInvitationReceived` | `(const FFWGuildInvitation& Invitation)` | New invitation received. |
| `OnInvitationExpired` | `(const FString& InvitationId)` | Invitation expired. |

---

## UFWGuildChatIntegrationComponent

### BlueprintCallable Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `SetChatChannelPrefix` | `Prefix: FString` | `void` | Sets the guild channel name prefix. |
| `SendGuildMessage` | `Message: FString` | `void` | Sends a player message to guild chat. |
| `SendGuildSystemMessage` | `Message: FString` | `void` | Sends a system message to guild chat. |

### BlueprintPure Functions

| Function | Returns | Description |
|---|---|---|
| `GetGuildChannelName` | `FString` | Full guild chat channel name. |
| `IsChatSystemAvailable` | `bool` | Whether FWChatSystem is loaded. |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnGuildChatMessageReceived` | `(const FString& SenderId, const FString& Message)` | Chat message received. |
| `OnGuildChatChannelCreated` | `(const FString& ChannelName)` | Channel created. |
| `OnGuildChatChannelDestroyed` | `(const FString& ChannelName)` | Channel destroyed. |

---

## FFWGuildTypeUtils (Static)

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `PermissionsToString` | `Permissions: uint8` | `FString` | Human-readable permission list. |
| `ParseGuildDetails` | `JsonString: FString`, `OutDetails: FFWGuildDetails&` | `bool` | Parses JSON to guild details. |
| `SerializeGuildDetails` | `Details: const FFWGuildDetails&` | `FString` | Serializes guild details to JSON. |
| `GetDefaultRanks` | -- | `TArray<FFWGuildRank>` | Returns default 4-rank hierarchy. |
| `CanActOnRank` | `SourceRank: FFWGuildRank`, `TargetRank: FFWGuildRank` | `bool` | Checks rank priority for actions. |
| `GetResultMessage` | `Result: EFWGuildOperationResult` | `FText` | User-facing result message. |
| `UpdateTypeToString` | `UpdateType: EFWGuildUpdateType` | `FString` | Display string for update type. |

---

## Operation Flow Diagram

The following shows the typical flow for a guild operation:

```
Player Action
    |
    v
UFWGuildManagerComponent (BlueprintCallable)
    |
    +--> Permission check via UFWGuildStateComponent::HasPermission()
    |       |
    |       +--> Denied: OnGuildOperationFailed delegate fires
    |
    +--> HTTP request to backend API
            |
            +--> Success: Operation-specific delegate fires
            |               |
            |               +--> UFWGuildStateComponent updates cached state
            |               |
            |               +--> UFWGuildChatIntegrationComponent posts system message
            |               |
            |               +--> OnGuildStateChanged fires (UI refresh)
            |
            +--> Failure: OnGuildOperationFailed delegate fires
```

---

## Error Handling

All operations follow the same error handling pattern:

1. **Local validation** -- permission checks, state preconditions.
2. **Network request** -- HTTP call to the backend API.
3. **Result delegate** -- operation-specific delegate with `EFWGuildOperationResult`.
4. **Fallback delegate** -- `OnGuildOperationFailed` fires for any non-success result.

```cpp
// Both the specific and generic delegates fire on failure
GuildManager->OnMemberKicked.AddDynamic(this, &AMyPC::HandleKickResult);
GuildManager->OnGuildOperationFailed.AddDynamic(this, &AMyPC::HandleAnyError);
```

!!! tip "UI Error Display"
    Use `FFWGuildTypeUtils::GetResultMessage()` to convert result codes to localized user-facing text for display in UI popups.
