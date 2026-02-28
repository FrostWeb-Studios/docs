---
title: Quick Start - Faction System
---

# Quick Start

This guide gets you from zero to a working faction system in under 10 minutes. By the end, you will have two factions with a hostile relationship, NPCs that attack enemy-faction players, and reputation that changes at runtime.

---

## 1. Create Faction Definitions

Create two `UFWFactionDefinition` DataAssets in your Content Browser:

### Ironclad Vanguard

| Property | Value |
|----------|-------|
| FactionId | `Ironclad` |
| DisplayName | `Ironclad Vanguard` |
| TeamId | `1` |
| FactionColor | Blue |
| bPlayerJoinable | `true` |

### Shadow Covenant

| Property | Value |
|----------|-------|
| FactionId | `Shadow` |
| DisplayName | `Shadow Covenant` |
| TeamId | `2` |
| FactionColor | Red |
| bPlayerJoinable | `true` |

---

## 2. Configure Relationships

On the **Ironclad** DataAsset, add a `DefaultRelationships` entry:

| OtherFaction | BaseReputation |
|-------------|----------------|
| `FactionDef:DA_Faction_Shadow` | `-100.0` |

On the **Shadow** DataAsset, add the reverse:

| OtherFaction | BaseReputation |
|-------------|----------------|
| `FactionDef:DA_Faction_Ironclad` | `-100.0` |

With the default thresholds (Hostile < -75), both factions start as Hostile to each other.

---

## 3. Add Components to Actors

### Player Character

Add a `UFWFactionComponent` to your player character Blueprint:

```cpp
// In your character's constructor (C++)
FactionComponent = CreateDefaultSubobject<UFWFactionComponent>(TEXT("FactionComponent"));
```

Or simply add it in the Blueprint editor via **Add Component > FW Faction Component**.

Set the faction at runtime when the player joins:

```cpp
// Server-side, e.g., in GameMode after login
UFWFactionComponent* FactionComp = PlayerCharacter->FindComponentByClass<UFWFactionComponent>();
if (FactionComp)
{
    FactionComp->SetFaction(TEXT("Ironclad"));
}
```

### NPC Characters

Add a `UFWFactionComponent` to your NPC Blueprint and set `DefaultFactionId` to `Shadow` in the editor details panel.

---

## 4. Wire Up AI Perception

For NPCs to react to faction hostility, register the static attitude solver with your AI Controller:

```cpp
// In your AI Controller constructor
AMyAIController::AMyAIController()
{
    // Set the team attitude solver
    SetGenericTeamId(FGenericTeamId(0)); // Will be overridden by component
}
```

Override `GetTeamAttitudeTowards` to use the faction system:

```cpp
ETeamAttitude::Type AMyAIController::GetTeamAttitudeTowards(const AActor& Other) const
{
    return UFWFactionComponent::SolveTeamAttitude(GetPawn(), &Other);
}
```

!!! info "Automatic Registration"
    `SolveTeamAttitude` finds the `UFWFactionComponent` on both actors, queries the subsystem for their attitude, and converts the five-level `EFWFactionAttitude` to Unreal's three-state `ETeamAttitude`. If either actor lacks a component, it returns `Neutral`.

---

## 5. Test It

1. Place a player start and an NPC with an AI Controller in your level.
2. Ensure the player has `FactionId = Ironclad` and the NPC has `FactionId = Shadow`.
3. Press Play. The NPC's AI Perception should detect the player as Hostile.

---

## 6. Modify Reputation at Runtime

Grant reputation when a player completes a quest or kills an enemy:

=== "C++"

    ```cpp
    // Award 10 rep with Shadow faction
    FactionComp->ModifyPersonalReputation(TEXT("Shadow"), 10.0f);
    ```

=== "Blueprint"

    Call **Modify Personal Reputation** on the Faction Component node:

    - **Faction Id**: `Shadow`
    - **Delta**: `10.0`

Bind to the `OnReputationChanged` event to update UI:

```cpp
FactionComp->OnReputationChanged.AddDynamic(this, &AMyHUD::HandleReputationChanged);

void AMyHUD::HandleReputationChanged(FName FactionId, float OldScore, float NewScore)
{
    // Update reputation bar
}
```

---

## Next Steps

- [Configuration](configuration.md) -- Customize reputation thresholds and propagation rules.
- [API Reference](api-reference.md) -- Full documentation of all classes and functions.
- [Tutorials](tutorials.md) -- In-depth guide on setting up a faction war scenario.
