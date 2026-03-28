---
title: API Reference - FWPartySystem
description: Complete function and delegate reference for all FWPartySystem classes.
---

# Party System API Reference

Complete reference for all public functions, properties, and delegates across FWPartySystem classes.

---

## UFWPartyManagerComponent

The primary interface for all party operations. Attach to your Player Controller.

### Party Creation and Joining

| Function | Parameters | Description |
|---|---|---|
| `CreateParty` | -- | Creates a new party with the caller as leader |
| `JoinPartyByCode` | `JoinCode: FString` | Attempts to join a party using a join code |
| `AcceptInvitation` | `InvitationId: FString` | Accepts a pending invitation |
| `DeclineInvitation` | `InvitationId: FString` | Declines a pending invitation |
| `LeaveParty` | -- | Removes the local player from their current party |

### Leader Actions

| Function | Parameters | Description |
|---|---|---|
| `InvitePlayer` | `PlayerId: FUniqueNetIdRepl` | Sends a party invitation to a player |
| `KickMember` | `MemberId: FUniqueNetIdRepl` | Removes a member from the party (leader only) |
| `TransferLeadership` | `NewLeaderId: FUniqueNetIdRepl` | Transfers party leadership to another member |

### Queries

| Function | Returns | Description |
|---|---|---|
| `IsInParty` | `bool` | Whether the local player is currently in a party |
| `IsPartyLeader` | `bool` | Whether the local player is the party leader |
| `GetCurrentPartyInfo` | `FFWPartyInfo` | Returns the cached party state |
| `GetPendingInvitations` | `TArray<FFWPartyInvitation>` | Returns all pending invitations |
| `GetLocalMemberInfo` | `FFWPartyMemberInfo` | Returns the local player's member info |
| `GetMemberCount` | `int32` | Returns the current number of party members |

### Chat Integration

| Function | Parameters | Description |
|---|---|---|
| `SendPartyMessage` | `Message: FString` | Sends a chat message to the party channel (requires FWChatSystem) |

### Events

| Delegate | Signature | Category |
|---|---|---|
| `OnPartyCreated` | `(FFWPartyInfo PartyInfo)` | Lifecycle |
| `OnPartyDisbanded` | `()` | Lifecycle |
| `OnPartyUpdated` | `(FFWPartyInfo PartyInfo)` | Lifecycle |
| `OnMemberJoined` | `(FFWPartyMemberInfo MemberInfo)` | Membership |
| `OnMemberLeft` | `(FFWPartyMemberInfo MemberInfo)` | Membership |
| `OnMemberKicked` | `(FFWPartyMemberInfo MemberInfo)` | Membership |
| `OnLeaderChanged` | `(FFWPartyMemberInfo NewLeader)` | Membership |
| `OnJoinPartyResult` | `(EFWPartyJoinResult Result)` | Local |
| `OnLocalPlayerJoinedParty` | `(FFWPartyInfo PartyInfo)` | Local |
| `OnLocalPlayerLeftParty` | `()` | Local |
| `OnInvitationReceived` | `(FFWPartyInvitation Invitation)` | Invitation |
| `OnInvitationExpired` | `(FString InvitationId)` | Invitation |

---

## UFWPartyManagerSubsystem

Game Instance subsystem that manages the beacon lifecycle and provides global party configuration.

### Beacon Management

| Function | Parameters | Description |
|---|---|---|
| `StartHostBeacon` | `ListenPort: int32` | Starts the party beacon host on the specified port |
| `StopHostBeacon` | -- | Shuts down the party beacon host |
| `ConnectToHost` | `HostAddress: FString` | Connects the local beacon client to a remote host |
| `Disconnect` | -- | Disconnects the beacon client |

### Queries

| Function | Returns | Description |
|---|---|---|
| `IsHostRunning` | `bool` | Whether the beacon host is currently active |
| `IsConnected` | `bool` | Whether the beacon client is connected to a host |
| `GetActivePartyCount` | `int32` | Number of active parties on this host |

### Configuration

| Property | Type | Description |
|---|---|---|
| `DefaultMaxPartySize` | `int32` | Default maximum members per party |
| `DefaultPrivacy` | `EFWPartyPrivacy` | Default privacy mode for new parties |
| `InvitationExpirySeconds` | `float` | How long invitations remain valid |
| `JoinCodeLength` | `int32` | Length of generated join codes |

---

## AFWPartyBeaconHost

Server-side beacon that holds authoritative state for all active parties. Managed by `UFWPartyManagerSubsystem`.

### Functions

| Function | Parameters | Description |
|---|---|---|
| `FindPartyByCode` | `JoinCode: FString` | Looks up a party by its join code |
| `FindPartyByMember` | `MemberId: FUniqueNetIdRepl` | Looks up which party a player belongs to |
| `GetActivePartyCount` | -- | Returns the number of active parties |

### Properties

| Property | Type | Description |
|---|---|---|
| `PartySettings` | `FFWPartySettings` | Server-wide party configuration defaults |

---

## AFWPartyBeaconClient

Client-side beacon for communicating party requests to the host. Managed internally by `UFWPartyManagerComponent`.

### Functions

| Function | Description |
|---|---|
| `SendCreatePartyRequest` | Sends a create request to the host |
| `SendJoinByCodeRequest` | Sends a join-by-code request to the host |
| `SendLeaveRequest` | Sends a leave request to the host |
| `SendKickRequest` | Sends a kick request to the host |
| `SendInviteRequest` | Sends an invite request to the host |
| `SendAcceptInvitationRequest` | Sends an accept-invitation request to the host |
| `SendDeclineInvitationRequest` | Sends a decline-invitation request to the host |

---

## AFWPartyHostObject

Per-party state object managed by the beacon host. One instance exists for each active party.

### Functions

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `AddMember` | `MemberInfo: FFWPartyMemberInfo` | `bool` | Adds a member. Returns false if the party is full |
| `RemoveMember` | `MemberId: FUniqueNetIdRepl` | `FFWPartyMemberInfo` | Removes and returns the member info |
| `TransferLeadership` | `NewLeaderId: FUniqueNetIdRepl` | `void` | Transfers leadership to a member |
| `IsFull` | -- | `bool` | Whether the party has reached max members |
| `FindMember` | `MemberId: FUniqueNetIdRepl` | `FFWPartyMemberInfo` | Finds a member by net ID |
| `GetLeader` | -- | `FFWPartyMemberInfo` | Returns the current party leader |

### Properties

| Property | Type | Description |
|---|---|---|
| `PartyInfo` | `FFWPartyInfo` | Complete party state |
| `PendingInvitations` | `TArray<FFWPartyInvitation>` | Active invitations for this party |
