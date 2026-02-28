---
title: Guild Types Reference
description: Complete reference for all enums, structs, and type utilities in FWGuildTypes.h.
---

# Guild Types Reference

**Header:** `FWGuildTypes.h`

All type definitions used by the FWGuildSystem plugin. Every struct is marked `BlueprintType` for full Blueprint compatibility.

---

## Enums

### EFWGuildPermission

Bitmask enum defining granular permissions assignable to guild ranks. Multiple permissions are combined using bitwise OR.

```cpp
UENUM(BlueprintType, Meta = (Bitflags, UseEnumValuesAsBitmaskValues = "true"))
enum class EFWGuildPermission : uint8
{
    None           = 0       UMETA(Hidden),
    Invite         = 1 << 0, // 1   - Invite new members
    Kick           = 1 << 1, // 2   - Remove members
    Promote        = 1 << 2, // 4   - Promote members to higher rank
    Demote         = 1 << 3, // 8   - Demote members to lower rank
    EditInfo       = 1 << 4, // 16  - Edit guild name and description
    EditRanks      = 1 << 5, // 32  - Modify rank hierarchy and permissions
    Disband        = 1 << 6, // 64  - Disband the guild permanently
    ViewAuditLog   = 1 << 7  // 128 - View the guild audit log
};
ENUM_CLASS_FLAGS(EFWGuildPermission);
```

**Permission Bitmask Examples:**

| Role | Bitmask | Value | Permissions |
|---|---|---|---|
| Guild Leader | `0xFF` | 255 | All permissions |
| Officer | `Invite | Kick | ViewAuditLog` | 131 | Invite, kick, view audit log |
| Veteran | `Invite` | 1 | Invite only |
| Member | `None` | 0 | No special permissions |

!!! tip "Bitmask Operations"
    ```cpp
    // Combine permissions
    uint8 OfficerPerms = static_cast<uint8>(EFWGuildPermission::Invite)
                       | static_cast<uint8>(EFWGuildPermission::Kick)
                       | static_cast<uint8>(EFWGuildPermission::ViewAuditLog);

    // Check permission
    bool bCanKick = (RankPermissions & static_cast<uint8>(EFWGuildPermission::Kick)) != 0;
    ```

---

### EFWGuildUpdateType

Describes the type of guild state change in update events.

```cpp
UENUM(BlueprintType)
enum class EFWGuildUpdateType : uint8
{
    MemberJoined,
    MemberLeft,
    MemberKicked,
    MemberPromoted,
    MemberDemoted,
    GuildInfoEdited,
    RanksEdited,
    GuildDisbanded,
    InvitationSent,
    InvitationAccepted,
    InvitationDeclined,
    InvitationExpired
};
```

---

### EFWGuildOperationResult

Result codes returned by guild operations via delegates.

```cpp
UENUM(BlueprintType)
enum class EFWGuildOperationResult : uint8
{
    Success,
    AlreadyInGuild,
    NotInGuild,
    InsufficientPermission,
    GuildNotFound,
    PlayerNotFound,
    PlayerAlreadyInGuild,
    PlayerAlreadyInvited,
    GuildFull,
    InvalidGuildName,
    GuildNameTaken,
    RankNotFound,
    CannotModifyHigherRank,
    CannotModifySelf,
    NetworkError,
    ServerError,
    Unknown
};
```

---

## Structs

### FFWGuildRank

Defines a single rank in the guild hierarchy.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildRank
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Guild")
    FString RankId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Guild")
    FString Name;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Guild")
    int32 Priority;  // 0 = highest (leader)

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Guild",
        Meta = (Bitmask, BitmaskEnum = "/Script/FWGuildSystem.EFWGuildPermission"))
    uint8 Permissions;
};
```

| Field | Type | Description |
|---|---|---|
| `RankId` | `FString` | Unique identifier for the rank. |
| `Name` | `FString` | Display name (e.g., "Guild Leader", "Officer", "Member"). |
| `Priority` | `int32` | Rank ordering. `0` is the highest rank (guild leader). Higher values are lower ranks. |
| `Permissions` | `uint8` | Bitmask of `EFWGuildPermission` flags. |

---

### FFWGuildMember

Represents a single guild member.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildMember
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString PlayerId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString DisplayName;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FFWGuildRank Rank;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    bool bIsOnline;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FDateTime JoinDate;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FDateTime LastSeen;
};
```

| Field | Type | Description |
|---|---|---|
| `PlayerId` | `FString` | Unique player identifier. |
| `DisplayName` | `FString` | Player's display name. |
| `Rank` | `FFWGuildRank` | The member's current rank. |
| `bIsOnline` | `bool` | Whether the player is currently online. |
| `JoinDate` | `FDateTime` | When the player joined the guild. |
| `LastSeen` | `FDateTime` | Last time the player was seen online. |

---

### FFWGuildInvitation

Represents a pending guild invitation.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildInvitation
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString InvitationId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString GuildId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString GuildName;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString InvitedByPlayerId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString InvitedByDisplayName;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FDateTime SentAt;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FDateTime ExpiresAt;
};
```

| Field | Type | Description |
|---|---|---|
| `InvitationId` | `FString` | Unique invitation identifier. |
| `GuildId` | `FString` | ID of the guild that sent the invitation. |
| `GuildName` | `FString` | Display name of the inviting guild. |
| `InvitedByPlayerId` | `FString` | Player ID of the member who sent the invitation. |
| `InvitedByDisplayName` | `FString` | Display name of the inviting member. |
| `SentAt` | `FDateTime` | When the invitation was sent. |
| `ExpiresAt` | `FDateTime` | When the invitation expires. |

---

### FFWGuildInfo

Summary information about a guild, used in search results and lightweight displays.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildInfo
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString GuildId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString Name;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString Description;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    int32 MemberCount;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    int32 MaxMembers;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString LeaderDisplayName;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FDateTime CreatedAt;
};
```

---

### FFWGuildDetails

Complete guild data including rank hierarchy and full member list. Extends the information in `FFWGuildInfo`.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildDetails
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FFWGuildInfo Info;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    TArray<FFWGuildRank> Ranks;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    TArray<FFWGuildMember> Members;
};
```

| Field | Type | Description |
|---|---|---|
| `Info` | `FFWGuildInfo` | Basic guild information. |
| `Ranks` | `TArray<FFWGuildRank>` | Ordered rank hierarchy (index 0 = leader). |
| `Members` | `TArray<FFWGuildMember>` | Complete member list. |

---

### FFWGuildUpdateEvent

Payload for guild state change notifications pushed from the server.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildUpdateEvent
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    EFWGuildUpdateType UpdateType;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString ActorPlayerId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString TargetPlayerId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString GuildId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FDateTime Timestamp;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString AdditionalData;  // JSON string for type-specific payload
};
```

---

### FFWGuildSearchResult

Paginated search results returned by `SearchGuilds`.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildSearchResult
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    TArray<FFWGuildInfo> Guilds;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    int32 TotalResults;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    int32 CurrentPage;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    int32 TotalPages;
};
```

---

### FFWGuildAuditLogEntry

A single audit log entry recording a guild operation.

```cpp
USTRUCT(BlueprintType)
struct FFWGuildAuditLogEntry
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString EntryId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    EFWGuildUpdateType ActionType;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString ActorPlayerId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString ActorDisplayName;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString TargetPlayerId;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString TargetDisplayName;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FDateTime Timestamp;

    UPROPERTY(BlueprintReadOnly, Category = "Guild")
    FString Description;
};
```

---

## Utility Class

### FFWGuildTypeUtils

Static helper functions for working with guild types.

```cpp
struct FWGUILDSYSTEM_API FFWGuildTypeUtils
{
    /** Converts a permission bitmask to a human-readable string list. */
    static FString PermissionsToString(uint8 Permissions);

    /** Parses a JSON string into an FFWGuildDetails struct. */
    static bool ParseGuildDetails(const FString& JsonString, FFWGuildDetails& OutDetails);

    /** Serializes an FFWGuildDetails struct to JSON. */
    static FString SerializeGuildDetails(const FFWGuildDetails& Details);

    /** Returns the default rank hierarchy for a newly created guild. */
    static TArray<FFWGuildRank> GetDefaultRanks();

    /** Checks if SourceRank can perform an action on TargetRank based on priority. */
    static bool CanActOnRank(const FFWGuildRank& SourceRank, const FFWGuildRank& TargetRank);

    /** Converts EFWGuildOperationResult to a user-facing error message. */
    static FText GetResultMessage(EFWGuildOperationResult Result);

    /** Converts EFWGuildUpdateType to a display string. */
    static FString UpdateTypeToString(EFWGuildUpdateType UpdateType);
};
```

!!! example "Default Rank Hierarchy"
    `GetDefaultRanks()` returns the following four ranks:

    | Priority | Name | Permissions |
    |---|---|---|
    | 0 | Guild Leader | All (255) |
    | 1 | Officer | Invite, Kick, Promote, Demote, ViewAuditLog (143) |
    | 2 | Veteran | Invite (1) |
    | 3 | Member | None (0) |
