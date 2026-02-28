---
title: Blueprints - FWSkillSystem
---

# Blueprint Reference

All Blueprint-exposed nodes in FWSkillSystem, organized by component and category.

---

## UFWSkillProgressionComponent

### Callable Functions

These nodes perform actions and should be connected to execution pins.

#### Award Skill XP

Awards XP to a specific skill on this component's owning actor.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill receives the XP |
| **Amount** | `Float` | How much XP to award |

!!! note "Authority Required"
    This node only executes on the server. Calling it on a client with no authority produces no effect.

---

#### Award Combat XP

Distributes XP across combat skills (Warfare, Marksmanship, Magic, Bounty) based on the active combat context.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Amount** | `Float` | Total combat XP to distribute |

---

#### Load Skill Data

Loads a map of skill data into the component, replacing current state. Use this when restoring from a save file or backend response.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **In Skill Data** | `Map of EFWSkillType to FFWSkillData` | The saved skill data to load |

---

### Pure Functions

These nodes return values and do not have execution pins.

#### Get Skill Level

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `Integer` | Current level of the skill |

---

#### Get Skill Total XP

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `Float` | Total accumulated XP |

---

#### Get Skill Level Progress

Returns a normalized value (0.0 to 1.0) representing progress toward the next level. Ideal for driving progress bars.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `Float` | 0.0 (just leveled) to 1.0 (about to level) |

---

#### Get XP To Next Level

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `Float` | XP remaining until next level. Returns 0 at max level. |

---

#### Get Max Skill Level

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `Integer` | Maximum achievable level for this skill |

---

#### Get Total Level

Returns the sum of all 15 skill levels.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Return Value** | `Integer` | Combined level across all skills |

---

#### Get Skill Data

Returns the full skill data struct for the specified skill.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `FFWSkillData` | Struct containing Level, TotalXP, and XPToNextLevel |

---

#### Get Skill Definition

Returns the data asset definition for the specified skill.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `FW Skill Definition` | The skill definition asset, or None |

---

#### Get Skill Milestones

Returns all milestone definitions for the specified skill.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Skill Type** | `EFWSkillType` | Which skill to query |
| **Return Value** | `Array of FW Skill Milestone Definition` | Sorted by required level |

---

#### Meets Requirement

Evaluates a skill requirement object against the current skill state.

| Pin | Type | Description |
|---|---|---|
| **Target** | `FW Skill Progression Component` | The component reference |
| **Requirement** | `FW Skill Requirement` | The requirement to check |
| **Return Value** | `Boolean` | True if the requirement is met |

---

### Static Functions

#### Get (Skill Progression Component)

Finds the `UFWSkillProgressionComponent` on the given actor.

| Pin | Type | Description |
|---|---|---|
| **Actor** | `Actor` | The actor to search |
| **Return Value** | `FW Skill Progression Component` | The component, or None |

---

### Events

#### On Skill XP Gained

Fires whenever XP is awarded to any skill.

| Pin | Type | Description |
|---|---|---|
| **Skill Type** | `EFWSkillType` | Which skill received XP |
| **Amount** | `Float` | XP amount awarded |
| **New Total XP** | `Float` | Updated total XP for this skill |

---

#### On Skill Level Up

Fires whenever a skill gains a level. If multiple levels are gained in a single award, this fires once per level.

| Pin | Type | Description |
|---|---|---|
| **Skill Type** | `EFWSkillType` | Which skill leveled up |
| **New Level** | `Integer` | The new level reached |

---

## UFWSkillTypeLibrary

Static Blueprint function library nodes. These appear under the **Skill System | Utils** category in the Blueprint action menu.

### Get Skill Category

| Pin | Type | Description |
|---|---|---|
| **Skill Type** | `EFWSkillType` | The skill to categorize |
| **Return Value** | `EFWSkillCategory` | Combat, Gathering, or Artisan |

---

### Get Skill Display Name

| Pin | Type | Description |
|---|---|---|
| **Skill Type** | `EFWSkillType` | The skill to query |
| **Return Value** | `Text` | Localized display name |

---

### Is Combat Skill

| Pin | Type | Description |
|---|---|---|
| **Skill Type** | `EFWSkillType` | The skill to check |
| **Return Value** | `Boolean` | True for Warfare, Marksmanship, Magic, Bounty |

---

### Get Skill Database

Returns the global skill database from project settings.

| Pin | Type | Description |
|---|---|---|
| **Return Value** | `FW Skill Database` | The global database, or None |

---

## Blueprint Usage Patterns

### Pattern: XP Award on Item Pickup

Connect your item pickup event to **Award Skill XP** on the player's Skill Progression Component:

```
[On Item Collected] --> [Get Player Character] --> [Get (Skill Progression)]
    --> [Award Skill XP: Mining, 50.0]
```

### Pattern: Skill-Gated Door

Use **Meets Requirement** to check if the player can pass through:

```
[On Interact] --> [Get (Skill Progression)] --> [Meets Requirement: SkillReq_Mining30]
    --> [Branch]
        True --> [Open Door]
        False --> [Show "Requires Mining Level 30" UI]
```

### Pattern: XP Progress Bar

Bind a progress bar widget to the **Get Skill Level Progress** node:

```
[Tick or Timer] --> [Get (Skill Progression)]
    --> [Get Skill Level Progress: Mining]
    --> [Set Percent on ProgressBar_Mining]
```

!!! tip "Event-Driven UI Updates"
    Rather than polling on Tick, bind to **On Skill XP Gained** and update the progress bar only when XP changes. This is more efficient and avoids unnecessary widget updates.

### Pattern: Skill Level Display

Combine **Get Skill Level** with **Get Skill Display Name** for UI text:

```
[On Skill XP Gained] --> [Get Skill Display Name: SkillType]
    --> [Format Text: "{Name}: Level {Level}"]
    <-- [Get Skill Level: SkillType]
    --> [Set Text on SkillLabel]
```
