---
title: Types Reference - FWQuestSystem
description: Complete reference for all enums, structs, and delegate types defined in FWQuestTypes.h.
---

# Types Reference

All types are defined in `FWQuestTypes.h` and are available to both C++ and Blueprints.

---

## Enums

### EFWQuestState

The lifecycle state of a quest instance.

```cpp
UENUM(BlueprintType)
enum class EFWQuestState : uint8
{
    Available      UMETA(DisplayName = "Available"),
    Active         UMETA(DisplayName = "Active"),
    ReadyToTurnIn  UMETA(DisplayName = "Ready To Turn In"),
    Completed      UMETA(DisplayName = "Completed"),
    Failed         UMETA(DisplayName = "Failed"),
    Abandoned      UMETA(DisplayName = "Abandoned")
};
```

| Value | Description |
|---|---|
| `Available` | Quest conditions are met and the quest can be accepted |
| `Active` | Quest has been accepted and tasks are being tracked |
| `ReadyToTurnIn` | All tasks are complete, waiting for player to turn in |
| `Completed` | Quest has been turned in and rewards granted |
| `Failed` | Quest was failed (timer expired, game logic, etc.) |
| `Abandoned` | Quest was abandoned by the player |

**State Transitions:**

```
Available --> Active --> ReadyToTurnIn --> Completed
    ^           |                            |
    |           +-------> Failed             | (if repeatable)
    |           |                            |
    |           +-------> Abandoned          |
    |                         |              |
    +-------------------------+--------------+
              (reset/repeat)
```

---

### EFWQuestCategory

Organizational category for quest journal grouping.

```cpp
UENUM(BlueprintType)
enum class EFWQuestCategory : uint8
{
    MainStory   UMETA(DisplayName = "Main Story"),
    Side        UMETA(DisplayName = "Side"),
    Daily       UMETA(DisplayName = "Daily"),
    Weekly      UMETA(DisplayName = "Weekly"),
    World       UMETA(DisplayName = "World"),
    Dungeon     UMETA(DisplayName = "Dungeon"),
    Guild       UMETA(DisplayName = "Guild"),
    Event       UMETA(DisplayName = "Event"),
    Profession  UMETA(DisplayName = "Profession"),
    Challenge   UMETA(DisplayName = "Challenge")
};
```

| Value | Typical Use |
|---|---|
| `MainStory` | Critical-path narrative quests |
| `Side` | Optional story or exploration quests |
| `Daily` | Quests that reset every day |
| `Weekly` | Quests that reset every week |
| `World` | Open-world objectives available to all players |
| `Dungeon` | Quests tied to specific dungeon instances |
| `Guild` | Quests that require guild membership |
| `Event` | Time-limited seasonal or special event quests |
| `Profession` | Crafting, gathering, or profession-specific quests |
| `Challenge` | Achievement-style skill challenges |

---

### EFWQuestRepeatType

Controls whether and how a quest can be repeated after completion.

```cpp
UENUM(BlueprintType)
enum class EFWQuestRepeatType : uint8
{
    None       UMETA(DisplayName = "None"),
    Cooldown   UMETA(DisplayName = "Cooldown"),
    Daily      UMETA(DisplayName = "Daily"),
    Weekly     UMETA(DisplayName = "Weekly"),
    Unlimited  UMETA(DisplayName = "Unlimited")
};
```

| Value | Description |
|---|---|
| `None` | Quest can only be completed once |
| `Cooldown` | Quest can be repeated after a configurable cooldown period |
| `Daily` | Quest resets at the daily reset time (configurable, default midnight UTC) |
| `Weekly` | Quest resets at the weekly reset time (configurable, default Monday midnight UTC) |
| `Unlimited` | Quest can be immediately re-accepted after turn-in |

---

### EFWQuestPriority

Sorting and notification priority for quests.

```cpp
UENUM(BlueprintType)
enum class EFWQuestPriority : uint8
{
    Low     UMETA(DisplayName = "Low"),
    Normal  UMETA(DisplayName = "Normal"),
    High    UMETA(DisplayName = "High"),
    Urgent  UMETA(DisplayName = "Urgent")
};
```

---

### EFWQuestPartyMode

Controls how quest progress interacts with party members.

```cpp
UENUM(BlueprintType)
enum class EFWQuestPartyMode : uint8
{
    None          UMETA(DisplayName = "None"),
    ShareOnly     UMETA(DisplayName = "Share Only"),
    SyncProgress  UMETA(DisplayName = "Sync Progress"),
    RequireAll    UMETA(DisplayName = "Require All")
};
```

| Value | Description |
|---|---|
| `None` | Quest is fully individual. Party membership has no effect. |
| `ShareOnly` | Party members can accept the same quest, but progress is tracked individually. |
| `SyncProgress` | Kills, collections, and other events by any party member count for all members who have the quest active. |
| `RequireAll` | All party members must be present and have the quest active for progress to count. |

---

### EFWTaskState

The lifecycle state of an individual task within a quest.

```cpp
UENUM(BlueprintType)
enum class EFWTaskState : uint8
{
    Pending    UMETA(DisplayName = "Pending"),
    Locked     UMETA(DisplayName = "Locked"),
    Active     UMETA(DisplayName = "Active"),
    Completed  UMETA(DisplayName = "Completed"),
    Failed     UMETA(DisplayName = "Failed")
};
```

| Value | Description |
|---|---|
| `Pending` | Task is part of the quest but not yet tracking (quest not accepted) |
| `Locked` | Task is gated by a previous task in the sequence |
| `Active` | Task is actively tracking progress |
| `Completed` | Task has reached its required progress |
| `Failed` | Task has failed (timer expired, etc.) |

---

## Structs

### FFWQuestInstance

Runtime state of an active quest, stored per-player.

```cpp
USTRUCT(BlueprintType)
struct FFWQuestInstance
{
    GENERATED_BODY()

    /** Reference to the quest definition. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    TObjectPtr<UFWQuestDefinition> QuestDefinition;

    /** Current quest state. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    EFWQuestState State;

    /** Runtime task instances with progress data. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    TArray<FFWQuestTaskInstance> TaskInstances;

    /** When this quest was accepted (UTC). */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    FDateTime AcceptedAt;

    /** When this quest was completed (UTC). Zero if not completed. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    FDateTime CompletedAt;

    /** Number of times this quest has been completed (for repeatable quests). */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    int32 CompletionCount;
};
```

---

### FFWQuestTaskInstance

Runtime state of an individual task within an active quest.

```cpp
USTRUCT(BlueprintType)
struct FFWQuestTaskInstance
{
    GENERATED_BODY()

    /** Display name of the task. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    FText TaskName;

    /** Current progress toward completion. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    int32 CurrentProgress;

    /** Required progress for completion. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    int32 RequiredProgress;

    /** Current task state. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    EFWTaskState State;

    /** Index of the task in the quest definition's task array. */
    UPROPERTY(BlueprintReadOnly, Category = "Quest")
    int32 TaskIndex;
};
```

---

### FFWQuestEvent

Event payload used with `ProcessQuestEvent` to report game events to the quest system.

```cpp
USTRUCT(BlueprintType)
struct FFWQuestEvent
{
    GENERATED_BODY()

    /** The type of event (e.g., Quest.Event.EnemyKilled). */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest")
    FGameplayTag EventTag;

    /** The target of the event (e.g., Enemy.Wolf, Item.HerbBundle). */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest")
    FGameplayTag TargetTag;

    /** Count associated with this event (e.g., number killed, items collected). */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest")
    int32 Count = 1;

    /** Optional actor reference for context (e.g., the killed enemy, the interacted NPC). */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest")
    TObjectPtr<AActor> ContextActor;

    /** Optional location for proximity-based tasks. */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Quest")
    FVector EventLocation = FVector::ZeroVector;
};
```

---

## Delegates

### Quest Lifecycle Delegates

```cpp
/** Fired when a quest is accepted. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnQuestAccepted, UFWQuestDefinition*, QuestDefinition);

/** Fired when a quest is turned in and rewards granted. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnQuestCompleted, UFWQuestDefinition*, QuestDefinition);

/** Fired when a quest is abandoned. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnQuestAbandoned, UFWQuestDefinition*, QuestDefinition);
```

### Progress Delegates

```cpp
/** Fired when a task's progress changes. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(
    FOnTaskProgressChanged,
    UFWQuestDefinition*, QuestDefinition,
    int32, TaskIndex,
    int32, CurrentProgress,
    int32, RequiredProgress);
```

### State Change Delegates

```cpp
/** Fired on any quest state transition. */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnQuestStateChanged,
    UFWQuestDefinition*, QuestDefinition,
    EFWQuestState, OldState,
    EFWQuestState, NewState);
```

---

## Next Steps

- See [Data Definitions](data-definitions.md) for how types are used in quest asset configuration.
- See [API Reference](api-reference.md) for the complete function listing.
- See [Events and Delegates](events-delegates.md) for event flow diagrams.
