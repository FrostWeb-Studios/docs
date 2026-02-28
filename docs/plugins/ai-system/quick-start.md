---
title: Quick Start - FWAISystem
---

# Quick Start

Get an NPC with threat tracking, leashing, and patrol running in under 10 minutes.

---

## Prerequisites

- FWAISystem plugin [installed and enabled](installation.md)
- A Character or Pawn class for your NPC
- An AI Controller assigned to your NPC
- A Behavior Tree asset (or willingness to create one)

---

## Step 1: Create an NPC Definition Data Asset

The `UFWNpcDefinition` data asset is the central configuration for each NPC archetype.

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWNpcDefinition` as the class.
3. Name it `DA_ForestWolf` (or whatever fits your NPC).
4. Configure the following fields:

| Section | Field | Value |
|---------|-------|-------|
| Identity | NpcId | `ForestWolf` |
| Identity | DisplayName | `Forest Wolf` |
| Identity | Category | `Trash` |
| Identity | NpcClass | Your wolf Pawn Blueprint |
| AI Behavior | BehaviorTree | Your behavior tree asset |
| AI Behavior | PatrolMode | `Random Roam` |
| AI Behavior | RoamRadius | `1000` |
| AI Behavior | AggroRadius | `1500` |
| Spawning | RespawnTime | `30` |
| Spawning | MaxActiveInstances | `1` |

!!! tip "Start Simple"
    Leave Leash, Threat, Scaling, and Faction at their defaults for now. You can fine-tune them later through the [Configuration](configuration.md) guide.

---

## Step 2: Place a Spawn Point

1. In the Level Editor, search the **Place Actors** panel for `FWNpcSpawnPoint`.
2. Drag it into your level at the desired spawn location.
3. In the Details panel, set:
    - **NpcDefinition**: Select your `DA_ForestWolf` asset.
    - **bSpawnOnBeginPlay**: `true`
    - **MaxInstances**: `1`

The spawn point will show a billboard icon in the editor for easy identification.

---

## Step 3: Add Threat and Leash Components to Your NPC

Open your NPC's Blueprint or C++ class and add both components:

=== "Blueprint"

    1. Open your NPC Blueprint.
    2. Click **Add Component** and search for `FW Threat Component`. Add it.
    3. Click **Add Component** again and search for `FW Leash Component`. Add it.

=== "C++ (Constructor)"

    ```cpp
    #include "Components/FWThreatComponent.h"
    #include "Components/FWLeashComponent.h"

    AForestWolf::AForestWolf()
    {
        ThreatComponent = CreateDefaultSubobject<UFWThreatComponent>(TEXT("ThreatComponent"));
        LeashComponent = CreateDefaultSubobject<UFWLeashComponent>(TEXT("LeashComponent"));
    }
    ```

!!! info "Automatic Configuration"
    When spawned by `AFWNpcSpawnPoint`, the spawn point automatically configures the Threat and Leash components with values from the `UFWNpcDefinition` data asset. You do not need to manually set leash radii or threat decay rates if they are defined in the data asset.

---

## Step 4: Set Up the Behavior Tree

Create a basic behavior tree that uses the built-in FWAISystem nodes.

### Recommended Behavior Tree Structure

```
Root
  Selector
    |
    +-- Sequence [Decorator: IsLeashing]
    |     +-- Task: ReturnToSpawn
    |
    +-- Sequence [Decorator: HasThreat]
    |     +-- Service: UpdateThreat
    |     +-- Service: CheckLeash
    |     +-- Task: FindThreatTarget
    |     +-- Task: MoveToTarget
    |
    +-- Task: Patrol
```

### Adding Nodes

1. **Right-click** in the Behavior Tree editor to add nodes.
2. Under **Tasks**, you will find:
    - `FW Find Threat Target` -- Sets `TargetActor` blackboard key to highest-threat actor
    - `FW Move To Target` -- Moves toward the `TargetActor`, aborts on leash
    - `FW Return To Spawn` -- Walks back to spawn, calls `ResetToSpawn` on arrival
    - `FW Patrol` -- Random roam or waypoint patrol
3. Under **Services**, you will find:
    - `FW Update Threat` -- Adds proximity threat from perceived hostiles, updates blackboard
    - `FW Check Leash` -- Monitors leash state, updates `NpcState` blackboard key
4. Under **Decorators**, you will find:
    - `FW Has Threat` -- Gates combat subtree on threat table having entries
    - `FW Is Leashing` -- Gates leashing subtree on hard leash exceeded

### Blackboard Setup

Create a Blackboard asset with the following keys:

| Key Name | Type | Purpose |
|----------|------|---------|
| `TargetActor` | Object (Actor) | Current threat target |
| `bHasThreat` | Bool | Whether the threat table has entries |
| `NpcState` | Enum (EFWNpcState) | Current NPC behavior state |
| `PatrolIndex` | Int | Current waypoint index for patrol |
| `SpawnLocation` | Vector | Spawn anchor location |
| `HomeLocation` | Vector | Home/patrol center location |

---

## Step 5: Test in PIE

1. Press **Play in Editor** (PIE).
2. The spawn point should spawn your NPC automatically.
3. Approach the NPC -- if you have AI Perception configured with sight sensing, the `UpdateThreat` service will generate proximity threat.
4. Attack the NPC to generate damage threat. The NPC should chase you.
5. Run far enough away (beyond `LeashRadius`, default 4000 cm) -- the NPC should disengage and walk back to its spawn point.

!!! warning "AI Perception Required"
    The `FWBTService_UpdateThreat` relies on the NPC's `UAIPerceptionComponent` to detect nearby hostile actors. Ensure your AI Controller has perception configured with at least a **Sight** sense.

---

## Step 6: Verify Threat and Leash Behavior

To debug threat and leash at runtime:

1. Open the **Gameplay Debugger** (press `'` in PIE by default).
2. Select your NPC.
3. Check the **BehaviorTree** category to see which nodes are active.
4. Use `ShowDebug AI` in the console to visualize the behavior tree.

You can also print threat values in Blueprint by calling:

- `GetHighestThreat()` -- Returns the actor with the most threat
- `GetThreatValue(Actor)` -- Returns a specific actor's threat value
- `HasThreat()` -- Returns true if anyone is on the threat table
- `GetDistanceFromSpawn()` -- Returns distance from spawn anchor
- `IsInSoftLeash()` / `IsLeashTriggered()` -- Check leash state

---

## Next Steps

- [Configuration](configuration.md) -- Tune threat decay rates, leash radii, and scaling parameters
- [Tutorials](tutorials.md) -- Build a full MMO boss with threat, scaling, and faction aggro
- [Blueprints](blueprints.md) -- Complete Blueprint node reference
- [API Reference](api-reference.md) -- Full C++ API documentation
