---
title: API Reference - Faction System
---

# API Reference

Complete C++ reference for all public classes, structs, enums, delegates, and utility functions in FWFactionSystem.

---

## UFWFactionComponent

`#include "Components/FWFactionComponent.h"`

Per-actor component managing faction membership, personal reputation, and the bridge to `IGenericTeamAgentInterface` for AI targeting. Place on any actor that needs faction identity -- players, NPCs, destructibles, etc.

**Class:** `UActorComponent` | **Specifiers:** `BlueprintSpawnableComponent`

### Properties

| Property | Type | Access | Replication | Description |
|----------|------|--------|-------------|-------------|
| `DefaultFactionId` | `FName` | `EditAnywhere`, `BlueprintReadOnly` | None | Faction assigned on BeginPlay. Set in editor for NPCs. |
| `CurrentFactionId` | `FName` | `VisibleInstanceOnly`, `BlueprintReadOnly` | `ReplicatedUsing = OnRep_FactionId` | Runtime faction ID. Replicated to all clients. |
| `PersonalReputations` | `TArray<FFWPersonalReputation>` | `VisibleInstanceOnly`, `BlueprintReadOnly` | `ReplicatedUsing = OnRep_PersonalReputations` | Per-actor reputation overrides. Replicated to owning client only. |

### BlueprintPure Functions

#### `GetCurrentFactionId`

```cpp
FName GetCurrentFactionId() const;
```

Returns the current faction ID for this actor.

---

#### `GetGenericTeamId`

```cpp
FGenericTeamId GetGenericTeamId() const;
```

Returns the `FGenericTeamId` for this actor's faction, looked up from the faction definition's `TeamId`.

---

#### `GetPersonalReputation`

```cpp
float GetPersonalReputation(FName FactionId) const;
```

Returns the personal reputation score toward a specific faction. Returns `0.0f` if no personal override exists.

| Parameter | Type | Description |
|-----------|------|-------------|
| `FactionId` | `FName` | The faction to query reputation for. |

---

#### `GetFactionPlayerData`

```cpp
FFWFactionPlayerData GetFactionPlayerData() const;
```

Exports the current faction state as an API-serializable struct containing the primary faction ID and all personal reputation entries.

---

### BlueprintCallable Functions

#### `SetFaction`

```cpp
void SetFaction(FName NewFactionId);
```

Sets the actor's faction. **Server-authoritative** (`BlueprintAuthorityOnly`). Fires `OnFactionChanged`.

| Parameter | Type | Description |
|-----------|------|-------------|
| `NewFactionId` | `FName` | The new faction to assign. |

---

#### `ModifyPersonalReputation`

```cpp
void ModifyPersonalReputation(FName FactionId, float Delta);
```

Modifies personal reputation toward a faction by a delta value. **Server-authoritative**. Fires `OnReputationChanged`.

| Parameter | Type | Description |
|-----------|------|-------------|
| `FactionId` | `FName` | Target faction. |
| `Delta` | `float` | Amount to add (positive) or subtract (negative). |

---

#### `SetPersonalReputation`

```cpp
void SetPersonalReputation(FName FactionId, float NewScore);
```

Sets personal reputation toward a faction to an absolute value. **Server-authoritative**. Fires `OnReputationChanged`.

---

#### `ApplyFactionPlayerData`

```cpp
void ApplyFactionPlayerData(const FFWFactionPlayerData& Data);
```

Applies loaded data from a persistence backend. **Server-authoritative**. Sets the primary faction and all personal reputation entries.

---

### Static Functions

#### `SolveTeamAttitude`

```cpp
static ETeamAttitude::Type SolveTeamAttitude(
    const AActor* SourceActor,
    const AActor* TargetActor
);
```

Static attitude solver for `IGenericTeamAgentInterface`. Finds `UFWFactionComponent` on both actors, resolves attitude through the subsystem (checking personal rep first, then global matrix), and converts the result to `ETeamAttitude`. Returns `ETeamAttitude::Neutral` if either actor lacks a component.

---

### Server RPCs

| RPC | Parameters | Description |
|-----|------------|-------------|
| `Server_SetFaction` | `FName NewFactionId` | Client-to-server request for faction change. Validated. |
| `Server_ModifyPersonalReputation` | `FName FactionId, float Delta` | Client-to-server request for reputation modification. Validated. |

---

### Events (BlueprintAssignable Delegates)

#### `OnFactionChanged`

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnFactionChanged,
    FName, OldFactionId,
    FName, NewFactionId
);
```

Fired when this actor's faction changes.

---

#### `OnReputationChanged`

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnReputationChanged,
    FName, FactionId,
    float, OldScore,
    float, NewScore
);
```

Fired when this actor's personal reputation toward any faction changes.

---

#### `OnAttitudeChanged`

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnAttitudeChanged,
    const FFWAttitudeChangeEvent&, Event
);
```

Fired when an attitude change affects this actor.

---

## UFWFactionSubsystem

`#include "Subsystems/FWFactionSubsystem.h"`

World subsystem managing global faction state: definition caching, reputation matrix, attitude resolution, component registry, and persistence.

**Class:** `UWorldSubsystem`

### Definition Access

#### `GetFactionDefinition`

```cpp
const UFWFactionDefinition* GetFactionDefinition(FName FactionId) const;
```

Returns the faction definition for a given FactionId, or `nullptr` if not found. `BlueprintPure`.

---

#### `GetFactionByTeamId`

```cpp
const UFWFactionDefinition* GetFactionByTeamId(uint8 TeamId) const;
```

Returns the faction definition for a given TeamId, or `nullptr` if not found. `BlueprintPure`.

---

#### `GetAllFactions`

```cpp
TArray<UFWFactionDefinition*> GetAllFactions() const;
```

Returns all loaded faction definitions. `BlueprintPure`.

---

### Reputation Matrix

#### `GetFactionReputation`

```cpp
float GetFactionReputation(FName SourceFactionId, FName TargetFactionId) const;
```

Returns the global reputation score between two factions. `BlueprintPure`.

---

#### `GetFactionAttitude`

```cpp
EFWFactionAttitude GetFactionAttitude(FName SourceFactionId, FName TargetFactionId) const;
```

Returns the attitude between two factions based on the global reputation matrix. `BlueprintPure`.

---

#### `GetAttitudeBetweenActors`

```cpp
EFWFactionAttitude GetAttitudeBetweenActors(AActor* SourceActor, AActor* TargetActor) const;
```

Returns the attitude between two actors. Checks personal reputation first, then falls back to the global matrix. `BlueprintPure`.

---

#### `ModifyFactionReputation`

```cpp
void ModifyFactionReputation(
    FName SourceFactionId,
    FName TargetFactionId,
    float Delta,
    bool bPropagate = true
);
```

Modifies the global reputation between two factions. **Server-authoritative**. If `bPropagate` is true, applies propagation rules to allied factions.

---

#### `SetFactionReputation`

```cpp
void SetFactionReputation(
    FName SourceFactionId,
    FName TargetFactionId,
    float NewValue
);
```

Sets the global reputation between two factions to an absolute value. **Server-authoritative**.

---

### Component Registry

#### `GetComponentsInFaction`

```cpp
TArray<UFWFactionComponent*> GetComponentsInFaction(FName FactionId) const;
```

Returns all faction components belonging to a specific faction. `BlueprintPure`.

---

### Persistence

#### `SavePlayerFactionData`

```cpp
void SavePlayerFactionData(const FString& PlayerId, const FFWFactionPlayerData& Data);
```

Saves player faction data via the configured persistence handler. Fires `OnFactionDataSaved` on completion.

---

#### `LoadPlayerFactionData`

```cpp
void LoadPlayerFactionData(const FString& PlayerId);
```

Loads player faction data via the configured persistence handler. Fires `OnFactionDataLoaded` on completion.

---

#### `GetPersistenceHandler`

```cpp
UFWFactionPersistenceHandler* GetPersistenceHandler() const;
```

Returns the active persistence handler, or `nullptr` if none configured. `BlueprintPure`.

---

### Subsystem Events

| Event | Signature | Description |
|-------|-----------|-------------|
| `OnFactionWarDeclared` | `(FName FactionA, FName FactionB)` | Fired when two factions cross into Hostile attitude. |
| `OnFactionAllianceFormed` | `(FName FactionA, FName FactionB)` | Fired when two factions cross into Allied attitude. |
| `OnGlobalAttitudeChanged` | `(const FFWAttitudeChangeEvent& Event)` | Fired on any global attitude change. |
| `OnFactionDataLoaded` | `(EFWFactionOperationResult Result, const FFWFactionPlayerData& Data)` | Fired when player data finishes loading. |
| `OnFactionDataSaved` | `(EFWFactionOperationResult Result)` | Fired when player data finishes saving. |

---

## UFWFactionDefinition

`#include "DataAssets/FWFactionDefinition.h"`

Primary data asset defining a faction. Create instances in the Content Browser and register with AssetManager as PrimaryAssetType `FactionDef`.

**Class:** `UPrimaryDataAsset` | **Specifiers:** `BlueprintType`

### Properties

| Property | Type | Category | Description |
|----------|------|----------|-------------|
| `FactionId` | `FName` | Identity | Unique identifier used in code and API. |
| `DisplayName` | `FText` | Identity | Localized display name. |
| `Description` | `FText` | Identity | Localized lore text. Multiline. |
| `Icon` | `TSoftObjectPtr<UTexture2D>` | Identity | Faction icon for UI. |
| `FactionColor` | `FLinearColor` | Identity | Color for debug/UI. Default: White. |
| `TeamId` | `uint8` | Identity | Maps to `FGenericTeamId` (0--254). Must be unique. 255 = NoTeam. |
| `FactionTags` | `FGameplayTagContainer` | Classification | Tags for categorization (e.g., `Faction.Type.Military`). |
| `bPlayerJoinable` | `bool` | Classification | Whether players can join through gameplay. |
| `bHidden` | `bool` | Classification | Whether hidden from UI listings. |
| `ReputationThresholds` | `FFWReputationThresholds` | Reputation | Threshold breakpoints for score-to-attitude conversion. |
| `DefaultRelationships` | `TArray<FFWFactionRelationship>` | Relationships | Default reputation values toward other factions. |
| `PropagationRules` | `TArray<FFWReputationPropagationRule>` | Relationships | Rules for propagating rep changes to allies. |

---

## UFWFactionPersistenceHandler

`#include "FWFactionPersistenceHandler.h"`

Abstract base class for faction data persistence. Override in your project to implement save/load for your backend.

**Class:** `UObject` | **Specifiers:** `Abstract, Blueprintable, EditInlineNew`

### Virtual Methods

#### `SavePlayerFactionData`

```cpp
virtual void SavePlayerFactionData(
    const FString& PlayerId,
    const FFWFactionPlayerData& Data,
    FOnFactionSaveComplete OnComplete
);
```

#### `LoadPlayerFactionData`

```cpp
virtual void LoadPlayerFactionData(
    const FString& PlayerId,
    FOnFactionLoadComplete OnComplete
);
```

Both methods have default no-op implementations that return `Success`.

---

## UFWFactionBlueprintLibrary

`#include "FWFactionBlueprintLibrary.h"`

Static Blueprint helpers for common faction queries.

| Function | Return Type | Description |
|----------|-------------|-------------|
| `GetFactionComponent(AActor*)` | `UFWFactionComponent*` | Finds the faction component on an actor (checks actor, pawn, and controller). |
| `AreActorsHostile(AActor*, AActor*)` | `bool` | True if the two actors are hostile. |
| `AreActorsFriendly(AActor*, AActor*)` | `bool` | True if the two actors are friendly. |
| `AreActorsSameFaction(AActor*, AActor*)` | `bool` | True if both actors share the same faction. |
| `GetAttitudeDisplayName(EFWFactionAttitude)` | `FText` | Human-readable name for an attitude level. |
| `GetAttitudeColor(EFWFactionAttitude)` | `FLinearColor` | Color representing an attitude level. |

---

## UFWFactionSettings

`#include "FWFactionSettings.h"`

Project-wide settings. Accessible at **Project Settings > Game > Faction System**.

**Class:** `UDeveloperSettings`

| Property | Type | Description |
|----------|------|-------------|
| `PersistenceHandlerClass` | `TSoftClassPtr<UFWFactionPersistenceHandler>` | Class to instantiate for save/load. Leave empty to disable persistence. |

---

## Enums

### EFWFactionAttitude

```cpp
UENUM(BlueprintType)
enum class EFWFactionAttitude : uint8
{
    Allied,
    Friendly,
    Neutral,
    Unfriendly,
    Hostile
};
```

Five-level attitude model. Maps to Unreal's three-state `ETeamAttitude`:

| FWFactionAttitude | ETeamAttitude |
|-------------------|---------------|
| Allied | Friendly |
| Friendly | Friendly |
| Neutral | Neutral |
| Unfriendly | Hostile |
| Hostile | Hostile |

---

### EFWFactionOperationResult

```cpp
UENUM(BlueprintType)
enum class EFWFactionOperationResult : uint8
{
    Success,
    NetworkError,
    Unauthorized,
    NotFound,
    AlreadyMember,
    NotMember,
    InvalidRequest,
    Failed
};
```

Result codes for faction operations (API calls, membership changes, persistence).

---

## Structs

### FFWReputationThresholds

Threshold breakpoints mapping reputation scores to attitudes.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `AlliedThreshold` | `float` | `75.0` | Score >= this is Allied. |
| `FriendlyThreshold` | `float` | `25.0` | Score >= this is Friendly. |
| `UnfriendlyThreshold` | `float` | `-25.0` | Score < this is Unfriendly. |
| `HostileThreshold` | `float` | `-75.0` | Score < this is Hostile. |

**Method:** `GetAttitudeFromScore(float Score)` -- Resolves a numeric score into `EFWFactionAttitude`.

---

### FFWReputationPropagationRule

Rule for propagating reputation changes to allied factions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `TargetFaction` | `FPrimaryAssetId` | -- | The allied faction receiving propagated changes. |
| `FalloffMultiplier` | `float` | `0.5` | Multiplier applied to the original delta (0..1). |

---

### FFWFactionRelationship

Default relationship entry in a faction DataAsset.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `OtherFaction` | `FPrimaryAssetId` | -- | The other faction. |
| `BaseReputation` | `float` | `0.0` | Default reputation score toward that faction. |

---

### FFWFactionMembership

Faction membership entry for actors belonging to multiple factions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `FactionId` | `FName` | -- | Faction identifier. |
| `bIsPrimary` | `bool` | `false` | Whether this is the primary faction. |

---

### FFWPersonalReputation

Per-actor personal reputation override toward a faction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `FactionId` | `FName` | -- | Target faction. |
| `Score` | `float` | `0.0` | Reputation score. |

---

### FFWFactionPlayerData

Serialization struct for player faction data (API save/load).

| Field | Type | Description |
|-------|------|-------------|
| `PrimaryFactionId` | `FName` | The player's primary faction. |
| `PersonalReputations` | `TArray<FFWPersonalReputation>` | All personal reputation entries. |

---

### FFWAttitudeChangeEvent

Event data when an attitude change occurs.

| Field | Type | Description |
|-------|------|-------------|
| `SourceFactionId` | `FName` | Source faction. |
| `TargetFactionId` | `FName` | Target faction. |
| `OldAttitude` | `EFWFactionAttitude` | Previous attitude. |
| `NewAttitude` | `EFWFactionAttitude` | New attitude. |
| `NewScore` | `float` | The reputation score after the change. |

---

## Utility Namespace: FWFactionUtils

```cpp
namespace FWFactionUtils
{
    ETeamAttitude::Type ConvertToTeamAttitude(EFWFactionAttitude Attitude);
    FName MakeRelationshipKey(FName SourceId, FName TargetId);
}
```

| Function | Description |
|----------|-------------|
| `ConvertToTeamAttitude` | Converts five-state faction attitude to UE's three-state `ETeamAttitude`. |
| `MakeRelationshipKey` | Makes a composite key from two faction IDs for the reputation matrix lookup. |
