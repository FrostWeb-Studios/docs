---
title: Guild Events and Delegates
description: Complete reference for all multicast delegates, event types, and event handling patterns in FWGuildSystem.
---

# Guild Events and Delegates

All delegates in FWGuildSystem are `DECLARE_DYNAMIC_MULTICAST_DELEGATE` variants and are marked `BlueprintAssignable`, making them available for both C++ binding and Blueprint event binding.

---

## Guild Manager Delegates

These delegates are broadcast by `UFWGuildManagerComponent` in response to completed operations.

### OnGuildCreated

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnGuildCreated,
    EFWGuildOperationResult, Result,
    const FFWGuildDetails&, GuildDetails);
```

Fires when `CreateGuild()` completes. On success, `GuildDetails` contains the full guild data including the default rank hierarchy and the creator as the sole member.

---

### OnGuildDisbanded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildDisbanded,
    EFWGuildOperationResult, Result);
```

Fires when `DisbandGuild()` completes. On success, the guild state component is cleared automatically.

---

### OnPlayerInvited

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnPlayerInvited,
    EFWGuildOperationResult, Result,
    const FString&, TargetPlayerId);
```

Fires when `InvitePlayer()` completes. The `TargetPlayerId` echoes back the player who was invited.

---

### OnMemberKicked

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnMemberKicked,
    EFWGuildOperationResult, Result,
    const FString&, MemberId);
```

Fires when `KickMember()` completes. On success, the kicked member is removed from the cached member list.

---

### OnMemberPromoted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnMemberPromoted,
    EFWGuildOperationResult, Result,
    const FString&, MemberId,
    const FFWGuildRank&, NewRank);
```

Fires when `PromoteMember()` completes. `NewRank` contains the member's new rank after promotion.

---

### OnMemberDemoted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnMemberDemoted,
    EFWGuildOperationResult, Result,
    const FString&, MemberId,
    const FFWGuildRank&, NewRank);
```

Fires when `DemoteMember()` completes. `NewRank` contains the member's new rank after demotion.

---

### OnGuildInfoEdited

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildInfoEdited,
    EFWGuildOperationResult, Result);
```

Fires when `EditGuildInfo()` completes.

---

### OnGuildRanksEdited

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildRanksEdited,
    EFWGuildOperationResult, Result);
```

Fires when `EditGuildRanks()` completes. The guild state component updates its cached rank hierarchy automatically.

---

### OnGuildSearchCompleted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildSearchCompleted,
    const FFWGuildSearchResult&, SearchResult);
```

Fires when `SearchGuilds()` completes. Contains paginated results.

---

### OnAuditLogReceived

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnAuditLogReceived,
    const TArray<FFWGuildAuditLogEntry>&, Entries,
    int32, TotalEntries,
    int32, CurrentPage);
```

Fires when `ViewAuditLog()` completes. Contains a page of audit log entries with pagination metadata.

---

### OnGuildOperationFailed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnGuildOperationFailed,
    EFWGuildOperationResult, Result,
    const FString&, ErrorMessage);
```

Generic failure delegate that fires alongside any operation-specific delegate when the result is not `Success`. Useful for centralized error handling and UI error display.

---

## Guild State Delegates

These delegates are broadcast by `UFWGuildStateComponent` in response to state changes.

### OnGuildStateChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnGuildStateChanged);
```

Fires whenever any aspect of the cached guild state is modified. This is the primary delegate for UI refresh -- bind to this in any widget that displays guild data.

!!! tip "Debouncing"
    Multiple state changes can fire in quick succession (e.g., a batch rank update affecting several members). Consider debouncing your UI refresh to avoid redundant rebuilds.

---

### OnMemberOnlineStatusChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnGuildMemberOnlineStatusChanged,
    const FString&, PlayerId,
    bool, bIsOnline);
```

Fires when a guild member comes online or goes offline. Useful for updating presence indicators in the guild roster UI.

---

### OnInvitationReceived

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildInvitationReceived,
    const FFWGuildInvitation&, Invitation);
```

Fires when the owning player receives a new guild invitation. Use this to show an invitation popup or notification.

---

### OnInvitationExpired

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildInvitationExpired,
    const FString&, InvitationId);
```

Fires when a pending invitation reaches its expiry time. Remove the invitation from your UI when this fires.

---

## Chat Integration Delegates

These delegates are broadcast by `UFWGuildChatIntegrationComponent`.

### OnGuildChatMessageReceived

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnGuildChatMessageReceived,
    const FString&, SenderId,
    const FString&, Message);
```

Fires when a message is received in the guild chat channel.

---

### OnGuildChatChannelCreated

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildChatChannelCreated,
    const FString&, ChannelName);
```

Fires when the guild chat channel is successfully provisioned.

---

### OnGuildChatChannelDestroyed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildChatChannelDestroyed,
    const FString&, ChannelName);
```

Fires when the guild chat channel is destroyed (guild disbanded).

---

## Binding Patterns

### C++ Binding

```cpp
void AMyPlayerController::SetupGuildBindings()
{
    auto* GuildManager = FindComponentByClass<UFWGuildManagerComponent>();
    auto* GuildState = FindComponentByClass<UFWGuildStateComponent>();

    if (GuildManager)
    {
        // Bind specific operation delegates
        GuildManager->OnGuildCreated.AddDynamic(
            this, &AMyPlayerController::OnGuildCreated);
        GuildManager->OnPlayerInvited.AddDynamic(
            this, &AMyPlayerController::OnPlayerInvited);

        // Bind generic error handler
        GuildManager->OnGuildOperationFailed.AddDynamic(
            this, &AMyPlayerController::OnGuildError);
    }

    if (GuildState)
    {
        // Bind state change for UI refresh
        GuildState->OnGuildStateChanged.AddDynamic(
            this, &AMyPlayerController::RefreshGuildUI);
        GuildState->OnInvitationReceived.AddDynamic(
            this, &AMyPlayerController::ShowInvitationPopup);
    }
}
```

### Blueprint Binding

1. In your widget or actor Blueprint, get a reference to the guild component.
2. Drag from the component pin and search for **Bind Event**.
3. Select the desired delegate (e.g., **Bind Event to On Guild State Changed**).
4. Connect the event node to your handler logic.

### Unbinding

Always unbind delegates when the listening object is destroyed to prevent dangling references:

```cpp
void AMyPlayerController::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
    if (auto* GuildManager = FindComponentByClass<UFWGuildManagerComponent>())
    {
        GuildManager->OnGuildCreated.RemoveDynamic(
            this, &AMyPlayerController::OnGuildCreated);
        GuildManager->OnGuildOperationFailed.RemoveDynamic(
            this, &AMyPlayerController::OnGuildError);
    }

    Super::EndPlay(EndPlayReason);
}
```

---

## Server Push Events

In addition to delegates that fire in response to player-initiated operations, the guild system receives server push events for actions performed by other guild members. These are delivered via the `FFWGuildUpdateEvent` struct and processed by the guild state component, which fires `OnGuildStateChanged` after updating the cache.

### Push Event Flow

```
Backend Server
    |
    v
WebSocket / Long-Poll
    |
    v
UFWGuildManagerComponent::HandleServerPushEvent()
    |
    v
UFWGuildStateComponent (cache update)
    |
    +--> OnGuildStateChanged
    +--> OnMemberOnlineStatusChanged (if applicable)
    +--> UFWGuildChatIntegrationComponent (system message)
```

### Supported Push Events

| Event Type | Trigger | State Update |
|---|---|---|
| `MemberJoined` | Another player accepted an invitation | Member added to list |
| `MemberLeft` | A member voluntarily left | Member removed from list |
| `MemberKicked` | An officer kicked a member | Member removed from list |
| `MemberPromoted` | A member was promoted | Member rank updated |
| `MemberDemoted` | A member was demoted | Member rank updated |
| `GuildInfoEdited` | Guild name/description changed | Info updated |
| `RanksEdited` | Rank hierarchy changed | Ranks and member ranks updated |
| `GuildDisbanded` | Guild leader disbanded the guild | Entire state cleared |
