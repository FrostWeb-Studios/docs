---
title: FAQ - FWSkillSystem
---

# Frequently Asked Questions

Common questions about XP formulas, max levels, combat XP distribution, milestone triggers, and general usage.

---

## XP and Leveling

### What XP formula does FWSkillSystem use?

FWSkillSystem does not hardcode any XP formula. Instead, it uses `UFWSkillXPCurve` data assets that define the cumulative XP required for each level as an array of thresholds. You can implement any curve shape -- linear, polynomial, exponential, or completely custom -- by populating the array with the appropriate values.

!!! info "RS-Style Curve"
    If you want to replicate the classic RS XP curve, calculate your thresholds using:
    ```
    XP(L) = floor( (1/4) * sum(floor(L + 300 * 2^(L/7)), L=1..N-1) )
    ```
    Then enter the resulting values into your `UFWSkillXPCurve` asset.

### What is the default max level?

The default max level is **99**, configurable per-skill in `UFWSkillDefinition` or globally via `UFWSkillSystemSettings` in Project Settings. You can set any maximum level you want -- 50, 99, 120, or any other value.

### Can different skills have different max levels?

Yes. Each `UFWSkillDefinition` has its own `MaxLevel` property. For example, you could set Combat skills to max at 99 and Gathering skills to max at 120.

### Can different skills use different XP curves?

Yes. Each `UFWSkillDefinition` has an **XP Curve Override** property. If set, the skill uses that curve instead of the global default. This lets you make some skills faster or slower to level than others.

### What happens when a skill reaches max level?

When a skill reaches its maximum level, `GetXPToNextLevel` returns 0, `GetSkillLevelProgress` returns 1.0, and further calls to `AwardSkillXP` for that skill are silently ignored. The `OnSkillXPGained` event does not fire for XP awarded to a maxed skill.

### Can a single XP award cause multiple level-ups?

Yes. If a large XP award crosses multiple level thresholds, the `OnSkillLevelUp` event fires once for each level gained. For example, if a player at Level 3 receives enough XP to reach Level 7, the event fires for levels 4, 5, 6, and 7 in sequence.

### How is Total Level calculated?

`GetTotalLevel` returns the sum of all 15 skill levels. With a default max of 99 per skill, the theoretical maximum total level is 1,485.

---

## Combat XP

### How does AwardCombatXP distribute XP?

`AwardCombatXP` distributes the given XP amount across the four combat skills: Warfare, Marksmanship, Magic, and Bounty. The distribution depends on the combat context -- for example, a melee-focused encounter might weight more XP toward Warfare, while a ranged encounter weights toward Marksmanship.

### Does AwardCombatXP split or duplicate the XP amount?

The total amount is **split** across eligible combat skills, not duplicated. If you call `AwardCombatXP(100.0f)`, the total XP awarded across all combat skills sums to 100.

### Can I award XP to a specific combat skill instead?

Yes. Use `AwardSkillXP` with the specific `EFWSkillType` value (e.g., `EFWSkillType::Warfare`). `AwardCombatXP` is a convenience function for distributing XP based on combat context; you are not required to use it.

### What determines which combat skill receives the most XP?

The distribution is based on the skill definitions and the active combat state. The primary weapon or ability type used in the encounter determines the dominant skill. If no combat context is available, XP is distributed evenly.

### Can I customize the combat XP distribution logic?

Yes. Subclass `UFWSkillProgressionComponent` and override the combat XP distribution behavior to implement custom logic based on your game's combat system.

---

## Milestones

### When do milestones trigger?

Milestones trigger when `OnSkillLevelUp` fires and the new level equals or exceeds a milestone's `RequiredLevel`. The `UFWSkillMilestoneComponent` listens for level-up events and checks all milestones for the affected skill.

### Can a milestone trigger retroactively?

Yes. When `LoadSkillData` is called (e.g., loading a save file), the milestone component checks all milestones against the loaded levels and awards any that have not been previously granted.

### Are milestone rewards granted automatically?

The `UFWSkillMilestoneComponent` detects when milestones are reached. The actual reward delivery depends on your `UFWSkillMilestoneReward` subclass implementation. The component marks milestones as awarded to prevent duplicate grants.

### Can I define milestones in Blueprint?

Yes. Create Blueprint subclasses of `UFWSkillMilestoneReward` to define reward logic without C++. The `UFWSkillMilestoneDefinition` data asset can reference both C++ and Blueprint reward subclasses.

### How do I prevent duplicate milestone rewards on repeated logins?

The `UFWSkillMilestoneComponent` tracks which milestones have been awarded. As long as this tracking state is persisted alongside skill data (via your save system or backend), milestones will not be re-awarded.

---

## Skill Requirements

### How do I gate content behind a skill level?

Create a `UFWSkillRequirement` subclass (see [Configuration](configuration.md#skill-requirements)) and assign it to any object that needs gating -- items, quest steps, abilities, doors, etc. At runtime, call `MeetsRequirement` on the player's `UFWSkillProgressionComponent` to check.

### Can I combine multiple skill requirements?

Yes. Create a composite `UFWSkillRequirement` subclass that holds an array of child requirements and checks all of them:

```cpp
UCLASS()
class UFWSkillRequirement_All : public UFWSkillRequirement
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, Instanced, Category = "Requirement")
    TArray<TObjectPtr<UFWSkillRequirement>> Requirements;

    virtual bool IsMet(const UFWSkillProgressionComponent* Component) const override
    {
        for (const UFWSkillRequirement* Req : Requirements)
        {
            if (Req && !Req->IsMet(Component))
                return false;
        }
        return true;
    }
};
```

### Do skill requirements work with other FrostWeb plugins?

Yes. `UFWSkillRequirement` is a standalone UObject that any system can evaluate. FWInventorySystem, FWQuestSystem, and other plugins can hold `UFWSkillRequirement*` properties and call `MeetsRequirement` against the player's skill component.

---

## Data and Persistence

### How do I save and load skill data?

Use `GetSkillData` for each skill to collect the current state, serialize it to your save format, and restore it later with `LoadSkillData`. The component does not handle persistence internally -- this is delegated to your game's save system or backend API.

```cpp
// Save
TMap<EFWSkillType, FFWSkillData> SaveData;
for (uint8 i = 0; i <= static_cast<uint8>(EFWSkillType::Artifice); ++i)
{
    EFWSkillType Type = static_cast<EFWSkillType>(i);
    SaveData.Add(Type, SkillProgression->GetSkillData(Type));
}

// Load
SkillProgression->LoadSkillData(SaveData);
```

### Can I add new skills after launch?

Adding new entries to `EFWSkillType` requires a C++ change and recompilation. However, since skills are data-driven via `UFWSkillDefinition` assets, you can prepare "placeholder" enum entries and enable them later by adding definitions to the database.

---

## General

### Does FWSkillSystem replicate in multiplayer?

The progression component performs server authority checks on XP mutation functions (`AwardSkillXP`, `AwardCombatXP`, `LoadSkillData`). Skill state replication depends on your actor's replication setup. For dedicated server games, skill state should be server-authoritative and replicated to owning clients.

### Can I use FWSkillSystem without GameplayAbilities?

No. GameplayAbilities is a required dependency. The plugin uses GAS for skill-gated ability activation and attribute integration.

### How do I access the skill component from any Blueprint?

Use the static **Get** node with a reference to the target actor:

```
[Get (Skill Progression Component)] --> Actor: [Get Player Character]
    --> Returns the component or None
```

### Is there a performance concern with 15 skills per player?

No. Skill data is stored as a lightweight map of 15 `FFWSkillData` structs (each containing three numeric fields). XP calculations only run when XP is awarded, not on tick. The system has negligible runtime cost.
