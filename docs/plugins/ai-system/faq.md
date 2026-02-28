---
title: FAQ - FWAISystem
---

# Frequently Asked Questions

---

## Threat System

### How is threat priority determined?

The threat table is sorted by descending `ThreatValue`. The actor with the highest cumulative threat becomes the NPC's target. When two actors have equal threat, the one who generated threat most recently takes priority (based on `LastThreatTime`).

### Does threat decay when a player is still attacking?

No. `ThreatDecayRate` only applies when a source has not generated any new threat. Each time `AddThreat()` is called for an actor, their `LastThreatTime` is updated, resetting the decay clock for that entry.

### How do taunts work?

When `AddThreat()` is called with `Reason = EFWThreatReason::Taunt`, the threat amount is multiplied by `TauntThreatMultiplier` (default 1.5x) and applied to the current top threat value. This means a taunt generates `TopThreatValue * TauntThreatMultiplier` threat, ensuring the taunting player reliably becomes the highest-threat target.

### What happens when a player dies?

Dead players are automatically pruned from the threat table during the `DecayAndPrune` tick. Their `TWeakObjectPtr<AActor>` becomes invalid when the actor is destroyed, and the entry is removed on the next tick. If the dead player was the highest-threat target, `OnNewHighestThreat` fires with the next actor in the table.

### Can I have multiple threat components on one NPC?

Technically yes, but it is not recommended. Each threat component maintains its own independent table. The behavior tree nodes (`FindThreatTarget`, `UpdateThreat`) query the first `UFWThreatComponent` found via `GetComponentByClass`, so only one table will be used for AI decisions.

### How does healing generate threat?

Healing generates threat on ALL NPCs that have the healer's allies on their threat tables. The threat amount is `HealAmount * HealingThreatMultiplier` (default 0.5, meaning healing generates half the threat of equivalent damage). You must call `AddThreat()` manually with `Reason = EFWThreatReason::Healing` -- the system does not automatically detect healing.

### What is the maximum threat table size?

The default is 40 entries (`MaxThreatEntries`). For world bosses with many participants, increase this value. When the table is full and a new actor generates threat, the lowest-threat entry is evicted.

---

## Leashing

### What is the difference between soft leash and hard leash?

- **Soft leash** (default: 3000 cm / 30 meters): The NPC's pursuit speed is reduced by `PursuitSpeedMultiplier` (default 0.6, meaning 60% speed). The NPC is still chasing but slowing down.
- **Hard leash** (default: 4000 cm / 40 meters): The NPC fully disengages. It stops chasing, clears its threat table, walks back to spawn, and regenerates HP.

### Can I prevent a boss from leashing?

Yes. Set `bCanLeash = false` on either the `UFWLeashComponent` or the `UFWNpcDefinition`. This is the recommended approach for instanced dungeon and raid bosses.

### How fast does HP regenerate during leash reset?

At the default `RegenRatePercent` of 0.05 (5% per second), an NPC fully heals in 20 seconds. This happens while the NPC walks back to spawn. If the NPC reaches spawn before fully healing, `bResetHealthOnReturn = true` instantly restores it to full HP.

### What happens if a player re-engages an NPC that is leashing?

If a player generates new threat (by dealing damage) while the NPC is walking back to spawn but before the `ResetDelay` timer expires, the NPC will re-engage. The `ShouldReset()` function returns `false` until `ResetDelay` seconds (default 8) have passed with no new threat.

### How do I adjust leash distances per NPC type?

Set `LeashRadius` and `SoftLeashRadius` on the `UFWNpcDefinition` data asset. Values of `0` mean "use the component default." The spawn point automatically applies these overrides when spawning the NPC.

---

## Party Scaling

### How is the scaling formula calculated?

The scaling formula is:

```
EffectiveMultiplier = BaseMultiplier * (1 + PerPlayerScale * max(0, PlayerCount - ExpectedPlayerCount))
```

This means:

- At the expected player count, no additional scaling is applied
- Each extra player beyond the expected count adds a flat percentage
- Fewer players than expected does NOT reduce the multiplier (it floors at 0)

### What are the default scaling values?

| Parameter | Default |
|-----------|---------|
| ExpectedPlayerCount | 1 |
| PerPlayerHpScale | 0.3 (+30% HP per extra player) |
| PerPlayerDamageScale | 0.1 (+10% damage per extra player) |

### Can scaling be applied dynamically (e.g., when a player joins mid-fight)?

Yes. Call `UFWInstanceScalingLibrary::ApplyScalingToNpc()` at any time to update the NPC's stats. However, this recalculates from the base stats, so any temporary buffs or debuffs on the NPC will be overwritten.

!!! tip "Mid-Fight Scaling"
    For mid-fight scaling, cache the current health percentage before applying new scaling, then restore it afterward. This prevents the boss from suddenly gaining or losing a large chunk of current HP.

### What if I want different scaling formulas?

Override `GetTierMultipliers()` or `ComputePartyScaling()` by creating a child class of `UFWInstanceScalingLibrary` in your game module. The Blueprint function library pattern makes this straightforward.

---

## Spawning

### When does the spawn point auto-spawn NPCs?

If `bSpawnOnBeginPlay = true` (the default), the spawn point calls `SpawnNpc()` during `BeginPlay()`. If `InitialSpawnDelay` is set, it waits that many seconds before the first spawn.

### How does the respawn timer work?

When a spawned NPC is destroyed (dies), the spawn point's `OnNpcDestroyed` callback fires and calls `StartRespawnTimer()`. After the respawn time elapses (from `RespawnTime` override or `NpcDefinition->RespawnTime`), `SpawnNpc()` is called again.

### What is the difference between AFWNpcSpawnPoint and AFWNpcSpawnVolume?

| Feature | SpawnPoint | SpawnVolume |
|---------|:----------:|:-----------:|
| Placement | Exact location | Anywhere in box volume |
| NPC Source | Single definition | Weighted spawn table |
| Patrol Support | Waypoint array | No (uses definition roam) |
| Density Control | MaxInstances | MaxNpcs |
| Player Proximity | No | Yes (spawn/despawn distance) |
| Use Case | Named NPCs, bosses, quest givers | Zone population, roaming mobs |

### Do spawn volumes check for valid spawn locations?

Yes, when `bCheckNavmesh = true` (the default). The volume finds random points within its bounds and validates them against the NavMesh. If no valid location is found after several attempts, that spawn check is skipped and retried at the next interval.

---

## Behavior Tree

### What blackboard keys do I need?

At minimum, create these blackboard keys:

| Key | Type | Used By |
|-----|------|---------|
| `TargetActor` | Object (Actor) | FindThreatTarget, MoveToTarget, FactionAttitude |
| `bHasThreat` | Bool | UpdateThreat, HasThreat decorator |
| `NpcState` | Enum (int) | CheckLeash |
| `PatrolIndex` | Int | Patrol task |

### Can I mix FWAISystem BT nodes with custom nodes?

Yes. FWAISystem BT nodes are standard `UBTTaskNode`, `UBTService`, and `UBTDecorator` subclasses. They integrate with any behavior tree and can be freely mixed with custom nodes, standard UE nodes, or nodes from other plugins.

### What happens if FWGASSystem is not installed and I use PlayAbility?

The `UFWBTTask_PlayAbility` task is guarded by `WITH_FWGASSYSTEM`. When FWGASSystem is not present, the task immediately returns `EBTNodeResult::Failed`, and the behavior tree continues to the next branch in the selector.

---

## Integration

### Does FWAISystem replicate?

The threat table and leash state are server-only. No replication is performed. Target selection and AI decisions run entirely on the server, which is the standard approach for authoritative multiplayer games.

### Can I use FWAISystem without FWFactionSystem?

Yes. FWFactionSystem is optional. Without it, the `FWBTDecorator_FactionAttitude` falls back to Unreal's `IGenericTeamAgentInterface`. You can set team IDs on your AI Controller and player controller to control hostility.

### Can I use FWAISystem without FWGASSystem?

Yes. FWGASSystem is optional. Without it, `FWBTTask_PlayAbility` fails gracefully, and `ApplyScalingToNpc` stores scaling data for manual application instead of modifying GAS attributes.
