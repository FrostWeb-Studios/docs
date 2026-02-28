---
title: Configuration - FWAISystem
---

# Configuration

This guide covers how to configure NPC definitions, difficulty tiers, scaling parameters, and spawn tables.

---

## NPC Definition Data Assets

The `UFWNpcDefinition` data asset is the primary configuration point for each NPC archetype. Create one per NPC type (e.g., "Forest Wolf", "Lich Guard", "Raid Boss").

### Creating a Definition

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWNpcDefinition` as the parent class.
3. Name it following the convention `DA_<NpcName>` (e.g., `DA_LichGuard`).

### Configuration Sections

#### Identity

| Field | Description | Example |
|-------|-------------|---------|
| `NpcId` | Unique FName identifier. Used as the primary asset ID. | `LichGuard` |
| `DisplayName` | Localized display name shown to players. | `Lich Guard` |
| `Category` | Classification tier affecting loot tables and UI. | `Elite` |
| `NpcClass` | Soft reference to the Pawn Blueprint to spawn. | `BP_LichGuard` |

!!! tip "NpcId Naming"
    Use PascalCase without spaces for `NpcId`. This value is used in the Asset Manager and must be unique across all NPC definitions.

#### AI Behavior

| Field | Description | Default |
|-------|-------------|---------|
| `BehaviorTree` | Soft reference to the Behavior Tree to run. | -- |
| `PatrolMode` | `None`, `Waypoint`, or `RandomRoam`. | `None` |
| `RoamRadius` | Radius for random roam in centimeters. Only used when `PatrolMode = RandomRoam`. | `1000` |
| `AggroRadius` | AI perception sight radius for aggro detection in centimeters. | `1500` |

=== "No Patrol"

    Set `PatrolMode = None` for NPCs that stand in place until aggro'd.

=== "Random Roam"

    Set `PatrolMode = RandomRoam` and configure `RoamRadius`. The NPC picks random navmesh-reachable points within the radius and walks between them with randomized wait times.

=== "Waypoint Patrol"

    Set `PatrolMode = Waypoint`. Waypoints are defined on the `AFWNpcSpawnPoint` actor's `PatrolPoints` array, not in the definition. This allows the same NPC definition to patrol different routes in different levels.

#### Leash Overrides

| Field | Description | Default |
|-------|-------------|---------|
| `LeashRadius` | Override for hard leash distance (cm). `0` = use component default (4000). | `0` |
| `SoftLeashRadius` | Override for soft leash distance (cm). `0` = use component default (3000). | `0` |
| `bCanLeash` | Set to `false` to prevent this NPC from ever leashing. Use for raid/dungeon bosses. | `true` |

!!! warning "Boss Leashing"
    Always set `bCanLeash = false` for bosses in instanced content. If a boss leashes mid-fight, it creates a frustrating player experience. For open-world bosses where leashing is acceptable, leave it enabled with a generous radius.

#### Threat Overrides

| Field | Description | Default |
|-------|-------------|---------|
| `ThreatDecayRate` | Override threat lost per second. `0` = use component default (2.0). | `0` |
| `TauntThreatMultiplier` | Override taunt multiplier. `0` = use component default (1.5). | `0` |

#### Combat

| Field | Description | Default |
|-------|-------------|---------|
| `BaseLevel` | Base NPC level. Used for scaling and combat calculations. | `1` |
| `AbilityTags` | GAS ability tags granted on spawn. Requires FWGASSystem. | -- |
| `AttributeDefaults` | Data table for GAS base attribute defaults. | -- |

#### Faction

| Field | Description | Default |
|-------|-------------|---------|
| `DefaultFactionId` | FName of the faction to assign on spawn. Requires FWFactionSystem. | -- |

#### Spawning

| Field | Description | Default |
|-------|-------------|---------|
| `RespawnTime` | Seconds before respawning after death. | `60` |
| `MaxActiveInstances` | Max concurrent instances per spawn point. | `1` |

---

## Difficulty Tiers

FWAISystem supports three difficulty tiers with built-in default multipliers.

### Default Tier Multipliers

| Tier | HP Multiplier | Damage Multiplier | XP Multiplier |
|------|:-------------:|:-----------------:|:-------------:|
| Normal | 1.0 | 1.0 | 1.0 |
| Hard | 2.0 | 1.5 | 1.75 |
| Mythic | 4.0 | 2.5 | 3.0 |

### Per-Definition Tier Overrides

You can override the default tier multipliers on a per-NPC basis using the `TierScaling` map in `UFWNpcDefinition`:

```cpp
// In your NPC definition setup
UFWNpcDefinition* BossDef = ...;

FFWInstanceScalingConfig MythicConfig;
MythicConfig.DifficultyTier = EFWDifficultyTier::Mythic;
MythicConfig.HpMultiplier = 6.0f;      // Override: 6x HP instead of 4x
MythicConfig.DamageMultiplier = 3.0f;   // Override: 3x damage instead of 2.5x
MythicConfig.XpMultiplier = 5.0f;       // Override: 5x XP instead of 3x
MythicConfig.ExpectedPlayerCount = 5;
MythicConfig.PerPlayerHpScale = 0.2f;   // +20% HP per extra player
MythicConfig.PerPlayerDamageScale = 0.05f; // +5% damage per extra player

BossDef->TierScaling.Add(EFWDifficultyTier::Mythic, MythicConfig);
```

In the Editor, expand the `TierScaling` map and add entries for each tier you want to customize.

---

## Party Scaling

Party scaling adjusts NPC stats based on the number of players in the group. The formula is additive:

```
EffectiveHP = BaseHP * HpMultiplier * (1 + PerPlayerHpScale * max(0, PlayerCount - ExpectedPlayerCount))
EffectiveDmg = BaseDmg * DamageMultiplier * (1 + PerPlayerDamageScale * max(0, PlayerCount - ExpectedPlayerCount))
```

### Scaling Configuration Fields

| Field | Description | Default |
|-------|-------------|---------|
| `ExpectedPlayerCount` | Baseline player count (no scaling at this count). | `1` |
| `PerPlayerHpScale` | Fractional HP increase per extra player. `0.3` = +30% per player. | `0.3` |
| `PerPlayerDamageScale` | Fractional damage increase per extra player. `0.1` = +10% per player. | `0.1` |

### Example: 5-Player Mythic Dungeon Boss

| Setting | Value |
|---------|-------|
| Base HP | 10,000 |
| HpMultiplier (Mythic) | 4.0 |
| ExpectedPlayerCount | 5 |
| PerPlayerHpScale | 0.2 |

With 5 players (expected): `10,000 * 4.0 * (1 + 0.2 * 0) = 40,000 HP`

With 6 players: `10,000 * 4.0 * (1 + 0.2 * 1) = 48,000 HP`

With 10 players: `10,000 * 4.0 * (1 + 0.2 * 5) = 80,000 HP`

---

## Spawn Tables

Spawn tables define weighted lists of NPC definitions for zone-based spawning via `AFWNpcSpawnVolume`.

### Creating a Spawn Table

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWSpawnTable` as the parent class.
3. Name it following the convention `DA_SpawnTable_<ZoneName>` (e.g., `DA_SpawnTable_DarkForest`).

### Configuring Entries

Each entry in the spawn table has the following fields:

| Field | Description |
|-------|-------------|
| `NpcDefinition` | Reference to the NPC definition data asset (preferred). |
| `NpcClass` | Direct class reference (fallback if no definition is needed). |
| `Weight` | Relative spawn weight. Higher values mean more frequent spawns. |
| `MinLevel` | Level floor override (-1 = use definition default). |
| `MaxLevel` | Level ceiling override (-1 = use definition default). |

### Weight Distribution Example

| NPC | Weight | Spawn Chance |
|-----|:------:|:------------:|
| Forest Wolf | 5.0 | 50% |
| Forest Bear | 3.0 | 30% |
| Forest Spider | 1.5 | 15% |
| Rare Forest Treant | 0.5 | 5% |
| **Total** | **10.0** | **100%** |

!!! info "Weight Calculation"
    Spawn chance = `EntryWeight / TotalWeight`. Weights are relative, not percentages. A table with weights [2, 2] produces the same 50/50 distribution as [1, 1].

### Level Filtering

Use `MinLevel` and `MaxLevel` on entries to restrict spawns to appropriate player levels:

```
Entry: DA_ForestWolfPup
  MinLevel: 1
  MaxLevel: 10
  Weight: 5.0

Entry: DA_ForestWolfAlpha
  MinLevel: 8
  MaxLevel: 20
  Weight: 3.0
```

This creates overlapping level ranges so players see a gradual transition between NPC types.

---

## Spawn Point Configuration

### AFWNpcSpawnPoint Properties

| Property | Description | Recommended Values |
|----------|-------------|--------------------|
| `NpcDefinition` | What NPC to spawn | Required |
| `RespawnTime` | Override definition respawn time (0 = use definition) | `0` for most, `300` for bosses |
| `InitialSpawnDelay` | Delay before first spawn on level load | `0` to `5` for staggering |
| `MaxInstances` | Max concurrent NPCs from this point | `1` for most, `3-5` for packs |
| `WanderRadius` | Override roam radius | `0` for most |
| `PatrolPoints` | Local-space waypoints for waypoint patrol | Define in editor viewport |
| `bSpawnOnBeginPlay` | Auto-spawn on level load | `true` for most |

### AFWNpcSpawnVolume Properties

| Property | Description | Recommended Values |
|----------|-------------|--------------------|
| `SpawnTable` | Spawn table data asset | Required |
| `MaxNpcs` | Maximum NPCs in the volume | `5-20` depending on zone size |
| `SpawnInterval` | Seconds between spawn checks | `3-10` |
| `DespawnDistance` | Despawn when no player within (cm) | `15000-30000` |
| `SpawnDistance` | Only spawn when player within (cm) | `8000-15000` |
| `bCheckNavmesh` | Validate spawn locations on navmesh | `true` (always) |

!!! warning "Performance"
    Keep `SpawnInterval` at 3 seconds or higher to avoid excessive spawn checks. For large zones with many volumes, consider 5-10 seconds. The `SpawnDistance` and `DespawnDistance` settings are critical for performance -- NPCs far from any player are despawned automatically.

---

## Threat Component Configuration

### Properties

| Property | Default | Description | Tuning Notes |
|----------|---------|-------------|--------------|
| `ThreatDecayRate` | `2.0` | Threat lost per second of inactivity | Increase for faster threat forgetting; decrease for stickier aggro |
| `TauntThreatMultiplier` | `1.5` | Multiplier applied to taunt-generated threat | Values above 1.0 ensure taunts reliably grab aggro |
| `HealingThreatMultiplier` | `0.5` | Threat per point of healing relative to damage | Keep below 1.0 to avoid healers pulling aggro too easily |
| `ProximityThreatRadius` | `0.0` | Range for automatic proximity threat (0 = disabled) | Set to aggro radius for auto-aggro on approach |
| `MaxThreatEntries` | `40` | Maximum table size | Increase for world bosses with many participants |

---

## Leash Component Configuration

### Properties

| Property | Default | Description | Tuning Notes |
|----------|---------|-------------|--------------|
| `LeashRadius` | `4000` | Hard leash distance (cm) | 40 meters default; increase for bosses with large arenas |
| `SoftLeashRadius` | `3000` | Soft leash distance (cm) | Must be less than LeashRadius |
| `RegenRatePercent` | `0.05` | HP regen per second as fraction of max HP | 5% per second = full heal in 20 seconds |
| `PursuitSpeedMultiplier` | `0.6` | Speed multiplier in the soft leash zone | Lower = more aggressive slowdown |
| `ResetDelay` | `8.0` | Seconds of no threat before full reset | Increase for zones with sparse enemies |
| `bResetHealthOnReturn` | `true` | Full heal on reaching spawn | Set false for NPCs that should retain damage |
| `bCanLeash` | `true` | Whether leashing is enabled | Set false for instanced bosses |
