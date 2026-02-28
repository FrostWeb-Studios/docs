---
title: Types Reference - FWPartySystem
description: Complete reference for all enums, structs, and delegate types defined in FWPartyTypes.h.
---

# Types Reference

All types are defined in `FWPartyTypes.h` and are available to both C++ and Blueprints.

---

## Enums

### EFWPartyRole

Defines the role of a party member.

```cpp
UENUM(BlueprintType)
enum class EFWPartyRole : uint8
{
    Leader   UMETA(DisplayName = "Leader"),
    Member   UMETA(DisplayName = "Member"),
    Pending  UMETA(DisplayName = "Pending")
};
```

| Value | Description |
|---|---|
| `Leader` | The party leader. Only one per party. Can kick, invite, and change settings. |
| `Member` | A regular party member. Can leave and (optionally) invite others. |
| `Pending` | A player who has been invited but has not yet accepted. Not visible to other members. |

---

### EFWPartyJoinResult

Result of a join attempt returned by `OnJoinPartyResult`.

```cpp
UENUM(BlueprintType)
enum class EFWPartyJoinResult : uint8
{
    Success         UMETA(DisplayName = "Success"),
    PartyFull       UMETA(DisplayName = "Party Full"),
    InvalidCode     UMETA(DisplayName = "Invalid Code"),
    InviteOnly      UMETA(DisplayName = "Invite Only"),
    AlreadyInParty  UMETA(DisplayName = "Already In Party"),
    Banned          UMETA(DisplayName = "Banned"),
    ServerError     UMETA(DisplayName = "Server Error")
};
```

| Value | Description |
|---|---|
| `Success` | The player successfully joined the party |
| `PartyFull` | The party has reached its maximum member count |
| `InvalidCode` | No active party matches the provided join code |
| `InviteOnly` | The party is invite-only and the player has no pending invitation |
| `AlreadyInParty` | The player is already in a party (must leave first) |
| `Banned` | The player was previously kicked and is temporarily banned from this party |
| `ServerError` | An unexpected server-side error occurred |

---

### EFWPartyPrivacy

Controls how players can join the party.

```cpp
UENUM(BlueprintType)
enum class EFWPartyPrivacy : uint8
{
    Open       UMETA(DisplayName = "Open"),
    InviteOnly UMETA(DisplayName = "Invite Only")
};
```

| Value | Description |
|---|---|
| `Open` | Any player with the join code can join directly |
| `InviteOnly` | Players must receive and accept an invitation before joining |

---

## Structs

### FFWPartyMemberInfo

Information about a single party member. Replicated to all party members.

```cpp
USTRUCT(BlueprintType)
struct FFWPartyMemberInfo
{
    GENERATED_BODY()

    /** Server-assigned member identifier. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString MemberId;

    /** Display name for UI rendering. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString DisplayName;

    /** Platform-specific unique network identity. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FUniqueNetIdRepl UniqueNetId;

    /** Current role in the party. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    EFWPartyRole Role;

    /** Whether the member is currently connected. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    bool bIsOnline;

    /** Timestamp when the member joined the party (UTC). */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FDateTime JoinedAt;
};
```

---

### FFWPartyInfo

Complete party state. Replicated from the server to all party members.

```cpp
USTRUCT(BlueprintType)
struct FFWPartyInfo
{
    GENERATED_BODY()

    /** Unique party identifier assigned by the server. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString PartyId;

    /** Optional display name for the party. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString PartyName;

    /** Short alphanumeric code for direct join. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString JoinCode;

    /** Whether the party is open or invite-only. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    EFWPartyPrivacy Privacy;

    /** Maximum number of members allowed. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    int32 MaxMembers;

    /** Current member list. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    TArray<FFWPartyMemberInfo> Members;

    /** Timestamp when the party was created (UTC). */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FDateTime CreatedAt;
};
```

---

### FFWPartyInvitation

Represents a pending party invitation sent to a player.

```cpp
USTRUCT(BlueprintType)
struct FFWPartyInvitation
{
    GENERATED_BODY()

    /** Unique invitation identifier. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString InvitationId;

    /** The party this invitation is for. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString PartyId;

    /** Display name of the player who sent the invitation. */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FString SenderDisplayName;

    /** When this invitation expires (UTC). */
    UPROPERTY(BlueprintReadOnly, Category = "Party")
    FDateTime ExpiresAt;
};
```

---

### FFWPartySettings

Configurable defaults for party creation and behavior.

```cpp
USTRUCT(BlueprintType)
struct FFWPartySettings
{
    GENERATED_BODY()

    /** Default maximum party size. */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Party")
    int32 DefaultMaxMembers = 4;

    /** Default privacy mode for new parties. */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Party")
    EFWPartyPrivacy DefaultPrivacy = EFWPartyPrivacy::Open;

    /** How long invitations remain valid (seconds). */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Party")
    float InvitationExpirySeconds = 120.0f;

    /** Grace period before disconnected members are removed (seconds). */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Party")
    float DisconnectGracePeriod = 60.0f;

    /** Whether non-leader members can send invitations. */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Party")
    bool bAllowMemberInvites = true;
};
```

---

## Delegates

### Party Lifecycle Delegates

```cpp
/** Fired when a party is successfully created. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnPartyCreated, const FFWPartyInfo&, PartyInfo);

/** Fired when the current party is disbanded. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnPartyDisbanded);

/** Fired when any party property changes. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnPartyUpdated, const FFWPartyInfo&, PartyInfo);
```

### Membership Delegates

```cpp
/** Fired when a new member joins the party. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMemberJoined, const FFWPartyMemberInfo&, MemberInfo);

/** Fired when a member leaves the party. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMemberLeft, const FFWPartyMemberInfo&, MemberInfo);

/** Fired when a member is kicked from the party. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMemberKicked, const FFWPartyMemberInfo&, MemberInfo);

/** Fired when party leadership changes. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnLeaderChanged, const FFWPartyMemberInfo&, NewLeader);
```

### Local Player Delegates

```cpp
/** Fired on the local player when they join a party. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnLocalPlayerJoinedParty, const FFWPartyInfo&, PartyInfo);

/** Fired on the local player when they leave or are kicked. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnLocalPlayerLeftParty);

/** Fired with the result of a join attempt. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnJoinPartyResult, EFWPartyJoinResult, Result);
```

### Invitation Delegates

```cpp
/** Fired when an invitation is received from another player. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnInvitationReceived, const FFWPartyInvitation&, Invitation);

/** Fired when a pending invitation expires. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnInvitationExpired, const FString&, InvitationId);
```

---

## Next Steps

- See [API Reference](api-reference.md) for the complete function listing.
- See [Events and Delegates](events-delegates.md) for event flow diagrams and binding patterns.
