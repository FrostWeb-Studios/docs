---
title: Changelog - FWAISystem
---

# Changelog

All notable changes to FWAISystem are documented in this file.

---

## v1.0 -- Initial Release

### Components

- **UFWThreatComponent** -- Per-NPC threat table with cumulative tracking, decay, transfer, and modification. Supports threat from damage, healing, taunts, proximity, and AoE. Server-authoritative with automatic pruning of invalid entries.
- **UFWLeashComponent** -- Spawn anchor with soft and hard leash radii, pursuit speed reduction, configurable reset delay, HP regeneration during reset, and optional full heal on return to spawn.

### Data Assets

- **UFWNpcDefinition** -- Data-driven NPC configuration asset covering identity, AI behavior, leash overrides, threat overrides, combat, scaling, faction, and spawning parameters. Registered as a PrimaryDataAsset for Asset Manager integration.
- **UFWSpawnTable** -- Weighted list of NPC definitions for zone-based spawning with level filtering and weighted random selection.

### Spawning

- **AFWNpcSpawnPoint** -- Level-placed actor for specific NPC placement with auto-spawn on BeginPlay, respawn timers, configurable initial delay, max instances, and optional patrol waypoints. Editor billboard visualization.
- **AFWNpcSpawnVolume** -- Box-volume density spawner with weighted spawn tables, player proximity-based spawn/despawn, configurable density and interval, and navmesh validation.

### Instance Scaling

- **UFWInstanceScalingLibrary** -- Static Blueprint function library with difficulty tier multipliers (Normal, Hard, Mythic), party-size scaling with additive per-player HP and damage modifiers, and direct GAS attribute application.

### Behavior Tree Nodes

Tasks:

- **UFWBTTask_FindThreatTarget** -- Queries threat component and sets TargetActor blackboard key.
- **UFWBTTask_MoveToTarget** -- Moves toward TargetActor with leash-aware abort.
- **UFWBTTask_ReturnToSpawn** -- Returns NPC to spawn location and triggers reset.
- **UFWBTTask_Patrol** -- Dual-mode patrol (waypoint and random roam) with configurable wait times.
- **UFWBTTask_PlayAbility** -- GAS ability activation by gameplay tag with optional wait-for-end. Requires FWGASSystem (graceful fallback when absent).

Services:

- **UFWBTService_UpdateThreat** -- Ticking service for AI Perception-based proximity threat and blackboard updates.
- **UFWBTService_CheckLeash** -- Ticking service for leash state monitoring and NpcState blackboard updates.

Decorators:

- **UFWBTDecorator_HasThreat** -- Condition check for threat table entries (direct query or blackboard-based).
- **UFWBTDecorator_IsLeashing** -- Condition check for leash state with optional soft leash inclusion.
- **UFWBTDecorator_FactionAttitude** -- Faction attitude gate using FWFactionSystem (falls back to IGenericTeamAgentInterface).

### Types and Enums

- `EFWNpcState` -- Idle, Patrol, Combat, Leashing, Resetting, Dead
- `EFWPatrolMode` -- None, Waypoint, RandomRoam
- `EFWDifficultyTier` -- Normal, Hard, Mythic
- `EFWThreatReason` -- Damage, Healing, Taunt, Proximity, AoE
- `EFWNpcCategory` -- Trash, Elite, Rare, Boss, WorldBoss
- `FFWThreatEntry` -- Threat table entry struct
- `FFWThreatEvent` -- Threat event data struct
- `FFWSpawnTableEntry` -- Spawn table entry struct
- `FFWInstanceScalingConfig` -- Instance scaling configuration struct

### Delegates

- `FOnThreatChanged` -- Threat added/modified
- `FOnThreatTableEmpty` -- Threat table emptied
- `FOnNewHighestThreat` -- Highest-threat target changed
- `FOnNpcStateChanged` -- NPC behavior state changed
- `FOnLeashTriggered` -- Hard leash exceeded
- `FOnLeashResetBegin` -- Leash reset started
- `FOnLeashResetComplete` -- Leash reset completed

### Blackboard Key Constants

- `FWAIBBKeys::TargetActor`, `SpawnLocation`, `NpcState`, `PatrolIndex`, `HasThreat`, `HomeLocation`

### Optional Integrations

- **FWFactionSystem** -- Auto-detected at compile time via `WITH_FWFACTIONSYSTEM`. Enables `FWBTDecorator_FactionAttitude` faction queries.
- **FWGASSystem** -- Auto-detected at compile time via `WITH_FWGASSYSTEM`. Enables `FWBTTask_PlayAbility` and GAS attribute scaling in `ApplyScalingToNpc`.
