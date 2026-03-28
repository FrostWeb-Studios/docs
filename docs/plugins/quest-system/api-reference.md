---
title: API Reference - FWQuestSystem
description: Complete function, property, and delegate reference for all FWQuestSystem classes.
---

# API Reference

Complete reference for all public functions, properties, and delegates across FWQuestSystem classes.

---

## UFWQuestManagerComponent

**Parent:** `UActorComponent`

The primary interface for quest operations. Add this component to your Player Controller or Pawn to manage quests at runtime.

### Quest Actions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `AcceptQuest` | `QuestDef` | `void` | Accepts a quest after validating all conditions |
| `AbandonQuest` | `QuestDef` | `void` | Abandons an active quest and resets task progress |
| `TurnInQuest` | `QuestDef` | `void` | Turns in a completed quest and grants all rewards |
| `FailQuest` | `QuestDef` | `void` | Marks a quest as failed |
| `SetQuestDatabase` | `Database` | `void` | Sets the active quest database |

### Event-Based Progress

| Function | Parameters | Return | Description |
|---|---|---|---|
| `ProcessQuestEvent` | `QuestEvent` | `void` | Dispatches a quest event to all active tasks for matching |
| `OnEnemyKilled` | `EnemyTag, Count` | `void` | Convenience method to report an enemy kill |
| `OnLocationReached` | `LocationTag` | `void` | Convenience method to report location arrival |
| `OnNPCInteracted` | `NPCTag` | `void` | Convenience method to report NPC interaction |
| `OnItemCollected` | `ItemTag, Count` | `void` | Convenience method to report item collection |

### Queries

| Function | Return | Description |
|---|---|---|
| `GetQuestDatabase` | `QuestDatabase` | Returns the active quest database |
| `GetQuestState` | `EFWQuestState` | Returns the quest's current state |
| `GetQuestTasks` | `Array of TaskInstances` | Returns runtime task instances for a quest |
| `CanAcceptQuest` | `bool` | Evaluates all prerequisites and conditions |
| `IsQuestAvailable` | `bool` | Whether the quest is in the Available state |
| `GetActiveQuests` | `Array of QuestDefs` | Returns all quests in Active state |
| `GetCompletedQuests` | `Array of QuestDefs` | Returns all quests in Completed state |
| `GetQuestInstance` | `QuestInstance` | Returns the full runtime quest instance |
| `GetQuestsByCategory` | `Array of QuestDefs` | Returns quests filtered by category |
| `GetQuestCompletionCount` | `int32` | Returns how many times a repeatable quest has been completed |

### Party Integration

| Function | Parameters | Return | Description |
|---|---|---|---|
| `ShareQuestWithParty` | `QuestDef` | `void` | Shares a quest with party members based on party mode |
| `SyncPartyProgress` | `QuestDef` | `void` | Synchronizes task progress across party members |

### Save / Load

| Function | Parameters | Return | Description |
|---|---|---|---|
| `SaveQuestData` | -- | `FString` | Serializes all quest state to a JSON string |
| `LoadQuestData` | `JsonData` | `bool` | Restores quest state from a JSON string |

### Events (Delegates)

All delegates are `BlueprintAssignable`.

| Delegate | Signature | Description |
|---|---|---|
| `OnQuestAccepted` | `(QuestDef)` | Fired when a quest is accepted |
| `OnQuestCompleted` | `(QuestDef)` | Fired when a quest is turned in and rewards granted |
| `OnQuestAbandoned` | `(QuestDef)` | Fired when a quest is abandoned |
| `OnQuestFailed` | `(QuestDef)` | Fired when a quest fails |
| `OnQuestAvailable` | `(QuestDef)` | Fired when a quest becomes available |
| `OnQuestStateChanged` | `(QuestDef, OldState, NewState)` | Fired on any state transition |
| `OnTaskProgressChanged` | `(QuestDef, TaskIndex, Current, Required)` | Fired when task progress updates |
| `OnTaskCompleted` | `(QuestDef, TaskIndex)` | Fired when an individual task is completed |
| `OnQuestChainAdvanced` | `(QuestDef, NextQuestDef)` | Fired when a quest chain advances to the next quest |
| `OnDailyQuestsReset` | `()` | Fired when daily quests reset |
| `OnWeeklyQuestsReset` | `()` | Fired when weekly quests reset |
| `OnQuestDatabaseChanged` | `(Database)` | Fired when the active quest database changes |

---

## UFWQuestStateSubsystem

**Parent:** `UWorldSubsystem`

World-level subsystem that manages global quest state, completion history, cooldowns, and daily/weekly reset scheduling.

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `FindQuest` | `QuestTag` | `QuestDefinition` | Looks up a quest definition by gameplay tag |
| `RecordQuestCompletion` | `PlayerController, QuestDef` | `void` | Records a quest completion for history tracking |
| `IsQuestEverCompleted` | `PlayerController, QuestDef` | `bool` | Whether the player has ever completed this quest |
| `GetDailyResetTime` | -- | `FDateTime` | Returns the next daily reset time |
| `GetWeeklyResetTime` | -- | `FDateTime` | Returns the next weekly reset time |
| `ProcessDailyReset` | -- | `void` | Resets all daily quests for all registered players |
| `ProcessWeeklyReset` | -- | `void` | Resets all weekly quests for all registered players |
| `RegisterQuestManager` | `Manager` | `void` | Registers a player's quest manager with the subsystem |
| `UnregisterQuestManager` | `Manager` | `void` | Unregisters a player's quest manager |

---

## Data Assets

### UFWQuestDatabase

A collection asset that holds references to all quest definitions in your project or a specific content set.

| Property | Type | Description |
|---|---|---|
| `Quests` | `Array of QuestDefinitions` | All quest definitions in this database |

| Function | Return | Description |
|---|---|---|
| `GetAllQuests` | `Array of QuestDefinitions` | Returns all quest definitions |
| `FindQuestByTag` | `QuestDefinition` | Finds a quest by its gameplay tag |
| `GetQuestsByCategory` | `Array of QuestDefinitions` | Filters quests by category |
| `GetQuestCount` | `int32` | Returns total number of quests |

### UFWQuestDefinition

A data asset representing a single quest with all its configuration.

#### Identity

| Property | Type | Description |
|---|---|---|
| `QuestTag` | `FGameplayTag` | Unique identifier tag for this quest |
| `QuestId` | `FName` | Unique name identifier |

#### Classification

| Property | Type | Description |
|---|---|---|
| `Category` | `EFWQuestCategory` | Organizational category (MainStory, Side, Daily, etc.) |
| `Priority` | `EFWQuestPriority` | Sort and notification priority |

#### Display

| Property | Type | Description |
|---|---|---|
| `QuestName` | `FText` | Localized display name |
| `Description` | `FText` | Journal description |
| `ObjectiveText` | `FText` | Short objective summary |
| `Icon` | `Texture2D` | Optional quest icon |

#### Tasks

| Property | Type | Description |
|---|---|---|
| `Tasks` | `Array of QuestTasks` | Instanced task definitions (Slay, Collect, Travel, etc.) |
| `bRequireAllTasks` | `bool` | Whether all tasks must be completed, or just one |

#### Prerequisites

| Property | Type | Description |
|---|---|---|
| `Conditions` | `Array of Conditions` | All conditions that must be met to accept the quest |

#### Rewards

| Property | Type | Description |
|---|---|---|
| `Rewards` | `Array of Rewards` | Rewards granted on quest completion |

#### Time

| Property | Type | Description |
|---|---|---|
| `bHasTimeLimit` | `bool` | Whether the quest has a time limit |
| `TimeLimitSeconds` | `float` | Time limit duration in seconds |

#### Repeat

| Property | Type | Description |
|---|---|---|
| `RepeatType` | `EFWQuestRepeatType` | Repeat behavior (None, Cooldown, Daily, Weekly, Unlimited) |
| `RepeatCooldownSeconds` | `float` | Cooldown between repeats (Cooldown type only) |
| `MaxCompletions` | `int32` | Maximum number of completions (0 = unlimited) |

#### Party

| Property | Type | Description |
|---|---|---|
| `PartyMode` | `EFWQuestPartyMode` | Party synchronization mode |

#### Chain

| Property | Type | Description |
|---|---|---|
| `NextQuestInChain` | `QuestDefinition` | Next quest to auto-offer when this quest completes |
| `ChainId` | `FGameplayTag` | Identifier for the quest chain this quest belongs to |

### UFWQuestCategory

A data asset for defining custom quest categories beyond the built-in set.

| Property | Type | Description |
|---|---|---|
| `CategoryName` | `FText` | Display name |
| `CategoryTag` | `FGameplayTag` | Unique tag identifier |
| `Icon` | `Texture2D` | Category icon |
| `SortOrder` | `int32` | UI sort priority |

---

## Task System

### Base Task

All tasks share the following interface and base properties.

| Property | Type | Description |
|---|---|---|
| `TaskDescription` | `FText` | Display description for the task |
| `bIsOptional` | `bool` | Whether this task is optional for quest completion |
| `TriggerType` | `EFWTaskTrigger` | How this task receives progress (Event, Automatic, Manual) |

| Function | Return | Description |
|---|---|---|
| `ProcessEvent` | `int32` | Returns progress increment for a given quest event (0 = no match) |
| `GetTaskDescription` | `FText` | Returns the formatted task description |
| `IsRelevantEvent` | `bool` | Quick check before full event processing |

### Built-In Task Types

| Task | Key Properties | Description |
|---|---|---|
| **Slay** | `TargetTag`, `RequiredCount` | Kill a number of enemies matching the tag |
| **Collect** | `ItemTag`, `RequiredCount` | Collect items matching the tag |
| **Travel** | `LocationTag`, `ProximityRadius` | Reach a location within the specified radius |
| **TalkTo** | `NPCTag` | Talk to a specific NPC |
| **Interact** | `InteractTag` | Interact with a specific object |
| **Timer** | `TimeLimitSeconds`, `bShowCountdown` | Complete the quest within a time limit |
| **Custom** | *(user-defined)* | Override `ProcessEvent` in Blueprints for custom logic |

---

## Condition System

### Base Condition

| Function | Return | Description |
|---|---|---|
| `Evaluate` | `bool` | Returns whether the condition is met |
| `GetFailureReason` | `FText` | Returns a user-facing explanation of why the condition failed |

### Built-In Condition Types

| Condition | Key Properties | Description |
|---|---|---|
| **Level** | `MinLevel` | Requires a minimum player level |
| **QuestComplete** | `RequiredQuestTag` | Requires a specific quest to have been completed |
| **HasItem** | `ItemTag`, `RequiredCount` | Requires the player to possess specific items |
| **Reputation** | `FactionTag`, `MinReputation` | Requires a minimum faction reputation |
| **Time** | `StartTime`, `EndTime` | Requires the current time to be within a window |
| **And** | `ChildConditions` | All child conditions must pass |
| **Or** | `ChildConditions` | Any child condition must pass |
| **Custom** | *(user-defined)* | Override `Evaluate` in Blueprints for custom logic |

---

## Reward System

### Base Reward

| Function | Return | Description |
|---|---|---|
| `Grant` | `void` | Applies the reward to the quest owner |
| `GetDescription` | `FText` | Returns a display-friendly reward description |
| `GetIcon` | `Texture2D` | Returns an optional icon for UI display |

### Built-In Reward Types

| Reward | Key Properties | Description |
|---|---|---|
| **Experience** | `Amount` | Grants experience points |
| **Currency** | `CurrencyType`, `Amount` | Grants a specified currency |
| **Item** | `ItemDefinition`, `Quantity` | Grants items to the player's inventory |
| **Reputation** | `FactionTag`, `Amount` | Grants faction reputation |
| **SkillXP** | `SkillTag`, `Amount` | Grants experience to a specific skill |
| **Unlock** | `UnlockTag` | Unlocks content (recipes, areas, features, etc.) |
| **Custom** | *(user-defined)* | Override `Grant` in Blueprints for custom logic |

---

## Core Types

### Quest States (EFWQuestState)

| Value | Description |
|---|---|
| `Unavailable` | Conditions not met |
| `Available` | Ready to accept |
| `Active` | In progress |
| `Completed` | Turned in, rewards granted |
| `Failed` | Quest failed |

### Quest Categories (EFWQuestCategory)

| Value |
|---|
| `MainStory` |
| `Side` |
| `Daily` |
| `Weekly` |
| `World` |
| `Dungeon` |
| `Guild` |
| `Event` |
| `Profession` |
| `Challenge` |

### Quest Repeat Types (EFWQuestRepeatType)

| Value | Description |
|---|---|
| `None` | One-time quest |
| `Cooldown` | Repeatable after a cooldown period |
| `Daily` | Resets with the daily reset |
| `Weekly` | Resets with the weekly reset |
| `Unlimited` | Immediately repeatable |

### Task Trigger Types (EFWTaskTrigger)

| Value | Description |
|---|---|
| `Event` | Progress via quest events |
| `Automatic` | Progress tracked automatically by the system |
| `Manual` | Progress set manually via code or Blueprints |

### FFWQuestEvent

The payload dispatched to active tasks for progress matching.

| Field | Type | Description |
|---|---|---|
| `EventTag` | `FGameplayTag` | Identifies the type of event |
| `Instigator` | `Actor` | The actor that caused the event |
| `Target` | `Actor` | The target of the event (if applicable) |
| `Count` | `int32` | Quantity associated with the event |
| `Payload` | `GameplayTagContainer` | Additional tags for complex matching |

### FFWQuestContext

Runtime context for quest evaluation and processing.

| Field | Type | Description |
|---|---|---|
| `QuestManager` | `QuestManagerComponent` | The owning quest manager |
| `QuestDefinition` | `QuestDefinition` | The quest being evaluated |
| `PlayerController` | `PlayerController` | The owning player |

---

## Next Steps

- See [Types Reference](types.md) for additional struct and enum documentation.
- See [Events and Delegates](events-delegates.md) for event flow diagrams.
- See [Configuration](configuration.md) for tuning quest behavior.
