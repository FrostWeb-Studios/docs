---
title: Tutorial - Building a Quest Chain
description: Step-by-step guide to building a complete quest chain with prerequisites, kill/collect/travel tasks, party sync, daily resets, and custom rewards.
---

# Tutorial: Building a Quest Chain

This tutorial walks through building a complete quest chain from scratch. You will create quest definitions with prerequisite conditions, configure multiple task types, set up party synchronization, implement daily reset quests, and create a custom reward type.

---

## Prerequisites

Before starting, ensure you have:

- [x] FWQuestSystem installed and enabled ([Installation](installation.md))
- [x] A Player Controller with `UFWQuestManagerComponent` attached ([Quick Start](quick-start.md))
- [x] GameplayAbilities plugin enabled
- [x] Basic familiarity with Gameplay Tags

---

## Part 1: Creating Quest Definitions with Prerequisites

We will build a three-quest chain where each quest requires the previous one to be completed.

### Step 1 -- Define the Quest Chain

| Quest | Tasks | Prerequisite |
|---|---|---|
| The Wolf Problem | Kill 5 wolves | None |
| Tracking the Alpha | Travel to Wolf Den, Kill Alpha Wolf | The Wolf Problem |
| The Elder's Gratitude | Talk to Village Elder | Tracking the Alpha |

### Step 2 -- Create the First Quest (No Prerequisites)

1. Create a Data Asset of type `FWQuestDefinition`.
2. Name it `QD_WolfProblem`.
3. Configure:

```
Quest Tag:        Quest.Side.WolfProblem
Quest Name:       The Wolf Problem
Description:      Wolves have been attacking travelers on the forest road. Thin their numbers.
Category:         Side
Priority:         Normal
Repeat Type:      None
Party Mode:       SyncProgress
```

4. Add a `FWTask_Slay` task:

```
Task Name:        Kill Forest Wolves
Target Tag:       Enemy.Wolf.Forest
Required Count:   5
Description:      Slay 5 forest wolves along the road to the village.
```

5. Add a `FWReward_Experience` reward:

```
XP Amount:        250
```

6. Add a `FWReward_Currency` reward:

```
Currency Type:    Currency.Gold
Amount:           100
```

### Step 3 -- Create the Second Quest (With Prerequisite)

1. Create `QD_TrackingTheAlpha`.
2. Configure:

```
Quest Tag:        Quest.Side.TrackingTheAlpha
Quest Name:       Tracking the Alpha
Description:      The wolves were driven by an alpha. Track it to its den and put it down.
Category:         Side
Priority:         High
Repeat Type:      None
Party Mode:       SyncProgress
```

3. Add a **condition** -- `FWCondition_QuestComplete`:

```
Required Quest Tag:   Quest.Side.WolfProblem
```

This ensures the player must complete "The Wolf Problem" before this quest becomes available.

4. Add tasks (in order):

**Task 1: Travel to the Wolf Den**

```
Type:             FWTask_Travel
Task Name:        Find the Wolf Den
Location Tag:     Location.WolfDen
Description:      Follow the wolf tracks to their den in the Whispering Woods.
```

**Task 2: Kill the Alpha Wolf**

```
Type:             FWTask_Slay
Task Name:        Slay the Alpha Wolf
Target Tag:       Enemy.Wolf.Alpha
Required Count:   1
Description:      Defeat the alpha wolf.
```

!!! note "Task Ordering"
    Tasks are processed in array order. By default, all tasks are active simultaneously. To make tasks sequential (Task 2 locked until Task 1 completes), set the `bSequentialTasks` property on the quest definition to `true`.

5. Add rewards:

```
FWReward_Experience:    XP Amount = 500
FWReward_Item:          Item Tag = Item.AlphaWolfPelt, Count = 1
FWReward_Reputation:    Faction Tag = Faction.Village, Amount = 15
```

### Step 4 -- Create the Third Quest (Chained Prerequisite)

1. Create `QD_EldersGratitude`.
2. Add condition: `FWCondition_QuestComplete` with `RequiredQuestTag = Quest.Side.TrackingTheAlpha`.
3. Add task: `FWTask_TalkTo` with `NPCTag = NPC.VillageElder`.
4. Add rewards: XP + Gold + Reputation.

### Step 5 -- Add All Quests to the Database

Open your `DA_QuestDatabase` and add all three quest definitions.

---

## Part 2: Setting Up Kill, Collect, and Travel Tasks

### Reporting Kills from Your Combat System

When an enemy dies, report it to the quest manager:

```cpp
void AMyGameMode::OnEnemyKilled(AEnemyCharacter* Enemy, APlayerController* Killer)
{
    // Get the quest manager from the killer's controller
    AMyPlayerController* PC = Cast<AMyPlayerController>(Killer);
    if (!PC || !PC->QuestManager) return;

    // Report the kill with the enemy's gameplay tag
    FGameplayTag EnemyTag = Enemy->GetEnemyTag();  // e.g., "Enemy.Wolf.Forest"
    PC->QuestManager->OnEnemyKilled(EnemyTag, 1);
}
```

### Reporting Item Collection

When a player picks up an item:

```cpp
void AMyPlayerController::OnItemCollected(const FGameplayTag& ItemTag, int32 Count)
{
    FFWQuestEvent Event;
    Event.EventTag = FGameplayTag::RequestGameplayTag("Quest.Event.ItemCollected");
    Event.TargetTag = ItemTag;  // e.g., "Item.HerbBundle"
    Event.Count = Count;

    QuestManager->ProcessQuestEvent(Event);
}
```

### Reporting Location Arrival

Use a trigger volume to detect when a player reaches a quest location:

```cpp
// ATriggerVolume in the Wolf Den area
void AQuestLocationTrigger::OnOverlapBegin(AActor* OtherActor)
{
    AMyPlayerController* PC = Cast<AMyPlayerController>(
        Cast<APawn>(OtherActor)->GetController());
    if (!PC) return;

    PC->QuestManager->OnLocationReached(LocationTag);  // "Location.WolfDen"
}
```

=== "Blueprint"

    1. Place a **Box Collision** volume at the quest location.
    2. On **On Component Begin Overlap**, get the overlapping Pawn's Controller.
    3. Get the Quest Manager Component and call **On Location Reached** with the location's Gameplay Tag.

---

## Part 3: Configuring Party Synchronization

### Set Up Party Mode on the Quest

In the quest definition's Details panel, set **Party Mode** to `SyncProgress`. This means kills by any party member within range count for everyone.

### Configure Party Settings

In `DefaultGame.ini`:

```ini
[/Script/FWQuestSystem.FWQuestManagerSettings]
PartyProgressRadius=5000.0
bRequirePartyMemberInRange=true
```

### How Party Sync Works at Runtime

When Player A kills a wolf:

1. Player A's quest manager receives the kill event.
2. The quest manager checks active quests with `SyncProgress` party mode.
3. For each matching quest, it queries the party system for nearby members.
4. Each nearby party member with the same quest active receives the same event.

```cpp
// This happens automatically inside the quest manager when Party Mode is SyncProgress.
// You do not need to write this code -- it is shown for understanding.

void UFWQuestManagerComponent::DispatchToPartyMembers(const FFWQuestEvent& Event,
    UFWQuestDefinition* QuestDef)
{
    // Get party members from FWPartySystem (if available)
    TArray<APlayerController*> PartyMembers = GetNearbyPartyMembers(
        PartyProgressRadius);

    for (APlayerController* Member : PartyMembers)
    {
        UFWQuestManagerComponent* MemberQM = Member->FindComponentByClass<UFWQuestManagerComponent>();
        if (MemberQM && MemberQM->GetQuestState(QuestDef) == EFWQuestState::Active)
        {
            MemberQM->ProcessQuestEvent(Event);
        }
    }
}
```

### Testing Party Sync

1. Create a party with two players (see [FWPartySystem Tutorial](../party-system/tutorial.md)).
2. Accept the same quest on both players.
3. Have Player A kill enemies while Player B is nearby.
4. Verify both players' `OnTaskProgressChanged` fires.

---

## Part 4: Daily Reset Quests

### Step 6 -- Create a Daily Quest

1. Create `QD_Daily_GatherHerbs`.
2. Configure:

```
Quest Tag:        Quest.Daily.GatherHerbs
Quest Name:       Herb Gathering
Description:      Gather fresh herbs for the village alchemist.
Category:         Daily
Priority:         Normal
Repeat Type:      Daily
Party Mode:       None
```

3. Add task: `FWTask_Collect` with `ItemTag = Item.Herb`, `RequiredCount = 10`.
4. Add rewards: `FWReward_Experience` (100 XP) + `FWReward_Currency` (50 Gold).

### Step 7 -- Configure the Reset Schedule

```ini
[/Script/FWQuestSystem.FWQuestStateSubsystem]
DailyResetHourUTC=6
```

Quests with `RepeatType::Daily` reset at 06:00 UTC every day.

### Step 8 -- Handle Reset Events

```cpp
QuestManager->OnQuestStateChanged.AddDynamic(
    this, &AMyPlayerController::HandleQuestStateChanged);

void AMyPlayerController::HandleQuestStateChanged(UFWQuestDefinition* QuestDef,
    EFWQuestState OldState, EFWQuestState NewState)
{
    if (NewState == EFWQuestState::Available && OldState == EFWQuestState::Completed)
    {
        // Quest was reset -- show notification
        if (QuestDef->RepeatType == EFWQuestRepeatType::Daily)
        {
            ShowNotification(FText::Format(
                LOCTEXT("DailyReset", "Daily quest available: {0}"),
                QuestDef->QuestName));
        }
    }
}
```

### Weekly Quests

Weekly quests work identically. Set `RepeatType` to `Weekly` and configure:

```ini
[/Script/FWQuestSystem.FWQuestStateSubsystem]
WeeklyResetDayOfWeek=1
WeeklyResetHourUTC=6
```

This resets weekly quests at Monday 06:00 UTC.

---

## Part 5: Custom Rewards

### Step 9 -- Create a Custom Reward in C++

Create a reward that unlocks a new ability:

```cpp
// FWReward_UnlockAbility.h
#pragma once

#include "Rewards/FWQuestRewardBase.h"
#include "AbilitySystemComponent.h"
#include "FWReward_UnlockAbility.generated.h"

UCLASS(BlueprintType, EditInlineNew, DefaultToInstanced)
class UFWReward_UnlockAbility : public UFWQuestRewardBase
{
    GENERATED_BODY()

public:
    /** The gameplay ability class to grant. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Reward|Ability")
    TSubclassOf<UGameplayAbility> AbilityClass;

    /** The level to grant the ability at. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Reward|Ability")
    int32 AbilityLevel = 1;

    virtual void Grant_Implementation(AActor* QuestOwner) const override
    {
        UAbilitySystemComponent* ASC = QuestOwner->FindComponentByClass<UAbilitySystemComponent>();
        if (ASC && AbilityClass)
        {
            FGameplayAbilitySpec AbilitySpec(AbilityClass, AbilityLevel);
            ASC->GiveAbility(AbilitySpec);
        }
    }

    virtual FText GetDescription_Implementation() const override
    {
        return FText::Format(
            LOCTEXT("UnlockAbilityReward", "Unlocks ability: {0} (Level {1})"),
            AbilityClass ? AbilityClass->GetDefaultObject<UGameplayAbility>()->GetAbilityName() : FText::GetEmpty(),
            FText::AsNumber(AbilityLevel));
    }
};
```

### Step 10 -- Create a Custom Reward in Blueprints

1. Create a new Blueprint with parent class `FWQuestRewardBase`.
2. Name it `BP_Reward_UnlockRecipe`.
3. Add a variable: `RecipeTag` (Gameplay Tag).
4. Override **Grant**:

```
Event Grant(Quest Owner)
    |
    v
Get Crafting Component from Quest Owner
    |
    v
Call "Unlock Recipe" with Recipe Tag
    |
    v
Print String: "Recipe unlocked!"
```

5. Override **Get Description**:

```
Return "Unlocks crafting recipe: {RecipeTag}"
```

6. Use this reward in any quest definition by adding it to the Rewards array.

---

## Complete Quest Chain Summary

```
Quest Chain: The Wolf Menace

[Quest 1: The Wolf Problem]
  Conditions: (none)
  Tasks: Kill 5 Forest Wolves
  Rewards: 250 XP, 100 Gold
  Party Mode: SyncProgress
        |
        v (prerequisite)
[Quest 2: Tracking the Alpha]
  Conditions: Quest.Side.WolfProblem completed
  Tasks: Travel to Wolf Den -> Kill Alpha Wolf (sequential)
  Rewards: 500 XP, Alpha Wolf Pelt x1, +15 Village Reputation
  Party Mode: SyncProgress
        |
        v (prerequisite)
[Quest 3: The Elder's Gratitude]
  Conditions: Quest.Side.TrackingTheAlpha completed
  Tasks: Talk to Village Elder
  Rewards: 200 XP, 250 Gold, +10 Village Reputation, Unlock Ability (Wolf's Instinct)
  Party Mode: None

[Daily Quest: Herb Gathering]
  Conditions: (none)
  Tasks: Collect 10 Herbs
  Rewards: 100 XP, 50 Gold
  Repeat Type: Daily (resets at 06:00 UTC)
```

---

## Testing the Quest Chain

### Step-by-Step Verification

1. Start the game. Verify `QD_WolfProblem` shows as `Available`.
2. Verify `QD_TrackingTheAlpha` is NOT available (condition not met).
3. Accept `QD_WolfProblem`. Kill 5 wolves. Verify progress events fire.
4. Turn in the quest. Verify rewards are granted.
5. Verify `QD_TrackingTheAlpha` is now `Available`.
6. Accept it. Travel to the wolf den. Verify the travel task completes.
7. Kill the alpha wolf. Verify quest moves to `ReadyToTurnIn`.
8. Turn in. Verify `QD_EldersGratitude` is now available.
9. Talk to the Village Elder. Turn in. Verify the ability reward is granted.

### Testing Daily Reset

```cpp
// Force a daily reset for testing (server-side or console command)
UFWQuestStateSubsystem* QuestState = GetWorld()->GetSubsystem<UFWQuestStateSubsystem>();
QuestState->ProcessDailyReset();
```

After reset, verify the daily quest returns to `Available` with zero task progress.

### Testing Party Sync

1. Create a party with Player A and Player B.
2. Both accept `QD_WolfProblem`.
3. Player A kills 3 wolves, Player B kills 2 wolves.
4. Both players should show 5/5 progress (SyncProgress mode).

---

## Next Steps

- See [Data Definitions](data-definitions.md) for all task, condition, and reward types.
- See [Configuration](configuration.md) for reset schedules and serialization.
- See [Events and Delegates](events-delegates.md) for the complete event reference.
- See [API Reference](api-reference.md) for all available functions.
