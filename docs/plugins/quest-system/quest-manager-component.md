---
title: Quest Manager Component - FWQuestSystem
description: Complete API reference for UFWQuestManagerComponent, the primary Blueprint-facing interface for quest operations.
---

# UFWQuestManagerComponent

**Class:** `UFWQuestManagerComponent` | **Parent:** `UActorComponent` | **Specifier:** `BlueprintSpawnableComponent`

The central component for all quest operations. Attach this to your Player Controller or Pawn to give players the ability to accept, track, complete, and abandon quests. All functions are exposed to Blueprints.

---

## Component Setup

```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Quest")
TObjectPtr<UFWQuestManagerComponent> QuestManager;

// In constructor
QuestManager = CreateDefaultSubobject<UFWQuestManagerComponent>(TEXT("QuestManager"));
```

Set the quest database before accepting quests:

```cpp
void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    UFWQuestDatabase* DB = LoadObject<UFWQuestDatabase>(
        nullptr, TEXT("/Game/Data/DA_QuestDatabase"));
    QuestManager->SetQuestDatabase(DB);
}
```

---

## Blueprint Callable Functions

### Quest Lifecycle

#### `AcceptQuest`

Accepts a quest and begins tracking its tasks. Validates all conditions before acceptance.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void AcceptQuest(UFWQuestDefinition* QuestDefinition);
```

| Parameter | Type | Description |
|---|---|---|
| `QuestDefinition` | `UFWQuestDefinition*` | The quest to accept |

Fires `OnQuestAccepted` on success. Fires `OnQuestStateChanged` with transition to `Active`.

!!! warning "Condition Check"
    `AcceptQuest` calls `CanAcceptQuest` internally. If conditions are not met, the quest is not accepted and no event fires. Always call `CanAcceptQuest` first in your UI to provide feedback to the player.

---

#### `AbandonQuest`

Abandons an active quest, resetting all task progress.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void AbandonQuest(UFWQuestDefinition* QuestDefinition);
```

| Parameter | Type | Description |
|---|---|---|
| `QuestDefinition` | `UFWQuestDefinition*` | The quest to abandon |

Fires `OnQuestAbandoned`. For repeatable quests, the state resets to `Available`.

---

#### `TurnInQuest`

Turns in a completed quest and grants all configured rewards.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void TurnInQuest(UFWQuestDefinition* QuestDefinition);
```

| Parameter | Type | Description |
|---|---|---|
| `QuestDefinition` | `UFWQuestDefinition*` | The quest to turn in (must be in `ReadyToTurnIn` state) |

Fires `OnQuestCompleted` after rewards are granted. For repeatable quests, starts the cooldown or schedules the next reset.

---

#### `FailQuest`

Marks a quest as failed. Typically called by timer tasks or external game logic.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void FailQuest(UFWQuestDefinition* QuestDefinition);
```

| Parameter | Type | Description |
|---|---|---|
| `QuestDefinition` | `UFWQuestDefinition*` | The quest to fail |

Fires `OnQuestStateChanged` with transition to `Failed`.

---

### Event Reporting

#### `ProcessQuestEvent`

Dispatches a generic quest event to all active tasks for matching.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void ProcessQuestEvent(const FFWQuestEvent& QuestEvent);
```

| Parameter | Type | Description |
|---|---|---|
| `QuestEvent` | `FFWQuestEvent` | The event payload containing tag, count, and optional context |

This is the primary method for reporting game events to the quest system. All convenience methods (`OnEnemyKilled`, `OnLocationReached`, `OnNPCInteracted`) call this internally.

---

#### `OnEnemyKilled`

Convenience method for reporting enemy kills.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void OnEnemyKilled(const FGameplayTag& EnemyTag, int32 Count = 1);
```

| Parameter | Type | Description |
|---|---|---|
| `EnemyTag` | `FGameplayTag` | Tag identifying the enemy type (e.g., `Enemy.Wolf`) |
| `Count` | `int32` | Number of kills to report (default 1) |

---

#### `OnLocationReached`

Convenience method for reporting location arrival.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void OnLocationReached(const FGameplayTag& LocationTag);
```

| Parameter | Type | Description |
|---|---|---|
| `LocationTag` | `FGameplayTag` | Tag identifying the location (e.g., `Location.WhisperingWoods`) |

---

#### `OnNPCInteracted`

Convenience method for reporting NPC interactions.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void OnNPCInteracted(const FGameplayTag& NPCTag);
```

| Parameter | Type | Description |
|---|---|---|
| `NPCTag` | `FGameplayTag` | Tag identifying the NPC (e.g., `NPC.Blacksmith`) |

---

### Database Management

#### `SetQuestDatabase`

Sets the quest database used for quest lookups and availability checks.

```cpp
UFUNCTION(BlueprintCallable, Category = "Quest")
void SetQuestDatabase(UFWQuestDatabase* Database);
```

| Parameter | Type | Description |
|---|---|---|
| `Database` | `UFWQuestDatabase*` | The quest database asset to use |

---

## Blueprint Pure Functions

#### `GetQuestDatabase`

```cpp
UFUNCTION(BlueprintPure, Category = "Quest")
UFWQuestDatabase* GetQuestDatabase() const;
```

Returns the currently assigned quest database.

---

#### `GetQuestState`

```cpp
UFUNCTION(BlueprintPure, Category = "Quest")
EFWQuestState GetQuestState(UFWQuestDefinition* QuestDefinition) const;
```

Returns the current state of a quest (`Available`, `Active`, `ReadyToTurnIn`, `Completed`, `Failed`, `Abandoned`).

---

#### `GetQuestTasks`

```cpp
UFUNCTION(BlueprintPure, Category = "Quest")
TArray<FFWQuestTaskInstance> GetQuestTasks(UFWQuestDefinition* QuestDefinition) const;
```

Returns the runtime task instances for an active quest, including current progress.

---

#### `CanAcceptQuest`

```cpp
UFUNCTION(BlueprintPure, Category = "Quest")
bool CanAcceptQuest(UFWQuestDefinition* QuestDefinition) const;
```

Evaluates all conditions on the quest definition and returns whether the player can accept it.

---

#### `IsQuestAvailable`

```cpp
UFUNCTION(BlueprintPure, Category = "Quest")
bool IsQuestAvailable(UFWQuestDefinition* QuestDefinition) const;
```

Returns whether a quest is in the `Available` state (not active, not completed unless repeatable and reset).

---

## Events (Delegates)

All delegates are `BlueprintAssignable`.

| Delegate | Signature | Description |
|---|---|---|
| `OnQuestAccepted` | `(UFWQuestDefinition* QuestDef)` | Fired when a quest is accepted |
| `OnQuestCompleted` | `(UFWQuestDefinition* QuestDef)` | Fired when a quest is turned in and rewards granted |
| `OnQuestAbandoned` | `(UFWQuestDefinition* QuestDef)` | Fired when a quest is abandoned |
| `OnTaskProgressChanged` | `(UFWQuestDefinition* QuestDef, int32 TaskIndex, int32 Current, int32 Required)` | Fired when a task's progress changes |
| `OnQuestStateChanged` | `(UFWQuestDefinition* QuestDef, EFWQuestState OldState, EFWQuestState NewState)` | Fired on any state transition |

---

## Usage Patterns

### Quest Journal Query

```cpp
// Get all active quests
TArray<UFWQuestDefinition*> ActiveQuests;
for (UFWQuestDefinition* QuestDef : QuestManager->GetQuestDatabase()->GetAllQuests())
{
    if (QuestManager->GetQuestState(QuestDef) == EFWQuestState::Active)
    {
        ActiveQuests.Add(QuestDef);
    }
}

// Sort by priority
ActiveQuests.Sort([](const UFWQuestDefinition& A, const UFWQuestDefinition& B)
{
    return static_cast<uint8>(A.Priority) > static_cast<uint8>(B.Priority);
});
```

### Task Progress Display

```cpp
TArray<FFWQuestTaskInstance> Tasks = QuestManager->GetQuestTasks(QuestDef);
for (const FFWQuestTaskInstance& Task : Tasks)
{
    UE_LOG(LogTemp, Log, TEXT("  [%s] %s: %d/%d"),
        *UEnum::GetValueAsString(Task.State),
        *Task.TaskName,
        Task.CurrentProgress,
        Task.RequiredProgress);
}
```

---

## Next Steps

- See [Data Definitions](data-definitions.md) for how to create quest definitions, tasks, conditions, and rewards.
- See [Events and Delegates](events-delegates.md) for detailed event flow diagrams.
- See [Types Reference](types.md) for all struct and enum definitions.
