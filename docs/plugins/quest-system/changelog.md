---
title: Changelog - FWQuestSystem
description: Version history and release notes for the FWQuestSystem plugin.
---

# Changelog

All notable changes to FWQuestSystem are documented on this page.

---

## Version 3.0

**Release type:** Major | **Engine:** UE 5.3+

### Added

- **Party synchronization modes.** `EFWQuestPartyMode` with four modes: `None`, `ShareOnly`, `SyncProgress`, `RequireAll`. Party members can now share quest progress based on proximity and quest configuration.
- **Daily and weekly reset system.** `EFWQuestRepeatType::Daily` and `EFWQuestRepeatType::Weekly` with configurable reset times managed by `UFWQuestStateSubsystem`. Automatic reset scheduling with server-authoritative time tracking.
- **Quest priority system.** `EFWQuestPriority` (Low, Normal, High, Urgent) for UI sorting and notification filtering.
- **Expanded quest categories.** Added `World`, `Dungeon`, `Guild`, `Event`, `Profession`, and `Challenge` to `EFWQuestCategory`.
- **UFWQuestStateSubsystem.** New World Subsystem that manages quest state persistence, reset scheduling, and cross-player coordination at the world level.
- **JSON serialization.** Full quest state serialization via `SaveQuestState` and `LoadQuestState` on the state subsystem. Versioned JSON format with active quests, completed history, and reset timestamps.
- **Completion count tracking.** `FFWQuestInstance::CompletionCount` tracks how many times a repeatable quest has been completed.
- **New condition types.** `FWCondition_Time` (time window checks), `FWCondition_And` (boolean AND combinator), `FWCondition_Or` (boolean OR combinator), `FWCondition_Custom` (Blueprint-defined).
- **New reward types.** `FWReward_SkillXP` (skill-specific XP), `FWReward_Currency` (configurable currency types), `FWReward_Unlock` (content unlocks), `FWReward_Custom` (Blueprint-defined).
- **UFWTask_Timer.** Time-limited task type that auto-fails the quest when the timer expires.
- **UFWTask_Custom.** Blueprint-extensible task type with overridable `ProcessEvent`.
- **`GetFailureReason` on conditions.** Returns user-facing text explaining why a condition is not met, for better UI feedback.
- **`GetIcon` on rewards.** Optional icon texture for reward display in UI.
- **`GetQuestsByCategory` on both database and manager.** Filter quests by category for journal organization.
- **`OnQuestStateChanged` delegate.** Fires on any state transition with both old and new states.
- **`MaxCompletionHistory` setting.** Configurable limit on completed quest history to control memory usage.

### Changed

- **Quest definitions are now `UPrimaryDataAsset`.** Upgraded from `UDataAsset` to enable Asset Manager integration, async loading, and bundle management.
- **Tasks, conditions, and rewards are instanced.** All sub-objects use `EditInlineNew` + `DefaultToInstanced` specifiers, allowing per-quest configuration without shared state issues.
- **`ProcessQuestEvent` is the primary event entry point.** All convenience methods (`OnEnemyKilled`, `OnLocationReached`, `OnNPCInteracted`) now route through `ProcessQuestEvent` with standardized `FFWQuestEvent` payloads.
- **Task matching uses `MatchesTag` instead of exact match.** A task targeting `Enemy.Wolf` now matches events tagged `Enemy.Wolf.Forest` or `Enemy.Wolf.Alpha`, enabling tag hierarchy-based matching.
- **Condition evaluation passes the quest manager.** `Evaluate` now receives a `const UFWQuestManagerComponent*` parameter, allowing conditions to query quest state (e.g., `FWCondition_QuestComplete`).
- **Reward `Grant` receives `AActor*` instead of `APlayerController*`.** This supports granting rewards to non-player actors in AI or NPC quest scenarios.

### Removed

- **`FFWQuestProgress` struct.** Replaced by `FFWQuestInstance` and `FFWQuestTaskInstance` for clearer separation of quest-level and task-level state.
- **`UQuestSubsystem` (Game Instance subsystem).** Replaced by `UFWQuestStateSubsystem` (World Subsystem) for proper per-world isolation in PIE and server scenarios.
- **Hard-coded condition checks.** All conditions are now data-driven via `UFWQuestConditionBase` subclasses. The previous `MinLevel` and `RequiredQuestId` properties on the quest definition have been removed in favor of composable condition objects.

### Migration Guide

!!! warning "Breaking Changes"
    Version 3.0 changes the quest definition format, state management, and event processing pipeline. Manual migration is required.

**Key migration steps:**

1. **Convert quest definitions to `UPrimaryDataAsset`.** Re-create quest definition assets or use the Asset Manager's re-parenting tool. Move `MinLevel` and `RequiredQuestId` properties into condition objects (`FWCondition_Level`, `FWCondition_QuestComplete`).

2. **Update task references.** Tasks are now instanced sub-objects. If you had shared task assets, create instanced copies in each quest definition.

3. **Replace `FFWQuestProgress` usage.** Use `GetQuestInstance` to retrieve `FFWQuestInstance` and access `TaskInstances` for per-task progress.

4. **Replace `UQuestSubsystem` references.** Use `GetWorld()->GetSubsystem<UFWQuestStateSubsystem>()` instead of the Game Instance subsystem.

5. **Update event dispatching.** Replace direct task-type event calls with `ProcessQuestEvent(FFWQuestEvent)` for consistency. The convenience methods still exist but now wrap `ProcessQuestEvent`.

6. **Update condition evaluation.** If you had custom condition subclasses, update `Evaluate` to accept `const UFWQuestManagerComponent*` instead of `APlayerController*`.

7. **Update reward application.** If you had custom reward subclasses, update `Grant` to accept `AActor*` instead of `APlayerController*`. Cast to `APlayerController` internally if needed.

---

## Version 2.0

**Release type:** Major | **Engine:** UE 5.2+

### Added

- **DataAsset-based quest definitions.** Complete rewrite from Blueprint-only quest classes to `UDataAsset`-based definitions.
- **Instanced task system.** `UFWQuestTaskBase` with built-in `FWTask_Slay`, `FWTask_Travel`, `FWTask_Interact`, `FWTask_TalkTo`, `FWTask_Collect`.
- **Condition system.** `UFWQuestConditionBase` with `FWCondition_QuestComplete`, `FWCondition_Level`, `FWCondition_HasItem`, `FWCondition_Reputation`.
- **Reward system.** `UFWQuestRewardBase` with `FWReward_Experience`, `FWReward_Item`, `FWReward_Reputation`.
- **`UFWQuestDatabase`.** Central collection of quest definitions.
- **`OnTaskProgressChanged` delegate.** Per-task progress reporting.
- **`EFWQuestRepeatType::Cooldown` and `Unlimited`.** Basic repeatable quest support.
- **`EFWQuestCategory`.** MainStory, Side, Daily, Weekly categories.

### Changed

- **`UFWQuestManagerComponent`** completely rewritten with event-driven architecture.

### Removed

- **Blueprint-only quest classes.** Replaced by DataAsset definitions.

---

## Version 1.0

**Release type:** Initial | **Engine:** UE 5.1+

### Added

- Initial release of FWQuestSystem.
- `UFWQuestManagerComponent` for quest acceptance and tracking.
- Blueprint-only quest definition classes.
- Basic kill and collect task tracking.
- `OnQuestAccepted` and `OnQuestCompleted` delegates.
- Simple quest state machine (Available, Active, Completed).
