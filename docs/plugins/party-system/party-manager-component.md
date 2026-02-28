---
title: Party Manager Component - FWPartySystem
description: Complete API reference for UFWPartyManagerComponent, the primary Blueprint-facing interface for party operations.
---

# UFWPartyManagerComponent

**Class:** `UFWPartyManagerComponent` | **Parent:** `UActorComponent` | **Specifier:** `BlueprintSpawnableComponent`

The central component for all party operations. Attach this to your Player Controller or Pawn to give players the ability to create, join, manage, and leave parties. All functions are exposed to Blueprints.

---

## Component Setup

```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Party")
TObjectPtr<UFWPartyManagerComponent> PartyManager;

// In constructor
PartyManager = CreateDefaultSubobject<UFWPartyManagerComponent>(TEXT("PartyManager"));
```

!!! info "Authority Model"
    All mutating functions (Create, Join, Leave, Kick, Invite) send requests to the server via `AFWPartyBeaconClient`. The component does not modify local state until the server confirms the operation and replicates the updated state back. Read-only queries operate on the local cached state.

---

## Blueprint Callable Functions

### Party Lifecycle

#### `CreateParty`

Creates a new party with the calling player as leader.

```cpp
UFUNCTION(BlueprintCallable, Category = "Party")
void CreateParty();
```

| Parameter | Type | Description |
|---|---|---|
| *(none)* | | Uses default party settings from configuration |

Fires `OnPartyCreated` on success with the new `FFWPartyInfo`.

---

#### `LeaveParty`

Removes the local player from their current party.

```cpp
UFUNCTION(BlueprintCallable, Category = "Party")
void LeaveParty();
```

If the leaving player is the party leader:

- If other members remain, leadership transfers to the member who joined earliest.
- If no members remain, the party is disbanded and `OnPartyDisbanded` fires.

Fires `OnLocalPlayerLeftParty` on the leaving player's component. Fires `OnMemberLeft` on all remaining members' components.

---

### Joining

#### `JoinPartyByCode`

Attempts to join an existing party using a join code.

```cpp
UFUNCTION(BlueprintCallable, Category = "Party")
void JoinPartyByCode(const FString& JoinCode);
```

| Parameter | Type | Description |
|---|---|---|
| `JoinCode` | `FString` | The alphanumeric join code shared by the party leader |

Fires `OnJoinPartyResult` with the outcome. On success, also fires `OnLocalPlayerJoinedParty` on the joining player and `OnMemberJoined` on all existing members.

!!! warning "Privacy Check"
    If the party's privacy is set to `EFWPartyPrivacy::InviteOnly`, join-by-code requests are rejected with `EFWPartyJoinResult::InviteOnly`.

---

### Invitations

#### `InvitePlayer`

Sends a party invitation to another player. Only callable by party members (leader restriction is configurable).

```cpp
UFUNCTION(BlueprintCallable, Category = "Party")
void InvitePlayer(const FUniqueNetIdRepl& PlayerId);
```

| Parameter | Type | Description |
|---|---|---|
| `PlayerId` | `FUniqueNetIdRepl` | The unique net ID of the player to invite |

Fires `OnInvitationReceived` on the target player's component.

---

#### `AcceptInvitation`

Accepts a pending party invitation.

```cpp
UFUNCTION(BlueprintCallable, Category = "Party")
void AcceptInvitation(const FString& InvitationId);
```

| Parameter | Type | Description |
|---|---|---|
| `InvitationId` | `FString` | The ID of the invitation to accept (from `FFWPartyInvitation`) |

Fires `OnJoinPartyResult` and `OnLocalPlayerJoinedParty` on success.

---

#### `DeclineInvitation`

Declines a pending party invitation.

```cpp
UFUNCTION(BlueprintCallable, Category = "Party")
void DeclineInvitation(const FString& InvitationId);
```

| Parameter | Type | Description |
|---|---|---|
| `InvitationId` | `FString` | The ID of the invitation to decline |

The invitation is removed from the local pending list. No event fires on the sender.

---

### Member Management

#### `KickMember`

Removes a member from the party. Only the party leader can kick members.

```cpp
UFUNCTION(BlueprintCallable, Category = "Party")
void KickMember(const FUniqueNetIdRepl& MemberId);
```

| Parameter | Type | Description |
|---|---|---|
| `MemberId` | `FUniqueNetIdRepl` | The unique net ID of the member to remove |

Fires `OnMemberKicked` on all remaining members' components. The kicked player receives `OnLocalPlayerLeftParty`.

---

## Events (Delegates)

All delegates are `BlueprintAssignable` and can be bound in both C++ and Blueprints.

### Party Lifecycle Events

| Delegate | Signature | Description |
|---|---|---|
| `OnPartyCreated` | `(const FFWPartyInfo& PartyInfo)` | Fired on the leader when a party is successfully created |
| `OnPartyDisbanded` | `()` | Fired on all members when the party is disbanded |
| `OnPartyUpdated` | `(const FFWPartyInfo& PartyInfo)` | Fired when any party property changes (privacy, name, max members) |

### Membership Events

| Delegate | Signature | Description |
|---|---|---|
| `OnMemberJoined` | `(const FFWPartyMemberInfo& MemberInfo)` | Fired on all members when a new player joins |
| `OnMemberLeft` | `(const FFWPartyMemberInfo& MemberInfo)` | Fired on all remaining members when someone leaves |
| `OnMemberKicked` | `(const FFWPartyMemberInfo& MemberInfo)` | Fired on all remaining members when someone is kicked |
| `OnLeaderChanged` | `(const FFWPartyMemberInfo& NewLeader)` | Fired on all members when leadership transfers |

### Local Player Events

| Delegate | Signature | Description |
|---|---|---|
| `OnLocalPlayerJoinedParty` | `(const FFWPartyInfo& PartyInfo)` | Fired on the local player when they successfully join a party |
| `OnLocalPlayerLeftParty` | `()` | Fired on the local player when they leave or are kicked |
| `OnJoinPartyResult` | `(EFWPartyJoinResult Result)` | Fired on the local player after a join attempt |

### Invitation Events

| Delegate | Signature | Description |
|---|---|---|
| `OnInvitationReceived` | `(const FFWPartyInvitation& Invitation)` | Fired when an invitation is received from another player |
| `OnInvitationExpired` | `(const FString& InvitationId)` | Fired when a pending invitation expires |

---

## Event Flow Diagram

```
Player A: CreateParty()
    |
    v
Server: AFWPartyBeaconHost validates and creates party state
    |
    v
Player A: OnPartyCreated(PartyInfo)
    |
Player B: JoinPartyByCode("ABC123")
    |
    v
Server: Validates code, checks capacity and privacy
    |
    +--[Success]---> Player B: OnJoinPartyResult(Success)
    |                Player B: OnLocalPlayerJoinedParty(PartyInfo)
    |                Player A: OnMemberJoined(PlayerB_Info)
    |
    +--[Failure]---> Player B: OnJoinPartyResult(PartyFull|InvalidCode|InviteOnly)
```

---

## Usage Patterns

### Checking Current Party State

```cpp
// The component caches the current party info locally
if (PartyManager->IsInParty())
{
    const FFWPartyInfo& Info = PartyManager->GetCurrentPartyInfo();
    UE_LOG(LogTemp, Log, TEXT("In party: %s (%d/%d members)"),
        *Info.PartyName, Info.Members.Num(), Info.MaxMembers);
}
```

### Iterating Party Members

```cpp
const FFWPartyInfo& Info = PartyManager->GetCurrentPartyInfo();
for (const FFWPartyMemberInfo& Member : Info.Members)
{
    UE_LOG(LogTemp, Log, TEXT("  %s [%s] - %s"),
        *Member.DisplayName,
        *UEnum::GetValueAsString(Member.Role),
        Member.bIsOnline ? TEXT("Online") : TEXT("Offline"));
}
```

---

## Next Steps

- See [Beacon Architecture](beacon-architecture.md) for how server-side state management works.
- See [Events and Delegates](events-delegates.md) for detailed event flow and binding patterns.
- See [Types Reference](types.md) for all struct and enum definitions.
