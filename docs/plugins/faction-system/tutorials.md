---
title: Tutorials - Faction System
---

# Tutorial: Setting Up a Faction War

This tutorial walks through creating a complete faction war scenario where two opposing factions fight for territory. NPCs attack hostile-faction players on sight, reputation changes propagate to allies, and players can shift allegiances through gameplay.

---

## Overview

By the end of this tutorial you will have:

- Three faction definitions with configured relationships
- AI NPCs that attack hostile-faction players via AI Perception
- Reputation propagation rules so allied factions share standing
- A reputation reward system tied to killing enemy NPCs
- Custom thresholds controlling when factions flip between war and peace

---

## Step 1: Define Your Factions

Create three `UFWFactionDefinition` DataAssets in `/Game/Data/Factions/`:

### DA_Faction_Ironclad

| Property | Value |
|----------|-------|
| FactionId | `Ironclad` |
| DisplayName | `Ironclad Vanguard` |
| TeamId | `1` |
| FactionColor | `(R=0.2, G=0.4, B=0.9, A=1.0)` |
| bPlayerJoinable | `true` |
| FactionTags | `Faction.Type.Military` |

### DA_Faction_Shadow

| Property | Value |
|----------|-------|
| FactionId | `Shadow` |
| DisplayName | `Shadow Covenant` |
| TeamId | `2` |
| FactionColor | `(R=0.7, G=0.1, B=0.1, A=1.0)` |
| bPlayerJoinable | `true` |
| FactionTags | `Faction.Type.Military` |

### DA_Faction_Merchants

| Property | Value |
|----------|-------|
| FactionId | `Merchants` |
| DisplayName | `Merchant Confederacy` |
| TeamId | `3` |
| FactionColor | `(R=0.9, G=0.8, B=0.2, A=1.0)` |
| bPlayerJoinable | `false` |
| bHidden | `false` |
| FactionTags | `Faction.Type.Civilian` |

---

## Step 2: Configure Relationships

### Ironclad Relationships

| Other Faction | Base Reputation | Starting Attitude |
|--------------|-----------------|-------------------|
| Shadow | `-100.0` | Hostile |
| Merchants | `50.0` | Friendly |

### Shadow Relationships

| Other Faction | Base Reputation | Starting Attitude |
|--------------|-----------------|-------------------|
| Ironclad | `-100.0` | Hostile |
| Merchants | `-10.0` | Neutral |

### Merchants Relationships

| Other Faction | Base Reputation | Starting Attitude |
|--------------|-----------------|-------------------|
| Ironclad | `50.0` | Friendly |
| Shadow | `-10.0` | Neutral |

This creates the war dynamic: Ironclad and Shadow are mutually hostile, Merchants are friendly to Ironclad and neutral to Shadow.

---

## Step 3: Set Up Reputation Propagation

On the **Ironclad** DataAsset, add Propagation Rules:

| Target Faction | Falloff Multiplier |
|---------------|-------------------|
| Merchants | `0.5` |

On the **Shadow** DataAsset, add Propagation Rules:

| Target Faction | Falloff Multiplier |
|---------------|-------------------|
| (none) | -- |

Now when a player gains reputation with Ironclad, 50% of that gain also applies to Merchants. Shadow has no allies, so its reputation changes do not propagate.

---

## Step 4: Customize Reputation Thresholds

To make the faction war feel dynamic, customize thresholds so that peace is possible but hard-won.

### Ironclad Thresholds

| Threshold | Value | Notes |
|-----------|-------|-------|
| Allied | `90.0` | Very hard to achieve full alliance |
| Friendly | `25.0` | Standard |
| Unfriendly | `-25.0` | Standard |
| Hostile | `-50.0` | Easier to become hostile (military faction) |

### Shadow Thresholds

| Threshold | Value | Notes |
|-----------|-------|-------|
| Allied | `90.0` | Nearly impossible -- they trust no one |
| Friendly | `40.0` | Harder to befriend |
| Unfriendly | `-15.0` | Quick to distrust |
| Hostile | `-40.0` | Very quick to become hostile |

---

## Step 5: Create the AI Controller

Create a C++ AI Controller that uses the faction system for attitude solving.

```cpp
// FactionAIController.h
#pragma once

#include "CoreMinimal.h"
#include "AIController.h"
#include "FactionAIController.generated.h"

UCLASS()
class AFactionAIController : public AAIController
{
    GENERATED_BODY()

public:
    AFactionAIController();

    virtual ETeamAttitude::Type GetTeamAttitudeTowards(const AActor& Other) const override;
};
```

```cpp
// FactionAIController.cpp
#include "FactionAIController.h"
#include "Components/FWFactionComponent.h"

AFactionAIController::AFactionAIController()
{
    // AI Perception uses this to determine team affiliation
    SetGenericTeamId(FGenericTeamId(255)); // Overridden by component
}

ETeamAttitude::Type AFactionAIController::GetTeamAttitudeTowards(const AActor& Other) const
{
    return UFWFactionComponent::SolveTeamAttitude(GetPawn(), &Other);
}
```

!!! info "How SolveTeamAttitude Works"
    1. Finds `UFWFactionComponent` on both actors.
    2. If either is missing, returns `ETeamAttitude::Neutral`.
    3. Queries the subsystem for the attitude between the two actors (personal rep takes priority over global matrix).
    4. Converts the five-level `EFWFactionAttitude` to UE's three-state `ETeamAttitude` using `FWFactionUtils::ConvertToTeamAttitude`.

---

## Step 6: Configure AI Perception

On your NPC Blueprint (or AI Controller):

1. Add an **AI Perception** component.
2. Add an **AI Sight** sense configuration.
3. Set **Detection by Affiliation**:
    - Detect Enemies: **Checked**
    - Detect Neutrals: **Checked** (optional)
    - Detect Friendlies: **Unchecked**

4. In the AI Controller's `OnPerceptionUpdated` event, filter for hostile targets:

```cpp
void AFactionAIController::OnPerceptionUpdated(const TArray<AActor*>& UpdatedActors)
{
    for (AActor* Actor : UpdatedActors)
    {
        if (GetTeamAttitudeTowards(*Actor) == ETeamAttitude::Hostile)
        {
            // Add to threat list, begin combat behavior
            SetFocus(Actor);
            MoveToActor(Actor);
        }
    }
}
```

Alternatively, in a Behavior Tree, use the **AI Perception** decorator with the `Sense` set to `AISense_Sight` and filter by detected actor's team attitude.

---

## Step 7: Award Reputation on Kill

When a player kills an enemy NPC, award positive reputation with the player's faction and negative reputation with the NPC's faction.

```cpp
void AMyGameMode::OnNPCKilled(ACharacter* Killer, ACharacter* Victim)
{
    UFWFactionComponent* KillerFaction = Killer->FindComponentByClass<UFWFactionComponent>();
    UFWFactionComponent* VictimFaction = Victim->FindComponentByClass<UFWFactionComponent>();

    if (!KillerFaction || !VictimFaction)
    {
        return;
    }

    FName KillerFactionId = KillerFaction->GetCurrentFactionId();
    FName VictimFactionId = VictimFaction->GetCurrentFactionId();

    // Award positive rep with killer's own faction
    KillerFaction->ModifyPersonalReputation(KillerFactionId, 5.0f);

    // Award negative rep with victim's faction
    KillerFaction->ModifyPersonalReputation(VictimFactionId, -10.0f);

    // Optionally modify the global faction matrix
    UFWFactionSubsystem* Subsystem = GetWorld()->GetSubsystem<UFWFactionSubsystem>();
    if (Subsystem)
    {
        // Small global shift: the war intensifies
        Subsystem->ModifyFactionReputation(
            KillerFactionId,
            VictimFactionId,
            -1.0f,    // Delta
            true       // bPropagate -- ripples to allied factions
        );
    }
}
```

---

## Step 8: Listen for War and Peace Events

The subsystem fires events when factions cross critical attitude thresholds.

```cpp
void AMyGameState::BeginPlay()
{
    Super::BeginPlay();

    if (UFWFactionSubsystem* Subsystem = GetWorld()->GetSubsystem<UFWFactionSubsystem>())
    {
        Subsystem->OnFactionWarDeclared.AddDynamic(this, &AMyGameState::HandleWarDeclared);
        Subsystem->OnFactionAllianceFormed.AddDynamic(this, &AMyGameState::HandleAllianceFormed);
    }
}

void AMyGameState::HandleWarDeclared(FName FactionA, FName FactionB)
{
    // Global announcement: "War has broken out between X and Y!"
    BroadcastWorldMessage(FString::Printf(
        TEXT("War declared between %s and %s!"), *FactionA.ToString(), *FactionB.ToString()
    ));
}

void AMyGameState::HandleAllianceFormed(FName FactionA, FName FactionB)
{
    // Global announcement: "X and Y have formed an alliance!"
    BroadcastWorldMessage(FString::Printf(
        TEXT("Alliance formed between %s and %s!"), *FactionA.ToString(), *FactionB.ToString()
    ));
}
```

---

## Step 9: Test with Debug Tools

Use console commands to verify your setup at runtime:

```
FWFaction.ListFactions
```

Expected output:

```
[FWFactionSystem] Loaded factions:
  Ironclad (TeamId=1) - Ironclad Vanguard
  Shadow (TeamId=2) - Shadow Covenant
  Merchants (TeamId=3) - Merchant Confederacy
```

```
FWFaction.ShowMatrix
```

Displays the full reputation matrix with color-coded attitude levels.

Toggle the debug HUD for a visual overlay:

```
FWFaction.ToggleHUD
```

---

## Step 10: Putting It All Together

Your faction war is now fully operational:

1. **Ironclad and Shadow start hostile** -- NPCs from both factions attack players of the opposing faction on sight.
2. **Killing enemy NPCs awards reputation** -- positive with the player's faction, negative with the victim's faction.
3. **Reputation propagates** -- gaining Ironclad rep also gives 50% to Merchants.
4. **Dynamic thresholds** -- Shadow faction is quick to become hostile but hard to befriend. Peace requires sustained effort.
5. **Global events** -- When factions cross critical thresholds, the entire server is notified.
6. **Personal overrides** -- Individual players can have unique standing with any faction, independent of the global matrix.

!!! tip "Advanced: Faction Switching"
    Players can defect to another faction by calling `SetFaction()`. Their personal reputations persist, so a former Shadow player who joins Ironclad retains their hard-earned standing with Merchants.
