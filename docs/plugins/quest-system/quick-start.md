---
title: Quick Start - FWQuestSystem
description: Get a working quest system running in under 10 minutes.
---

# Quick Start

Get a working quest system running in under 10 minutes with a quest database, a kill quest, and progress tracking.

---

## Overview

By the end of this guide you will have:

1. A quest database with one quest definition
2. A Player Controller with `UFWQuestManagerComponent` attached
3. A kill quest that tracks enemy deaths and grants XP on completion
4. Event-driven UI updates for quest progress

---

## Step 1 -- Create the Quest Database

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWQuestDatabase` as the class.
3. Name it `DA_QuestDatabase`.

This asset holds references to all quest definitions in your game.

---

## Step 2 -- Create a Quest Definition

1. Create another Data Asset, this time choosing `FWQuestDefinition`.
2. Name it `QD_SlayWolves`.
3. Open it and configure:

| Property | Value |
|---|---|
| Quest Name | Slay the Wolves |
| Description | The wolves near the village have become aggressive. Thin their numbers. |
| Category | `Side` |
| Priority | `Normal` |
| Repeat Type | `None` |
| Party Mode | `None` |

### Add a Task

1. In the **Tasks** array, click the **+** button.
2. Set the task class to `FWTask_Slay`.
3. Configure:

| Property | Value |
|---|---|
| Task Name | Kill Wolves |
| Target Tag | `Enemy.Wolf` |
| Required Count | 5 |
| Description | Slay 5 wolves in the Whispering Woods. |

### Add a Reward

1. In the **Rewards** array, click the **+** button.
2. Set the reward class to `FWReward_Experience`.
3. Configure:

| Property | Value |
|---|---|
| XP Amount | 250 |

### Add to Database

4. Open `DA_QuestDatabase` and add `QD_SlayWolves` to its quest definition list.

---

## Step 3 -- Add the Component to Your Player Controller

### In C++

```cpp
// MyPlayerController.h
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Quest")
TObjectPtr<UFWQuestManagerComponent> QuestManager;

// MyPlayerController.cpp (constructor)
QuestManager = CreateDefaultSubobject<UFWQuestManagerComponent>(TEXT("QuestManager"));
```

Then in `BeginPlay`:

```cpp
void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    // Load the quest database
    UFWQuestDatabase* DB = LoadObject<UFWQuestDatabase>(
        nullptr, TEXT("/Game/Data/DA_QuestDatabase"));
    QuestManager->SetQuestDatabase(DB);

    // Bind events
    QuestManager->OnQuestAccepted.AddDynamic(this, &AMyPlayerController::HandleQuestAccepted);
    QuestManager->OnTaskProgressChanged.AddDynamic(this, &AMyPlayerController::HandleTaskProgress);
    QuestManager->OnQuestCompleted.AddDynamic(this, &AMyPlayerController::HandleQuestCompleted);
}
```

### In Blueprints

1. Open your Player Controller Blueprint.
2. Click **Add Component** and search for `FWQuestManager`.
3. Select **FW Quest Manager Component**.
4. In the Details panel, set **Quest Database** to `DA_QuestDatabase`.

---

## Step 4 -- Accept the Quest

=== "C++"

    ```cpp
    // Load the quest definition and accept it
    UFWQuestDefinition* SlayWolves = LoadObject<UFWQuestDefinition>(
        nullptr, TEXT("/Game/Data/QD_SlayWolves"));

    if (QuestManager->CanAcceptQuest(SlayWolves))
    {
        QuestManager->AcceptQuest(SlayWolves);
    }
    ```

=== "Blueprint"

    1. Get a reference to the Quest Definition asset.
    2. Call **Can Accept Quest** to check prerequisites.
    3. Call **Accept Quest** to begin tracking.

---

## Step 5 -- Report Enemy Kills

When an enemy dies, call `OnEnemyKilled` on the quest manager:

```cpp
void AMyPlayerController::HandleEnemyDeath(AActor* Enemy)
{
    // Get the enemy's gameplay tag (e.g., "Enemy.Wolf")
    FGameplayTag EnemyTag = GetEnemyTag(Enemy);

    // Report the kill to the quest system
    QuestManager->OnEnemyKilled(EnemyTag, 1);
}
```

The `FWTask_Slay` task automatically matches the `EnemyTag` against its `TargetTag` and increments progress.

---

## Step 6 -- Handle Events

```cpp
void AMyPlayerController::HandleQuestAccepted(UFWQuestDefinition* QuestDef)
{
    UE_LOG(LogTemp, Log, TEXT("Quest accepted: %s"), *QuestDef->QuestName);
    // Show quest accepted notification in UI
}

void AMyPlayerController::HandleTaskProgress(UFWQuestDefinition* QuestDef,
    int32 TaskIndex, int32 CurrentProgress, int32 RequiredProgress)
{
    UE_LOG(LogTemp, Log, TEXT("Task %d: %d/%d"), TaskIndex, CurrentProgress, RequiredProgress);
    // Update quest tracker widget
}

void AMyPlayerController::HandleQuestCompleted(UFWQuestDefinition* QuestDef)
{
    UE_LOG(LogTemp, Log, TEXT("Quest completed: %s"), *QuestDef->QuestName);
    // Show completion notification, rewards are granted automatically
}
```

---

## Step 7 -- Turn In the Quest

When all tasks are complete, the quest moves to `ReadyToTurnIn` state. Call `TurnInQuest` to grant rewards:

=== "C++"

    ```cpp
    // Check if quest is ready (typically from a UI button or NPC interaction)
    if (QuestManager->GetQuestState(SlayWolves) == EFWQuestState::ReadyToTurnIn)
    {
        QuestManager->TurnInQuest(SlayWolves);
        // Rewards (250 XP) are automatically applied
    }
    ```

=== "Blueprint"

    Check the quest state with **Get Quest State**, and if it returns **Ready To Turn In**, call **Turn In Quest**.

---

## Result

You now have a quest system that:

- Loads quest definitions from a data-driven database
- Accepts quests and tracks kill progress
- Fires events for UI updates on each kill
- Grants XP rewards on quest turn-in

---

## Next Steps

- Read the [Data Definitions](data-definitions.md) guide for all task, condition, and reward types.
- See the [Quest Manager Component](quest-manager-component.md) for the full API reference.
- Follow the [Tutorial](tutorial.md) to build a complete quest chain with prerequisites, party sync, and daily resets.
