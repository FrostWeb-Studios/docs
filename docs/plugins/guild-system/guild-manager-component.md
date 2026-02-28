---
title: UFWGuildManagerComponent
description: Blueprint-exposed component providing all guild operations including creation, management, invitations, and searches.
---

# UFWGuildManagerComponent

**Header:** `FWGuildManagerComponent.h` | **Parent:** `UActorComponent` | **Specifier:** `BlueprintSpawnableComponent`

The primary interface for all guild operations. This component exposes every guild action to both C++ and Blueprint, communicates with the backend HTTP API, and broadcasts results via multicast delegates.

---

## Class Declaration

```cpp
UCLASS(ClassGroup = (FrostWeb), meta = (BlueprintSpawnableComponent))
class FWGUILDSYSTEM_API UFWGuildManagerComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFWGuildManagerComponent();

    // --- Guild Lifecycle ---
    UFUNCTION(BlueprintCallable, Category = "Guild|Lifecycle")
    void CreateGuild(const FString& GuildName, const FString& Description);

    UFUNCTION(BlueprintCallable, Category = "Guild|Lifecycle")
    void DisbandGuild();

    // --- Member Management ---
    UFUNCTION(BlueprintCallable, Category = "Guild|Members")
    void InvitePlayer(const FString& TargetPlayerId);

    UFUNCTION(BlueprintCallable, Category = "Guild|Members")
    void KickMember(const FString& MemberId);

    UFUNCTION(BlueprintCallable, Category = "Guild|Members")
    void PromoteMember(const FString& MemberId);

    UFUNCTION(BlueprintCallable, Category = "Guild|Members")
    void DemoteMember(const FString& MemberId);

    // --- Guild Configuration ---
    UFUNCTION(BlueprintCallable, Category = "Guild|Config")
    void EditGuildInfo(const FString& NewName, const FString& NewDescription);

    UFUNCTION(BlueprintCallable, Category = "Guild|Config")
    void EditGuildRanks(const TArray<FFWGuildRank>& NewRanks);

    // --- Discovery ---
    UFUNCTION(BlueprintCallable, Category = "Guild|Search")
    void SearchGuilds(const FString& Query, int32 Page = 0, int32 PageSize = 20);

    // --- Audit ---
    UFUNCTION(BlueprintCallable, Category = "Guild|Audit")
    void ViewAuditLog(int32 Page = 0, int32 PageSize = 50);
};
```

---

## Functions

### Guild Lifecycle

#### CreateGuild

```cpp
void CreateGuild(const FString& GuildName, const FString& Description);
```

Creates a new guild with the calling player as the guild leader. The guild is persisted via the HTTP API. On completion, broadcasts `OnGuildCreated` with the operation result.

| Parameter | Type | Description |
|---|---|---|
| `GuildName` | `FString` | Display name for the guild. Must be unique and between 3-32 characters. |
| `Description` | `FString` | Guild description. Maximum 256 characters. |

!!! note "Prerequisite"
    The calling player must not already belong to a guild. If they do, the operation returns `EFWGuildOperationResult::AlreadyInGuild`.

---

#### DisbandGuild

```cpp
void DisbandGuild();
```

Permanently disbands the guild. All members are removed and the guild record is deleted from the backend. Requires `EFWGuildPermission::Disband` (guild leader only by default).

!!! danger "Irreversible"
    Disbanding a guild cannot be undone. All rank configurations, member lists, and audit logs are permanently deleted.

---

### Member Management

#### InvitePlayer

```cpp
void InvitePlayer(const FString& TargetPlayerId);
```

Sends a guild invitation to the specified player. The invitation expires after the configured timeout (default: 72 hours). Requires `EFWGuildPermission::Invite`.

| Parameter | Type | Description |
|---|---|---|
| `TargetPlayerId` | `FString` | The unique player ID of the invitation target. |

**Possible Results:**

| Result | Condition |
|---|---|
| `Success` | Invitation sent successfully |
| `InsufficientPermission` | Caller lacks `Invite` permission |
| `PlayerAlreadyInGuild` | Target is already in a guild |
| `PlayerAlreadyInvited` | Target has a pending invitation from this guild |
| `GuildFull` | Guild has reached maximum member count |

---

#### KickMember

```cpp
void KickMember(const FString& MemberId);
```

Removes a member from the guild. Requires `EFWGuildPermission::Kick`. Cannot kick members of equal or higher rank.

| Parameter | Type | Description |
|---|---|---|
| `MemberId` | `FString` | The unique player ID of the member to remove. |

---

#### PromoteMember

```cpp
void PromoteMember(const FString& MemberId);
```

Promotes a member to the next higher rank. Requires `EFWGuildPermission::Promote`. Cannot promote a member to your own rank or above.

| Parameter | Type | Description |
|---|---|---|
| `MemberId` | `FString` | The unique player ID of the member to promote. |

---

#### DemoteMember

```cpp
void DemoteMember(const FString& MemberId);
```

Demotes a member to the next lower rank. Requires `EFWGuildPermission::Demote`. Cannot demote members of equal or higher rank.

| Parameter | Type | Description |
|---|---|---|
| `MemberId` | `FString` | The unique player ID of the member to demote. |

---

### Guild Configuration

#### EditGuildInfo

```cpp
void EditGuildInfo(const FString& NewName, const FString& NewDescription);
```

Updates the guild's display name and description. Requires `EFWGuildPermission::EditInfo`.

| Parameter | Type | Description |
|---|---|---|
| `NewName` | `FString` | New guild name. Pass empty string to keep current name. |
| `NewDescription` | `FString` | New guild description. Pass empty string to keep current description. |

---

#### EditGuildRanks

```cpp
void EditGuildRanks(const TArray<FFWGuildRank>& NewRanks);
```

Replaces the guild's entire rank hierarchy. Requires `EFWGuildPermission::EditRanks`. The first rank in the array is the highest (guild leader), the last is the lowest (new member default).

| Parameter | Type | Description |
|---|---|---|
| `NewRanks` | `TArray<FFWGuildRank>` | The complete ordered list of ranks. |

!!! warning "Rank Reordering"
    When ranks are edited, existing members retain their rank by name match. If a rank name is removed, affected members are moved to the lowest rank. Always confirm rank changes with the guild leader via UI before submitting.

---

### Discovery

#### SearchGuilds

```cpp
void SearchGuilds(const FString& Query, int32 Page = 0, int32 PageSize = 20);
```

Searches for guilds by name. Results are returned via `OnGuildSearchCompleted` and contain basic guild info for display in a browse/search UI.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `Query` | `FString` | | Search term to match against guild names. |
| `Page` | `int32` | `0` | Zero-based page index for pagination. |
| `PageSize` | `int32` | `20` | Number of results per page. Maximum 100. |

---

### Audit

#### ViewAuditLog

```cpp
void ViewAuditLog(int32 Page = 0, int32 PageSize = 50);
```

Retrieves the guild's audit log. Requires `EFWGuildPermission::ViewAuditLog`. Entries are returned in reverse chronological order via `OnAuditLogReceived`.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `Page` | `int32` | `0` | Zero-based page index. |
| `PageSize` | `int32` | `50` | Entries per page. Maximum 200. |

---

## Delegates

All delegates are `DECLARE_DYNAMIC_MULTICAST_DELEGATE` and are BlueprintAssignable.

```cpp
UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnGuildCreated OnGuildCreated;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnGuildDisbanded OnGuildDisbanded;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnPlayerInvited OnPlayerInvited;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnMemberKicked OnMemberKicked;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnMemberPromoted OnMemberPromoted;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnMemberDemoted OnMemberDemoted;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnGuildInfoEdited OnGuildInfoEdited;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnGuildRanksEdited OnGuildRanksEdited;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnGuildSearchCompleted OnGuildSearchCompleted;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnAuditLogReceived OnAuditLogReceived;

UPROPERTY(BlueprintAssignable, Category = "Guild|Events")
FOnGuildOperationFailed OnGuildOperationFailed;
```

See [Events and Delegates](events-delegates.md) for full delegate signatures and payload details.

---

## Usage Example

=== "C++"

    ```cpp
    void AMyPlayerController::BeginPlay()
    {
        Super::BeginPlay();

        GuildManager = FindComponentByClass<UFWGuildManagerComponent>();
        if (GuildManager)
        {
            GuildManager->OnGuildCreated.AddDynamic(
                this, &AMyPlayerController::HandleGuildCreated);
            GuildManager->OnGuildOperationFailed.AddDynamic(
                this, &AMyPlayerController::HandleGuildError);
        }
    }

    void AMyPlayerController::HandleGuildCreated(
        EFWGuildOperationResult Result, const FFWGuildDetails& Details)
    {
        if (Result == EFWGuildOperationResult::Success)
        {
            UE_LOG(LogGuild, Log, TEXT("Guild '%s' created with %d ranks."),
                *Details.Info.Name, Details.Ranks.Num());
        }
    }

    void AMyPlayerController::HandleGuildError(
        EFWGuildOperationResult Result, const FString& Message)
    {
        UE_LOG(LogGuild, Warning, TEXT("Guild operation failed: %s"), *Message);
    }
    ```

=== "Blueprint"

    1. Add a **FW Guild Manager Component** to your Player Controller.
    2. In the Event Graph, drag from the component and select **Bind Event to On Guild Created**.
    3. In the bound event, check the **Result** enum for `Success` before accessing guild details.
    4. Call **Create Guild** with a name and description to trigger the flow.
