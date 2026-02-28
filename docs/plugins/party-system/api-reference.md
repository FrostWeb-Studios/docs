---
title: API Reference - FWPartySystem
description: Complete function and delegate reference for all FWPartySystem classes.
---

# API Reference

Complete reference for all public functions, properties, and delegates across FWPartySystem classes.

---

## UFWPartyManagerComponent

**Parent:** `UActorComponent` | **Specifier:** `BlueprintSpawnableComponent`

### Functions

#### BlueprintCallable

| Function | Parameters | Return | Description |
|---|---|---|---|
| `CreateParty` | *(none)* | `void` | Creates a new party with the caller as leader |
| `JoinPartyByCode` | `const FString& JoinCode` | `void` | Attempts to join a party using a join code |
| `AcceptInvitation` | `const FString& InvitationId` | `void` | Accepts a pending invitation |
| `DeclineInvitation` | `const FString& InvitationId` | `void` | Declines a pending invitation |
| `LeaveParty` | *(none)* | `void` | Removes the local player from their current party |
| `InvitePlayer` | `const FUniqueNetIdRepl& PlayerId` | `void` | Sends a party invitation to a player |
| `KickMember` | `const FUniqueNetIdRepl& MemberId` | `void` | Removes a member (leader only) |

#### BlueprintPure

| Function | Parameters | Return | Description |
|---|---|---|---|
| `IsInParty` | *(none)* | `bool` | Whether the local player is currently in a party |
| `IsPartyLeader` | *(none)* | `bool` | Whether the local player is the party leader |
| `GetCurrentPartyInfo` | *(none)* | `const FFWPartyInfo&` | Returns the cached party state |
| `GetPendingInvitations` | *(none)* | `const TArray<FFWPartyInvitation>&` | Returns all pending invitations |
| `GetLocalMemberInfo` | *(none)* | `const FFWPartyMemberInfo&` | Returns the local player's member info |
| `GetMemberCount` | *(none)* | `int32` | Returns the current number of party members |

### Delegates

| Delegate | Signature | Category |
|---|---|---|
| `OnPartyCreated` | `(const FFWPartyInfo& PartyInfo)` | Lifecycle |
| `OnPartyDisbanded` | `()` | Lifecycle |
| `OnPartyUpdated` | `(const FFWPartyInfo& PartyInfo)` | Lifecycle |
| `OnMemberJoined` | `(const FFWPartyMemberInfo& MemberInfo)` | Membership |
| `OnMemberLeft` | `(const FFWPartyMemberInfo& MemberInfo)` | Membership |
| `OnMemberKicked` | `(const FFWPartyMemberInfo& MemberInfo)` | Membership |
| `OnLeaderChanged` | `(const FFWPartyMemberInfo& NewLeader)` | Membership |
| `OnJoinPartyResult` | `(EFWPartyJoinResult Result)` | Local |
| `OnLocalPlayerJoinedParty` | `(const FFWPartyInfo& PartyInfo)` | Local |
| `OnLocalPlayerLeftParty` | `()` | Local |
| `OnInvitationReceived` | `(const FFWPartyInvitation& Invitation)` | Invitation |
| `OnInvitationExpired` | `(const FString& InvitationId)` | Invitation |

---

## AFWPartyBeaconHost

**Parent:** `AOnlineBeaconHost`

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `HandleCreatePartyRequest` | `AFWPartyBeaconClient* Client` | `void` | Processes a party creation request |
| `HandleJoinByCodeRequest` | `AFWPartyBeaconClient* Client, const FString& JoinCode` | `void` | Processes a join-by-code request |
| `HandleLeaveRequest` | `AFWPartyBeaconClient* Client` | `void` | Processes a leave request |
| `HandleKickRequest` | `AFWPartyBeaconClient* Client, const FUniqueNetIdRepl& TargetId` | `void` | Processes a kick request (validates leader authority) |
| `HandleInviteRequest` | `AFWPartyBeaconClient* Client, const FUniqueNetIdRepl& TargetId` | `void` | Processes an invitation request |
| `HandleAcceptInvitationRequest` | `AFWPartyBeaconClient* Client, const FString& InvitationId` | `void` | Processes an invitation acceptance |
| `HandleDeclineInvitationRequest` | `AFWPartyBeaconClient* Client, const FString& InvitationId` | `void` | Processes an invitation decline |
| `FindPartyByCode` | `const FString& JoinCode` | `AFWPartyHostObject*` | Looks up a party by its join code |
| `FindPartyByMember` | `const FUniqueNetIdRepl& MemberId` | `AFWPartyHostObject*` | Looks up which party a player belongs to |
| `GenerateJoinCode` | *(none)* | `FString` | Generates a unique 6-character join code |

### Properties

| Property | Type | Description |
|---|---|---|
| `ActiveParties` | `TArray<AFWPartyHostObject*>` | All currently active party host objects |
| `ActiveJoinCodes` | `TSet<FString>` | Set of join codes in use (for uniqueness) |
| `PartySettings` | `FFWPartySettings` | Server-wide party configuration defaults |

---

## AFWPartyBeaconClient

**Parent:** `AOnlineBeaconClient`

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `SendCreatePartyRequest` | *(none)* | `void` | Sends a create request to the server |
| `SendJoinByCodeRequest` | `const FString& JoinCode` | `void` | Sends a join-by-code request to the server |
| `SendLeaveRequest` | *(none)* | `void` | Sends a leave request to the server |
| `SendKickRequest` | `const FUniqueNetIdRepl& TargetId` | `void` | Sends a kick request to the server |
| `SendInviteRequest` | `const FUniqueNetIdRepl& TargetId` | `void` | Sends an invite request to the server |
| `SendAcceptInvitationRequest` | `const FString& InvitationId` | `void` | Sends an accept request to the server |
| `SendDeclineInvitationRequest` | `const FString& InvitationId` | `void` | Sends a decline request to the server |

### Client RPCs

| RPC | Parameters | Description |
|---|---|---|
| `ClientReceivePartyUpdate` | `const FFWPartyInfo& UpdatedPartyInfo` | Server pushes updated party state |
| `ClientReceiveJoinResult` | `EFWPartyJoinResult Result` | Server pushes join attempt result |
| `ClientReceiveInvitation` | `const FFWPartyInvitation& Invitation` | Server pushes incoming invitation |
| `ClientReceiveInvitationExpired` | `const FString& InvitationId` | Server pushes expiration notice |
| `ClientReceiveKicked` | `()` | Server notifies the client they were kicked |

---

## AFWPartyHostObject

**Parent:** `UObject`

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `AddMember` | `const FFWPartyMemberInfo& MemberInfo, AFWPartyBeaconClient* Client` | `bool` | Adds a member. Returns false if full. |
| `RemoveMember` | `const FUniqueNetIdRepl& MemberId` | `FFWPartyMemberInfo` | Removes and returns the member info |
| `TransferLeadership` | `const FUniqueNetIdRepl& NewLeaderId` | `void` | Transfers leadership to a member |
| `BroadcastStateUpdate` | *(none)* | `void` | Sends current state to all connected clients |
| `AddInvitation` | `const FFWPartyInvitation& Invitation` | `void` | Adds a pending invitation |
| `RemoveInvitation` | `const FString& InvitationId` | `bool` | Removes an invitation by ID |
| `IsFull` | *(none)* | `bool` | Whether the party has reached max members |
| `FindMember` | `const FUniqueNetIdRepl& MemberId` | `FFWPartyMemberInfo*` | Finds a member by net ID |
| `GetLeader` | *(none)* | `FFWPartyMemberInfo*` | Returns the current party leader |

### Properties

| Property | Type | Description |
|---|---|---|
| `PartyInfo` | `FFWPartyInfo` | Complete party state |
| `PendingInvitations` | `TArray<FFWPartyInvitation>` | Active invitations |
| `MemberClients` | `TMap<FUniqueNetIdRepl, AFWPartyBeaconClient*>` | Member-to-client mapping |

---

## Static Utility Functions

### FFWPartyTypeUtils

| Function | Parameters | Return | Description |
|---|---|---|---|
| `IsValidJoinCode` | `const FString& Code` | `bool` | Validates code format (6 chars, alphanumeric) |
| `GetRoleDisplayName` | `EFWPartyRole Role` | `FText` | Returns localized role name |
| `GetJoinResultMessage` | `EFWPartyJoinResult Result` | `FText` | Returns localized result message |

---

## Next Steps

- See [Types Reference](types.md) for detailed struct and enum documentation.
- See [Events and Delegates](events-delegates.md) for event flow and binding patterns.
- See [Configuration](configuration.md) for tuning party behavior.
