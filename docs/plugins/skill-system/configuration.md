---
title: Configuration - FWSkillSystem
---

# Configuration

Complete guide to setting up the Skill Database, XP curves, milestones, and project settings.

---

## Project Settings (UFWSkillSystemSettings)

Navigate to **Edit > Project Settings > Plugins > FW Skill System** to access global configuration.

| Setting | Type | Default | Description |
|---|---|---|---|
| Skill Database | `UFWSkillDatabase` | None | Reference to the global skill database containing all skill definitions |
| Default XP Curve | `UFWSkillXPCurve` | None | Default XP curve used by skills that do not specify an override |
| Default Max Level | `int32` | 99 | Default maximum level when not specified per-skill |

!!! warning "Skill Database Required"
    The skill system will not function without a `UFWSkillDatabase` assigned. All calls to `AwardSkillXP`, `GetSkillLevel`, and other progression functions require a valid database.

### Accessing Settings in C++

```cpp
#include "Settings/FWSkillSystemSettings.h"

const UFWSkillSystemSettings* Settings = GetDefault<UFWSkillSystemSettings>();
UFWSkillDatabase* Database = Settings->SkillDatabase.LoadSynchronous();
```

### Accessing Settings in Blueprint

Use the **Get Skill Database** node from `UFWSkillTypeLibrary` to retrieve the globally configured database.

---

## Skill Database Setup

The `UFWSkillDatabase` is a `UDataAsset` that holds references to all 15 skill definitions. It serves as the single source of truth for what skills exist in your game.

### Creating the Database

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWSkillDatabase` as the class.
3. Name it `DA_SkillDatabase`.
4. Open it and add one `UFWSkillDefinition` entry for each skill.

### Recommended Asset Organization

```
Content/
    Data/
        Skills/
            DA_SkillDatabase.uasset
            Definitions/
                DA_Skill_Warfare.uasset
                DA_Skill_Marksmanship.uasset
                DA_Skill_Magic.uasset
                DA_Skill_Bounty.uasset
                DA_Skill_Woodcutting.uasset
                DA_Skill_Mining.uasset
                DA_Skill_Fishing.uasset
                DA_Skill_Husbandry.uasset
                DA_Skill_Cooking.uasset
                DA_Skill_Woodworking.uasset
                DA_Skill_Crafting.uasset
                DA_Skill_Smithing.uasset
                DA_Skill_Alchemy.uasset
                DA_Skill_Construction.uasset
                DA_Skill_Artifice.uasset
            Curves/
                DA_DefaultXPCurve.uasset
                DA_CombatXPCurve.uasset
            Milestones/
                DA_Milestone_Mining_10.uasset
                DA_Milestone_Mining_50.uasset
                DA_Milestone_Mining_99.uasset
```

---

## Skill Definitions

Each `UFWSkillDefinition` is a `UPrimaryDataAsset` that configures a single skill.

### Creating a Skill Definition

1. Create a Data Asset of class `FWSkillDefinition`.
2. Name it `DA_Skill_<SkillName>`.
3. Configure the following properties:

| Property | Required | Description |
|---|---|---|
| Skill Type | Yes | Select the `EFWSkillType` enum value this definition represents |
| Display Name | Yes | Localized name shown in UI (e.g., "Mining") |
| Description | No | Localized description of the skill |
| Icon | No | `UTexture2D` reference for UI display |
| Category | Yes | `EFWSkillCategory` -- automatically determined by skill type, but can be verified here |
| Max Level | Yes | Maximum level for this skill (typically 99) |
| XP Curve Override | No | If set, this skill uses a custom XP curve instead of the default |
| Milestones | No | Array of `UFWSkillMilestoneDefinition` assets for this skill |

### Example: Mining Skill Definition

| Property | Value |
|---|---|
| Skill Type | `Mining` |
| Display Name | "Mining" |
| Description | "Extract ores and minerals from rock formations" |
| Category | `Gathering` |
| Max Level | 99 |
| XP Curve Override | (empty -- uses default) |

---

## XP Curves

The `UFWSkillXPCurve` data asset defines how much cumulative XP is required for each level. This is the core progression tuning mechanism.

### Creating an XP Curve

1. Create a Data Asset of class `FWSkillXPCurve`.
2. Name it `DA_DefaultXPCurve` (or `DA_<SkillName>XPCurve` for per-skill overrides).
3. Populate the `LevelXPThresholds` array.

### How the Curve Works

The `LevelXPThresholds` array maps array indices to cumulative XP requirements:

| Array Index | Level | Cumulative XP |
|---|---|---|
| 0 | 1 | 0 |
| 1 | 2 | 83 |
| 2 | 3 | 174 |
| 3 | 4 | 276 |
| ... | ... | ... |
| 98 | 99 | 13,034,431 |

The XP required to go from Level N to Level N+1 is `LevelXPThresholds[N] - LevelXPThresholds[N-1]`.

### Curve Design Strategies

=== "Exponential"

    An exponential curve where each level requires increasingly more XP:

    ```
    XP(L) = floor( (1/4) * sum(floor(L + 300 * 2^(L/7)), L=1..N-1) )
    ```

    This produces a curve where early levels are quick and late levels require significant grinding.

=== "Linear"

    A flat increase per level (e.g., 100 XP more per level):

    | Level | XP Required | Cumulative |
    |---|---|---|
    | 2 | 100 | 100 |
    | 3 | 200 | 300 |
    | 4 | 300 | 600 |

=== "Custom"

    Enter any arbitrary values in the array. This gives designers full control over the feel of each level transition.

!!! tip "Per-Skill Overrides"
    Combat skills might use a steeper curve to slow down power progression, while gathering skills might use a gentler curve to keep the activity loop rewarding. Assign different `UFWSkillXPCurve` assets to each `UFWSkillDefinition` via the **XP Curve Override** property.

---

## Milestones

Milestones are special rewards triggered when a player reaches a specific level in a skill.

### Creating a Milestone Definition

1. Create a Data Asset of class `FWSkillMilestoneDefinition`.
2. Name it `DA_Milestone_<Skill>_<Level>` (e.g., `DA_Milestone_Mining_10`).
3. Configure:

| Property | Description |
|---|---|
| Milestone Name | Display name (e.g., "Novice Miner") |
| Required Level | The skill level that triggers this milestone (e.g., 10) |
| Rewards | Array of `UFWSkillMilestoneReward` assets |

### Creating Milestone Rewards

`UFWSkillMilestoneReward` is a base class. Subclass it in C++ or Blueprint to define custom reward types:

=== "C++ Subclass"

    ```cpp
    UCLASS()
    class UFWSkillMilestoneReward_GrantItem : public UFWSkillMilestoneReward
    {
        GENERATED_BODY()

    public:
        UPROPERTY(EditAnywhere, Category = "Reward")
        TSubclassOf<AMyItemActor> ItemClass;

        UPROPERTY(EditAnywhere, Category = "Reward")
        int32 Quantity = 1;
    };
    ```

=== "Blueprint Subclass"

    1. Create a new Blueprint class with `FWSkillMilestoneReward` as the parent.
    2. Add variables for the reward data (item references, currency amounts, etc.).
    3. Override the reward description.

### Assigning Milestones to Skills

Open the `UFWSkillDefinition` for the target skill and add your milestone definition to its **Milestones** array.

### Example: Mining Milestones

| Level | Milestone Name | Rewards |
|---|---|---|
| 10 | Novice Miner | Bronze pickaxe, 500 gold |
| 25 | Journeyman Miner | Access to iron ore deposits |
| 50 | Expert Miner | Mithril mining ability unlocked |
| 75 | Master Miner | Chance to find gems while mining |
| 99 | Grandmaster Miner | Legendary pickaxe, mining cape |

---

## Milestone Component Setup

The `UFWSkillMilestoneComponent` tracks which milestones have been awarded and triggers reward delivery when new milestones are reached.

### Adding the Component

=== "Blueprint"

    1. Open your player character or controller Blueprint.
    2. Click **Add Component** and search for `FWSkillMilestone`.
    3. Add the `FW Skill Milestone Component`.
    4. The component automatically binds to the `UFWSkillProgressionComponent` on the same actor.

=== "C++"

    ```cpp
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Skills")
    TObjectPtr<UFWSkillMilestoneComponent> SkillMilestones;

    // In constructor
    SkillMilestones = CreateDefaultSubobject<UFWSkillMilestoneComponent>(TEXT("SkillMilestones"));
    ```

!!! info "Component Dependency"
    `UFWSkillMilestoneComponent` requires a `UFWSkillProgressionComponent` on the same actor. If the progression component is missing, the milestone component logs a warning and disables itself during `BeginPlay`.

---

## Combat XP Configuration

Combat XP distribution is handled by `AwardCombatXP` on the progression component. When called, XP is distributed to the four combat skills (Warfare, Marksmanship, Magic, Bounty) based on the combat context.

### Distribution Logic

The distribution is determined by the skill definitions and the active combat state. By default:

- **Primary combat skill** receives the largest share of XP
- **Secondary combat skills** receive a reduced share
- Skills not actively used in the combat encounter may receive no XP

!!! tip "Customizing Distribution"
    To customize how combat XP is distributed, override the distribution behavior in a subclass of `UFWSkillProgressionComponent` or configure distribution weights in your skill definitions.

---

## Skill Requirements

`UFWSkillRequirement` objects define skill level prerequisites that other systems can check against.

### Creating a Requirement

```cpp
UCLASS(EditInlineNew, DefaultToInstanced)
class UFWSkillRequirement_MinLevel : public UFWSkillRequirement
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, Category = "Requirement")
    EFWSkillType SkillType;

    UPROPERTY(EditAnywhere, Category = "Requirement")
    int32 MinimumLevel;

    virtual bool IsMet(const UFWSkillProgressionComponent* Component) const override
    {
        return Component && Component->GetSkillLevel(SkillType) >= MinimumLevel;
    }

    virtual FText GetDescription() const override
    {
        return FText::Format(
            NSLOCTEXT("Skills", "ReqMinLevel", "Requires {0} Level {1}"),
            UFWSkillTypeLibrary::GetSkillDisplayName(SkillType),
            FText::AsNumber(MinimumLevel)
        );
    }
};
```

### Using Requirements in Other Systems

Any system that gates content behind skills can hold a `UFWSkillRequirement*` property and evaluate it:

```cpp
// On an item, quest, door, or ability
UPROPERTY(EditAnywhere, Instanced, Category = "Requirements")
TObjectPtr<UFWSkillRequirement> SkillRequirement;

// Check at runtime
UFWSkillProgressionComponent* Skills = UFWSkillProgressionComponent::Get(PlayerActor);
if (Skills && Skills->MeetsRequirement(SkillRequirement))
{
    // Player meets the skill requirement
}
```
