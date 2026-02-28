---
title: Tutorials - FWAISystem
---

# Tutorials

## Creating Your First MMO Boss

This tutorial walks through creating a fully functional MMO dungeon boss with threat management, leash prevention, party scaling, and faction-based aggro. By the end, you will have a boss NPC that:

- Maintains a threat table and targets the highest-threat player
- Cannot be leashed (boss fights should not reset from kiting)
- Scales HP and damage based on difficulty tier and party size
- Only aggros hostile faction players
- Uses GAS abilities during combat (optional)

---

### Prerequisites

- FWAISystem installed and enabled
- A Character Blueprint for your boss (e.g., `BP_LichKing`)
- An AI Controller Blueprint assigned to your boss
- Basic familiarity with Behavior Trees

Optional:

- FWFactionSystem for faction aggro gating
- FWGASSystem for ability activation

---

### Step 1: Create the NPC Definition

Create a data asset for the Lich King boss:

1. Content Browser: right-click > **Miscellaneous > Data Asset** > `FWNpcDefinition`.
2. Name it `DA_LichKing`.
3. Configure as follows:

=== "Identity"

    | Field | Value |
    |-------|-------|
    | NpcId | `LichKing` |
    | DisplayName | `The Lich King` |
    | Category | `Boss` |
    | NpcClass | `BP_LichKing` |

=== "AI Behavior"

    | Field | Value |
    |-------|-------|
    | BehaviorTree | `BT_LichKing` (create in next step) |
    | PatrolMode | `None` |
    | AggroRadius | `2500` |

=== "Leash"

    | Field | Value |
    |-------|-------|
    | bCanLeash | `false` |

=== "Threat"

    | Field | Value |
    |-------|-------|
    | ThreatDecayRate | `1.0` (slow decay for boss fights) |
    | TauntThreatMultiplier | `2.0` (strong taunts for tank role) |

=== "Combat"

    | Field | Value |
    |-------|-------|
    | BaseLevel | `50` |
    | AbilityTags | `Ability.Boss.LichKing.Frost`, `Ability.Boss.LichKing.Shadow` |

=== "Scaling"

    Add entries to the TierScaling map:

    **Normal:**

    | Field | Value |
    |-------|-------|
    | HpMultiplier | `1.0` |
    | DamageMultiplier | `1.0` |
    | XpMultiplier | `1.0` |
    | ExpectedPlayerCount | `5` |
    | PerPlayerHpScale | `0.15` |
    | PerPlayerDamageScale | `0.05` |

    **Hard:**

    | Field | Value |
    |-------|-------|
    | HpMultiplier | `2.5` |
    | DamageMultiplier | `1.8` |
    | XpMultiplier | `2.0` |
    | ExpectedPlayerCount | `5` |
    | PerPlayerHpScale | `0.2` |
    | PerPlayerDamageScale | `0.08` |

    **Mythic:**

    | Field | Value |
    |-------|-------|
    | HpMultiplier | `5.0` |
    | DamageMultiplier | `3.0` |
    | XpMultiplier | `4.0` |
    | ExpectedPlayerCount | `5` |
    | PerPlayerHpScale | `0.25` |
    | PerPlayerDamageScale | `0.1` |

=== "Faction"

    | Field | Value |
    |-------|-------|
    | DefaultFactionId | `Undead` |

=== "Spawning"

    | Field | Value |
    |-------|-------|
    | RespawnTime | `600` (10 minutes) |
    | MaxActiveInstances | `1` |

---

### Step 2: Set Up Components on the Boss

Open `BP_LichKing` and add the following components:

1. **FW Threat Component** -- Configure:
    - `MaxThreatEntries`: `40` (supports large raids)
    - `ProximityThreatRadius`: `0` (boss does not proximity aggro; controlled by perception)

2. **FW Leash Component** -- Configure:
    - `bCanLeash`: `false` (boss cannot leash)

!!! info "Definition Overrides"
    The `AFWNpcSpawnPoint` will automatically apply `DA_LichKing`'s threat and leash settings to these components when the NPC spawns. You only need to set defaults here for testing without a spawn point.

---

### Step 3: Build the Behavior Tree

Create `BT_LichKing` with this structure:

```
Root
  Selector
    |
    +-- Sequence [Decorator: HasThreat]
    |     +-- Service: UpdateThreat
    |     |     TargetActorKey: TargetActor
    |     |     HasThreatKey: bHasThreat
    |     +-- Selector
    |           |
    |           +-- Sequence [Decorator: FactionAttitude (Hostile)]
    |           |     +-- Task: FindThreatTarget
    |           |     |     TargetActorKey: TargetActor
    |           |     +-- Task: PlayAbility
    |           |     |     AbilityTag: Ability.Boss.LichKing.Frost
    |           |     |     bWaitForEnd: true
    |           |     +-- Task: MoveToTarget
    |           |           TargetActorKey: TargetActor
    |           |           AcceptanceRadius: 150
    |           |
    |           +-- Task: FindThreatTarget
    |                 TargetActorKey: TargetActor
    |
    +-- Task: Wait (5 seconds, idle)
```

!!! tip "Ability Rotation"
    For more complex boss fights, use a `Sequence` of multiple `PlayAbility` tasks with different cooldown decorators to create an ability rotation. The boss will cycle through abilities while chasing the highest-threat target.

---

### Step 4: Apply Scaling at Runtime

When the dungeon instance initializes, apply scaling based on the difficulty tier and party size:

=== "Blueprint"

    ```
    Event: On Dungeon Initialized
      |
      Get Tier Multipliers (DifficultyTier)
        |
        Compute Party Scaling (PartySize, TierConfig)
          |
          For Each Loop (BossActors)
            |
            Apply Scaling To NPC (BossActor, ScaledConfig)
    ```

=== "C++"

    ```cpp
    void AMyDungeonGameMode::InitializeBoss(AActor* Boss, EFWDifficultyTier Tier, int32 PartySize)
    {
        FFWInstanceScalingConfig BaseConfig = UFWInstanceScalingLibrary::GetTierMultipliers(Tier);
        FFWInstanceScalingConfig ScaledConfig = UFWInstanceScalingLibrary::ComputePartyScaling(PartySize, BaseConfig);
        UFWInstanceScalingLibrary::ApplyScalingToNpc(Boss, ScaledConfig);
    }
    ```

---

### Step 5: Place the Spawn Point

1. In your dungeon level, place an `AFWNpcSpawnPoint`.
2. Set `NpcDefinition` to `DA_LichKing`.
3. Set `bSpawnOnBeginPlay` to `false` (boss spawns on dungeon trigger, not level load).
4. Set `RespawnTime` to `0` (use definition default of 600 seconds).

Trigger spawning from your dungeon logic:

```cpp
// When players enter the boss room
AFWNpcSpawnPoint* BossSpawn = FindBossSpawnPoint();
APawn* Boss = BossSpawn->SpawnNpc();

// Apply scaling
InitializeBoss(Boss, CurrentDifficultyTier, PartyMembers.Num());
```

---

### Step 6: Listen to Threat Events

Connect to threat events for boss-specific mechanics:

=== "Blueprint"

    ```
    Event BeginPlay
      |
      Get Component by Class (FW Threat Component)
      |
      Bind Event to On New Highest Threat
        -> Custom Event "OnTankSwap"
             |
             Print String: "New tank: " + NewTarget.GetDisplayName()
             // Trigger tank-swap mechanic
    ```

=== "C++"

    ```cpp
    void ALichKingCharacter::BeginPlay()
    {
        Super::BeginPlay();

        if (UFWThreatComponent* ThreatComp = FindComponentByClass<UFWThreatComponent>())
        {
            ThreatComp->OnNewHighestThreat.AddDynamic(this, &ALichKingCharacter::OnTankSwap);
            ThreatComp->OnThreatTableEmpty.AddDynamic(this, &ALichKingCharacter::OnEvade);
        }
    }

    void ALichKingCharacter::OnTankSwap(AActor* NewTarget)
    {
        // Trigger tank-swap debuff mechanic
        UE_LOG(LogTemp, Log, TEXT("New highest threat: %s"), *NewTarget->GetName());
    }

    void ALichKingCharacter::OnEvade(void)
    {
        // All players dead or out of range
        UE_LOG(LogTemp, Log, TEXT("Boss evading -- no threat targets"));
    }
    ```

---

### Step 7: Test the Boss

1. Play in Editor with multiple player windows (or use a listen server with multiple clients).
2. Enter the boss room to trigger the spawn.
3. Verify:
    - [x] Boss spawns at the spawn point
    - [x] Boss targets the player who deals the most damage
    - [x] Taunting transfers threat correctly
    - [x] Boss does NOT leash when players run far away
    - [x] Scaling applies correctly based on tier and party size
    - [x] Boss uses abilities during combat (if FWGASSystem is present)
    - [x] Boss resets threat when all players die

!!! success "Congratulations"
    You now have a fully functional MMO boss with production-quality threat management, instance scaling, and faction integration. Customize the behavior tree, add more abilities, and tune the scaling parameters to match your game's balance requirements.

---

## Next Steps

- [Configuration](configuration.md) -- Fine-tune all parameters
- [API Reference](api-reference.md) -- Detailed method documentation
- [FAQ](faq.md) -- Common questions and troubleshooting
