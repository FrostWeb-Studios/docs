---
title: Events and Delegates - FWPartySystem
description: All multicast delegates, event flow diagrams, and binding patterns for FWPartySystem.
---

# Events and Delegates

This page documents all events fired by FWPartySystem, their flow through the beacon architecture, and patterns for binding them in C++ and Blueprints.

---

## Delegate Summary

### Party Lifecycle

| Delegate | Fires On | Signature |
|---|---|---|
| `OnPartyCreated` | Leader only | `(const FFWPartyInfo& PartyInfo)` |
| `OnPartyDisbanded` | All members | `()` |
| `OnPartyUpdated` | All members | `(const FFWPartyInfo& PartyInfo)` |

### Membership

| Delegate | Fires On | Signature |
|---|---|---|
| `OnMemberJoined` | All existing members | `(const FFWPartyMemberInfo& MemberInfo)` |
| `OnMemberLeft` | All remaining members | `(const FFWPartyMemberInfo& MemberInfo)` |
| `OnMemberKicked` | All remaining members | `(const FFWPartyMemberInfo& MemberInfo)` |
| `OnLeaderChanged` | All members | `(const FFWPartyMemberInfo& NewLeader)` |

### Local Player

| Delegate | Fires On | Signature |
|---|---|---|
| `OnLocalPlayerJoinedParty` | Joining player | `(const FFWPartyInfo& PartyInfo)` |
| `OnLocalPlayerLeftParty` | Leaving/kicked player | `()` |
| `OnJoinPartyResult` | Joining player | `(EFWPartyJoinResult Result)` |

### Invitations

| Delegate | Fires On | Signature |
|---|---|---|
| `OnInvitationReceived` | Target player | `(const FFWPartyInvitation& Invitation)` |
| `OnInvitationExpired` | Target player | `(const FString& InvitationId)` |

---

## Binding Patterns

### C++ Dynamic Binding

```cpp
void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    // Bind lifecycle events
    PartyManager->OnPartyCreated.AddDynamic(this, &AMyPlayerController::HandlePartyCreated);
    PartyManager->OnPartyDisbanded.AddDynamic(this, &AMyPlayerController::HandlePartyDisbanded);
    PartyManager->OnPartyUpdated.AddDynamic(this, &AMyPlayerController::HandlePartyUpdated);

    // Bind membership events
    PartyManager->OnMemberJoined.AddDynamic(this, &AMyPlayerController::HandleMemberJoined);
    PartyManager->OnMemberLeft.AddDynamic(this, &AMyPlayerController::HandleMemberLeft);
    PartyManager->OnMemberKicked.AddDynamic(this, &AMyPlayerController::HandleMemberKicked);
    PartyManager->OnLeaderChanged.AddDynamic(this, &AMyPlayerController::HandleLeaderChanged);

    // Bind local player events
    PartyManager->OnJoinPartyResult.AddDynamic(this, &AMyPlayerController::HandleJoinResult);
    PartyManager->OnLocalPlayerJoinedParty.AddDynamic(this, &AMyPlayerController::HandleLocalJoined);
    PartyManager->OnLocalPlayerLeftParty.AddDynamic(this, &AMyPlayerController::HandleLocalLeft);

    // Bind invitation events
    PartyManager->OnInvitationReceived.AddDynamic(this, &AMyPlayerController::HandleInvitationReceived);
    PartyManager->OnInvitationExpired.AddDynamic(this, &AMyPlayerController::HandleInvitationExpired);
}
```

!!! warning "UFUNCTION Requirement"
    All dynamic delegate handler functions must be marked with `UFUNCTION()`. Forgetting this macro will compile but crash at runtime when the delegate attempts to fire.

### Blueprint Binding

In Blueprints, drag from the Party Manager Component reference and select **Assign** on the desired event. The editor will create an event node automatically.

Common pattern for a party HUD widget:

1. Get a reference to the owning Player Controller's Party Manager Component.
2. Bind **On Party Updated** to refresh the member roster.
3. Bind **On Invitation Received** to show an invitation popup.
4. Bind **On Local Player Left Party** to hide the party UI.

---

## Event Flow Diagrams

### Create Party Flow

```
Player A calls CreateParty()
    |
    v
BeaconClient -> SendCreatePartyRequest() -> [Network] -> BeaconHost
    |
    v
BeaconHost: Validate (not already in party)
    |
    v
BeaconHost: Create AFWPartyHostObject
    |
    v
BeaconHost: Generate JoinCode, assign PartyId
    |
    v
BeaconHost -> ClientReceivePartyUpdate() -> [Network] -> BeaconClient
    |
    v
PartyManagerComponent:
    +-- OnPartyCreated(PartyInfo)
    +-- OnLocalPlayerJoinedParty(PartyInfo)
```

### Join by Code Flow

```
Player B calls JoinPartyByCode("ABC123")
    |
    v
BeaconClient -> SendJoinByCodeRequest("ABC123") -> [Network] -> BeaconHost
    |
    v
BeaconHost: FindPartyByCode("ABC123")
    |
    +--[Not Found]-> ClientReceiveJoinResult(InvalidCode)
    |                    |
    |                    v
    |                Player B: OnJoinPartyResult(InvalidCode)
    |
    +--[Found, Full]-> ClientReceiveJoinResult(PartyFull)
    |
    +--[Found, InviteOnly]-> ClientReceiveJoinResult(InviteOnly)
    |
    +--[Found, OK]--> HostObject: AddMember(PlayerB_Info)
                         |
                         v
                      HostObject: BroadcastStateUpdate()
                         |
                         +---> Player B: OnJoinPartyResult(Success)
                         |               OnLocalPlayerJoinedParty(PartyInfo)
                         |
                         +---> Player A: OnMemberJoined(PlayerB_Info)
                                         OnPartyUpdated(PartyInfo)
```

### Invitation Flow

```
Player A calls InvitePlayer(PlayerB_NetId)
    |
    v
BeaconClient -> SendInviteRequest(PlayerB_NetId) -> [Network] -> BeaconHost
    |
    v
BeaconHost: Validate (A is in party, B is not, party not full)
    |
    v
BeaconHost: Create FFWPartyInvitation, start expiry timer
    |
    v
BeaconHost -> ClientReceiveInvitation(Invitation) -> [Network] -> Player B's BeaconClient
    |
    v
Player B: OnInvitationReceived(Invitation)
    |
    +--[Accept]-> Player B calls AcceptInvitation(InvitationId)
    |                |
    |                v
    |             (Same flow as JoinByCode, but bypasses privacy check)
    |
    +--[Decline]-> Player B calls DeclineInvitation(InvitationId)
    |                |
    |                v
    |             Invitation removed locally, no notification to sender
    |
    +--[Timeout]-> Expiry timer fires on BeaconHost
                     |
                     v
                  BeaconHost -> ClientReceiveInvitationExpired(InvitationId)
                     |
                     v
                  Player B: OnInvitationExpired(InvitationId)
```

### Leave / Kick Flow

```
Leave:
  Player calls LeaveParty()
      |
      v
  BeaconClient -> SendLeaveRequest() -> BeaconHost
      |
      v
  BeaconHost: RemoveMember()
      |
      +--[Was Leader, others remain]-> TransferLeadership()
      |                                   |
      |                                   v
      |                                All members: OnLeaderChanged(NewLeader)
      |
      +--[No members remain]-> Disband party
      |                           |
      |                           v
      |                        (All clients already left)
      |
      v
  Leaving player: OnLocalPlayerLeftParty()
  Remaining members: OnMemberLeft(LeavingMemberInfo)
                     OnPartyUpdated(PartyInfo)

Kick:
  Leader calls KickMember(TargetId)
      |
      v
  (Same as Leave, but kicked player receives OnLocalPlayerLeftParty
   and remaining members receive OnMemberKicked instead of OnMemberLeft)
```

---

## Event Ordering Guarantees

The following ordering guarantees are maintained:

1. `OnJoinPartyResult(Success)` always fires before `OnLocalPlayerJoinedParty`.
2. `OnMemberJoined` always fires on existing members before `OnPartyUpdated`.
3. `OnLeaderChanged` always fires before `OnMemberLeft` when a leader leaves.
4. `OnLocalPlayerLeftParty` always fires after all server-side cleanup is complete.
5. `OnPartyDisbanded` fires only once per party lifecycle, and only after all members have been notified.

!!! note "No Guaranteed Cross-Client Ordering"
    Events are delivered reliably but the order in which different clients receive the same event is not guaranteed. Client A may see `OnMemberJoined` before Client B, depending on network conditions.

---

## Next Steps

- See [Party Manager Component](party-manager-component.md) for the full function API.
- See [Types Reference](types.md) for delegate type declarations.
- See [Tutorial](tutorial.md) for practical examples of event handling in a complete implementation.
