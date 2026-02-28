---
title: Events and Delegates - FWQuestSystem
description: All multicast delegates, event flow diagrams, and binding patterns for FWQuestSystem.
---

# Events and Delegates

This page documents all events fired by FWQuestSystem, their flow through the quest lifecycle, and patterns for binding them in C++ and Blueprints.

---

## Delegate Summary

| Delegate | Signature | When It Fires |
|---|---|---|
| `OnQuestAccepted` | `(UFWQuestDefinition* QuestDef)` | After a quest is successfully accepted |
| `OnQuestCompleted` | `(UFWQuestDefinition* QuestDef)` | After a quest is turned in and rewards granted |
| `OnQuestAbandoned` | `(UFWQuestDefinition* QuestDef)` | After a quest is abandoned |
| `OnTaskProgressChanged` | `(UFWQuestDefinition* QuestDef, int32 TaskIndex, int32 Current, int32 Required)` | When a task's progress changes |
| `OnQuestStateChanged` | `(UFWQuestDefinition* QuestDef, EFWQuestState OldState, EFWQuestState NewState)` | On any state transition |

---

## Binding Patterns

### C++ Dynamic Binding

```cpp
void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    QuestManager->OnQuestAccepted.AddDynamic(
        this, &AMyPlayerController::HandleQuestAccepted);
    QuestManager->OnQuestCompleted.AddDynamic(
        this, &AMyPlayerController::HandleQuestCompleted);
    QuestManager->OnQuestAbandoned.AddDynamic(
        this, &AMyPlayerController::HandleQuestAbandoned);
    QuestManager->OnTaskProgressChanged.AddDynamic(
        this, &AMyPlayerController::HandleTaskProgress);
    QuestManager->OnQuestStateChanged.AddDynamic(
        this, &AMyPlayerController::HandleQuestStateChanged);
}
```

!!! warning "UFUNCTION Requirement"
    All dynamic delegate handler functions must be marked with `UFUNCTION()`. Forgetting this macro will compile but crash at runtime when the delegate attempts to fire.

### Blueprint Binding

In Blueprints, drag from the Quest Manager Component reference and select **Assign** on the desired event. Common pattern for a quest tracker widget:

1. Get a reference to the owning Player Controller's Quest Manager Component.
2. Bind **On Task Progress Changed** to update progress bars.
3. Bind **On Quest State Changed** to add/remove quests from the tracker.
4. Bind **On Quest Completed** to show a completion notification.

---

## Event Flow Diagrams

### Accept Quest Flow

```
Player calls AcceptQuest(QuestDef)
    |
    v
QuestManager: CanAcceptQuest(QuestDef)?
    |
    +--[No]---> (No event fires, operation rejected)
    |
    +--[Yes]--> Create FFWQuestInstance
                    |
                    v
                Initialize task instances (Active or Locked)
                    |
                    v
                Register with UFWQuestStateSubsystem
                    |
                    v
                OnQuestStateChanged(QuestDef, Available, Active)
                    |
                    v
                OnQuestAccepted(QuestDef)
```

### Quest Event Processing Flow

```
Game event occurs (enemy killed, location reached, etc.)
    |
    v
QuestManager->ProcessQuestEvent(FFWQuestEvent)
    |
    v
For each active quest:
    |
    v
    For each active task in the quest:
        |
        v
        Task->ProcessEvent(QuestEvent) -> returns progress increment
            |
            +--[0]---> (No match, skip)
            |
            +--[>0]--> Update task progress
                           |
                           v
                        OnTaskProgressChanged(QuestDef, TaskIndex, Current, Required)
                           |
                           +--[Current >= Required]
                           |       |
                           |       v
                           |   Task state -> Completed
                           |       |
                           |       v
                           |   Unlock next sequential task (if any)
                           |       |
                           |       v
                           |   All tasks completed?
                           |       |
                           |       +--[No]---> Continue tracking
                           |       |
                           |       +--[Yes]--> Quest state -> ReadyToTurnIn
                           |                       |
                           |                       v
                           |                   OnQuestStateChanged(QuestDef, Active, ReadyToTurnIn)
                           |
                           +--[Current < Required]
                                   |
                                   v
                               Continue tracking
```

### Turn In Quest Flow

```
Player calls TurnInQuest(QuestDef)
    |
    v
QuestManager: Is quest state == ReadyToTurnIn?
    |
    +--[No]---> (No event fires, operation rejected)
    |
    +--[Yes]--> Grant all rewards in sequence
                    |
                    v
                Rewards[0]->Grant(QuestOwner)
                Rewards[1]->Grant(QuestOwner)
                Rewards[N]->Grant(QuestOwner)
                    |
                    v
                Quest state -> Completed
                CompletionCount++
                CompletedAt = Now
                    |
                    v
                OnQuestStateChanged(QuestDef, ReadyToTurnIn, Completed)
                    |
                    v
                OnQuestCompleted(QuestDef)
                    |
                    v
                Is quest repeatable?
                    |
                    +--[None]-------> Quest stays Completed
                    |
                    +--[Cooldown]---> Start cooldown timer
                    |                     |
                    |                     v (after cooldown)
                    |                 Quest state -> Available
                    |
                    +--[Daily]------> Schedule for next daily reset
                    |
                    +--[Weekly]-----> Schedule for next weekly reset
                    |
                    +--[Unlimited]--> Quest state -> Available immediately
```

### Abandon Quest Flow

```
Player calls AbandonQuest(QuestDef)
    |
    v
QuestManager: Is quest state == Active or ReadyToTurnIn?
    |
    +--[No]---> (No event fires, operation rejected)
    |
    +--[Yes]--> Clear all task progress
                    |
                    v
                Quest state -> Abandoned
                    |
                    v
                OnQuestStateChanged(QuestDef, Active/ReadyToTurnIn, Abandoned)
                    |
                    v
                OnQuestAbandoned(QuestDef)
                    |
                    v
                Is quest repeatable?
                    |
                    +--[Yes]--> Quest state -> Available (can be re-accepted)
                    +--[No]---> Quest stays Abandoned
```

### Daily/Weekly Reset Flow

```
UFWQuestStateSubsystem detects reset time reached
    |
    v
For each registered UFWQuestManagerComponent:
    |
    v
    For each quest in Completed/Failed/Abandoned state:
        |
        v
        Does quest RepeatType match reset type (Daily/Weekly)?
            |
            +--[No]---> Skip
            |
            +--[Yes]--> Clear task progress
                            |
                            v
                        Quest state -> Available
                            |
                            v
                        OnQuestStateChanged(QuestDef, OldState, Available)
```

---

## Party Event Synchronization

When a quest has `PartyMode` set to `SyncProgress`, quest events are distributed to party members:

```
Player A kills an enemy
    |
    v
Player A's QuestManager->OnEnemyKilled(EnemyTag, 1)
    |
    v
QuestManager checks active quests with SyncProgress party mode
    |
    v
For each matching quest:
    |
    v
    Find party members within PartyProgressRadius
        |
        v
        For each nearby party member with the same quest active:
            |
            v
            Dispatch the same FFWQuestEvent to their QuestManager
            -> OnTaskProgressChanged fires on each member's component
```

!!! note "Server Authority"
    In multiplayer, party progress synchronization should execute on the server. The server validates proximity, quest state, and party membership before dispatching events to other players' quest managers.

---

## Event Ordering Guarantees

1. `OnQuestStateChanged` always fires before `OnQuestAccepted`, `OnQuestCompleted`, or `OnQuestAbandoned`.
2. `OnTaskProgressChanged` fires for each task before `OnQuestStateChanged` transitions to `ReadyToTurnIn`.
3. All reward `Grant` calls complete before `OnQuestCompleted` fires.
4. During resets, `OnQuestStateChanged` fires once per reset quest, not in batch.

---

## Common UI Patterns

### Quest Tracker Widget

```cpp
void UQuestTrackerWidget::BindToQuestManager(UFWQuestManagerComponent* QM)
{
    QM->OnTaskProgressChanged.AddDynamic(this, &UQuestTrackerWidget::UpdateTaskProgress);
    QM->OnQuestStateChanged.AddDynamic(this, &UQuestTrackerWidget::HandleStateChange);
}

void UQuestTrackerWidget::UpdateTaskProgress(UFWQuestDefinition* QuestDef,
    int32 TaskIndex, int32 Current, int32 Required)
{
    // Find the progress bar for this task and update it
    if (UProgressBar* Bar = FindProgressBar(QuestDef, TaskIndex))
    {
        Bar->SetPercent(static_cast<float>(Current) / static_cast<float>(Required));
    }
}

void UQuestTrackerWidget::HandleStateChange(UFWQuestDefinition* QuestDef,
    EFWQuestState OldState, EFWQuestState NewState)
{
    if (NewState == EFWQuestState::ReadyToTurnIn)
    {
        // Highlight the quest as ready to turn in
        MarkQuestReady(QuestDef);
    }
    else if (NewState == EFWQuestState::Completed || NewState == EFWQuestState::Abandoned)
    {
        // Remove from tracker
        RemoveQuestFromTracker(QuestDef);
    }
}
```

---

## Next Steps

- See [Quest Manager Component](quest-manager-component.md) for the full function API.
- See [Types Reference](types.md) for delegate type declarations.
- See [Tutorial](tutorial.md) for practical examples of event handling in a complete implementation.
