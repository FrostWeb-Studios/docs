---
title: Tutorials - FWSkillSystem
---

# Tutorials

## Building a Skill Progression UI

This tutorial walks through creating a complete skill panel that displays all 15 skills organized by category, with level numbers, XP progress bars, milestone rewards, and skill-gated content indicators.

---

### What You Will Build

A full-screen skill panel with:

- Three category tabs (Combat, Gathering, Artisan)
- A row per skill showing icon, name, level, and XP progress bar
- A detail panel for the selected skill showing milestones and their reward status
- Skill requirement checks that gray out locked content

---

### Prerequisites

- FWSkillSystem installed and configured (see [Installation](installation.md) and [Configuration](configuration.md))
- A player character with `UFWSkillProgressionComponent` and `UFWSkillMilestoneComponent`
- Basic familiarity with UMG widget Blueprints

---

### Step 1 -- Create the Skill Panel Widget

1. Create a new Widget Blueprint: **Content Browser > User Interface > Widget Blueprint**.
2. Name it `WBP_SkillPanel`.
3. Set up the root layout:

```
[Canvas Panel]
    [Vertical Box - "Root"]
        [Horizontal Box - "CategoryTabs"]
            [Button - "Btn_Combat"]     Text: "Combat"
            [Button - "Btn_Gathering"]  Text: "Gathering"
            [Button - "Btn_Artisan"]    Text: "Artisan"
        [Scroll Box - "SkillList"]
            (populated at runtime)
        [Vertical Box - "DetailPanel"]
            [Text Block - "Txt_SkillName"]
            [Text Block - "Txt_SkillLevel"]
            [Progress Bar - "PB_SkillXP"]
            [Text Block - "Txt_XPNumbers"]
            [Vertical Box - "MilestoneList"]
                (populated at runtime)
```

---

### Step 2 -- Create the Skill Row Widget

Create a separate widget for individual skill rows.

1. Create `WBP_SkillRow`.
2. Layout:

```
[Horizontal Box]
    [Image - "Img_SkillIcon"]       Size: 48x48
    [Vertical Box]
        [Text Block - "Txt_Name"]   Font: Bold
        [Progress Bar - "PB_XP"]    Fill: Accent Color
    [Text Block - "Txt_Level"]      Font: Large Bold
```

3. Add a variable `SkillType` of type `EFWSkillType` (Instance Editable, Expose on Spawn).
4. Add a variable `SkillProgression` of type `FW Skill Progression Component` (Instance Editable, Expose on Spawn).

---

### Step 3 -- Populate the Skill Row

In `WBP_SkillRow`, implement the **Event Construct** or a custom `Refresh` function:

=== "Blueprint"

    ```
    [Event Construct]
        --> [Get Skill Display Name: SkillType] --> [Set Text: Txt_Name]
        --> [Get Skill Level: SkillType] --> [Format Text: "Lv. {0}"] --> [Set Text: Txt_Level]
        --> [Get Skill Level Progress: SkillType] --> [Set Percent: PB_XP]
        --> [Get Skill Definition: SkillType] --> [Get Icon] --> [Set Brush: Img_SkillIcon]
    ```

=== "C++"

    ```cpp
    void USkillRowWidget::NativeConstruct()
    {
        Super::NativeConstruct();
        RefreshDisplay();
    }

    void USkillRowWidget::RefreshDisplay()
    {
        if (!SkillProgression) return;

        Txt_Name->SetText(UFWSkillTypeLibrary::GetSkillDisplayName(SkillType));

        int32 Level = SkillProgression->GetSkillLevel(SkillType);
        Txt_Level->SetText(FText::Format(
            NSLOCTEXT("UI", "SkillLevel", "Lv. {0}"),
            FText::AsNumber(Level)));

        PB_XP->SetPercent(SkillProgression->GetSkillLevelProgress(SkillType));

        if (UFWSkillDefinition* Def = SkillProgression->GetSkillDefinition(SkillType))
        {
            if (UTexture2D* Icon = Def->Icon.LoadSynchronous())
            {
                Img_SkillIcon->SetBrushFromTexture(Icon);
            }
        }
    }
    ```

---

### Step 4 -- Populate the Skill List by Category

In `WBP_SkillPanel`, create a function `ShowCategory(EFWSkillCategory Category)`:

=== "Blueprint"

    ```
    [ShowCategory(Category)]
        --> [Clear Children: SkillList]
        --> [For Each: All EFWSkillType values]
            --> [Get Skill Category: SkillType]
            --> [Branch: Category == InputCategory]
                True --> [Create Widget: WBP_SkillRow]
                    Set SkillType, Set SkillProgression
                    --> [Add Child to Scroll Box: SkillList]
    ```

=== "C++"

    ```cpp
    void USkillPanelWidget::ShowCategory(EFWSkillCategory Category)
    {
        SkillList->ClearChildren();

        // Iterate all skill types
        for (uint8 i = 0; i < static_cast<uint8>(EFWSkillType::Artifice) + 1; ++i)
        {
            EFWSkillType SkillType = static_cast<EFWSkillType>(i);

            if (UFWSkillTypeLibrary::GetSkillCategory(SkillType) == Category)
            {
                USkillRowWidget* Row = CreateWidget<USkillRowWidget>(this, SkillRowClass);
                Row->SkillType = SkillType;
                Row->SkillProgression = SkillProgression;
                SkillList->AddChild(Row);
            }
        }
    }
    ```

Wire the three category buttons to call `ShowCategory` with the appropriate enum value:

- **Btn_Combat** On Clicked --> `ShowCategory(Combat)`
- **Btn_Gathering** On Clicked --> `ShowCategory(Gathering)`
- **Btn_Artisan** On Clicked --> `ShowCategory(Artisan)`

---

### Step 5 -- Build the Detail Panel

When a skill row is clicked, populate the detail panel with extended information.

1. Add an `OnClicked` event to `WBP_SkillRow` (or use a Button wrapper).
2. In the parent `WBP_SkillPanel`, handle the selection:

```
[On Skill Row Clicked (SkillType)]
    --> [Set Text: Txt_SkillName] <-- [Get Skill Display Name]
    --> [Set Text: Txt_SkillLevel] <-- [Format: "Level {0} / {1}"]
        <-- [Get Skill Level], [Get Max Skill Level]
    --> [Set Percent: PB_SkillXP] <-- [Get Skill Level Progress]
    --> [Set Text: Txt_XPNumbers] <-- [Format: "{0} / {1} XP"]
        <-- [Get Skill Total XP], [XP threshold for next level]
    --> [Show Milestones: SkillType]
```

---

### Step 6 -- Display Milestones

Create a function `ShowMilestones(EFWSkillType SkillType)`:

=== "Blueprint"

    ```
    [ShowMilestones(SkillType)]
        --> [Clear Children: MilestoneList]
        --> [Get Skill Milestones: SkillType] --> [For Each: Milestone]
            --> [Create Widget: WBP_MilestoneRow]
                Set MilestoneName, RequiredLevel, IsUnlocked
                --> [Add Child: MilestoneList]
    ```

=== "C++"

    ```cpp
    void USkillPanelWidget::ShowMilestones(EFWSkillType SkillType)
    {
        MilestoneList->ClearChildren();

        int32 CurrentLevel = SkillProgression->GetSkillLevel(SkillType);
        TArray<UFWSkillMilestoneDefinition*> Milestones =
            SkillProgression->GetSkillMilestones(SkillType);

        for (UFWSkillMilestoneDefinition* Milestone : Milestones)
        {
            UMilestoneRowWidget* Row = CreateWidget<UMilestoneRowWidget>(
                this, MilestoneRowClass);
            Row->MilestoneName = Milestone->MilestoneName;
            Row->RequiredLevel = Milestone->RequiredLevel;
            Row->bIsUnlocked = CurrentLevel >= Milestone->RequiredLevel;
            MilestoneList->AddChild(Row);
        }
    }
    ```

Create `WBP_MilestoneRow` with:

```
[Horizontal Box]
    [Image - "Img_LockIcon"]       Visible if not unlocked
    [Image - "Img_CheckIcon"]      Visible if unlocked
    [Vertical Box]
        [Text Block - "Txt_MilestoneName"]
        [Text Block - "Txt_RequiredLevel"]   "Requires Level 50"
    [Vertical Box - "RewardList"]
        (List reward descriptions)
```

---

### Step 7 -- Live Updates with Events

Bind to skill events so the UI updates in real time without polling:

```cpp
// In WBP_SkillPanel initialization
SkillProgression->OnSkillXPGained.AddDynamic(this, &USkillPanelWidget::HandleXPGained);
SkillProgression->OnSkillLevelUp.AddDynamic(this, &USkillPanelWidget::HandleLevelUp);
```

```cpp
void USkillPanelWidget::HandleXPGained(EFWSkillType SkillType, float Amount, float NewTotalXP)
{
    // Refresh the row for this skill
    RefreshSkillRow(SkillType);

    // If this skill is selected in the detail panel, refresh it too
    if (SkillType == SelectedSkillType)
    {
        RefreshDetailPanel(SkillType);
    }
}

void USkillPanelWidget::HandleLevelUp(EFWSkillType SkillType, int32 NewLevel)
{
    RefreshSkillRow(SkillType);
    RefreshDetailPanel(SkillType);

    // Play level-up animation or sound
    PlayLevelUpEffect(SkillType, NewLevel);
}
```

---

### Step 8 -- Skill-Gated Content Indicators

For UI elements that represent locked content (items, areas, abilities), use `MeetsRequirement` to visually indicate availability:

```cpp
void UContentSlotWidget::RefreshLockState()
{
    UFWSkillProgressionComponent* Skills =
        UFWSkillProgressionComponent::Get(GetOwningPlayerPawn());

    if (Skills && SkillRequirement)
    {
        bool bMet = Skills->MeetsRequirement(SkillRequirement);
        Img_LockOverlay->SetVisibility(bMet ? ESlateVisibility::Collapsed : ESlateVisibility::Visible);
        SetIsEnabled(bMet);

        if (!bMet)
        {
            Txt_RequirementLabel->SetText(SkillRequirement->GetDescription());
        }
    }
}
```

---

### Step 9 -- Total Level Display

Add a total level indicator to your HUD or skill panel header:

```
[Text Block - "Txt_TotalLevel"]
    Text: [Format: "Total Level: {0}"] <-- [Get Total Level]
```

This sums all 15 skill levels and provides a single overall progression number.

---

### Final Result

Your completed skill panel includes:

- **Category tabs** that filter skills by Combat, Gathering, and Artisan
- **Skill rows** showing icon, name, current level, and XP progress bar
- **Detail panel** with full XP numbers, level progress, and milestone list
- **Milestone indicators** showing locked/unlocked status and reward descriptions
- **Live updates** driven by `OnSkillXPGained` and `OnSkillLevelUp` events
- **Skill-gated content** with lock overlays and requirement descriptions
- **Total level display** aggregating all skills into a single number
