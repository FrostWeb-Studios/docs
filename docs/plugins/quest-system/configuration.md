---
title: Configuration - FWQuestSystem
description: Plugin settings, reset schedules, serialization options, and project-level configuration for FWQuestSystem.
---

# Configuration

This page covers all configurable settings for FWQuestSystem, from reset schedules to JSON serialization and party synchronization.

---

## Quest Manager Settings

Configure the quest manager component via `DefaultGame.ini` or at runtime.

### Default Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `MaxActiveQuests` | `int32` | `25` | Maximum number of simultaneously active quests |
| `bAutoTrackNewQuests` | `bool` | `true` | Whether newly accepted quests are automatically tracked in the HUD |
| `bAutoTurnIn` | `bool` | `false` | Whether quests auto-turn-in when all tasks complete (skips NPC turn-in) |
| `bNotifyOnProgress` | `bool` | `true` | Whether to fire progress notifications for UI display |
| `MaxCompletionHistory` | `int32` | `100` | Maximum number of completed quests to retain in history |

### DefaultGame.ini

```ini
[/Script/FWQuestSystem.FWQuestManagerSettings]
MaxActiveQuests=25
bAutoTrackNewQuests=true
bAutoTurnIn=false
bNotifyOnProgress=true
MaxCompletionHistory=100
```

### Runtime Configuration

=== "C++"

    ```cpp
    QuestManager->SetMaxActiveQuests(30);
    QuestManager->SetAutoTurnIn(true);
    ```

=== "Blueprint"

    Set properties on the Quest Manager Component in the Details panel or via Blueprint setter nodes.

---

## Reset Schedules

Daily and weekly quest resets are managed by `UFWQuestStateSubsystem`.

### Reset Times

| Property | Type | Default | Description |
|---|---|---|---|
| `DailyResetHourUTC` | `int32` | `0` | Hour (0-23) in UTC when daily quests reset |
| `WeeklyResetDayOfWeek` | `int32` | `1` | Day of week (0=Sunday, 1=Monday, ..., 6=Saturday) |
| `WeeklyResetHourUTC` | `int32` | `0` | Hour (0-23) in UTC when weekly quests reset |

```ini
[/Script/FWQuestSystem.FWQuestStateSubsystem]
DailyResetHourUTC=6
WeeklyResetDayOfWeek=2
WeeklyResetHourUTC=6
```

This example sets daily reset at 06:00 UTC and weekly reset at Tuesday 06:00 UTC.

### Reset Behavior

When a reset occurs:

1. All quests with matching `RepeatType` (`Daily` or `Weekly`) that are in `Completed`, `Failed`, or `Abandoned` state are transitioned to `Available`.
2. Task progress is cleared.
3. `OnQuestStateChanged` fires for each reset quest.
4. Completion count is preserved (for tracking total completions).

!!! info "Cooldown Resets"
    `EFWQuestRepeatType::Cooldown` does not use the global reset schedule. Each quest tracks its own cooldown timer independently, starting when the quest is turned in.

### Manual Reset Trigger

For testing or server-side administrative control:

```cpp
UFWQuestStateSubsystem* QuestState = GetWorld()->GetSubsystem<UFWQuestStateSubsystem>();
QuestState->ProcessDailyReset();   // Reset all daily quests now
QuestState->ProcessWeeklyReset();  // Reset all weekly quests now
```

---

## Party Synchronization Settings

Party quest behavior is configured per-quest via `EFWQuestPartyMode` on the quest definition. Global party settings:

| Property | Type | Default | Description |
|---|---|---|---|
| `PartyProgressRadius` | `float` | `5000.0` | Maximum distance for party progress sharing (SyncProgress mode) |
| `bRequirePartyMemberInRange` | `bool` | `true` | Whether party members must be within radius for shared progress |
| `bAutoShareQuestOnAccept` | `bool` | `false` | Whether accepting a ShareOnly quest notifies party members |

```ini
[/Script/FWQuestSystem.FWQuestManagerSettings]
PartyProgressRadius=5000.0
bRequirePartyMemberInRange=true
bAutoShareQuestOnAccept=false
```

### Party Mode Details

=== "None"

    No party interaction. Quest is fully individual.

=== "ShareOnly"

    Party members who have the quest see each other's progress in the UI but progress is tracked independently. Optionally, accepting the quest sends a notification to party members.

=== "SyncProgress"

    Quest events from any party member within `PartyProgressRadius` contribute to all party members' progress. Example: If Player A kills a wolf and Player B is within range, both get credit.

    ```cpp
    // Server-side: when an enemy is killed, the quest manager checks for
    // nearby party members with the same quest active and dispatches
    // the event to all of them.
    ```

=== "RequireAll"

    All party members must be present (within radius) and have the quest active for any progress to count. If one member is out of range or does not have the quest, the event is ignored for everyone.

    !!! warning "Design Consideration"
        RequireAll mode is strict. Consider using it only for group content (dungeons, world bosses) where party coordination is expected.

---

## JSON Serialization

Quest state can be serialized to JSON for save files, server persistence, or cloud saves.

### Save

```cpp
UFWQuestStateSubsystem* QuestState = GetWorld()->GetSubsystem<UFWQuestStateSubsystem>();
FString JsonData = QuestState->SaveQuestState(PlayerController);
// Store JsonData in your save system
```

### Load

```cpp
FString JsonData = LoadFromSaveSystem(PlayerController);
bool bSuccess = QuestState->LoadQuestState(PlayerController, JsonData);
```

### JSON Structure

```json
{
    "version": 3,
    "playerId": "player-unique-id",
    "savedAt": "2026-02-27T12:00:00Z",
    "activeQuests": [
        {
            "questTag": "Quest.Side.SlayWolves",
            "state": "Active",
            "acceptedAt": "2026-02-27T10:30:00Z",
            "tasks": [
                {
                    "taskIndex": 0,
                    "currentProgress": 3,
                    "requiredProgress": 5,
                    "state": "Active"
                }
            ]
        }
    ],
    "completedQuests": [
        {
            "questTag": "Quest.MainStory.TheBeginning",
            "completedAt": "2026-02-26T15:00:00Z",
            "completionCount": 1
        }
    ],
    "dailyResetTimestamp": "2026-02-28T00:00:00Z",
    "weeklyResetTimestamp": "2026-03-02T00:00:00Z"
}
```

### Serialization Settings

| Property | Type | Default | Description |
|---|---|---|---|
| `bIncludeCompletedQuests` | `bool` | `true` | Whether to include completed quest history in serialization |
| `bIncludeAbandonedQuests` | `bool` | `false` | Whether to include abandoned quest records |
| `bPrettyPrint` | `bool` | `false` | Whether to format JSON with indentation (development only) |

```ini
[/Script/FWQuestSystem.FWQuestStateSubsystem]
bIncludeCompletedQuests=true
bIncludeAbandonedQuests=false
bPrettyPrint=false
```

---

## Logging

FWQuestSystem uses the `LogFWQuest` log category. Configure verbosity in `DefaultEngine.ini`:

```ini
[Core.Log]
LogFWQuest=Verbose
```

| Level | What is logged |
|---|---|
| `Error` | Failed quest operations, serialization errors, missing definitions |
| `Warning` | Rejected quest acceptance (conditions not met), task mismatches |
| `Log` | Quest lifecycle events (accept, complete, abandon, reset) |
| `Verbose` | Event dispatching, task matching details, serialization traffic |

---

## Gameplay Tag Configuration

FWQuestSystem uses Gameplay Tags extensively for event matching. Ensure your project's tag hierarchy includes:

```
Quest
    Quest.Event
        Quest.Event.EnemyKilled
        Quest.Event.LocationReached
        Quest.Event.NPCInteracted
        Quest.Event.ItemCollected
        Quest.Event.ItemCrafted
        Quest.Event.TimerExpired
```

!!! tip "Custom Event Tags"
    You can define additional event tags for custom task types. The tag hierarchy is flexible -- tasks match using `MatchesTag` which supports parent-child matching. For example, a task listening for `Quest.Event.EnemyKilled` will match events tagged with `Quest.Event.EnemyKilled.Boss`.

---

## Next Steps

- See [Data Definitions](data-definitions.md) for how configuration affects quest asset setup.
- See [Tutorial](tutorial.md) for a complete implementation with daily resets and party sync.
- See [Quest Manager Component](quest-manager-component.md) for runtime configuration APIs.
