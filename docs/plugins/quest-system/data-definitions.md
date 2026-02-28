---
title: Data Definitions - FWQuestSystem
description: Quest definitions, databases, task types, condition types, and reward types for the FWQuestSystem plugin.
---

# Data Definitions

FWQuestSystem uses Unreal Engine's DataAsset system for all quest content. This page documents every asset type, its properties, and how they compose together.

---

## UFWQuestDatabase (DataAsset)

A collection of quest definitions. Assign this to the `UFWQuestManagerComponent` to make quests available to the player.

```cpp
UCLASS(BlueprintType)
class UFWQuestDatabase : public UDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quests")
    TArray<TObjectPtr<UFWQuestDefinition>> Quests;

    /** Returns all quest definitions in this database. */
    UFUNCTION(BlueprintPure, Category = "Quests")
    const TArray<UFWQuestDefinition*>& GetAllQuests() const;

    /** Finds a quest definition by its unique ID tag. */
    UFUNCTION(BlueprintPure, Category = "Quests")
    UFWQuestDefinition* FindQuestByTag(const FGameplayTag& QuestTag) const;
};
```

!!! tip "Multiple Databases"
    You can create multiple databases (e.g., per region, per expansion) and swap them at runtime using `SetQuestDatabase`. Only quests in the active database are available for acceptance.

---

## UFWQuestDefinition (PrimaryDataAsset)

The core quest definition. Each quest is a standalone Data Asset that defines its metadata, tasks, acceptance conditions, and rewards.

```cpp
UCLASS(BlueprintType)
class UFWQuestDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    /** Unique identifier tag for this quest. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest")
    FGameplayTag QuestTag;

    /** Display name shown in the quest journal. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest")
    FText QuestName;

    /** Detailed description shown in the quest journal. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest")
    FText Description;

    /** Quest category for journal organization. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest")
    EFWQuestCategory Category;

    /** Sorting and notification priority. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest")
    EFWQuestPriority Priority;

    /** Whether and how this quest can be repeated. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest")
    EFWQuestRepeatType RepeatType;

    /** Cooldown duration in seconds (only used with RepeatType::Cooldown). */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest",
        meta = (EditCondition = "RepeatType == EFWQuestRepeatType::Cooldown"))
    float RepeatCooldownSeconds;

    /** Party synchronization mode. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Quest")
    EFWQuestPartyMode PartyMode;

    /** Tasks that must be completed to finish this quest. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Instanced, Category = "Tasks")
    TArray<TObjectPtr<UFWQuestTaskBase>> Tasks;

    /** Conditions that must be met before this quest can be accepted. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Instanced, Category = "Conditions")
    TArray<TObjectPtr<UFWQuestConditionBase>> Conditions;

    /** Rewards granted when the quest is turned in. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Instanced, Category = "Rewards")
    TArray<TObjectPtr<UFWQuestRewardBase>> Rewards;
};
```

---

## Task Types

All tasks extend `UFWQuestTaskBase`. Tasks are **instanced** -- each quest definition contains its own configured task objects.

### UFWQuestTaskBase (Abstract)

```cpp
UCLASS(Abstract, BlueprintType, Blueprintable, EditInlineNew, DefaultToInstanced)
class UFWQuestTaskBase : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Task")
    FText TaskName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Task")
    FText Description;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Task")
    int32 RequiredProgress = 1;

    /** Override to handle incoming quest events. Return progress increment. */
    UFUNCTION(BlueprintNativeEvent, Category = "Task")
    int32 ProcessEvent(const FFWQuestEvent& QuestEvent) const;
};
```

### Built-in Task Types

| Task Class | Match Criteria | Properties |
|---|---|---|
| `UFWTask_Slay` | `Quest.Event.EnemyKilled` + `TargetTag` match | `TargetTag` (FGameplayTag), `RequiredCount` (int32) |
| `UFWTask_Travel` | `Quest.Event.LocationReached` + `LocationTag` match | `LocationTag` (FGameplayTag) |
| `UFWTask_Interact` | `Quest.Event.NPCInteracted` + `InteractTag` match | `InteractTag` (FGameplayTag) |
| `UFWTask_TalkTo` | `Quest.Event.NPCInteracted` + `NPCTag` match | `NPCTag` (FGameplayTag) |
| `UFWTask_Collect` | `Quest.Event.ItemCollected` + `ItemTag` match | `ItemTag` (FGameplayTag), `RequiredCount` (int32) |
| `UFWTask_Timer` | Time-based, auto-fails quest on expiry | `TimeLimitSeconds` (float) |
| `UFWTask_Custom` | Blueprint-defined `ProcessEvent` override | Any custom properties |

### UFWTask_Slay Example

```cpp
UCLASS(BlueprintType, EditInlineNew, DefaultToInstanced)
class UFWTask_Slay : public UFWQuestTaskBase
{
    GENERATED_BODY()

public:
    /** Gameplay tag that must match the killed enemy's tag. */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Task|Slay")
    FGameplayTag TargetTag;

    virtual int32 ProcessEvent_Implementation(const FFWQuestEvent& QuestEvent) const override
    {
        if (QuestEvent.EventTag.MatchesTag(FGameplayTag::RequestGameplayTag("Quest.Event.EnemyKilled"))
            && QuestEvent.TargetTag.MatchesTagExact(TargetTag))
        {
            return QuestEvent.Count;
        }
        return 0;
    }
};
```

### Creating Custom Tasks in Blueprint

1. Create a new Blueprint class with parent `FWQuestTaskBase`.
2. Add any custom properties you need.
3. Override **Process Event** to define your matching logic.
4. Return the progress increment (0 for no match).

=== "Example: Craft Task"

    ```
    Properties:
        RecipeTag: Gameplay Tag
        RequiredCount: Integer = 1

    Process Event:
        If Event Tag matches "Quest.Event.ItemCrafted"
        AND Event Target Tag matches RecipeTag
            Return Event Count
        Else
            Return 0
    ```

---

## Condition Types

All conditions extend `UFWQuestConditionBase`. Conditions are evaluated when `CanAcceptQuest` or `AcceptQuest` is called.

### UFWQuestConditionBase (Abstract)

```cpp
UCLASS(Abstract, BlueprintType, Blueprintable, EditInlineNew, DefaultToInstanced)
class UFWQuestConditionBase : public UObject
{
    GENERATED_BODY()

public:
    /** Override to evaluate whether this condition is met. */
    UFUNCTION(BlueprintNativeEvent, Category = "Condition")
    bool Evaluate(const UFWQuestManagerComponent* QuestManager) const;
};
```

### Built-in Condition Types

| Condition Class | Evaluates | Properties |
|---|---|---|
| `UFWCondition_QuestComplete` | Whether a specific quest has been completed | `RequiredQuestTag` (FGameplayTag) |
| `UFWCondition_Level` | Whether the player meets a minimum level | `MinimumLevel` (int32) |
| `UFWCondition_HasItem` | Whether the player has a specific item | `ItemTag` (FGameplayTag), `RequiredCount` (int32) |
| `UFWCondition_Reputation` | Whether faction reputation meets a threshold | `FactionTag` (FGameplayTag), `MinimumReputation` (float) |
| `UFWCondition_Time` | Whether the current time is within a window | `StartHour` (int32), `EndHour` (int32), `DaysOfWeek` (bitmask) |
| `UFWCondition_And` | Whether ALL child conditions pass | `Conditions` (TArray\<UFWQuestConditionBase*\>) |
| `UFWCondition_Or` | Whether ANY child condition passes | `Conditions` (TArray\<UFWQuestConditionBase*\>) |
| `UFWCondition_Custom` | Blueprint-defined Evaluate override | Any custom properties |

### Composable Conditions Example

Use `FWCondition_And` and `FWCondition_Or` to build complex prerequisite trees:

```
FWCondition_And
    +-- FWCondition_Level (MinimumLevel = 10)
    +-- FWCondition_Or
    |       +-- FWCondition_QuestComplete (RequiredQuestTag = "Quest.Wolves")
    |       +-- FWCondition_Reputation (FactionTag = "Faction.Village", MinReputation = 50)
    +-- FWCondition_HasItem (ItemTag = "Item.MapFragment", RequiredCount = 1)
```

This tree requires: level 10 AND (completed Wolves quest OR 50+ Village reputation) AND has a Map Fragment.

---

## Reward Types

All rewards extend `UFWQuestRewardBase`. Rewards are granted when `TurnInQuest` is called.

### UFWQuestRewardBase (Abstract)

```cpp
UCLASS(Abstract, BlueprintType, Blueprintable, EditInlineNew, DefaultToInstanced)
class UFWQuestRewardBase : public UObject
{
    GENERATED_BODY()

public:
    /** Override to grant this reward to the player. */
    UFUNCTION(BlueprintNativeEvent, Category = "Reward")
    void Grant(AActor* QuestOwner) const;

    /** Override to return a display-friendly description of this reward. */
    UFUNCTION(BlueprintNativeEvent, Category = "Reward")
    FText GetDescription() const;
};
```

### Built-in Reward Types

| Reward Class | Grants | Properties |
|---|---|---|
| `UFWReward_Experience` | Character XP | `XPAmount` (int32) |
| `UFWReward_Item` | Items to inventory | `ItemTag` (FGameplayTag), `Count` (int32) |
| `UFWReward_SkillXP` | XP to a specific skill | `SkillTag` (FGameplayTag), `XPAmount` (int32) |
| `UFWReward_Reputation` | Faction reputation | `FactionTag` (FGameplayTag), `Amount` (float) |
| `UFWReward_Currency` | In-game currency | `CurrencyType` (FGameplayTag), `Amount` (int32) |
| `UFWReward_Unlock` | Unlocks content | `UnlockTag` (FGameplayTag), `Description` (FText) |
| `UFWReward_Custom` | Blueprint-defined Grant override | Any custom properties |

### Multiple Rewards

Quests can grant multiple rewards. All rewards in the array are granted sequentially when the quest is turned in:

```
Quest: The Dark Cave
Rewards:
    +-- FWReward_Experience (XPAmount = 500)
    +-- FWReward_Item (ItemTag = "Item.DragonScale", Count = 3)
    +-- FWReward_Currency (CurrencyType = "Currency.Gold", Amount = 1000)
    +-- FWReward_Reputation (FactionTag = "Faction.DragonHunters", Amount = 25)
```

---

## Asset Organization

Recommended folder structure for quest content:

```
Content/
    Quests/
        DA_QuestDatabase.uasset
        MainStory/
            QD_MS_01_TheBeginning.uasset
            QD_MS_02_TheJourney.uasset
        Side/
            QD_Side_SlayWolves.uasset
            QD_Side_FindTheMap.uasset
        Daily/
            QD_Daily_GatherHerbs.uasset
            QD_Daily_PatrolRoute.uasset
        Weekly/
            QD_Weekly_DungeonClear.uasset
```

!!! tip "Naming Convention"
    Prefix quest definitions with `QD_` followed by the category abbreviation and a descriptive name. This makes assets easy to find in the Content Browser and avoids naming collisions.

---

## Next Steps

- See [Types Reference](types.md) for all enum and struct definitions.
- See [Quest Manager Component](quest-manager-component.md) for how to use these definitions at runtime.
- Follow the [Tutorial](tutorial.md) to build a complete quest chain using these asset types.
