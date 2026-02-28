---
title: Changelog - Faction System
---

# Changelog

All notable changes to FWFactionSystem are documented here. This project follows [Semantic Versioning](https://semver.org/).

---

## [1.0.0] - 2025-01-01

### Added

- **UFWFactionComponent** -- Per-actor component with faction identity, personal reputation, and replication.
    - `SetFaction`, `ModifyPersonalReputation`, `SetPersonalReputation`, `ApplyFactionPlayerData` (BlueprintCallable, server-authoritative).
    - `GetCurrentFactionId`, `GetGenericTeamId`, `GetPersonalReputation`, `GetFactionPlayerData` (BlueprintPure).
    - `SolveTeamAttitude` static function for AI integration.
    - `OnFactionChanged`, `OnReputationChanged`, `OnAttitudeChanged` events.
    - `Server_SetFaction`, `Server_ModifyPersonalReputation` RPCs.
    - Replication: `CurrentFactionId` to all clients, `PersonalReputations` to owning client.

- **UFWFactionSubsystem** -- World subsystem for global faction state.
    - Faction definition caching via AssetManager.
    - Global reputation matrix with get/set/modify operations.
    - Attitude resolution with personal reputation priority.
    - Component registry for per-faction queries.
    - Pluggable persistence via `UFWFactionPersistenceHandler`.
    - `OnFactionWarDeclared`, `OnFactionAllianceFormed`, `OnGlobalAttitudeChanged` events.
    - `OnFactionDataLoaded`, `OnFactionDataSaved` persistence events.
    - Debug HUD and console commands (non-shipping builds).

- **UFWFactionDefinition** -- Primary DataAsset for faction configuration.
    - Identity: FactionId, DisplayName, Description, Icon, Color, TeamId.
    - Classification: FactionTags, bPlayerJoinable, bHidden.
    - Custom reputation thresholds per faction.
    - Default relationships array.
    - Reputation propagation rules.

- **UFWFactionPersistenceHandler** -- Abstract base class for save/load backends.

- **UFWFactionBlueprintLibrary** -- Static Blueprint helpers.
    - `GetFactionComponent`, `AreActorsHostile`, `AreActorsFriendly`, `AreActorsSameFaction`.
    - `GetAttitudeDisplayName`, `GetAttitudeColor`.

- **UFWFactionSettings** -- Project settings (`Project Settings > Game > Faction System`).

- **Types** -- `EFWFactionAttitude`, `EFWFactionOperationResult`, `FFWReputationThresholds`, `FFWReputationPropagationRule`, `FFWFactionRelationship`, `FFWFactionMembership`, `FFWPersonalReputation`, `FFWFactionPlayerData`, `FFWAttitudeChangeEvent`.

- **Utilities** -- `FWFactionUtils::ConvertToTeamAttitude`, `FWFactionUtils::MakeRelationshipKey`.

- **Debug** -- `FFWFactionConsoleCommands`, `FFWFactionDebugHUD` (non-shipping only).
