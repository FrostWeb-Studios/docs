---
title: Blueprints - FWAISystem
---

# Blueprint Reference

Complete reference for all FWAISystem Blueprint-exposed functions and event dispatchers.

---

## UFWThreatComponent

Add to any NPC to enable threat table tracking.

**Category:** `FW AI|Threat`

### Functions

#### Threat Management

| Node | Type | Description |
|------|------|-------------|
| **Add Threat** | Callable | Add threat from a source actor. Automatically inserts new entries and re-sorts. |
| **Remove Threat** | Callable | Remove an actor from the threat table entirely. |
| **Clear All Threat** | Callable | Clear the entire threat table (use on death/reset). |
| **Modify Threat** | Callable | Multiply an actor's threat by a factor (e.g., 0.5 to halve). |
| **Transfer Threat** | Callable | Transfer a percentage of threat from one actor to another. |

##### Add Threat

```
Add Threat
  Target: FW Threat Component (self)
  Source: Actor Object Reference
  Amount: Float
  Reason: EFWThreatReason (default: Damage)
```

##### Remove Threat

```
Remove Threat
  Target: FW Threat Component (self)
  Source: Actor Object Reference
```

##### Clear All Threat

```
Clear All Threat
  Target: FW Threat Component (self)
```

##### Modify Threat

```
Modify Threat
  Target: FW Threat Component (self)
  Source: Actor Object Reference
  Multiplier: Float
```

##### Transfer Threat

```
Transfer Threat
  Target: FW Threat Component (self)
  From: Actor Object Reference
  To: Actor Object Reference
  Percentage: Float (0.0 - 1.0)
```

#### Queries

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Highest Threat** | Pure | Actor Reference | Returns the actor with the highest threat. |
| **Has Threat** | Pure | Boolean | Returns true if anyone is on the threat table. |
| **Get Threat Value** | Pure | Float | Returns threat value for a specific actor. |

##### Get Highest Threat

```
Get Highest Threat
  Target: FW Threat Component (self)
  Return: Actor Object Reference (or None)
```

##### Has Threat

```
Has Threat
  Target: FW Threat Component (self)
  Return: Boolean
```

##### Get Threat Value

```
Get Threat Value
  Target: FW Threat Component (self)
  Source: Actor Object Reference
  Return: Float
```

### Event Dispatchers

| Event | Parameters | Description |
|-------|------------|-------------|
| **On Threat Changed** | `Event: FFWThreatEvent` | Fired when threat is added or modified. |
| **On Threat Table Empty** | -- | Fired when the threat table becomes empty. |
| **On New Highest Threat** | `NewTarget: Actor` | Fired when the highest-threat target changes. |

#### Binding Example

```
Event BeginPlay
  |
  Get Component by Class (FW Threat Component)
  |
  Bind Event to On Threat Changed
    Event -> Custom Event "HandleThreatChanged"
      |
      Print String: "Threat changed!"
      Break FFWThreatEvent
        Source -> Get Display Name
        Amount -> Append " generated " + Amount + " threat"
```

---

## UFWLeashComponent

Add to any NPC to enable spawn anchor and leash behavior.

**Category:** `FW AI|Leash`

### Functions

#### Spawn Anchor

| Node | Type | Description |
|------|------|-------------|
| **Set Spawn Anchor** | Callable | Override the spawn anchor location and rotation. |
| **Get Spawn Location** | Pure | Returns the spawn anchor location. |
| **Get Spawn Rotation** | Pure | Returns the spawn anchor rotation. |

##### Set Spawn Anchor

```
Set Spawn Anchor
  Target: FW Leash Component (self)
  Location: Vector
  Rotation: Rotator
```

#### Distance Queries

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Distance From Spawn** | Pure | Float | Current distance from spawn anchor (cm). |
| **Is In Soft Leash** | Pure | Boolean | True if between soft and hard leash radius. |
| **Is Leash Triggered** | Pure | Boolean | True if exceeded hard leash radius. |
| **Should Reset** | Pure | Boolean | True if leash triggered and no threat for ResetDelay. |
| **Get Pursuit Speed Modifier** | Pure | Float | Speed modifier (1.0 normal, reduced in soft zone). |
| **Can Leash** | Pure | Boolean | False for bosses with leashing disabled. |

#### Reset

| Node | Type | Description |
|------|------|-------------|
| **Reset To Spawn** | Callable | Reset NPC to spawn point. Clears threat, optionally restores HP. |
| **Begin Leashing** | Callable | Mark that leash has started (for reset delay tracking). |

### Event Dispatchers

| Event | Parameters | Description |
|-------|------------|-------------|
| **On Leash Triggered** | `NPC: Actor, Distance: Float` | Fired when the NPC exceeds the hard leash distance. |
| **On Reset Begin** | -- | Fired when the NPC begins resetting to spawn. |
| **On Reset Complete** | -- | Fired when the NPC completes its reset. |

---

## UFWInstanceScalingLibrary

Static Blueprint functions available globally (no component reference needed).

**Category:** `FW AI|Scaling`

### Functions

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Tier Multipliers** | Pure | `FFWInstanceScalingConfig` | Default scaling for a difficulty tier. |
| **Compute Party Scaling** | Pure | `FFWInstanceScalingConfig` | Party-scaled config from player count + base config. |
| **Apply Scaling To NPC** | Callable | -- | Apply scaling to an NPC actor's attributes. |
| **Get Effective HP Multiplier** | Pure | Float | Combined HP multiplier for tier + player count. |
| **Get Effective Damage Multiplier** | Pure | Float | Combined damage multiplier for tier + player count. |

##### Get Tier Multipliers

```
Get Tier Multipliers
  Tier: EFWDifficultyTier
  Return: FFWInstanceScalingConfig
```

##### Compute Party Scaling

```
Compute Party Scaling
  Player Count: Integer
  Base: FFWInstanceScalingConfig
  Return: FFWInstanceScalingConfig
```

##### Apply Scaling To NPC

```
Apply Scaling To NPC
  NPC Actor: Actor Object Reference
  Config: FFWInstanceScalingConfig
```

##### Get Effective HP Multiplier

```
Get Effective HP Multiplier
  Tier: EFWDifficultyTier
  Player Count: Integer
  Return: Float
```

##### Get Effective Damage Multiplier

```
Get Effective Damage Multiplier
  Tier: EFWDifficultyTier
  Player Count: Integer
  Return: Float
```

---

## AFWNpcSpawnPoint

Level-placed spawner actor. Add to levels via the Place Actors panel.

**Category:** `FW AI|Spawning`

### Functions

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Spawn NPC** | Callable | Pawn Reference | Spawn an NPC from the assigned definition. |
| **Start Respawn Timer** | Callable | -- | Start the respawn countdown. |
| **Cancel Respawn Timer** | Callable | -- | Cancel a pending respawn. |
| **Get Active Count** | Pure | Integer | Number of currently active spawned NPCs. |

---

## UFWSpawnTable

Data asset for weighted NPC spawning.

**Category:** `FW AI|Spawning`

### Functions

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Roll** | Pure | `UFWNpcDefinition` | Weighted random pick from the table. |
| **Get Entry** | Pure | `FFWSpawnTableEntry` | Entry at a specific index. |

---

## Behavior Tree Nodes

All FWAISystem behavior tree nodes appear in the Behavior Tree editor under their respective categories.

### Tasks

| Node | Category | Description |
|------|----------|-------------|
| **FW Find Threat Target** | Tasks | Sets TargetActor BB key to highest-threat actor. |
| **FW Move To Target** | Tasks | Moves toward TargetActor, aborts on leash. |
| **FW Return To Spawn** | Tasks | Walks back to spawn, resets on arrival. |
| **FW Patrol** | Tasks | Random roam or waypoint patrol. |
| **FW Play Ability** | Tasks | Activates a GAS ability by gameplay tag. |

### Services

| Node | Category | Description |
|------|----------|-------------|
| **FW Update Threat** | Services | Adds proximity threat from perception, updates BB keys. |
| **FW Check Leash** | Services | Monitors leash state, updates NpcState BB key. |

### Decorators

| Node | Category | Description |
|------|----------|-------------|
| **FW Has Threat** | Decorators | Gates subtree on threat table having entries. |
| **FW Is Leashing** | Decorators | Gates subtree on hard leash exceeded. |
| **FW Faction Attitude** | Decorators | Gates subtree on faction attitude check. |

---

## Struct Types in Blueprints

### FFWThreatEntry

| Pin | Type | Description |
|-----|------|-------------|
| `Source` | Actor (Weak Ref) | The actor generating threat |
| `Threat Value` | Float | Cumulative threat value |
| `Last Threat Time` | Float | World time of last threat event |

### FFWThreatEvent

| Pin | Type | Description |
|-----|------|-------------|
| `Source` | Actor | The actor that caused the event |
| `Amount` | Float | Amount of threat generated |
| `Reason` | `EFWThreatReason` | Reason for threat generation |

### FFWSpawnTableEntry

| Pin | Type | Description |
|-----|------|-------------|
| `NPC Class` | Soft Class (Pawn) | Character class to spawn |
| `NPC Definition` | `UFWNpcDefinition` | Data asset for the NPC |
| `Weight` | Float | Relative spawn weight |
| `Min Level` | Integer | Minimum level override |
| `Max Level` | Integer | Maximum level override |

### FFWInstanceScalingConfig

| Pin | Type | Description |
|-----|------|-------------|
| `Difficulty Tier` | `EFWDifficultyTier` | Difficulty tier |
| `HP Multiplier` | Float | Base HP multiplier |
| `Damage Multiplier` | Float | Base damage multiplier |
| `XP Multiplier` | Float | XP reward multiplier |
| `Expected Player Count` | Integer | Baseline player count |
| `Per Player HP Scale` | Float | Additional HP per extra player |
| `Per Player Damage Scale` | Float | Additional damage per extra player |
