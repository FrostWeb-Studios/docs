---
title: API Reference - FWSkillSystem
---

# API Reference

Classes, methods, properties, structs, and enums for FWSkillSystem.

---

## Enums

### EFWSkillType

Identifies each of the 15 skills in the system.

| Value | Category |
|---|---|
| `Warfare` | Combat |
| `Marksmanship` | Combat |
| `Magic` | Combat |
| `Bounty` | Combat |
| `Woodcutting` | Gathering |
| `Mining` | Gathering |
| `Fishing` | Gathering |
| `Husbandry` | Gathering |
| `Cooking` | Artisan |
| `Woodworking` | Artisan |
| `Crafting` | Artisan |
| `Smithing` | Artisan |
| `Alchemy` | Artisan |
| `Construction` | Artisan |
| `Artifice` | Artisan |

### EFWSkillCategory

Groups skills into high-level categories.

| Value | Description |
|---|---|
| `Combat` | Skills advanced through combat encounters |
| `Gathering` | Skills advanced through resource collection |
| `Artisan` | Skills advanced through crafting and construction |

---

## Structs

### FFWSkillData

Runtime state for a single skill on a character.

| Field | Type | Description |
|---|---|---|
| `Level` | `int32` | Current level of the skill |
| `TotalXP` | `float` | Total accumulated XP for this skill |
| `XPToNextLevel` | `float` | XP remaining until the next level-up |

### FFWSkillXPEvent

Event payload broadcast when XP is gained.

| Field | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill that received XP |
| `XPAmount` | `float` | Amount of XP awarded |
| `NewTotalXP` | `float` | Total XP after the award |
| `PreviousLevel` | `int32` | Level before the XP award |
| `NewLevel` | `int32` | Level after the XP award |

### FFWSkillMilestoneContext

Context data passed to milestone reward handlers when a milestone is reached.

| Field | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill that triggered the milestone |
| `MilestoneLevel` | `int32` | The level at which the milestone was reached |
| `OwningActor` | `AActor*` | The actor that owns the progression component |

### FFWSkillRequirement

Defines a skill-level prerequisite for content gating.

| Field | Type | Description |
|---|---|---|
| `SkillType` | `EFWSkillType` | The skill being checked |
| `RequiredLevel` | `int32` | Minimum level required |

---

## Components

### UFWSkillProgressionComponent

**Inherits:** `UActorComponent`

The primary component for tracking skill XP and levels on an actor. Attach to your player character or controller.

#### Static Methods

| Method | Returns | Description |
|---|---|---|
| `Get(AActor*)` | `UFWSkillProgressionComponent*` | Returns the component on the given actor, or `nullptr` if not found |

#### XP API (BlueprintCallable)

| Method | Parameters | Description |
|---|---|---|
| `AwardSkillXP` | `EFWSkillType SkillType, float Amount` | Awards XP to a specific skill. Triggers `OnSkillXPGained` and, if a level threshold is crossed, `OnSkillLevelUp`. Server-authoritative -- client-side calls are ignored on replicated actors. |
| `AwardCombatXP` | `float Amount` | Awards XP distributed across combat skills (Warfare, Marksmanship, Magic, Bounty). Distribution is determined by the active combat context. Configurable via `XPPerDamage` and `KillBonusMultiplier`. |
| `LoadSkillData` | `TMap<EFWSkillType, FFWSkillData> InSkillData` | Loads previously saved skill data into the component. Use when restoring a player's skill state from a save file or backend. |

#### Query API (BlueprintPure)

| Method | Parameters | Returns | Description |
|---|---|---|---|
| `GetSkillLevel` | `EFWSkillType SkillType` | `int32` | Current level of the specified skill (starts at 1) |
| `GetSkillTotalXP` | `EFWSkillType SkillType` | `float` | Total accumulated XP for the specified skill |
| `GetSkillLevelProgress` | `EFWSkillType SkillType` | `float` | Fractional progress toward next level (0.0 to 1.0) |
| `GetXPToNextLevel` | `EFWSkillType SkillType` | `float` | XP remaining until next level-up. Returns 0 at max level. |
| `GetMaxSkillLevel` | `EFWSkillType SkillType` | `int32` | Maximum achievable level for the specified skill |
| `GetTotalLevel` | -- | `int32` | Sum of all 15 skill levels |
| `GetSkillData` | `EFWSkillType SkillType` | `FFWSkillData` | Full skill data struct (level, total XP, XP to next level) |
| `GetSkillDefinition` | `EFWSkillType SkillType` | `UFWSkillDefinition*` | The definition data asset for the specified skill |
| `GetSkillMilestones` | `EFWSkillType SkillType` | `TArray<UFWSkillMilestoneDefinition*>` | All milestone definitions for the skill, sorted by required level |

#### Requirements (BlueprintPure)

| Method | Parameters | Returns | Description |
|---|---|---|---|
| `MeetsRequirement` | `UFWSkillRequirement* Requirement` | `bool` | Evaluates a skill requirement against the current skill state |

#### Events

| Event | Parameters | Description |
|---|---|---|
| `OnSkillXPGained` | `EFWSkillType SkillType, float Amount, float NewTotalXP` | Broadcast whenever XP is awarded to any skill |
| `OnSkillLevelUp` | `EFWSkillType SkillType, int32 NewLevel` | Broadcast whenever a skill levels up. Fires once per level gained, even if multiple levels are gained in a single XP award. |

---

### UFWSkillMilestoneComponent

**Inherits:** `UActorComponent`

Tracks milestone progress and awards milestone rewards when skill levels are reached. Works in conjunction with `UFWSkillProgressionComponent`.

!!! info "Automatic Binding"
    `UFWSkillMilestoneComponent` automatically finds the `UFWSkillProgressionComponent` on the same actor during `BeginPlay` and binds to `OnSkillLevelUp` to check milestone conditions.

#### Milestone Reward Interface

The `IFWSkillMilestoneReward` interface allows you to implement custom milestone reward types. The milestone component calls `GrantReward` with an `FFWSkillMilestoneContext` when a milestone level is reached.

---

## Data Assets

### UFWSkillDefinition

**Inherits:** `UPrimaryDataAsset`

Defines configuration for a single skill. Create instances in the Content Browser.

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

Defines the XP required for each level using an RS-style formula. Supports per-skill overrides via `UFWSkillDefinition`.

| Property | Type | Description |
|---|---|---|
| `LevelXPThresholds` | `TArray<float>` | Cumulative XP required per level. Index 0 = Level 1, Index 1 = Level 2, etc. |

#### Configuration Parameters

The XP curve formula is influenced by the following parameters, configurable in Project Settings:

| Parameter | Type | Description |
|---|---|---|
| `GrindFactor` | `float` | Controls the steepness of the XP curve. Higher values = more XP required at higher levels. |
| `CompressionFactor` | `float` | Compresses or expands the overall XP requirements across all levels. |
| `PreviousMaxLevel` | `int32` | Used for curve recalculation when the max level cap changes. |

### UFWSkillMilestoneDefinition

**Inherits:** `UDataAsset`

Defines a milestone that triggers at a specific skill level.

| Property | Type | Description |
|---|---|---|
| `MilestoneName` | `FText` | Display name for the milestone |
| `RequiredLevel` | `int32` | Skill level required to trigger this milestone |
| `Rewards` | `TArray<IFWSkillMilestoneReward*>` | Array of rewards granted when the milestone is reached |

---

## Skill Requirements

### UFWSkillRequirement

**Inherits:** `UObject`

Base class for skill-based content gating. Used by other systems (quest system, inventory system, ability system) to check whether a player meets specific skill prerequisites.

| Method | Returns | Description |
|---|---|---|
| `IsMet(UFWSkillProgressionComponent*)` | `bool` | Override in subclasses to implement custom requirement logic |
| `GetDescription()` | `FText` | Returns a human-readable description of the requirement |

---

## Settings

### UFWSkillSystemSettings

Project-wide settings accessible via **Edit > Project Settings > Plugins > FW Skill System**.

| Property | Type | Description |
|---|---|---|
| `SkillDatabase` | `TSoftObjectPtr<UFWSkillDatabase>` | Reference to the global skill database |
| `DefaultXPCurve` | `TSoftObjectPtr<UFWSkillXPCurve>` | Default XP curve used when a skill definition does not specify an override |
| `DefaultMaxLevel` | `int32` | Default max level when not specified per-skill |
| `GrindFactor` | `float` | Controls the steepness of the XP curve |
| `CompressionFactor` | `float` | Compresses or expands overall XP requirements |
| `PreviousMaxLevel` | `int32` | Used for curve recalculation when max level cap changes |
| `XPPerDamage` | `float` | Base XP awarded per point of combat damage |
| `KillBonusMultiplier` | `float` | Multiplier applied to XP when a kill is scored |

---

## Blueprint Function Library

### UFWSkillTypeLibrary

Static helper functions for working with skill types in Blueprints and C++.

| Function | Parameters | Returns | Description |
|---|---|---|---|
| `GetSkillCategory` | `EFWSkillType SkillType` | `EFWSkillCategory` | Returns the category for the given skill type |
| `GetSkillDisplayName` | `EFWSkillType SkillType` | `FText` | Returns the localized display name for the given skill type |
| `IsCombatSkill` | `EFWSkillType SkillType` | `bool` | Returns `true` for Warfare, Marksmanship, Magic, Bounty |
| `GetSkillDatabase` | -- | `UFWSkillDatabase*` | Returns the global skill database as configured in project settings |
