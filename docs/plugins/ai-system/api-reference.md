---
title: API Reference - FWAISystem
---

# API Reference

Complete C++ API documentation for all FWAISystem classes, structs, enums, and delegates.

---

## Enums

### EFWNpcState

NPC behavior state machine. Defined in `FWAITypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWNpcState : uint8
{
    Idle,
    Patrol,
    Combat,
    Leashing,
    Resetting,
    Dead
};
```

| Value | Description |
|-------|-------------|
| `Idle` | NPC is stationary at spawn |
| `Patrol` | NPC is patrolling (waypoint or random roam) |
| `Combat` | NPC is engaged in combat |
| `Leashing` | NPC has exceeded hard leash and is returning |
| `Resetting` | NPC is resetting state after returning to spawn |
| `Dead` | NPC is dead |

### EFWPatrolMode

Patrol movement mode. Defined in `FWAITypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWPatrolMode : uint8
{
    None,
    Waypoint,
    RandomRoam
};
```

| Value | Description |
|-------|-------------|
| `None` | No patrol behavior |
| `Waypoint` | Move through ordered waypoints |
| `RandomRoam` | Pick random reachable points within a radius |

### EFWDifficultyTier

Instance difficulty tier. Defined in `FWAITypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWDifficultyTier : uint8
{
    Normal,
    Hard,
    Mythic
};
```

| Value | HP Multi | Dmg Multi | XP Multi |
|-------|:--------:|:---------:|:--------:|
| `Normal` | 1.0x | 1.0x | 1.0x |
| `Hard` | 2.0x | 1.5x | 1.75x |
| `Mythic` | 4.0x | 2.5x | 3.0x |

### EFWThreatReason

Reason for generating threat. Defined in `FWAITypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWThreatReason : uint8
{
    Damage,
    Healing,
    Taunt,
    Proximity,
    AoE
};
```

| Value | Description |
|-------|-------------|
| `Damage` | Direct damage dealt to NPC |
| `Healing` | Healing another player (generates reduced threat) |
| `Taunt` | Forced target switch (multiplied threat) |
| `Proximity` | Entering aggro range |
| `AoE` | Area-of-effect abilities |

### EFWNpcCategory

NPC classification category. Defined in `FWAITypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWNpcCategory : uint8
{
    Trash,
    Elite,
    Rare,
    Boss,
    WorldBoss
};
```

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

Single entry in an NPC's threat table. Defined in `FWAITypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWAISYSTEM_API FFWThreatEntry
{
    UPROPERTY(BlueprintReadOnly, Category = "Threat")
    TWeakObjectPtr<AActor> Source;

    UPROPERTY(BlueprintReadOnly, Category = "Threat")
    float ThreatValue = 0.f;

    UPROPERTY(BlueprintReadOnly, Category = "Threat")
    float LastThreatTime = 0.f;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `Source` | `TWeakObjectPtr<AActor>` | The actor generating threat |
| `ThreatValue` | `float` | Cumulative threat value |
| `LastThreatTime` | `float` | World time of last threat event |

### FFWThreatEvent

Threat event data passed to delegates. Defined in `FWAITypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWAISYSTEM_API FFWThreatEvent
{
    UPROPERTY(BlueprintReadOnly, Category = "Threat")
    TObjectPtr<AActor> Source = nullptr;

    UPROPERTY(BlueprintReadOnly, Category = "Threat")
    float Amount = 0.f;

    UPROPERTY(BlueprintReadOnly, Category = "Threat")
    EFWThreatReason Reason = EFWThreatReason::Damage;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `Source` | `TObjectPtr<AActor>` | The actor that caused the threat event |
| `Amount` | `float` | Amount of threat generated |
| `Reason` | `EFWThreatReason` | Reason for threat generation |

### FFWSpawnTableEntry

Weighted entry in a spawn table for zone-based spawning. Defined in `FWAITypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWAISYSTEM_API FFWSpawnTableEntry
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Spawn")
    TSoftClassPtr<APawn> NpcClass;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Spawn")
    TObjectPtr<UFWNpcDefinition> NpcDefinition = nullptr;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Spawn", meta = (ClampMin = "0.01"))
    float Weight = 1.f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Spawn")
    int32 MinLevel = -1;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Spawn")
    int32 MaxLevel = -1;
};
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `NpcClass` | `TSoftClassPtr<APawn>` | -- | Character class to spawn (if not using NpcDefinition) |
| `NpcDefinition` | `TObjectPtr<UFWNpcDefinition>` | `nullptr` | Data asset defining the NPC (preferred over NpcClass) |
| `Weight` | `float` | `1.0` | Relative spawn weight (higher = more likely) |
| `MinLevel` | `int32` | `-1` | Minimum level override (-1 = use definition default) |
| `MaxLevel` | `int32` | `-1` | Maximum level override (-1 = use definition default) |

### FFWInstanceScalingConfig

Instance scaling configuration for difficulty and party size. Defined in `FWAITypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWAISYSTEM_API FFWInstanceScalingConfig
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Scaling")
    EFWDifficultyTier DifficultyTier = EFWDifficultyTier::Normal;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Scaling", meta = (ClampMin = "0.1"))
    float HpMultiplier = 1.f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Scaling", meta = (ClampMin = "0.1"))
    float DamageMultiplier = 1.f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Scaling", meta = (ClampMin = "0.1"))
    float XpMultiplier = 1.f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Scaling", meta = (ClampMin = "1"))
    int32 ExpectedPlayerCount = 1;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Scaling", meta = (ClampMin = "0.0"))
    float PerPlayerHpScale = 0.3f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Scaling", meta = (ClampMin = "0.0"))
    float PerPlayerDamageScale = 0.1f;
};
```

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

## Delegates

All delegates are defined in `FWAITypes.h`.

| Delegate | Signature | Description |
|----------|-----------|-------------|
| `FOnThreatChanged` | `(const FFWThreatEvent& Event)` | Broadcast when threat is added or modified |
| `FOnThreatTableEmpty` | `()` | Broadcast when the threat table becomes empty |
| `FOnNewHighestThreat` | `(AActor* NewTarget)` | Broadcast when the highest-threat target changes |
| `FOnNpcStateChanged` | `(EFWNpcState NewState)` | Broadcast when the NPC behavior state changes |
| `FOnLeashTriggered` | `(AActor* NPC, float Distance)` | Broadcast when an NPC exceeds the hard leash radius |
| `FOnLeashResetBegin` | `()` | Broadcast when the NPC begins resetting to spawn |
| `FOnLeashResetComplete` | `()` | Broadcast when the NPC completes its reset |

---

## Blackboard Key Constants

Defined in namespace `FWAIBBKeys` in `FWAITypes.h`:

```cpp
namespace FWAIBBKeys
{
    inline const FName TargetActor   = TEXT("TargetActor");
    inline const FName SpawnLocation = TEXT("SpawnLocation");
    inline const FName NpcState      = TEXT("NpcState");
    inline const FName PatrolIndex   = TEXT("PatrolIndex");
    inline const FName HasThreat     = TEXT("bHasThreat");
    inline const FName HomeLocation  = TEXT("HomeLocation");
}
```

---

## Classes

### UFWThreatComponent

`UActorComponent` | Header: `Components/FWThreatComponent.h`

Per-NPC threat table component. Tracks cumulative threat from all attackers, providing WoW/FFXIV-style target priority. Server-authoritative.

```cpp
UCLASS(ClassGroup = (AI), meta = (BlueprintSpawnableComponent))
class FWAISYSTEM_API UFWThreatComponent : public UActorComponent
```

#### Methods

##### AddThreat

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Threat")
void AddThreat(AActor* Source, float Amount, EFWThreatReason Reason = EFWThreatReason::Damage);
```

Add threat from a source. Automatically inserts new entries and re-sorts the threat table.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Source` | `AActor*` | -- | The actor generating threat |
| `Amount` | `float` | -- | Amount of threat to add |
| `Reason` | `EFWThreatReason` | `Damage` | Reason for threat generation |

##### RemoveThreat

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Threat")
void RemoveThreat(AActor* Source);
```

Remove an actor from the threat table entirely.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Source` | `AActor*` | The actor to remove |

##### ClearAllThreat

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Threat")
void ClearAllThreat();
```

Clear the entire threat table. Typically called on NPC death or leash reset.

##### ModifyThreat

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Threat")
void ModifyThreat(AActor* Source, float Multiplier);
```

Multiply an actor's threat by a factor. Used for threat reduction abilities.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Source` | `AActor*` | The actor whose threat to modify |
| `Multiplier` | `float` | Factor to multiply by (e.g., 0.5 = halve threat) |

##### TransferThreat

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Threat")
void TransferThreat(AActor* From, AActor* To, float Percentage);
```

Transfer a percentage of threat from one actor to another. Used for taunt mechanics.

| Parameter | Type | Description |
|-----------|------|-------------|
| `From` | `AActor*` | Source actor to take threat from |
| `To` | `AActor*` | Target actor to give threat to |
| `Percentage` | `float` | Fraction to transfer (0.0 to 1.0) |

##### GetHighestThreat

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Threat")
AActor* GetHighestThreat() const;
```

**Returns:** The actor with the highest threat, or `nullptr` if the table is empty.

##### HasThreat

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Threat")
bool HasThreat() const;
```

**Returns:** `true` if the threat table has any entries.

##### GetThreatList

```cpp
const TArray<FFWThreatEntry>& GetThreatList() const;
```

**Returns:** The full threat table sorted by descending threat value.

!!! note "C++ Only"
    This method is not exposed to Blueprints. Use `GetHighestThreat()` and `GetThreatValue()` from Blueprints instead.

##### GetThreatValue

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Threat")
float GetThreatValue(AActor* Source) const;
```

**Returns:** The threat value for a specific actor, or `0.0` if not found.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Source` | `AActor*` | The actor to query |

#### Configuration Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ThreatDecayRate` | `float` | `2.0` | Threat lost per second when source is not attacking |
| `TauntThreatMultiplier` | `float` | `1.5` | Multiplier for taunt threat (applied to current top value) |
| `HealingThreatMultiplier` | `float` | `0.5` | Threat per point of healing (relative to damage) |
| `ProximityThreatRadius` | `float` | `0.0` | Range for initial proximity threat (0 = disabled) |
| `MaxThreatEntries` | `int32` | `40` | Maximum entries in the threat table |

#### Events

| Event | Signature | Description |
|-------|-----------|-------------|
| `OnThreatChanged` | `(const FFWThreatEvent& Event)` | Fired when threat is added or modified |
| `OnThreatTableEmpty` | `()` | Fired when the threat table becomes empty |
| `OnNewHighestThreat` | `(AActor* NewTarget)` | Fired when the highest-threat target changes |

---

### UFWLeashComponent

`UActorComponent` | Header: `Components/FWLeashComponent.h`

Soft leash component for NPC spawn-anchor behavior. Provides MMO-standard leash mechanics with two-zone distance falloff.

```cpp
UCLASS(ClassGroup = (AI), meta = (BlueprintSpawnableComponent))
class FWAISYSTEM_API UFWLeashComponent : public UActorComponent
```

#### Methods

##### SetSpawnAnchor

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Leash")
void SetSpawnAnchor(FVector Location, FRotator Rotation);
```

Override the spawn anchor location and rotation.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Location` | `FVector` | World location for the spawn anchor |
| `Rotation` | `FRotator` | Rotation for the spawn anchor |

##### GetSpawnLocation

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
FVector GetSpawnLocation() const;
```

**Returns:** The spawn anchor location.

##### GetSpawnRotation

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
FRotator GetSpawnRotation() const;
```

**Returns:** The spawn anchor rotation.

##### GetDistanceFromSpawn

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
float GetDistanceFromSpawn() const;
```

**Returns:** The owner's current distance from the spawn anchor in centimeters.

##### IsInSoftLeash

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
bool IsInSoftLeash() const;
```

**Returns:** `true` if the owner is between the soft and hard leash radius.

##### IsLeashTriggered

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
bool IsLeashTriggered() const;
```

**Returns:** `true` if the owner has exceeded the hard leash radius.

##### ShouldReset

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
bool ShouldReset() const;
```

**Returns:** `true` when leash is triggered AND no threat has been received for `ResetDelay` seconds.

##### GetPursuitSpeedModifier

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
float GetPursuitSpeedModifier() const;
```

**Returns:** Speed modifier -- `1.0` in normal range, reduced in soft zone (using `PursuitSpeedMultiplier`), `0.0` beyond hard leash.

##### CanLeash

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Leash")
bool CanLeash() const;
```

**Returns:** `true` if this NPC can leash. Returns `false` for bosses with `bCanLeash = false`.

##### ResetToSpawn

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Leash")
void ResetToSpawn();
```

Reset the NPC to its spawn point. Clears threat and optionally restores HP (based on `bResetHealthOnReturn`).

##### BeginLeashing

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Leash")
void BeginLeashing();
```

Mark that leash has started. Used for tracking the reset delay timer.

#### Configuration Properties

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

| Event | Signature | Description |
|-------|-----------|-------------|
| `OnLeashTriggered` | `(AActor* NPC, float Distance)` | Fired when the NPC exceeds the hard leash distance |
| `OnResetBegin` | `()` | Fired when the NPC begins resetting to spawn |
| `OnResetComplete` | `()` | Fired when the NPC completes its reset |

---

### UFWNpcDefinition

`UPrimaryDataAsset` | Header: `DataAssets/FWNpcDefinition.h`

Data-driven NPC configuration asset. One per NPC archetype.

```cpp
UCLASS(BlueprintType)
class FWAISYSTEM_API UFWNpcDefinition : public UPrimaryDataAsset
```

#### Properties

##### Identity

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `NpcId` | `FName` | -- | Unique identifier for this NPC type |
| `DisplayName` | `FText` | -- | Localized display name |
| `Category` | `EFWNpcCategory` | `Trash` | Classification category |
| `NpcClass` | `TSoftClassPtr<APawn>` | -- | Character class to spawn |

##### AI Behavior

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `BehaviorTree` | `TSoftObjectPtr<UBehaviorTree>` | -- | Behavior tree asset to run |
| `PatrolMode` | `EFWPatrolMode` | `None` | Patrol movement mode |
| `RoamRadius` | `float` | `1000.0` | Radius for random roam patrol (cm) |
| `AggroRadius` | `float` | `1500.0` | AI perception sight radius for aggro detection (cm) |

##### Leash

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `LeashRadius` | `float` | `0.0` | Max chase distance override (0 = use component default) |
| `SoftLeashRadius` | `float` | `0.0` | Soft leash distance override (0 = use component default) |
| `bCanLeash` | `bool` | `true` | Whether the NPC can leash (false for raid bosses) |

##### Threat

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ThreatDecayRate` | `float` | `0.0` | Threat decay rate override (0 = use component default) |
| `TauntThreatMultiplier` | `float` | `0.0` | Taunt multiplier override (0 = use component default) |

##### Combat

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `BaseLevel` | `int32` | `1` | Base level for this NPC type |
| `AbilityTags` | `FGameplayTagContainer` | -- | GAS ability tags to grant on spawn |
| `AttributeDefaults` | `TSoftObjectPtr<UDataTable>` | -- | Data table for base attribute defaults |

##### Scaling

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TierScaling` | `TMap<EFWDifficultyTier, FFWInstanceScalingConfig>` | -- | Per-difficulty-tier scaling overrides |

##### Faction

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `DefaultFactionId` | `FName` | -- | Faction ID to assign on spawn |

##### Spawning

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `RespawnTime` | `float` | `60.0` | Respawn time in seconds |
| `MaxActiveInstances` | `int32` | `1` | Max concurrent instances per spawn point |

#### Methods

##### GetPrimaryAssetId

```cpp
virtual FPrimaryAssetId GetPrimaryAssetId() const override;
```

**Returns:** `FPrimaryAssetId(TEXT("NpcDefinition"), NpcId)`

---

### UFWInstanceScalingLibrary

`UBlueprintFunctionLibrary` | Header: `Scaling/FWInstanceScaling.h`

Static blueprint function library for dungeon/raid scaling calculations.

```cpp
UCLASS()
class FWAISYSTEM_API UFWInstanceScalingLibrary : public UBlueprintFunctionLibrary
```

#### Methods

##### GetTierMultipliers

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Scaling")
static FFWInstanceScalingConfig GetTierMultipliers(EFWDifficultyTier Tier);
```

Get default scaling multipliers for a difficulty tier.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Tier` | `EFWDifficultyTier` | The difficulty tier to query |

**Returns:** `FFWInstanceScalingConfig` with default multipliers for the tier.

##### ComputePartyScaling

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Scaling")
static FFWInstanceScalingConfig ComputePartyScaling(
    int32 PlayerCount,
    const FFWInstanceScalingConfig& Base);
```

Compute party-scaled config based on player count. Applies additive per-player HP and damage scaling.

| Parameter | Type | Description |
|-----------|------|-------------|
| `PlayerCount` | `int32` | Number of players in the party |
| `Base` | `const FFWInstanceScalingConfig&` | Base scaling configuration |

**Returns:** `FFWInstanceScalingConfig` with adjusted multipliers.

##### ApplyScalingToNpc

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Scaling")
static void ApplyScalingToNpc(AActor* NpcActor, const FFWInstanceScalingConfig& Config);
```

Apply scaling configuration to an NPC actor. Modifies GAS attributes (MaxHealth, AttackPower) if FWGASSystem is available, otherwise stores scaling data for manual application.

| Parameter | Type | Description |
|-----------|------|-------------|
| `NpcActor` | `AActor*` | The NPC actor to scale |
| `Config` | `const FFWInstanceScalingConfig&` | Scaling configuration to apply |

##### GetEffectiveHpMultiplier

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Scaling")
static float GetEffectiveHpMultiplier(EFWDifficultyTier Tier, int32 PlayerCount);
```

Get effective HP multiplier for a tier and player count.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Tier` | `EFWDifficultyTier` | The difficulty tier |
| `PlayerCount` | `int32` | Number of players |

**Returns:** Combined HP multiplier as a `float`.

##### GetEffectiveDamageMultiplier

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Scaling")
static float GetEffectiveDamageMultiplier(EFWDifficultyTier Tier, int32 PlayerCount);
```

Get effective damage multiplier for a tier and player count.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Tier` | `EFWDifficultyTier` | The difficulty tier |
| `PlayerCount` | `int32` | Number of players |

**Returns:** Combined damage multiplier as a `float`.

---

### AFWNpcSpawnPoint

`AActor` | Header: `Spawning/FWNpcSpawnPoint.h`

Level-placed actor for specific NPC placement with respawn management.

```cpp
UCLASS(Blueprintable)
class FWAISYSTEM_API AFWNpcSpawnPoint : public AActor
```

#### Methods

##### SpawnNpc

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Spawning")
APawn* SpawnNpc();
```

Spawn an NPC from the assigned definition. Configures Threat and Leash components automatically.

**Returns:** The spawned `APawn*`, or `nullptr` if spawning fails or max instances reached.

##### StartRespawnTimer

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Spawning")
void StartRespawnTimer();
```

Start the respawn countdown using the effective respawn time.

##### CancelRespawnTimer

```cpp
UFUNCTION(BlueprintCallable, Category = "FW AI|Spawning")
void CancelRespawnTimer();
```

Cancel a pending respawn timer.

##### GetActiveCount

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Spawning")
int32 GetActiveCount() const;
```

**Returns:** The number of currently active (alive) spawned NPCs.

#### Configuration Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `NpcDefinition` | `TObjectPtr<UFWNpcDefinition>` | -- | What NPC type to spawn |
| `RespawnTime` | `float` | `0.0` | Respawn time override in seconds (0 = use definition default) |
| `InitialSpawnDelay` | `float` | `0.0` | Delay before first spawn on level load |
| `MaxInstances` | `int32` | `1` | Max concurrent NPCs from this spawn point |
| `WanderRadius` | `float` | `0.0` | Roam radius override |
| `PatrolPoints` | `TArray<FVector>` | -- | Optional patrol waypoints (local space) |
| `bSpawnOnBeginPlay` | `bool` | `true` | Whether to auto-spawn on BeginPlay |

---

### AFWNpcSpawnVolume

`AActor` | Header: `Spawning/FWNpcSpawnVolume.h`

Zone-based density spawner using a box volume. Spawns/despawns based on player proximity.

```cpp
UCLASS(Blueprintable)
class FWAISYSTEM_API AFWNpcSpawnVolume : public AActor
```

#### Configuration Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `SpawnTable` | `TObjectPtr<UFWSpawnTable>` | -- | Spawn table defining what NPCs to spawn |
| `MaxNpcs` | `int32` | `10` | Maximum NPC density within this volume |
| `SpawnInterval` | `float` | `5.0` | Seconds between spawn checks |
| `DespawnDistance` | `float` | `15000.0` | Despawn NPCs when no player is within this distance (cm) |
| `SpawnDistance` | `float` | `10000.0` | Only spawn if a player is within this distance (cm) |
| `bCheckNavmesh` | `bool` | `true` | Validate spawn points are on the navmesh |

---

### UFWSpawnTable

`UDataAsset` | Header: `DataAssets/FWSpawnTable.h`

Weighted list of NPC definitions for zone-based spawning.

```cpp
UCLASS(BlueprintType)
class FWAISYSTEM_API UFWSpawnTable : public UDataAsset
```

#### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Entries` | `TArray<FFWSpawnTableEntry>` | -- | Weighted list of NPC types |
| `MinLevel` | `int32` | `1` | Minimum level filter for this table |
| `MaxLevel` | `int32` | `100` | Maximum level filter for this table |

#### Methods

##### Roll

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Spawning")
UFWNpcDefinition* Roll() const;
```

Roll a weighted random pick from the spawn table.

**Returns:** The `UFWNpcDefinition` from the selected entry, or `nullptr` if the table is empty.

##### GetEntry

```cpp
UFUNCTION(BlueprintPure, Category = "FW AI|Spawning")
const FFWSpawnTableEntry& GetEntry(int32 Index) const;
```

Get the entry at a specific index for deterministic spawning.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Index` | `int32` | Index into the entries array |

**Returns:** Reference to the entry. Returns an empty entry if index is out of range.

---

## Behavior Tree Nodes

### Tasks

#### UFWBTTask_FindThreatTarget

Queries the NPC's threat component and sets the `TargetActor` blackboard key to the highest-threat actor.

| Property | Type | Description |
|----------|------|-------------|
| `TargetActorKey` | `FBlackboardKeySelector` | Blackboard key for the target actor |

**Result:** Succeeds if a target is found, fails if the threat table is empty.

#### UFWBTTask_MoveToTarget

Moves the NPC toward the `TargetActor` blackboard key. Checks leash component each tick and aborts if leash is triggered.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TargetActorKey` | `FBlackboardKeySelector` | -- | Blackboard key for the target actor |
| `AcceptanceRadius` | `float` | `100.0` | Distance to target before arrival |
| `bStopOnOverlap` | `bool` | `true` | Stop when overlapping target's collision |

#### UFWBTTask_ReturnToSpawn

Moves the NPC back to its spawn location. On arrival, calls `LeashComponent->ResetToSpawn()`.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `AcceptanceRadius` | `float` | `50.0` | Distance to spawn before arrival |

#### UFWBTTask_Patrol

Dual-mode patrol task supporting both waypoint and random-roam patterns.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `PatrolMode` | `EFWPatrolMode` | `RandomRoam` | Patrol movement mode |
| `WaitTimeMin` | `float` | `2.0` | Minimum wait time at each point (seconds) |
| `WaitTimeMax` | `float` | `5.0` | Maximum wait time at each point (seconds) |
| `RoamRadius` | `float` | `1000.0` | Radius for random roam |
| `AcceptanceRadius` | `float` | `100.0` | Acceptance radius for patrol point |
| `PatrolPoints` | `TArray<FVector>` | -- | Waypoint points (local space) |
| `PatrolIndexKey` | `FBlackboardKeySelector` | -- | Blackboard key for patrol index |

#### UFWBTTask_PlayAbility

Activates a GAS ability on the NPC's AbilitySystemComponent by gameplay tag. Guarded by `WITH_FWGASSYSTEM`.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `AbilityTag` | `FGameplayTag` | -- | Gameplay tag identifying the ability |
| `bWaitForEnd` | `bool` | `true` | Task remains InProgress until ability ends |

### Services

#### UFWBTService_UpdateThreat

Ticking service that queries AI Perception for newly sensed actors, adds proximity threat for hostiles, and updates blackboard keys.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TargetActorKey` | `FBlackboardKeySelector` | -- | Blackboard key for current threat target |
| `HasThreatKey` | `FBlackboardKeySelector` | -- | Blackboard key for whether threat exists |
| `ProximityThreatPerTick` | `float` | `1.0` | Proximity threat per tick for hostile actors |

#### UFWBTService_CheckLeash

Ticking service that monitors the leash component state and updates the `NpcState` blackboard key.

| Property | Type | Description |
|----------|------|-------------|
| `NpcStateKey` | `FBlackboardKeySelector` | Blackboard key for NPC state |

### Decorators

#### UFWBTDecorator_HasThreat

Checks whether the NPC's threat table has any entries.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `bQueryComponentDirectly` | `bool` | `true` | Query threat component or read from blackboard |
| `HasThreatKey` | `FBlackboardKeySelector` | -- | Blackboard key (used when not querying directly) |

#### UFWBTDecorator_IsLeashing

Checks the leash component to determine if the NPC should disengage.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `bIncludeSoftLeash` | `bool` | `false` | If true, triggers on soft leash zone too |

#### UFWBTDecorator_FactionAttitude

Checks the faction attitude between the NPC and a target actor. Uses FWFactionSystem when available, falls back to `IGenericTeamAgentInterface`.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `TargetActorKey` | `FBlackboardKeySelector` | -- | Blackboard key for the target actor |
| `RequiredAttitude` | `ETeamAttitude::Type` | `Hostile` | Required attitude for this decorator to pass |
