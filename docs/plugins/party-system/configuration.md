---
title: Configuration - FWPartySystem
description: Plugin settings, party defaults, beacon configuration, and project-level configuration for FWPartySystem.
---

# Configuration

This page covers all configurable settings for FWPartySystem, from party defaults to beacon networking parameters.

---

## Party Settings

Party behavior is configured through `FFWPartySettings`, which can be set in C++, Blueprints, or via `DefaultGame.ini`.

### FFWPartySettings Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `DefaultMaxMembers` | `int32` | `4` | Maximum members allowed per party |
| `DefaultPrivacy` | `EFWPartyPrivacy` | `Open` | Default privacy mode for new parties |
| `InvitationExpirySeconds` | `float` | `120.0` | How long invitations remain valid |
| `DisconnectGracePeriod` | `float` | `60.0` | Seconds before disconnected members are removed |
| `bAllowMemberInvites` | `bool` | `true` | Whether non-leader members can send invitations |

### Setting via DefaultGame.ini

```ini
[/Script/FWPartySystem.FWPartySettings]
DefaultMaxMembers=6
DefaultPrivacy=InviteOnly
InvitationExpirySeconds=180.0
DisconnectGracePeriod=90.0
bAllowMemberInvites=false
```

### Setting at Runtime

=== "C++"

    ```cpp
    FFWPartySettings Settings;
    Settings.DefaultMaxMembers = 6;
    Settings.DefaultPrivacy = EFWPartyPrivacy::InviteOnly;
    Settings.InvitationExpirySeconds = 180.0f;
    Settings.DisconnectGracePeriod = 90.0f;
    Settings.bAllowMemberInvites = false;

    // Apply to the beacon host (server-side only)
    BeaconHost->SetPartySettings(Settings);
    ```

=== "Blueprint"

    Set the **Party Settings** struct on the Party Beacon Host actor in your GameMode or server initialization logic.

---

## Beacon Configuration

### Beacon Port

The beacon host listens on a separate port from the game server. Configure this in `DefaultEngine.ini`:

```ini
[/Script/FWPartySystem.FWPartyBeaconHost]
BeaconPort=15000
```

!!! warning "Firewall"
    Ensure the beacon port is open in your firewall and any cloud provider security groups. The beacon uses a separate connection from the game port and will fail silently if blocked.

### Connection Settings

| Property | Type | Default | Description |
|---|---|---|---|
| `BeaconPort` | `int32` | `15000` | Port the beacon host listens on |
| `ConnectionTimeout` | `float` | `30.0` | Seconds before a beacon connection attempt times out |
| `HeartbeatInterval` | `float` | `10.0` | Seconds between client heartbeat pings |
| `MaxReconnectAttempts` | `int32` | `3` | Maximum reconnection attempts after connection loss |
| `ReconnectDelay` | `float` | `5.0` | Seconds between reconnection attempts |

```ini
[/Script/FWPartySystem.FWPartyBeaconHost]
BeaconPort=15000
ConnectionTimeout=30.0
HeartbeatInterval=10.0

[/Script/FWPartySystem.FWPartyBeaconClient]
MaxReconnectAttempts=3
ReconnectDelay=5.0
```

---

## Party Size Limits

The `DefaultMaxMembers` setting controls the global default, but individual parties can override this at creation time:

```cpp
// Create a party with a custom max size
FFWPartyInfo CustomParty;
CustomParty.MaxMembers = 8;
PartyManager->CreatePartyWithSettings(CustomParty);
```

!!! note "Hard Limit"
    The absolute maximum party size is capped at `16` regardless of configuration. This limit is enforced server-side in `AFWPartyBeaconHost` and cannot be overridden.

---

## Join Code Settings

| Property | Type | Default | Description |
|---|---|---|---|
| `JoinCodeLength` | `int32` | `6` | Number of characters in generated join codes |
| `JoinCodeCharSet` | `FString` | `ABCDEFGHJKLMNPQRSTUVWXYZ23456789` | Characters used in join codes |

```ini
[/Script/FWPartySystem.FWPartyBeaconHost]
JoinCodeLength=6
```

The character set intentionally excludes `0`, `O`, `1`, `I`, and `L` to prevent visual ambiguity.

---

## Chat Integration Settings

When FWChatSystem is present, party chat channels are created automatically. Configure this behavior:

| Property | Type | Default | Description |
|---|---|---|---|
| `bAutoCreateChatChannel` | `bool` | `true` | Whether to create a chat channel when a party forms |
| `ChatChannelPrefix` | `FString` | `party_` | Prefix for party chat channel names |

```ini
[/Script/FWPartySystem.FWPartyManagerComponent]
bAutoCreateChatChannel=true
ChatChannelPrefix=party_
```

!!! info "Channel Lifecycle"
    Party chat channels are created when the party is formed and destroyed when the party disbands. Members are automatically added to and removed from the channel as they join and leave the party.

---

## Logging

FWPartySystem uses the `LogFWParty` log category. To enable verbose logging for debugging:

```ini
[Core.Log]
LogFWParty=Verbose
```

Log levels:

| Level | What is logged |
|---|---|
| `Error` | Failed operations, connection failures, validation errors |
| `Warning` | Rejected requests (full party, invalid code), reconnection attempts |
| `Log` | Party lifecycle events (create, join, leave, disband) |
| `Verbose` | Beacon message traffic, heartbeats, state replication details |

---

## Development and Testing

### OnlineSubsystemNull Configuration

For local development without platform services:

```ini
[OnlineSubsystem]
DefaultPlatformService=Null

[OnlineSubsystemNull]
bEnabled=true
```

!!! tip "Multiple PIE Clients"
    When testing with multiple Play-In-Editor clients, each PIE instance gets a unique `OnlineSubsystemNull` identity. This allows you to test party creation, joining, and member management without needing Steam or EOS running.

### Listen Server vs Dedicated Server

| Mode | Beacon Host Location | Notes |
|---|---|---|
| Dedicated Server | Spawned by GameMode on server start | Beacon host and game server share the process |
| Listen Server | Spawned by GameMode on the host player's machine | Host player is both beacon host and beacon client |
| Standalone Beacon | Separate process running only the beacon host | Used for large-scale deployments where party state is isolated |

---

## Next Steps

- See [Beacon Architecture](beacon-architecture.md) for details on how configuration affects beacon behavior.
- See [Tutorial](tutorial.md) for a complete implementation walkthrough.
