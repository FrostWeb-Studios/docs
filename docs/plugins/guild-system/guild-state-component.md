---
title: UFWGuildStateComponent
description: Local guild state management component that caches guild data and provides read access to current guild information.
---

# UFWGuildStateComponent

**Header:** `FWGuildStateComponent.h` | **Parent:** `UActorComponent`

Manages the local cache of guild data for the owning player. This component stores the current guild details, member list, rank hierarchy, and pending invitations. It is updated automatically by `UFWGuildManagerComponent` after successful operations and via server push events.

---

## Class Declaration

```cpp
UCLASS(ClassGroup = (FrostWeb))
class FWGUILDSYSTEM_API UFWGuildStateComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFWGuildStateComponent();

    // --- State Queries ---
    UFUNCTION(BlueprintPure, Category = "Guild|State")
    bool IsInGuild() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    const FFWGuildInfo& GetGuildInfo() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    const FFWGuildDetails& GetGuildDetails() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    const FFWGuildRank& GetMyRank() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    bool HasPermission(EFWGuildPermission Permission) const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    const TArray<FFWGuildMember>& GetMembers() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    const TArray<FFWGuildRank>& GetRanks() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    const TArray<FFWGuildInvitation>& GetPendingInvitations() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    int32 GetMemberCount() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    int32 GetOnlineMemberCount() const;

    UFUNCTION(BlueprintPure, Category = "Guild|State")
    const FFWGuildMember* FindMemberById(const FString& PlayerId) const;

    // --- State Mutation (called internally by GuildManager) ---
    void SetGuildDetails(const FFWGuildDetails& InDetails);
    void ClearGuildState();
    void AddPendingInvitation(const FFWGuildInvitation& Invitation);
    void RemovePendingInvitation(const FString& InvitationId);
    void UpdateMember(const FFWGuildMember& UpdatedMember);
    void RemoveMember(const FString& MemberId);

    // --- Delegates ---
    UPROPERTY(BlueprintAssignable, Category = "Guild|State")
    FOnGuildStateChanged OnGuildStateChanged;

    UPROPERTY(BlueprintAssignable, Category = "Guild|State")
    FOnGuildMemberOnlineStatusChanged OnMemberOnlineStatusChanged;

    UPROPERTY(BlueprintAssignable, Category = "Guild|State")
    FOnGuildInvitationReceived OnInvitationReceived;

    UPROPERTY(BlueprintAssignable, Category = "Guild|State")
    FOnGuildInvitationExpired OnInvitationExpired;
};
```

---

## Functions

### State Queries

#### IsInGuild

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
bool IsInGuild() const;
```

Returns `true` if the owning player currently belongs to a guild.

---

#### GetGuildInfo

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
const FFWGuildInfo& GetGuildInfo() const;
```

Returns the basic guild information struct (name, description, member count, creation date). Returns a default-constructed struct if not in a guild.

!!! tip "Null Safety"
    Always check `IsInGuild()` before reading guild info to avoid operating on default values.

---

#### GetGuildDetails

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
const FFWGuildDetails& GetGuildDetails() const;
```

Returns the full guild details including the rank hierarchy and complete member list. This is a heavier struct than `FFWGuildInfo` -- use `GetGuildInfo()` when you only need summary data.

---

#### GetMyRank

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
const FFWGuildRank& GetMyRank() const;
```

Returns the current rank of the owning player within the guild.

---

#### HasPermission

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
bool HasPermission(EFWGuildPermission Permission) const;
```

Checks whether the owning player's current rank includes the specified permission flag. Uses bitmask comparison against the rank's permission field.

| Parameter | Type | Description |
|---|---|---|
| `Permission` | `EFWGuildPermission` | The permission flag to check. |

**Example:**

```cpp
if (GuildState->HasPermission(EFWGuildPermission::Kick))
{
    // Show kick button in member context menu
}
```

---

#### GetMembers

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
const TArray<FFWGuildMember>& GetMembers() const;
```

Returns the complete member list for the current guild. Each entry includes player ID, display name, rank, online status, join date, and last seen timestamp.

---

#### GetRanks

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
const TArray<FFWGuildRank>& GetRanks() const;
```

Returns the rank hierarchy in order from highest (index 0, guild leader) to lowest.

---

#### GetPendingInvitations

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
const TArray<FFWGuildInvitation>& GetPendingInvitations() const;
```

Returns all pending guild invitations for the owning player. This includes invitations from other guilds that the player has not yet accepted or declined.

---

#### GetMemberCount / GetOnlineMemberCount

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
int32 GetMemberCount() const;

UFUNCTION(BlueprintPure, Category = "Guild|State")
int32 GetOnlineMemberCount() const;
```

Convenience functions that return the total and currently-online member counts respectively.

---

#### FindMemberById

```cpp
UFUNCTION(BlueprintPure, Category = "Guild|State")
const FFWGuildMember* FindMemberById(const FString& PlayerId) const;
```

Looks up a specific member by player ID. Returns `nullptr` if the player is not a member of the current guild.

| Parameter | Type | Description |
|---|---|---|
| `PlayerId` | `FString` | The unique player ID to search for. |

---

## State Mutation

The following functions are called internally by `UFWGuildManagerComponent` and should not be called directly from game code.

| Function | Description |
|---|---|
| `SetGuildDetails` | Replaces the entire cached guild state. |
| `ClearGuildState` | Clears all guild data (called on leave/disband/kick). |
| `AddPendingInvitation` | Adds an incoming invitation to the pending list. |
| `RemovePendingInvitation` | Removes an invitation by ID (accepted, declined, or expired). |
| `UpdateMember` | Updates a single member entry (rank change, online status). |
| `RemoveMember` | Removes a member from the cached list. |

!!! warning "Internal API"
    The state mutation functions are public for component-to-component access but are not intended for direct use. Calling them without a corresponding backend operation will cause state desynchronization.

---

## Delegates

### OnGuildStateChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnGuildStateChanged);
```

Broadcast whenever the cached guild state is modified. Useful for refreshing UI panels.

### OnMemberOnlineStatusChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnGuildMemberOnlineStatusChanged,
    const FString&, PlayerId,
    bool, bIsOnline);
```

Broadcast when a guild member's online status changes.

### OnInvitationReceived

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildInvitationReceived,
    const FFWGuildInvitation&, Invitation);
```

Broadcast when a new guild invitation is received from another guild.

### OnInvitationExpired

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnGuildInvitationExpired,
    const FString&, InvitationId);
```

Broadcast when a pending invitation expires.

---

## Usage Example

=== "C++"

    ```cpp
    void UGuildRosterWidget::RefreshRoster()
    {
        UFWGuildStateComponent* GuildState =
            GetOwningPlayerController()->FindComponentByClass<UFWGuildStateComponent>();

        if (!GuildState || !GuildState->IsInGuild())
        {
            ShowNoGuildMessage();
            return;
        }

        const TArray<FFWGuildMember>& Members = GuildState->GetMembers();
        for (const FFWGuildMember& Member : Members)
        {
            AddMemberRow(Member.DisplayName, Member.Rank.Name, Member.bIsOnline);
        }

        SetGuildName(GuildState->GetGuildInfo().Name);
        SetMemberCountText(FString::Printf(TEXT("%d / %d online"),
            GuildState->GetOnlineMemberCount(),
            GuildState->GetMemberCount()));
    }
    ```

=== "Blueprint"

    1. Get a reference to the **FW Guild State Component** on your Player Controller.
    2. Call **Is In Guild** to check membership.
    3. Use **Get Members** to populate a roster list widget.
    4. Bind **On Guild State Changed** to automatically refresh the UI when data changes.
