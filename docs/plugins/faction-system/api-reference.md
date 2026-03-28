---
title: API Reference - Faction System
---

# API Reference

Classes, methods, properties, structs, and enums for FWFactionSystem.

---

## UFWFactionComponent

Per-actor component managing faction membership, personal reputation, and the bridge to `IGenericTeamAgentInterface` for AI targeting. Place on any actor that needs faction identity -- players, NPCs, destructibles, etc.

**Inherits:** `UActorComponent`

### Replicated Properties

| Property | Type | Replication | Description |
|----------|------|-------------|-------------|
| `DefaultFactionId` | `FName` | None | Faction assigned on BeginPlay. Set in editor for NPCs. |
| `CurrentFactionId` | `FName` | All Clients | Runtime faction ID. Replicated to all clients. |
| `PersonalReputations` | `TArray<FFWPersonalReputation>` | Owning Client Only | Per-actor reputation overrides. |

### Query Functions (BlueprintPure)

| Function | Returns | Description |
|----------|---------|-------------|
| `GetCurrentFactionId()` | `FName` | Returns the current faction ID for this actor. |
| `GetGenericTeamId()` | `FGenericTeamId` | Returns the team ID for this actor's faction, for AI integration. |
| `GetPersonalReputation(FName FactionId)` | `float` | Returns the personal reputation score toward a specific faction. Returns `0.0` if no override exists. |
| `GetFactionPlayerData()` | `FFWFactionPlayerData` | Exports current faction state as a serializable struct (faction ID + all personal reputations). |

### Mutation Functions (BlueprintCallable, Server-Authoritative)

| Function | Parameters | Description |
|----------|------------|-------------|
| `SetFaction` | `FName NewFactionId` | Sets the actor's faction. Fires `OnFactionChanged`. |
| `ModifyPersonalReputation` | `FName FactionId, float Delta` | Modifies personal reputation toward a faction by a delta value. Fires `OnReputationChanged`. |
| `SetPersonalReputation` | `FName FactionId, float NewScore` | Sets personal reputation toward a faction to an absolute value. Fires `OnReputationChanged`. |
| `ApplyFactionPlayerData` | `FFWFactionPlayerData Data` | Applies loaded data from a persistence backend. Sets the primary faction and all personal reputation entries. |

### Static Functions

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `SolveTeamAttitude` | `AActor* SourceActor, AActor* TargetActor` | `ETeamAttitude::Type` | Attitude solver for `IGenericTeamAgentInterface`. Resolves attitude through the subsystem (personal rep first, then global matrix) and converts to `ETeamAttitude`. Returns `Neutral` if either actor lacks a component. |

### Server RPCs

| RPC | Parameters | Description |
|-----|------------|-------------|
| `Server_SetFaction` | `FName NewFactionId` | Client-to-server request for faction change. Validated. |
| `Server_ModifyPersonalReputation` | `FName FactionId, float Delta` | Client-to-server request for reputation modification. Validated. |

### Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnFactionChanged` | `FName OldFactionId, FName NewFactionId` | Fired when this actor's faction changes. |
| `OnReputationChanged` | `FName FactionId, float OldScore, float NewScore` | Fired when this actor's personal reputation toward any faction changes. |
| `OnAttitudeChanged` | `FFWAttitudeChangeEvent Event` | Fired when an attitude change affects this actor. |

---

## UFWFactionSubsystem

World subsystem managing global faction state: definition caching, reputation matrix, attitude resolution, component registry, and persistence.

**Inherits:** `UWorldSubsystem`

### Definition Access (BlueprintPure)

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `GetFactionDefinition` | `FName FactionId` | `UFWFactionDefinition*` | Returns the faction definition for a given FactionId, or `nullptr` if not found. |
| `GetFactionByTeamId` | `uint8 TeamId` | `UFWFactionDefinition*` | Returns the faction definition for a given TeamId, or `nullptr` if not found. |
| `GetAllFactions` | -- | `TArray<UFWFactionDefinition*>` | Returns all loaded faction definitions. |

### Reputation Matrix

#### Query Functions (BlueprintPure)

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `GetFactionReputation` | `FName SourceFactionId, FName TargetFactionId` | `float` | Returns the global reputation score between two factions. |
| `GetFactionAttitude` | `FName SourceFactionId, FName TargetFactionId` | `EFWFactionAttitude` | Returns the attitude between two factions based on the global reputation matrix. |
| `GetAttitudeBetweenActors` | `AActor* SourceActor, AActor* TargetActor` | `EFWFactionAttitude` | Returns the attitude between two actors. Checks personal reputation first, then falls back to the global matrix. |

#### Mutation Functions (Server-Authoritative)

| Function | Parameters | Description |
|----------|------------|-------------|
| `ModifyFactionReputation` | `FName SourceFactionId, FName TargetFactionId, float Delta, bool bPropagate = true` | Modifies the global reputation between two factions. If `bPropagate` is true, applies propagation rules to allied factions. |
| `SetFactionReputation` | `FName SourceFactionId, FName TargetFactionId, float NewValue` | Sets the global reputation between two factions to an absolute value. |

### Component Registry (BlueprintPure)

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `GetComponentsInFaction` | `FName FactionId` | `TArray<UFWFactionComponent*>` | Returns all faction components belonging to a specific faction. |

### Persistence

| Function | Parameters | Description |
|----------|------------|-------------|
| `SavePlayerFactionData` | `FString PlayerId, FFWFactionPlayerData Data` | Saves player faction data via the configured persistence handler. Fires `OnFactionDataSaved`. |
| `LoadPlayerFactionData` | `FString PlayerId` | Loads player faction data via the configured persistence handler. Fires `OnFactionDataLoaded`. |
| `GetPersistenceHandler()` | -- | Returns the active `UFWFactionPersistenceHandler`, or `nullptr` if none configured. |

### Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnFactionWarDeclared` | `FName FactionA, FName FactionB` | Fired when two factions cross into Hostile attitude. |
| `OnFactionAllianceFormed` | `FName FactionA, FName FactionB` | Fired when two factions cross into Allied attitude. |
| `OnGlobalAttitudeChanged` | `FFWAttitudeChangeEvent Event` | Fired on any global attitude change. |
| `OnFactionDataLoaded` | `EFWFactionOperationResult Result, FFWFactionPlayerData Data` | Fired when player data finishes loading. |
| `OnFactionDataSaved` | `EFWFactionOperationResult Result` | Fired when player data finishes saving. |

---

## UFWFactionDefinition

Primary data asset defining a faction. Create instances in the Content Browser and register with AssetManager as PrimaryAssetType `FactionDef`.

**Inherits:** `UPrimaryDataAsset`

### Properties

| Property | Type | Category | Description |
|----------|------|----------|-------------|
| `FactionId` | `FName` | Identity | Unique identifier used in code and API. |
| `DisplayName` | `FText` | Identity | Localized display name. |
| `Description` | `FText` | Identity | Localized lore text. |
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

Abstract base class for faction data persistence. Subclass this to implement save/load for your backend (REST API, SaveGame, cloud storage, etc.).

**Inherits:** `UObject` (Abstract, Blueprintable)

### Virtual Methods

| Method | Parameters | Description |
|--------|------------|-------------|
| `SavePlayerFactionData` | `FString PlayerId, FFWFactionPlayerData Data, FOnFactionSaveComplete OnComplete` | Override to implement save logic. Default implementation returns `Success`. |
| `LoadPlayerFactionData` | `FString PlayerId, FOnFactionLoadComplete OnComplete` | Override to implement load logic. Default implementation returns `Success`. |

---

## UFWFactionBlueprintLibrary

Static Blueprint helpers for common faction queries.

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `GetFactionComponent` | `AActor*` | `UFWFactionComponent*` | Finds the faction component on an actor (checks actor, pawn, and controller). |
| `AreActorsHostile` | `AActor*, AActor*` | `bool` | True if the two actors are hostile. |
| `AreActorsFriendly` | `AActor*, AActor*` | `bool` | True if the two actors are friendly. |
| `AreActorsSameFaction` | `AActor*, AActor*` | `bool` | True if both actors share the same faction. |
| `GetAttitudeDisplayName` | `EFWFactionAttitude` | `FText` | Human-readable name for an attitude level. |
| `GetAttitudeColor` | `EFWFactionAttitude` | `FLinearColor` | Color representing an attitude level. |

---

## UFWFactionSettings

Project-wide settings accessible at **Project Settings > Game > Faction System**.

| Property | Type | Description |
|----------|------|-------------|
| `PersistenceHandlerClass` | `TSoftClassPtr<UFWFactionPersistenceHandler>` | Class to instantiate for save/load. Leave empty to disable persistence. |

---

## Enums

### EFWFactionAttitude

Five-level attitude model.

| Value | ETeamAttitude Mapping | Description |
|-------|----------------------|-------------|
| `Allied` | Friendly | Strongest positive relationship |
| `Friendly` | Friendly | Positive relationship |
| `Neutral` | Neutral | No particular disposition |
| `Unfriendly` | Hostile | Negative relationship |
| `Hostile` | Hostile | Strongest negative relationship |

### EFWFactionOperationResult

Result codes for faction operations (persistence, membership changes).

| Value | Description |
|-------|-------------|
| `Success` | Operation completed successfully |
| `NetworkError` | Network communication failure |
| `Unauthorized` | Caller lacks permission |
| `NotFound` | Requested resource not found |
| `AlreadyMember` | Actor is already a member of the faction |
| `NotMember` | Actor is not a member of the faction |
| `InvalidRequest` | Request parameters are invalid |
| `Failed` | Generic failure |

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

Includes a `GetAttitudeFromScore(float Score)` method that resolves a numeric score into `EFWFactionAttitude`.

### FFWReputationPropagationRule

Rule for propagating reputation changes to allied factions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `TargetFaction` | `FPrimaryAssetId` | -- | The allied faction receiving propagated changes. |
| `FalloffMultiplier` | `float` | `0.5` | Multiplier applied to the original delta (0..1). |

### FFWFactionRelationship

Default relationship entry in a faction DataAsset.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `OtherFaction` | `FPrimaryAssetId` | -- | The other faction. |
| `BaseReputation` | `float` | `0.0` | Default reputation score toward that faction. |

### FFWFactionMembership

Faction membership entry for actors belonging to multiple factions.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `FactionId` | `FName` | -- | Faction identifier. |
| `bIsPrimary` | `bool` | `false` | Whether this is the primary faction. |

### FFWPersonalReputation

Per-actor personal reputation override toward a faction.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `FactionId` | `FName` | -- | Target faction. |
| `Score` | `float` | `0.0` | Reputation score. |

### FFWFactionPlayerData

Serialization struct for player faction data (save/load).

| Field | Type | Description |
|-------|------|-------------|
| `PrimaryFactionId` | `FName` | The player's primary faction. |
| `PersonalReputations` | `TArray<FFWPersonalReputation>` | All personal reputation entries. |

### FFWAttitudeChangeEvent

Event data when an attitude change occurs.

| Field | Type | Description |
|-------|------|-------------|
| `SourceFactionId` | `FName` | Source faction. |
| `TargetFactionId` | `FName` | Target faction. |
| `OldAttitude` | `EFWFactionAttitude` | Previous attitude. |
| `NewAttitude` | `EFWFactionAttitude` | New attitude. |
| `NewScore` | `float` | The reputation score after the change. |
