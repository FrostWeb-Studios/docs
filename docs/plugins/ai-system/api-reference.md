---
title: API Reference
---

# API Reference

---

## Enums

### EFWNpcState

NPC behavior state machine.

| Value | Description |
|-------|-------------|
| `Idle` | NPC is stationary at spawn |
| `Patrol` | NPC is patrolling (waypoint or random roam) |
| `Combat` | NPC is engaged in combat |
| `Leashing` | NPC has exceeded hard leash and is returning |
| `Resetting` | NPC is resetting state after returning to spawn |
| `Dead` | NPC is dead |

### EFWPatrolMode

Patrol movement mode.

| Value | Description |
|-------|-------------|
| `None` | No patrol behavior |
| `Waypoint` | Move through ordered waypoints |
| `RandomRoam` | Pick random reachable points within a radius |

### EFWDifficultyTier

Instance difficulty tier.

| Value | HP Multiplier | Damage Multiplier | XP Multiplier |
|-------|:-------------:|:-----------------:|:-------------:|
| `Normal` | 1.0x | 1.0x | 1.0x |
| `Hard` | 2.0x | 1.5x | 1.75x |
| `Mythic` | 4.0x | 2.5x | 3.0x |

### EFWThreatReason

Reason for generating threat.

| Value | Description |
|-------|-------------|
| `Damage` | Direct damage dealt to NPC |
| `Healing` | Healing another player (generates reduced threat) |
| `Taunt` | Forced target switch (multiplied threat) |
| `Proximity` | Entering aggro range |
| `AoE` | Area-of-effect abilities |

### EFWNpcCategory

NPC classification category.

| Value | Description |
|-------|-------------|
| `Trash` | Standard encounter NPC |
| `Elite` | Stronger than normal, typically requires a group |
| `Rare` | Rare spawn with special loot |
| `Boss` | Dungeon or instanced boss |
| `WorldBoss` | Open-world boss requiring a large group |

---

## Structs

### FFWThreatEntry

Single entry in an NPC's threat table.

| Field | Type | Description |
|-------|------|-------------|
| `Source` | `Actor` (weak reference) | The actor generating threat |
| `ThreatValue` | `float` | Cumulative threat value |
| `LastThreatTime` | `float` | World time of last threat event |

### FFWInstanceScalingConfig

Instance scaling configuration for difficulty and party size.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `DifficultyTier` | `EFWDifficultyTier` | `Normal` | Difficulty tier |
| `HpMultiplier` | `float` | `1.0` | Base HP multiplier for this tier |
| `DamageMultiplier` | `float` | `1.0` | Base damage multiplier for this tier |
| `XpMultiplier` | `float` | `1.0` | XP reward multiplier for this tier |
| `ExpectedPlayerCount` | `int32` | `1` | Expected player count for baseline tuning |
| `PerPlayerHpScale` | `float` | `0.3` | Additional HP per extra player (0.3 = +30%) |
| `PerPlayerDamageScale` | `float` | `0.1` | Additional damage per extra player (0.1 = +10%) |

---

## Components

### UFWThreatComponent

Per-NPC threat table component. Tracks cumulative threat from all attackers, providing MMO-style target priority. Server-authoritative.

#### Functions

| Function | Description |
|----------|-------------|
| `AddThreat(Source, Amount, Reason)` | Add threat from a source actor. Automatically inserts new entries and re-sorts the threat table. `Reason` defaults to `Damage`. |
| `RemoveThreat(Source)` | Remove an actor from the threat table entirely. |
| `ClearAllThreat()` | Clear the entire threat table. Typically called on NPC death or leash reset. |
| `ModifyThreat(Source, Multiplier)` | Multiply an actor's threat by a factor (e.g., 0.5 = halve threat). Used for threat reduction abilities. |
| `TransferThreat(From, To, Percentage)` | Transfer a percentage of threat from one actor to another (0.0 to 1.0). Used for taunt mechanics. |
| `GetHighestThreat()` | Returns the actor with the highest threat, or null if the table is empty. |
| `HasThreat()` | Returns `true` if the threat table has any entries. |
| `GetThreatList()` | Returns the full threat table sorted by descending threat value. C++ only. |
| `GetThreatValue(Source)` | Returns the threat value for a specific actor, or `0.0` if not found. |

#### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ThreatDecayRate` | `float` | `2.0` | Threat lost per second when source is not attacking |
| `TauntThreatMultiplier` | `float` | `1.5` | Multiplier for taunt threat (applied to current top value) |
| `HealingThreatMultiplier` | `float` | `0.5` | Threat per point of healing (relative to damage) |
| `ProximityThreatRadius` | `float` | `0.0` | Range for initial proximity threat (0 = disabled) |
| `MaxThreatEntries` | `int32` | `40` | Maximum entries in the threat table |

#### Events

| Event | Payload | Description |
|-------|---------|-------------|
| `OnThreatChanged` | Threat event data (Source, Amount, Reason) | Fired when threat is added or modified |
| `OnThreatTableEmpty` | -- | Fired when the threat table becomes empty |
| `OnNewHighestThreat` | New target actor | Fired when the highest-threat target changes |

---

### UFWLeashComponent

Soft leash component for NPC spawn-anchor behavior. Provides MMO-standard leash mechanics with two-zone distance falloff.

#### Functions

| Function | Description |
|----------|-------------|
| `SetSpawnAnchor(Location, Rotation)` | Override the spawn anchor location and rotation. |
| `GetSpawnLocation()` | Returns the spawn anchor location. |
| `GetDistanceFromSpawn()` | Returns the owner's current distance from the spawn anchor in centimeters. |
| `IsInSoftLeash()` | Returns `true` if the owner is between the soft and hard leash radius. |
| `IsLeashTriggered()` | Returns `true` if the owner has exceeded the hard leash radius. |
| `ShouldReset()` | Returns `true` when leash is triggered and no threat has been received for `ResetDelay` seconds. |
| `GetPursuitSpeedModifier()` | Returns speed modifier: `1.0` in normal range, reduced in soft zone, `0.0` beyond hard leash. |
| `CanLeash()` | Returns `true` if this NPC can leash. Returns `false` for bosses with `bCanLeash = false`. |
| `ResetToSpawn()` | Reset the NPC to its spawn point. Clears threat and optionally restores HP. |
| `BeginLeashing()` | Mark that leash has started. Used for tracking the reset delay timer. |

#### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `LeashRadius` | `float` | `4000.0` | Max chase distance from spawn (cm) -- hard leash |
| `SoftLeashRadius` | `float` | `3000.0` | Distance where pursuit slows (cm) -- soft leash |
| `RegenRatePercent` | `float` | `0.05` | HP regen per second while leashing (fraction of max HP) |
| `PursuitSpeedMultiplier` | `float` | `0.6` | Speed multiplier in the soft leash zone |
| `ResetDelay` | `float` | `8.0` | Seconds of no threat before full reset |
| `bResetHealthOnReturn` | `bool` | `true` | Restore NPC to full health on reset |
| `bCanLeash` | `bool` | `true` | Whether this NPC can leash (set false for bosses) |

#### Events

| Event | Payload | Description |
|-------|---------|-------------|
| `OnLeashTriggered` | NPC actor, distance | Fired when the NPC exceeds the hard leash distance |
| `OnResetBegin` | -- | Fired when the NPC begins resetting to spawn |
| `OnResetComplete` | -- | Fired when the NPC completes its reset |

---

## Data Assets

### UFWNpcDefinition

Data-driven NPC configuration asset. One per NPC archetype.

#### Identity

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `NpcId` | `FName` | -- | Unique identifier for this NPC type |
| `DisplayName` | `FText` | -- | Localized display name |
| `Category` | `EFWNpcCategory` | `Trash` | Classification category |
| `NpcClass` | Pawn class reference | -- | Character class to spawn |

#### AI Behavior

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `BehaviorTree` | Behavior tree reference | -- | Behavior tree asset to run |
| `PatrolMode` | `EFWPatrolMode` | `None` | Patrol movement mode |
| `RoamRadius` | `float` | `1000.0` | Radius for random roam patrol (cm) |
| `AggroRadius` | `float` | `1500.0` | AI perception sight radius for aggro detection (cm) |

#### Leash

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `LeashRadius` | `float` | `0.0` | Max chase distance override (0 = use component default) |
| `SoftLeashRadius` | `float` | `0.0` | Soft leash distance override (0 = use component default) |
| `bCanLeash` | `bool` | `true` | Whether the NPC can leash (false for raid bosses) |

#### Threat

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ThreatDecayRate` | `float` | `0.0` | Threat decay rate override (0 = use component default) |
| `TauntThreatMultiplier` | `float` | `0.0` | Taunt multiplier override (0 = use component default) |

#### Combat

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `BaseLevel` | `int32` | `1` | Base level for this NPC type |
| `AbilityTags` | `GameplayTagContainer` | -- | GAS ability tags to grant on spawn |
| `AttributeDefaults` | DataTable reference | -- | Data table for base attribute defaults |

#### Scaling

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TierScaling` | Map of `EFWDifficultyTier` to `FFWInstanceScalingConfig` | -- | Per-difficulty-tier scaling overrides |

#### Faction

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `DefaultFactionId` | `FName` | -- | Faction ID to assign on spawn |

#### Spawning

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `RespawnTime` | `float` | `60.0` | Respawn time in seconds |
| `MaxActiveInstances` | `int32` | `1` | Max concurrent instances per spawn point |

---

### UFWSpawnTable

Weighted list of NPC definitions for zone-based spawning.

#### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Entries` | Array of spawn entries | -- | Weighted list of NPC types (each entry has NpcDefinition, Weight, MinLevel, MaxLevel) |
| `MinLevel` | `int32` | `1` | Minimum level filter for this table |
| `MaxLevel` | `int32` | `100` | Maximum level filter for this table |

#### Functions

| Function | Description |
|----------|-------------|
| `Roll()` | Roll a weighted random pick from the spawn table. Returns the selected NPC Definition, or null if the table is empty. |
| `GetEntry(Index)` | Get the entry at a specific index for deterministic spawning. |

---

## Spawning

### AFWNpcSpawnPoint

Level-placed actor for specific NPC placement with respawn management.

#### Functions

| Function | Description |
|----------|-------------|
| `SpawnNpc()` | Spawn an NPC from the assigned definition. Configures Threat and Leash components automatically. Returns the spawned Pawn, or null if spawning fails or max instances reached. |
| `StartRespawnTimer()` | Start the respawn countdown using the effective respawn time. |
| `CancelRespawnTimer()` | Cancel a pending respawn timer. |
| `GetActiveCount()` | Returns the number of currently active (alive) spawned NPCs. |

#### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `NpcDefinition` | NPC Definition reference | -- | What NPC type to spawn |
| `RespawnTime` | `float` | `0.0` | Respawn time override in seconds (0 = use definition default) |
| `InitialSpawnDelay` | `float` | `0.0` | Delay before first spawn on level load |
| `MaxInstances` | `int32` | `1` | Max concurrent NPCs from this spawn point |
| `WanderRadius` | `float` | `0.0` | Roam radius override |
| `PatrolPoints` | Array of vectors | -- | Optional patrol waypoints (local space) |
| `bSpawnOnBeginPlay` | `bool` | `true` | Whether to auto-spawn on BeginPlay |

---

### AFWNpcSpawnVolume

Zone-based density spawner using a box volume. Spawns and despawns based on player proximity.

#### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `SpawnTable` | Spawn Table reference | -- | Spawn table defining what NPCs to spawn |
| `MaxNpcs` | `int32` | `10` | Maximum NPC density within this volume |
| `SpawnInterval` | `float` | `5.0` | Seconds between spawn checks |
| `DespawnDistance` | `float` | `15000.0` | Despawn NPCs when no player is within this distance (cm) |
| `SpawnDistance` | `float` | `10000.0` | Only spawn if a player is within this distance (cm) |
| `bCheckNavmesh` | `bool` | `true` | Validate spawn points are on the navmesh |

---

## Instance Scaling

### UFWInstanceScalingLibrary

Static blueprint function library for dungeon/raid scaling calculations.

#### Functions

| Function | Description |
|----------|-------------|
| `GetTierMultipliers(Tier)` | Get default scaling multipliers for a difficulty tier. Returns an `FFWInstanceScalingConfig`. |
| `ComputePartyScaling(PlayerCount, BaseConfig)` | Compute party-scaled config based on player count. Applies additive per-player HP and damage scaling. |
| `ApplyScalingToNpc(NpcActor, Config)` | Apply scaling configuration to an NPC actor. Modifies GAS attributes (MaxHealth, AttackPower) if FWGASSystem is available, otherwise stores scaling data for manual application. |
| `GetEffectiveHpMultiplier(Tier, PlayerCount)` | Get effective HP multiplier for a tier and player count. Returns the combined multiplier. |
| `GetEffectiveDamageMultiplier(Tier, PlayerCount)` | Get effective damage multiplier for a tier and player count. Returns the combined multiplier. |

---

## Behavior Tree Nodes

### Tasks

#### FindThreatTarget

Queries the NPC's threat component and sets the target blackboard key to the highest-threat actor.

| Property | Type | Description |
|----------|------|-------------|
| `TargetActorKey` | Blackboard key | Blackboard key for the target actor |

**Result:** Succeeds if a target is found, fails if the threat table is empty.

#### MoveToTarget

Moves the NPC toward the target blackboard key. Checks leash component each tick and aborts if leash is triggered.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TargetActorKey` | Blackboard key | -- | Blackboard key for the target actor |
| `AcceptanceRadius` | `float` | `100.0` | Distance to target before arrival |
| `bStopOnOverlap` | `bool` | `true` | Stop when overlapping target's collision |

#### ReturnToSpawn

Moves the NPC back to its spawn location. On arrival, resets the leash component.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `AcceptanceRadius` | `float` | `50.0` | Distance to spawn before arrival |

#### Patrol

Dual-mode patrol task supporting both waypoint and random-roam patterns.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `PatrolMode` | `EFWPatrolMode` | `RandomRoam` | Patrol movement mode |
| `WaitTimeMin` | `float` | `2.0` | Minimum wait time at each point (seconds) |
| `WaitTimeMax` | `float` | `5.0` | Maximum wait time at each point (seconds) |
| `RoamRadius` | `float` | `1000.0` | Radius for random roam |
| `AcceptanceRadius` | `float` | `100.0` | Acceptance radius for patrol point |
| `PatrolPoints` | Array of vectors | -- | Waypoint points (local space) |
| `PatrolIndexKey` | Blackboard key | -- | Blackboard key for patrol index |

#### PlayAbility

Activates a GAS ability on the NPC's AbilitySystemComponent by gameplay tag. Requires FWGASSystem.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `AbilityTag` | `GameplayTag` | -- | Gameplay tag identifying the ability |
| `bWaitForEnd` | `bool` | `true` | Task remains InProgress until ability ends |

### Services

#### UpdateThreat

Ticking service that queries AI Perception for newly sensed actors, adds proximity threat for hostiles, and updates blackboard keys.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TargetActorKey` | Blackboard key | -- | Blackboard key for current threat target |
| `HasThreatKey` | Blackboard key | -- | Blackboard key for whether threat exists |
| `ProximityThreatPerTick` | `float` | `1.0` | Proximity threat per tick for hostile actors |

#### CheckLeash

Ticking service that monitors the leash component state and updates the NPC state blackboard key.

| Property | Type | Description |
|----------|------|-------------|
| `NpcStateKey` | Blackboard key | Blackboard key for NPC state |

### Decorators

#### HasThreat

Checks whether the NPC's threat table has any entries.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `bQueryComponentDirectly` | `bool` | `true` | Query threat component or read from blackboard |
| `HasThreatKey` | Blackboard key | -- | Blackboard key (used when not querying directly) |

#### IsLeashing

Checks the leash component to determine if the NPC should disengage.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `bIncludeSoftLeash` | `bool` | `false` | If true, triggers on soft leash zone too |

#### FactionAttitude

Checks the faction attitude between the NPC and a target actor. Uses FWFactionSystem when available, falls back to the generic team agent interface.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TargetActorKey` | Blackboard key | -- | Blackboard key for the target actor |
| `RequiredAttitude` | Team attitude | `Hostile` | Required attitude for this decorator to pass |

---

## Blackboard Keys

The following blackboard keys are used by the built-in behavior tree nodes:

| Key | Description |
|-----|-------------|
| `TargetActor` | Current threat target actor |
| `SpawnLocation` | NPC spawn anchor location |
| `NpcState` | Current NPC behavior state |
| `PatrolIndex` | Current waypoint index for patrol |
| `bHasThreat` | Whether the threat table has entries |
| `HomeLocation` | Return location for leashing |
