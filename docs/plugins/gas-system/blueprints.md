---
title: Blueprints - GAS System
---

# Blueprint Integration

FWGASSystem exposes its complete API to Blueprints through components, delegates, and function libraries. This page covers the most commonly used Blueprint patterns.

---

## Adding Components

Add these components to your character Blueprint via the **Components** panel:

| Component | Purpose | Required |
|-----------|---------|----------|
| **FW Ability System Component** | Core GAS component with auto-granting | Yes |
| **FW GAS Core Component** | Event bridge for attributes, abilities, effects | Recommended |
| **FW Ability Input Binding Component** | Enhanced Input to ability activation bridge | If using input binding |
| **FW Combo Manager Component** | Combo chain management | If using combos |
| **FW Ability Queue Component** | Ability queuing during animations | If using queuing |
| **FW Link Anim Layers Component** | Anim layer linking from Game Features | If using anim layers |

---

## FW Ability System Component -- Blueprint Nodes

### Configuration Properties

Configure these in the Details panel of the component:

#### Granted Abilities

An array of `FFWAbilityInputMapping` entries. Each entry specifies:

- **Ability** -- The `UGameplayAbility` subclass to grant.
- **Level** -- The ability level (default: 1).
- **Input Action** -- The `UInputAction` to bind (optional).
- **Trigger Event** -- `Started` (fires once on press) or `Triggered` (fires each frame while held).

!!! tip "Trigger Event Selection"
    Use `Started` for discrete actions (melee attack, dodge roll, cast spell). Use `Triggered` only for continuous actions where the Input Action itself is configured to fire once. If in doubt, always use `Started`.

#### Granted Attributes

An array of `FFWAttributeSetDefinition` entries:

- **Attribute Set** -- The `UAttributeSet` subclass to create.
- **Initialization Data** -- An optional `DataTable` (row structure: `AttributeMetaData`) for setting initial attribute values.

#### Granted Effects

An array of `TSubclassOf<UGameplayEffect>` -- effects applied on initialization (e.g., passive regen, base stats).

#### Granted Ability Sets

An array of soft references to `UFWAbilitySet` DataAssets. Each set is loaded and granted on `InitAbilityActorInfo`.

### Runtime Methods

| Node | Category | Authority | Description |
|------|----------|-----------|-------------|
| **Give Ability Set** | Ability Sets | Any | Grant an AbilitySet at runtime, returns a handle. |
| **Clear Ability Set** | Ability Sets | Any | Remove a previously granted AbilitySet by handle. |

---

## FW GAS Core Component -- Blueprint Events

The Core Component is the primary event surface for ability system activity. Bind to these events in your Blueprint's Event Graph.

### Attribute Change Events

=== "Health"

    **On Health Change**

    | Parameter | Type | Description |
    |-----------|------|-------------|
    | Delta Value | `Float` | Positive for heal, negative for damage. |
    | Event Tags | `GameplayTagContainer` | Tags from the causing effect. |

=== "Stamina"

    **On Stamina Change**

    | Parameter | Type | Description |
    |-----------|------|-------------|
    | Delta Value | `Float` | Positive for regen, negative for cost. |
    | Event Tags | `GameplayTagContainer` | Tags from the causing effect. |

=== "Mana"

    **On Mana Change**

    | Parameter | Type | Description |
    |-----------|------|-------------|
    | Delta Value | `Float` | Positive for regen, negative for cost. |
    | Event Tags | `GameplayTagContainer` | Tags from the causing effect. |

=== "Generic"

    **On Attribute Change**

    | Parameter | Type | Description |
    |-----------|------|-------------|
    | Attribute | `GameplayAttribute` | The attribute that changed. |
    | Delta Value | `Float` | Amount of change. |
    | Event Tags | `GameplayTagContainer` | Tags from the causing effect. |

### Ability Lifecycle Events

| Event | Parameters | When It Fires |
|-------|------------|---------------|
| **On Ability Activated** | `Ability` | Any ability successfully activates. |
| **On Ability Ended** | `Ability` | Any ability ends (success or cancel). |
| **On Ability Failed** | `Ability`, `ReasonTags` | An ability fails to activate. |
| **On Ability Commit** | `Ability` | An ability commits (cost/cooldown applied). |

### Effect Events

| Event | Parameters | When It Fires |
|-------|------------|---------------|
| **On Gameplay Effect Added** | `AssetTags`, `GrantedTags`, `ActiveHandle` | A new gameplay effect is applied. |
| **On Gameplay Effect Removed** | `AssetTags`, `GrantedTags`, `ActiveHandle` | A gameplay effect expires or is removed. |
| **On Gameplay Effect Stack Change** | `AssetTags`, `GrantedTags`, `Handle`, `NewCount`, `OldCount` | An effect's stack count changes. |
| **On Gameplay Effect Time Change** | `AssetTags`, `GrantedTags`, `Handle`, `NewStartTime`, `NewDuration` | An effect's duration is refreshed. |

### Cooldown Events

| Event | Parameters | When It Fires |
|-------|------------|---------------|
| **On Cooldown Start** | `Ability`, `CooldownTags`, `TimeRemaining`, `Duration` | An ability enters cooldown. |
| **On Cooldown End** | `Ability`, `CooldownTag`, `Duration` | A cooldown expires. |

### Combat Events

| Event | Parameters | When It Fires |
|-------|------------|---------------|
| **On Damage** | `DamageAmount`, `SourceCharacter`, `DamageTags` | The actor takes damage. |
| **On Death** | (none) | Health reaches zero. |

### Advanced Events

| Event | Parameters | When It Fires |
|-------|------------|---------------|
| **On Pre Attribute Change** | `AttributeSet`, `Attribute`, `NewValue` | Before an attribute is modified (can react but not change). |
| **On Post Gameplay Effect Execute** | `Attribute`, `SourceActor`, `TargetActor`, `SourceTags`, `Payload` | After a gameplay effect modifies an attribute's base value. |
| **On Gameplay Tag Change** | `GameplayTag`, `NewTagCount` | A tag is added (count goes to 1) or removed (count goes to 0). |

---

## Core Component -- Blueprint Functions

### Attribute Queries

| Node | Return Type | Description |
|------|-------------|-------------|
| **Get Health** | `Float` | Current health value. |
| **Get Max Health** | `Float` | Maximum health value. |
| **Get Stamina** | `Float` | Current stamina value. |
| **Get Max Stamina** | `Float` | Maximum stamina value. |
| **Get Mana** | `Float` | Current mana value. |
| **Get Max Mana** | `Float` | Maximum mana value. |
| **Get Attribute Value** | `Float` | Base value of any attribute. |
| **Get Current Attribute Value** | `Float` | Final (modified) value of any attribute. |
| **Is Alive** | `Bool` | True if Health > 0. |

### Attribute Mutations

| Node | Authority | Description |
|------|-----------|-------------|
| **Set Attribute Value** | Any | Sets the base value of an attribute. |
| **Clamp Attribute Value** | Any | Clamps an attribute between min and max. |
| **Adjust Attribute For Max Change** | Any | Proportionally adjusts an attribute when its max changes. |

### Ability Management

| Node | Authority | Description |
|------|-----------|-------------|
| **Grant Ability** | Server Only | Grants a gameplay ability at a specified level. |
| **Clear Ability** | Server Only | Removes a granted ability by class. |
| **Clear Abilities** | Server Only | Removes multiple abilities by class array. |
| **Activate Ability By Class** | Any | Attempts to activate an ability by class. Returns the activated ability if it is a `UFWGameplayAbility`. |
| **Activate Ability By Tags** | Any | Activates a single ability matching the provided tags. |

### Ability Queries

| Node | Return Type | Description |
|------|-------------|-------------|
| **Is Using Ability By Class** | `Bool` | Whether an active ability matches the class. |
| **Is Using Ability By Tags** | `Bool` | Whether an active ability matches the tags. |
| **Get Active Abilities By Class** | `Array<UGameplayAbility>` | Currently active instances of the specified class. |
| **Get Active Abilities By Tags** | `Array<UGameplayAbility>` | Currently active instances matching the tags. |
| **Has Any Matching Gameplay Tags** | `Bool` | Whether the ASC has any of the specified tags. |
| **Has Matching Gameplay Tag** | `Bool` | Whether the ASC has a specific tag. |

---

## Combo System -- Blueprint Usage

### Setup

1. Add a **FW Combo Manager Component** to your character.
2. Create a melee ability inheriting from `UFWGameplayAbility`.
3. In your attack montage, add anim notifies:
    - **FW Combo Window Notify State** -- marks where combo input is accepted.
    - **FW Trigger Combo Notify** -- triggers the next combo step.

### Blueprint Nodes

| Node | Description |
|------|-------------|
| **Increment Combo** | Advances the combo index (only works if combo window is open). |
| **Reset Combo** | Resets combo index to 0. |
| **Activate Combo Ability** | Activates a combo ability by class (handles server/multicast replication). |

### Reading Combo State

| Property | Type | Description |
|----------|------|-------------|
| **Combo Index** | `Int32` | Current combo step (0-based). Read-only in Blueprint. |
| **Combo Window Opened** | `Bool` | Whether the combo input window is active. |

---

## Ability Queue -- Blueprint Usage

### Setup

1. Add a **FW Ability Queue Component** to your character.
2. In your ability montages, add **FW Ability Queue Notify State** where you want to accept queued input.
3. Alternatively, set `bEnableAbilityQueue = true` on the ability itself (less granular control).

### Key Properties

| Property | Type | Description |
|----------|------|-------------|
| **Ability Queue Enabled** | `Bool` | Master toggle for the queue system. |

---

## Blueprint Example: Health Bar Update

```
Event On Health Change (from FW GAS Core Component)
  |
  +-- Delta Value, Event Tags
  |
  v
Get Health --> Divide --> Get Max Health
  |
  v
Set Progress Bar Percent (Health / MaxHealth)
  |
  v
Branch (Delta Value < 0)
  |
  True --> Play Animation (Damage Flash)
  False --> Play Animation (Heal Flash)
```

---

## Blueprint Example: Cooldown Display

```
Event On Cooldown Start
  |
  +-- Ability, CooldownTags, TimeRemaining, Duration
  |
  v
Set Timer by Event (TimeRemaining)
  |
  v
Update Cooldown Overlay (TimeRemaining / Duration)
  |
  v
Event On Cooldown End
  |
  v
Clear Cooldown Overlay
```
