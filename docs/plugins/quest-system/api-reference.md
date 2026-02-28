---
title: API Reference - FWQuestSystem
description: Complete function and delegate reference for all FWQuestSystem classes.
---

# API Reference

Complete reference for all public functions, properties, and delegates across FWQuestSystem classes.

---

## UFWQuestManagerComponent

**Parent:** `UActorComponent` | **Specifier:** `BlueprintSpawnableComponent`

### BlueprintCallable Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `AcceptQuest` | `UFWQuestDefinition* QuestDef` | `void` | Accepts a quest after validating conditions |
| `AbandonQuest` | `UFWQuestDefinition* QuestDef` | `void` | Abandons an active quest |
| `TurnInQuest` | `UFWQuestDefinition* QuestDef` | `void` | Turns in a completed quest and grants rewards |
| `FailQuest` | `UFWQuestDefinition* QuestDef` | `void` | Marks a quest as failed |
| `ProcessQuestEvent` | `const FFWQuestEvent& Event` | `void` | Dispatches a quest event to all active tasks |
| `OnEnemyKilled` | `const FGameplayTag& EnemyTag, int32 Count` | `void` | Convenience: report enemy kill |
| `OnLocationReached` | `const FGameplayTag& LocationTag` | `void` | Convenience: report location arrival |
| `OnNPCInteracted` | `const FGameplayTag& NPCTag` | `void` | Convenience: report NPC interaction |
| `SetQuestDatabase` | `UFWQuestDatabase* Database` | `void` | Sets the active quest database |

### BlueprintPure Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `GetQuestDatabase` | *(none)* | `UFWQuestDatabase*` | Returns the active quest database |
| `GetQuestState` | `UFWQuestDefinition* QuestDef` | `EFWQuestState` | Returns the quest's current state |
| `GetQuestTasks` | `UFWQuestDefinition* QuestDef` | `TArray<FFWQuestTaskInstance>` | Returns runtime task instances |
| `CanAcceptQuest` | `UFWQuestDefinition* QuestDef` | `bool` | Evaluates all conditions |
| `IsQuestAvailable` | `UFWQuestDefinition* QuestDef` | `bool` | Whether the quest is in Available state |
| `GetActiveQuests` | *(none)* | `TArray<UFWQuestDefinition*>` | Returns all quests in Active state |
| `GetCompletedQuests` | *(none)* | `TArray<UFWQuestDefinition*>` | Returns all quests in Completed state |
| `GetQuestInstance` | `UFWQuestDefinition* QuestDef` | `FFWQuestInstance` | Returns the full runtime quest instance |
| `GetQuestsByCategory` | `EFWQuestCategory Category` | `TArray<UFWQuestDefinition*>` | Returns quests filtered by category |
| `GetQuestCompletionCount` | `UFWQuestDefinition* QuestDef` | `int32` | Returns how many times a repeatable quest has been completed |

### Delegates

| Delegate | Signature | Description |
|---|---|---|
| `OnQuestAccepted` | `(UFWQuestDefinition* QuestDef)` | Quest accepted |
| `OnQuestCompleted` | `(UFWQuestDefinition* QuestDef)` | Quest turned in, rewards granted |
| `OnQuestAbandoned` | `(UFWQuestDefinition* QuestDef)` | Quest abandoned |
| `OnTaskProgressChanged` | `(UFWQuestDefinition* QuestDef, int32 TaskIndex, int32 Current, int32 Required)` | Task progress updated |
| `OnQuestStateChanged` | `(UFWQuestDefinition* QuestDef, EFWQuestState OldState, EFWQuestState NewState)` | Any state transition |

---

## UFWQuestDefinition

**Parent:** `UPrimaryDataAsset`

### Properties

| Property | Type | Description |
|---|---|---|
| `QuestTag` | `FGameplayTag` | Unique identifier tag |
| `QuestName` | `FText` | Display name |
| `Description` | `FText` | Journal description |
| `Category` | `EFWQuestCategory` | Organizational category |
| `Priority` | `EFWQuestPriority` | Sort and notification priority |
| `RepeatType` | `EFWQuestRepeatType` | Repeat behavior |
| `RepeatCooldownSeconds` | `float` | Cooldown duration (Cooldown repeat type only) |
| `PartyMode` | `EFWQuestPartyMode` | Party synchronization mode |
| `Tasks` | `TArray<UFWQuestTaskBase*>` | Instanced task definitions |
| `Conditions` | `TArray<UFWQuestConditionBase*>` | Instanced acceptance conditions |
| `Rewards` | `TArray<UFWQuestRewardBase*>` | Instanced reward definitions |

---

## UFWQuestDatabase

**Parent:** `UDataAsset`

### Properties

| Property | Type | Description |
|---|---|---|
| `Quests` | `TArray<UFWQuestDefinition*>` | All quest definitions in this database |

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `GetAllQuests` | *(none)* | `const TArray<UFWQuestDefinition*>&` | Returns all quest definitions |
| `FindQuestByTag` | `const FGameplayTag& QuestTag` | `UFWQuestDefinition*` | Finds a quest by its tag |
| `GetQuestsByCategory` | `EFWQuestCategory Category` | `TArray<UFWQuestDefinition*>` | Filters quests by category |
| `GetQuestCount` | *(none)* | `int32` | Returns total number of quests |

---

## UFWQuestStateSubsystem

**Parent:** `UWorldSubsystem`

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `SaveQuestState` | `APlayerController* PC` | `FString` | Serializes quest state to JSON |
| `LoadQuestState` | `APlayerController* PC, const FString& JsonData` | `bool` | Deserializes quest state from JSON |
| `GetDailyResetTime` | *(none)* | `FDateTime` | Returns the next daily reset time |
| `GetWeeklyResetTime` | *(none)* | `FDateTime` | Returns the next weekly reset time |
| `ProcessDailyReset` | *(none)* | `void` | Resets all daily quests for all players |
| `ProcessWeeklyReset` | *(none)* | `void` | Resets all weekly quests for all players |
| `RegisterQuestManager` | `UFWQuestManagerComponent* Manager` | `void` | Registers a player's quest manager |
| `UnregisterQuestManager` | `UFWQuestManagerComponent* Manager` | `void` | Unregisters a player's quest manager |

---

## Task Base Classes

### UFWQuestTaskBase

| Function | Parameters | Return | Description |
|---|---|---|---|
| `ProcessEvent` | `const FFWQuestEvent& QuestEvent` | `int32` | Returns progress increment for this event (0 = no match) |
| `GetTaskDescription` | *(none)* | `FText` | Returns formatted task description |
| `IsRelevantEvent` | `const FFWQuestEvent& QuestEvent` | `bool` | Quick check before full processing |

### Task Type Properties

=== "FWTask_Slay"

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `TargetTag` | `FGameplayTag` | *(none)* | Enemy tag to match |
    | `RequiredProgress` | `int32` | `1` | Kill count required |

=== "FWTask_Travel"

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `LocationTag` | `FGameplayTag` | *(none)* | Location tag to match |
    | `ProximityRadius` | `float` | `500.0` | Optional proximity check radius |

=== "FWTask_Interact"

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `InteractTag` | `FGameplayTag` | *(none)* | Interactable object tag |

=== "FWTask_TalkTo"

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `NPCTag` | `FGameplayTag` | *(none)* | NPC to talk to |

=== "FWTask_Collect"

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `ItemTag` | `FGameplayTag` | *(none)* | Item tag to match |
    | `RequiredProgress` | `int32` | `1` | Item count required |

=== "FWTask_Timer"

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `TimeLimitSeconds` | `float` | `300.0` | Time limit before quest fails |
    | `bShowCountdown` | `bool` | `true` | Whether to display a countdown UI |

=== "FWTask_Custom"

    No built-in properties. Override `ProcessEvent` in Blueprints to define custom logic.

---

## Condition Base Classes

### UFWQuestConditionBase

| Function | Parameters | Return | Description |
|---|---|---|---|
| `Evaluate` | `const UFWQuestManagerComponent* QM` | `bool` | Returns whether the condition is met |
| `GetFailureReason` | *(none)* | `FText` | Returns a user-facing explanation of why the condition failed |

---

## Reward Base Classes

### UFWQuestRewardBase

| Function | Parameters | Return | Description |
|---|---|---|---|
| `Grant` | `AActor* QuestOwner` | `void` | Applies the reward to the quest owner |
| `GetDescription` | *(none)* | `FText` | Returns a display-friendly reward description |
| `GetIcon` | *(none)* | `UTexture2D*` | Returns an optional icon for UI display |

---

## Next Steps

- See [Types Reference](types.md) for detailed struct and enum documentation.
- See [Events and Delegates](events-delegates.md) for event flow diagrams.
- See [Configuration](configuration.md) for tuning quest behavior.
