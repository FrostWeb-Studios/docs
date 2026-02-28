---
title: API Reference - FWSkillSystem
---

# API Reference

Complete C++ API documentation for all FWSkillSystem classes, methods, structs, and enums.

---

## Enums

### EFWSkillType

Identifies each of the 15 skills in the system.

```cpp
UENUM(BlueprintType)
enum class EFWSkillType : uint8
{
    // Combat
    Warfare,
    Marksmanship,
    Magic,
    Bounty,

    // Gathering
    Woodcutting,
    Mining,
    Fishing,
    Husbandry,

    // Artisan
    Cooking,
    Woodworking,
    Crafting,
    Smithing,
    Alchemy,
    Construction,
    Artifice
};
```

### EFWSkillCategory

Groups skills into high-level categories.

```cpp
UENUM(BlueprintType)
enum class EFWSkillCategory : uint8
{
    Combat,
    Gathering,
    Artisan
};
```

---

## Structs

### FFWSkillData

Runtime state for a single skill on a character.

```cpp
USTRUCT(BlueprintType)
struct FFWSkillData
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    int32 Level;

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    float TotalXP;

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    float XPToNextLevel;
};
```

| Field | Type | Description |
|---|---|---|
| `Level` | `int32` | Current level of the skill |
| `TotalXP` | `float` | Total accumulated XP for this skill |
| `XPToNextLevel` | `float` | XP remaining until the next level-up |

### FFWSkillXPEvent

Event payload broadcast when XP is gained.

```cpp
USTRUCT(BlueprintType)
struct FFWSkillXPEvent
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    EFWSkillType SkillType;

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    float XPAmount;

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    float NewTotalXP;

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    int32 PreviousLevel;

    UPROPERTY(BlueprintReadOnly, Category = "Skill")
    int32 NewLevel;
};
```

| Field | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill that received XP |
| `XPAmount` | `float` | Amount of XP awarded |
| `NewTotalXP` | `float` | Total XP after the award |
| `PreviousLevel` | `int32` | Level before the XP award |
| `NewLevel` | `int32` | Level after the XP award |

---

## Components

### UFWSkillProgressionComponent

**Inherits:** `UActorComponent`
**Specifiers:** `BlueprintSpawnableComponent`

The primary component for tracking skill XP and levels on an actor. Attach to your player character or controller.

#### Static Methods

##### Get

```cpp
UFUNCTION(BlueprintCallable, Category = "Skill System", meta = (DefaultToSelf = "Actor"))
static UFWSkillProgressionComponent* Get(AActor* Actor);
```

Returns the `UFWSkillProgressionComponent` on the given actor, or `nullptr` if not found.

| Parameter | Type | Description |
|---|---|---|
| `Actor` | `AActor*` | The actor to search for the component |
| **Returns** | `UFWSkillProgressionComponent*` | The component instance, or `nullptr` |

---

#### BlueprintCallable Methods

##### AwardSkillXP

```cpp
UFUNCTION(BlueprintCallable, Category = "Skill System")
void AwardSkillXP(EFWSkillType SkillType, float Amount);
```

Awards XP to a specific skill. Triggers `OnSkillXPGained` and, if a level threshold is crossed, `OnSkillLevelUp`.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to award XP to |
| `Amount` | `float` | The amount of XP to award (must be positive) |

!!! note "Server Authority"
    This function performs an authority check. On replicated actors, only the server can award XP. Client-side calls are silently ignored.

##### AwardCombatXP

```cpp
UFUNCTION(BlueprintCallable, Category = "Skill System")
void AwardCombatXP(float Amount);
```

Awards XP distributed across combat skills (Warfare, Marksmanship, Magic, Bounty). Distribution logic is determined by the active combat context and skill definitions.

| Parameter | Type | Description |
|---|---|---|
| `Amount` | `float` | Total combat XP to distribute |

##### LoadSkillData

```cpp
UFUNCTION(BlueprintCallable, Category = "Skill System")
void LoadSkillData(const TMap<EFWSkillType, FFWSkillData>& InSkillData);
```

Loads previously saved skill data into the component. Use this when restoring a player's skill state from a save file or backend.

| Parameter | Type | Description |
|---|---|---|
| `InSkillData` | `const TMap<EFWSkillType, FFWSkillData>&` | Map of skill types to their saved data |

---

#### BlueprintPure Methods

##### GetSkillLevel

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
int32 GetSkillLevel(EFWSkillType SkillType) const;
```

Returns the current level of the specified skill.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `int32` | Current level (starts at 1) |

##### GetSkillTotalXP

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
float GetSkillTotalXP(EFWSkillType SkillType) const;
```

Returns the total accumulated XP for the specified skill.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `float` | Total accumulated XP |

##### GetSkillLevelProgress

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
float GetSkillLevelProgress(EFWSkillType SkillType) const;
```

Returns the fractional progress toward the next level as a value between 0.0 and 1.0.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `float` | Progress from 0.0 (just leveled) to 1.0 (about to level) |

##### GetXPToNextLevel

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
float GetXPToNextLevel(EFWSkillType SkillType) const;
```

Returns the amount of XP remaining until the next level-up.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `float` | XP remaining to next level. Returns 0 if at max level. |

##### GetMaxSkillLevel

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
int32 GetMaxSkillLevel(EFWSkillType SkillType) const;
```

Returns the maximum achievable level for the specified skill, as defined in its `UFWSkillDefinition`.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `int32` | Maximum level for this skill |

##### GetTotalLevel

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
int32 GetTotalLevel() const;
```

Returns the sum of all skill levels. Useful for displaying an overall character progression metric.

| Parameter | Type | Description |
|---|---|---|
| **Returns** | `int32` | Sum of all 15 skill levels |

##### GetSkillData

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
FFWSkillData GetSkillData(EFWSkillType SkillType) const;
```

Returns the full `FFWSkillData` struct for the specified skill.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `FFWSkillData` | Full skill data including level, total XP, and XP to next level |

##### GetSkillDefinition

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
UFWSkillDefinition* GetSkillDefinition(EFWSkillType SkillType) const;
```

Returns the `UFWSkillDefinition` data asset for the specified skill from the active database.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `UFWSkillDefinition*` | The definition asset, or `nullptr` if not found |

##### GetSkillMilestones

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
TArray<UFWSkillMilestoneDefinition*> GetSkillMilestones(EFWSkillType SkillType) const;
```

Returns all milestone definitions for the specified skill, sorted by required level.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `TArray<UFWSkillMilestoneDefinition*>` | Array of milestone definitions |

##### MeetsRequirement

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System")
bool MeetsRequirement(const UFWSkillRequirement* Requirement) const;
```

Evaluates a skill requirement against the current skill state. Returns `true` if the player meets the requirement.

| Parameter | Type | Description |
|---|---|---|
| `Requirement` | `const UFWSkillRequirement*` | The requirement to evaluate |
| **Returns** | `bool` | `true` if the requirement is satisfied |

---

#### Events (Delegates)

##### OnSkillXPGained

```cpp
UPROPERTY(BlueprintAssignable, Category = "Skill System")
FOnSkillXPGained OnSkillXPGained;

DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnSkillXPGained,
    EFWSkillType, SkillType,
    float, Amount,
    float, NewTotalXP
);
```

Broadcast whenever XP is awarded to any skill.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill that received XP |
| `Amount` | `float` | XP amount awarded |
| `NewTotalXP` | `float` | New total XP after the award |

##### OnSkillLevelUp

```cpp
UPROPERTY(BlueprintAssignable, Category = "Skill System")
FOnSkillLevelUp OnSkillLevelUp;

DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnSkillLevelUp,
    EFWSkillType, SkillType,
    int32, NewLevel
);
```

Broadcast whenever a skill levels up. If multiple levels are gained in a single XP award, this fires once per level gained.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill that leveled up |
| `NewLevel` | `int32` | The new level reached |

---

### UFWSkillMilestoneComponent

**Inherits:** `UActorComponent`

Tracks milestone progress and awards milestone rewards when skill levels are reached. Works in conjunction with `UFWSkillProgressionComponent`.

!!! info "Automatic Binding"
    `UFWSkillMilestoneComponent` automatically finds the `UFWSkillProgressionComponent` on the same actor during `BeginPlay` and binds to `OnSkillLevelUp` to check milestone conditions.

---

## Data Assets

### UFWSkillDefinition

**Inherits:** `UPrimaryDataAsset`

Defines configuration for a single skill.

| Property | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | Which skill this definition represents |
| `DisplayName` | `FText` | Localized display name |
| `Description` | `FText` | Localized skill description |
| `Icon` | `TSoftObjectPtr<UTexture2D>` | Skill icon for UI display |
| `Category` | `EFWSkillCategory` | Category this skill belongs to |
| `MaxLevel` | `int32` | Maximum level for this skill |
| `XPCurveOverride` | `TSoftObjectPtr<UFWSkillXPCurve>` | Optional per-skill XP curve (overrides the default) |
| `Milestones` | `TArray<UFWSkillMilestoneDefinition*>` | Milestones defined for this skill |

### UFWSkillDatabase

**Inherits:** `UDataAsset`

Collection of all skill definitions. Assigned in project settings or directly on the progression component.

| Property | Type | Description |
|---|---|---|
| `SkillDefinitions` | `TArray<UFWSkillDefinition*>` | Array of all skill definitions |

### UFWSkillXPCurve

**Inherits:** `UDataAsset`

Defines the XP required for each level. The curve is represented as an array of XP thresholds where each index corresponds to a level.

| Property | Type | Description |
|---|---|---|
| `LevelXPThresholds` | `TArray<float>` | Cumulative XP required per level. Index 0 = Level 1, Index 1 = Level 2, etc. |

### UFWSkillMilestoneDefinition

**Inherits:** `UDataAsset`

Defines a milestone that triggers at a specific skill level.

| Property | Type | Description |
|---|---|---|
| `MilestoneName` | `FText` | Display name for the milestone |
| `RequiredLevel` | `int32` | Skill level required to trigger this milestone |
| `Rewards` | `TArray<UFWSkillMilestoneReward*>` | Array of rewards granted when the milestone is reached |

### UFWSkillMilestoneReward

**Inherits:** `UDataAsset`

Base class for milestone reward payloads. Subclass this to define custom reward types (items, abilities, currency, etc.).

| Property | Type | Description |
|---|---|---|
| `RewardDescription` | `FText` | Localized description of the reward |

---

## Requirements

### UFWSkillRequirement

**Inherits:** `UObject`

Base class for skill-based content gating. Used by other systems (quest system, inventory system, ability system) to check whether a player meets specific skill prerequisites.

Create subclasses to define custom requirement logic. The base class provides the interface that `MeetsRequirement` on the progression component evaluates against.

| Method | Signature | Description |
|---|---|---|
| `IsMet` | `virtual bool IsMet(const UFWSkillProgressionComponent* Component) const` | Override in subclasses to implement custom requirement logic |
| `GetDescription` | `virtual FText GetDescription() const` | Returns a human-readable description of the requirement |

---

## Settings

### UFWSkillSystemSettings

**Inherits:** `UDeveloperSettings`

Project-wide settings accessible via **Edit > Project Settings > Plugins > FW Skill System**.

| Property | Type | Description |
|---|---|---|
| `SkillDatabase` | `TSoftObjectPtr<UFWSkillDatabase>` | Reference to the global skill database |
| `DefaultXPCurve` | `TSoftObjectPtr<UFWSkillXPCurve>` | Default XP curve used when a skill definition does not specify an override |
| `DefaultMaxLevel` | `int32` | Default max level when not specified per-skill |

---

## Blueprint Function Library

### UFWSkillTypeLibrary

**Inherits:** `UBlueprintFunctionLibrary`

Static helper functions for working with skill types in Blueprints and C++.

##### GetSkillCategory

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System|Utils")
static EFWSkillCategory GetSkillCategory(EFWSkillType SkillType);
```

Returns the category for the given skill type.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to categorize |
| **Returns** | `EFWSkillCategory` | The category (Combat, Gathering, or Artisan) |

##### GetSkillDisplayName

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System|Utils")
static FText GetSkillDisplayName(EFWSkillType SkillType);
```

Returns the localized display name for the given skill type.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to query |
| **Returns** | `FText` | Display name |

##### IsCombatSkill

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System|Utils")
static bool IsCombatSkill(EFWSkillType SkillType);
```

Returns `true` if the given skill belongs to the Combat category.

| Parameter | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill to check |
| **Returns** | `bool` | `true` for Warfare, Marksmanship, Magic, Bounty |

##### GetSkillDatabase

```cpp
UFUNCTION(BlueprintPure, Category = "Skill System|Utils")
static UFWSkillDatabase* GetSkillDatabase();
```

Returns the global `UFWSkillDatabase` as configured in project settings.

| Parameter | Type | Description |
|---|---|---|
| **Returns** | `UFWSkillDatabase*` | The global skill database, or `nullptr` if not configured |
