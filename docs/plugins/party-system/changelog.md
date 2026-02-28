---
title: Changelog - FWPartySystem
description: Version history and release notes for the FWPartySystem plugin.
---

# Changelog

All notable changes to FWPartySystem are documented on this page.

---

## Version 2.0

**Release type:** Major | **Engine:** UE 5.3+

### Added

- **Beacon-based architecture.** Complete rewrite from GameState replication to Online Beacon framework (`AFWPartyBeaconHost`, `AFWPartyBeaconClient`, `AFWPartyHostObject`). Party state is now fully server-authoritative and decoupled from the game world.
- **Level transfer persistence.** Party state survives both seamless travel and hard map loads. Beacon connections persist independently of game world teardown.
- **Join codes.** 6-character alphanumeric codes generated server-side for direct party join without friend lists. Ambiguous characters (`0`, `O`, `1`, `I`, `L`) excluded from the character set.
- **Invitation system.** Full invitation workflow with `InvitePlayer`, `AcceptInvitation`, `DeclineInvitation`, and server-managed expiration timers.
- **Privacy modes.** `EFWPartyPrivacy::Open` and `EFWPartyPrivacy::InviteOnly` control whether join-by-code is available.
- **Kick system.** Leader can remove members via `KickMember` with automatic state cleanup and event broadcast.
- **Leader migration.** Automatic leadership transfer to the longest-tenured online member when the leader disconnects or leaves.
- **Disconnect grace period.** Configurable window (default 60s) during which disconnected members can rejoin without losing their party slot.
- **FWChatSystem integration.** Optional automatic party chat channel creation and teardown when FWChatSystem is present.
- **`OnMemberKicked` delegate.** Distinct from `OnMemberLeft` to allow UI differentiation between voluntary leave and kick.
- **`OnLocalPlayerJoinedParty` / `OnLocalPlayerLeftParty` delegates.** Separate local-only events for the player's own party transitions.
- **`OnInvitationExpired` delegate.** Server-driven expiration notification.
- **`FFWPartySettings` struct.** Centralized configuration for party defaults.

### Changed

- **`UFWPartyManagerComponent`** is now a pure client-side interface. All state mutations route through the beacon.
- **Member tracking** uses `FUniqueNetIdRepl` instead of `APlayerState` references, enabling cross-level identity.
- **Party state** is no longer replicated via `FFastArraySerializer`. The beacon pushes complete `FFWPartyInfo` snapshots on change.
- **`OnPartyUpdated`** now fires with the complete `FFWPartyInfo` instead of a delta.

### Removed

- **`UFWPartyStateComponent`** -- redundant with beacon-cached state in the manager component.
- **`FFastArraySerializer`-based replication** -- replaced by beacon RPC state pushes.
- **Direct `APlayerState` references** in party member info -- replaced by `FUniqueNetIdRepl`.

### Migration Guide

!!! warning "Breaking Changes"
    Version 2.0 is a complete architecture rewrite. There is no automated migration path from 1.x.

**Key migration steps:**

1. **Replace `UFWPartyStateComponent`** with `UFWPartyManagerComponent` only. The manager now handles both operations and local state caching.
2. **Spawn `AFWPartyBeaconHost`** in your GameMode's `InitGame`. See [Tutorial](tutorial.md) Part 1, Step 2.
3. **Update member references.** Replace any `APlayerState*` member lookups with `FUniqueNetIdRepl`-based lookups via `FFWPartyMemberInfo::UniqueNetId`.
4. **Update event handlers.** `OnPartyUpdated` now passes `FFWPartyInfo` instead of individual changed fields. Bind new delegates (`OnMemberKicked`, `OnLocalPlayerJoinedParty`, etc.) as needed.
5. **Configure beacon port.** Add `BeaconPort` to `DefaultEngine.ini` and ensure firewall rules allow the port.

---

## Version 1.0

**Release type:** Initial | **Engine:** UE 5.2+

### Added

- Initial release of FWPartySystem.
- `UFWPartyManagerComponent` for party creation and management.
- `UFWPartyStateComponent` for replicated party state via `FFastArraySerializer`.
- `EFWPartyRole` (Leader, Member).
- `FFWPartyMemberInfo` with `APlayerState` references.
- `FFWPartyInfo` with member list and party ID.
- `OnPartyCreated`, `OnPartyDisbanded`, `OnPartyUpdated`, `OnMemberJoined`, `OnMemberLeft`, `OnLeaderChanged` delegates.
- Basic party creation and leave functionality.
- Automatic leader migration on disconnect.
