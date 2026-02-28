---
title: Quick Start - FWSkillSystem
---

# Quick Start

Get a working skill progression system running in 10 minutes.

---

## Overview

By the end of this guide you will have:

1. A skill database with all 15 skills configured
2. An XP curve defining level progression
3. A player character with `UFWSkillProgressionComponent` attached
4. XP being awarded and skill levels displayed at runtime

---

## Step 1 -- Create the XP Curve

The XP curve defines how much total XP is required for each skill level.

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWSkillXPCurve` as the class.
3. Name it `DA_DefaultXPCurve`.
4. Open the asset and define XP thresholds per level.

The curve maps each level to the cumulative XP required to reach it. For example:

| Level | Total XP Required |
|---|---|
| 1 | 0 |
| 2 | 83 |
| 3 | 174 |
| 4 | 276 |
| 5 | 388 |
| 10 | 1,154 |
| 25 | 8,740 |
| 50 | 101,333 |
| 99 | 13,034,431 |

!!! tip "Curve Design"
    FWSkillSystem supports any XP curve shape. You can use linear, polynomial, or exponential scaling. The curve is defined as a data asset, so designers can iterate without recompiling.

---

## Step 2 -- Create Skill Definitions

For each of the 15 skills, create a `UFWSkillDefinition` data asset:

1. Right-click in the Content Browser and select **Miscellaneous > Data Asset**.
2. Choose `FWSkillDefinition` as the class.
3. Name it following the convention `DA_Skill_<SkillName>` (e.g., `DA_Skill_Mining`).
4. Open the asset and configure:
    - **Skill Type** -- Select the corresponding `EFWSkillType` value
    - **Display Name** -- Human-readable name (e.g., "Mining")
    - **XP Curve Override** -- Leave empty to use the default curve, or assign a custom `UFWSkillXPCurve`
    - **Max Level** -- Maximum achievable level for this skill (e.g., 99)

Repeat for all 15 skills: Warfare, Marksmanship, Magic, Bounty, Woodcutting, Mining, Fishing, Husbandry, Cooking, Woodworking, Crafting, Smithing, Alchemy, Construction, Artifice.

---

## Step 3 -- Create the Skill Database

1. Create another Data Asset, this time choosing `FWSkillDatabase`.
2. Name it `DA_SkillDatabase`.
3. Open it and add all 15 `UFWSkillDefinition` assets you created in Step 2.

---

## Step 4 -- Assign the Database in Project Settings

1. Navigate to **Edit > Project Settings > Plugins > FW Skill System**.
2. Set **Skill Database** to `DA_SkillDatabase`.
3. Set **Default XP Curve** to `DA_DefaultXPCurve`.

---

## Step 5 -- Add the Component to Your Character

### In Blueprints

1. Open your player character or controller Blueprint.
2. Click **Add Component** and search for `FWSkillProgression`.
3. Select `FW Skill Progression Component`.
4. The component automatically loads the skill database from project settings.

### In C++

```cpp
// In your character or controller header
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Skills")
TObjectPtr<UFWSkillProgressionComponent> SkillProgression;

// In your constructor
SkillProgression = CreateDefaultSubobject<UFWSkillProgressionComponent>(TEXT("SkillProgression"));
```

---

## Step 6 -- Award XP

=== "C++"

    ```cpp
    // Award 150 Mining XP
    SkillProgression->AwardSkillXP(EFWSkillType::Mining, 150.0f);

    // Award combat XP (distributed to relevant combat skills)
    SkillProgression->AwardCombatXP(500.0f);
    ```

=== "Blueprint"

    1. Get a reference to the Skill Progression Component.
    2. Call **Award Skill XP** with the skill type and XP amount.
    3. For combat encounters, call **Award Combat XP** instead.

---

## Step 7 -- Read Skill Levels

=== "C++"

    ```cpp
    // Get the current Mining level
    int32 MiningLevel = SkillProgression->GetSkillLevel(EFWSkillType::Mining);

    // Get XP progress toward the next level (0.0 to 1.0)
    float Progress = SkillProgression->GetSkillLevelProgress(EFWSkillType::Mining);

    // Get XP remaining to next level
    float Remaining = SkillProgression->GetXPToNextLevel(EFWSkillType::Mining);

    // Get total level across all skills
    int32 TotalLevel = SkillProgression->GetTotalLevel();
    ```

=== "Blueprint"

    Use the pure Blueprint nodes **Get Skill Level**, **Get Skill Level Progress**, **Get XP To Next Level**, and **Get Total Level** on the Skill Progression Component.

---

## Step 8 -- Listen for Events

Bind to events to react to XP gains and level-ups:

```cpp
SkillProgression->OnSkillXPGained.AddDynamic(this, &AMyCharacter::HandleXPGained);
SkillProgression->OnSkillLevelUp.AddDynamic(this, &AMyCharacter::HandleLevelUp);
```

```cpp
void AMyCharacter::HandleXPGained(EFWSkillType SkillType, float Amount, float NewTotalXP)
{
    // Update XP bar UI for the given skill
}

void AMyCharacter::HandleLevelUp(EFWSkillType SkillType, int32 NewLevel)
{
    // Show level-up notification, unlock new content
}
```

---

## Result

You now have a character that:

- Tracks XP and levels across 15 skills
- Awards XP through direct skill calls or combat XP distribution
- Broadcasts events on XP gain and level-up
- Provides pure getter functions for UI display

---

## Next Steps

- Read the full [API Reference](api-reference.md) for all available functions and events.
- See [Configuration](configuration.md) for detailed XP curve, milestone, and project settings setup.
- Follow the [Building a Skill Progression UI](tutorials.md) tutorial for a complete UMG integration walkthrough.
- Check [Blueprints](blueprints.md) for a visual reference of all exposed Blueprint nodes.
