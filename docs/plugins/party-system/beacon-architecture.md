---
title: Beacon Architecture - FWPartySystem
description: Server-side beacon host, client, and host object internals for the FWPartySystem plugin.
---

# Beacon Architecture

FWPartySystem uses Unreal Engine's Online Beacon framework to maintain server-authoritative party state. This page explains the three beacon classes, their responsibilities, and how they interact during party operations.

---

## Overview

The beacon architecture separates party logic into three classes:

| Class | Role | Runs On |
|---|---|---|
| `AFWPartyBeaconHost` | Authoritative party state, request validation | Server / Dedicated Server |
| `AFWPartyBeaconClient` | Request dispatch, response handling | Client |
| `AFWPartyHostObject` | Per-party state container, member tracking | Server (owned by beacon host) |

```
Client A                    Server                      Client B
+-----------------------+   +------------------------+  +-----------------------+
| AFWPartyBeaconClient  |-->| AFWPartyBeaconHost     |<-| AFWPartyBeaconClient  |
|   SendCreateRequest() |   |   HandleCreateRequest()|  |   SendJoinRequest()   |
|   SendJoinRequest()   |   |   HandleJoinRequest()  |  |   SendLeaveRequest()  |
|   SendLeaveRequest()  |   |   HandleLeaveRequest() |  |                       |
|   SendKickRequest()   |   |   HandleKickRequest()  |  |                       |
|   SendInviteRequest() |   |   HandleInviteRequest()|  |                       |
+-----------------------+   +---+--------------------+  +-----------------------+
                                |
                            +---v--------------------+
                            | AFWPartyHostObject     |
                            |   PartyInfo            |
                            |   Members[]            |
                            |   PendingInvitations[] |
                            +------------------------+
```

---

## AFWPartyBeaconHost

**Parent:** `AOnlineBeaconHost`

The server-side beacon that owns all party state and processes client requests. There is one beacon host per game server (or dedicated beacon server).

### Responsibilities

- Validate all incoming party operation requests
- Create and destroy `AFWPartyHostObject` instances for each party
- Generate unique party IDs and join codes
- Enforce party capacity limits and privacy settings
- Broadcast state changes to all connected beacon clients
- Handle player disconnection and leader migration

### Key Functions

```cpp
/** Called when a client requests party creation. */
void HandleCreatePartyRequest(AFWPartyBeaconClient* Client);

/** Called when a client requests to join via code. */
void HandleJoinByCodeRequest(AFWPartyBeaconClient* Client, const FString& JoinCode);

/** Called when a client requests to leave. */
void HandleLeaveRequest(AFWPartyBeaconClient* Client);

/** Called when a client requests to kick a member. Validates leader authority. */
void HandleKickRequest(AFWPartyBeaconClient* Client, const FUniqueNetIdRepl& TargetId);

/** Called when a client sends an invitation. */
void HandleInviteRequest(AFWPartyBeaconClient* Client, const FUniqueNetIdRepl& TargetId);
```

### Join Code Generation

Join codes are 6-character alphanumeric strings generated server-side. The beacon host maintains a set of active codes to guarantee uniqueness. Codes are released when a party is disbanded.

```cpp
// Internal code generation (simplified)
FString AFWPartyBeaconHost::GenerateJoinCode() const
{
    static const TCHAR Chars[] = TEXT("ABCDEFGHJKLMNPQRSTUVWXYZ23456789");
    FString Code;
    do {
        Code.Empty();
        for (int32 i = 0; i < 6; ++i)
        {
            Code.AppendChar(Chars[FMath::RandRange(0, 31)]);
        }
    } while (ActiveJoinCodes.Contains(Code));
    return Code;
}
```

!!! note "Character Set"
    The join code alphabet intentionally excludes `0`, `O`, `1`, `I`, and `L` to avoid visual ambiguity when players share codes verbally or via text.

---

## AFWPartyBeaconClient

**Parent:** `AOnlineBeaconClient`

The client-side counterpart that sends requests to the beacon host and receives state updates.

### Responsibilities

- Serialize and send party operation requests to the server
- Receive and deserialize state update RPCs from the server
- Forward received state to the local `UFWPartyManagerComponent` for event dispatch
- Handle connection loss and reconnection to the beacon host

### Key Functions

```cpp
/** Sends a create party request to the server. */
void SendCreatePartyRequest();

/** Sends a join-by-code request to the server. */
void SendJoinByCodeRequest(const FString& JoinCode);

/** Sends a leave request to the server. */
void SendLeaveRequest();

/** Sends a kick request to the server. */
void SendKickRequest(const FUniqueNetIdRepl& TargetId);

/** Sends an invitation request to the server. */
void SendInviteRequest(const FUniqueNetIdRepl& TargetId);
```

### Client RPCs (Server to Client)

```cpp
/** Called by the server when party state changes. */
UFUNCTION(Client, Reliable)
void ClientReceivePartyUpdate(const FFWPartyInfo& UpdatedPartyInfo);

/** Called by the server with the result of a join attempt. */
UFUNCTION(Client, Reliable)
void ClientReceiveJoinResult(EFWPartyJoinResult Result);

/** Called by the server when an invitation is received. */
UFUNCTION(Client, Reliable)
void ClientReceiveInvitation(const FFWPartyInvitation& Invitation);
```

---

## AFWPartyHostObject

**Parent:** `UObject`

A per-party state container created and owned by `AFWPartyBeaconHost`. Each active party has exactly one host object.

### State

```cpp
/** The complete party state. */
UPROPERTY()
FFWPartyInfo PartyInfo;

/** Pending invitations that have not yet been accepted or expired. */
UPROPERTY()
TArray<FFWPartyInvitation> PendingInvitations;

/** Mapping of member net IDs to their beacon client references. */
UPROPERTY()
TMap<FUniqueNetIdRepl, AFWPartyBeaconClient*> MemberClients;
```

### Member Operations

```cpp
/** Adds a member to the party. Returns false if party is full. */
bool AddMember(const FFWPartyMemberInfo& MemberInfo, AFWPartyBeaconClient* Client);

/** Removes a member by net ID. Returns the removed member info. */
FFWPartyMemberInfo RemoveMember(const FUniqueNetIdRepl& MemberId);

/** Promotes a member to leader. Demotes the current leader to member. */
void TransferLeadership(const FUniqueNetIdRepl& NewLeaderId);

/** Broadcasts the current PartyInfo to all connected member clients. */
void BroadcastStateUpdate();
```

---

## Level Transfer Persistence

Party state persists across seamless travel and hard map loads through the beacon architecture:

1. **Seamless Travel:** The beacon host runs independently of the game world. When the server performs seamless travel, the beacon host and all host objects survive because they are not part of the traveling world.

2. **Hard Map Load:** Before a hard map load, the beacon host serializes all `AFWPartyHostObject` state. After the new map loads, the beacon host restores party state and beacon clients reconnect automatically.

3. **Client Reconnection:** When a client's beacon connection drops during travel, the beacon client attempts reconnection using the stored party ID. The server matches the reconnecting player's net ID against the party member list and restores their session.

```
Seamless Travel Flow:

[Pre-Travel]
  BeaconHost: PartyHostObjects are NOT in the travel actor list
  BeaconHost: State persists in memory across the travel

[During Travel]
  BeaconClients: Connection maintained (beacon port is separate from game port)
  GameWorld: Actors destroyed and recreated

[Post-Travel]
  BeaconHost: Still active with all party state intact
  BeaconClients: Receive OnPartyUpdated to confirm state consistency
  PartyManagerComponents: Re-query cached state from beacon client
```

!!! tip "Testing Travel Persistence"
    To verify party persistence during development, create a party with multiple players, trigger a `ServerTravel` command, and confirm that `OnPartyUpdated` fires on all clients after the travel completes with the correct member list.

---

## Connection Lifecycle

### Initial Connection

```
1. Player spawns with UFWPartyManagerComponent
2. Component creates AFWPartyBeaconClient
3. BeaconClient connects to BeaconHost (uses configured beacon port)
4. BeaconHost registers the client
5. Component is ready for party operations
```

### Disconnection Handling

```
1. BeaconHost detects client disconnect (timeout or explicit)
2. If player was in a party:
   a. Mark member as offline (bIsOnline = false)
   b. Start grace period timer (configurable, default 60s)
   c. If player reconnects within grace period: restore session
   d. If grace period expires: remove from party, fire OnMemberLeft
3. If disconnected player was leader:
   a. Immediately transfer leadership to longest-tenured online member
   b. Fire OnLeaderChanged on all remaining members
```

---

## Next Steps

- See [Party Manager Component](party-manager-component.md) for the Blueprint-facing API that wraps beacon communication.
- See [Configuration](configuration.md) for beacon port, grace period, and other settings.
- See [Types Reference](types.md) for all struct and enum definitions used in beacon communication.
