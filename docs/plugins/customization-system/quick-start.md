---
title: Quick Start - FWCustomizationSystem
---

# Quick Start

Get a working character customization component running in under 10 minutes.

---

## Overview

By the end of this guide you will have:

1. A customization database with one race configuration
2. A character Blueprint with `UFWCustomizationComponent` attached
3. Skin tone and hair style selection working at runtime

---

## Step 1 -- Create the Customization Database

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWCustomizationDatabase` as the class.
3. Name it `DA_CustomizationDatabase`.

This asset will hold references to all your race configurations.

---

## Step 2 -- Create a Race Configuration

1. Create another Data Asset, this time choosing `FWRaceConfig`.
2. Name it `DA_RaceConfig_Human`.
3. Open it and set the **Race** field to `Human`.
4. Add at least one entry to each option array you want to support:

=== "Skin Tones"

    Create a `FWSkinToneOption` Data Asset for each skin tone. Each option contains an `FFWCustomizationTextureSet` with:

    - **Diffuse** -- Base color texture
    - **Normal** -- Normal map
    - **MetallicRoughness** -- Packed metallic/roughness
    - **Mask** -- Mask texture for blending

=== "Hair Styles"

    Create a `FWHairStyleOption` Data Asset for each hairstyle. Each option references:

    - **Mesh** -- The hair skeletal or static mesh
    - **Texture Set** -- Textures applied via the Hair DMI

=== "Eye Colors"

    Create a `FWEyeColorOption` Data Asset for each eye color variant, with its own texture set.

4. Back in `DA_RaceConfig_Human`, add your new option assets to the corresponding arrays.
5. Open `DA_CustomizationDatabase` and add `DA_RaceConfig_Human` to its race config list.

---

## Step 3 -- Add the Component to Your Character

### In Blueprints

1. Open your character Blueprint.
2. Click **Add Component** and search for `FWCustomization`.
3. Select `FW Customization Component`.
4. In the Details panel, set **Database** to `DA_CustomizationDatabase`.

### In C++

```cpp
// In your character header
UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
TObjectPtr<UFWCustomizationComponent> CustomizationComp;

// In your character constructor
CustomizationComp = CreateDefaultSubobject<UFWCustomizationComponent>(TEXT("CustomizationComp"));
```

Then in `BeginPlay` or your initialization logic:

```cpp
UFWCustomizationDatabase* DB = LoadObject<UFWCustomizationDatabase>(
    nullptr, TEXT("/Game/Data/DA_CustomizationDatabase"));
CustomizationComp->SetDatabase(DB);
CustomizationComp->SetRace(EFWRace::Human);
CustomizationComp->SetGender(EFWGender::Male);
CustomizationComp->RefreshVisuals();
```

---

## Step 4 -- Apply Customization at Runtime

To change a slot (e.g., skin tone), call `SetSlotSelection`:

=== "Blueprint"

    1. Get a reference to the Customization Component.
    2. Call **Set Slot Selection** with the slot enum and the index of the desired option.
    3. The component automatically updates DMIs and broadcasts `OnSlotChanged`.

=== "C++"

    ```cpp
    // Set the skin tone to index 2
    CustomizationComp->SetSlotSelection(EFWCustomizationSlot::SkinTone, 2);

    // Set hair style to index 0
    CustomizationComp->SetSlotSelection(EFWCustomizationSlot::HairStyle, 0);
    ```

---

## Step 5 -- Listen for Events

Bind to events to update your UI when selections change:

```cpp
CustomizationComp->OnSlotChanged.AddDynamic(this, &AMyCharacter::HandleSlotChanged);
CustomizationComp->OnProfileApplied.AddDynamic(this, &AMyCharacter::HandleProfileApplied);
```

```cpp
void AMyCharacter::HandleSlotChanged(EFWCustomizationSlot Slot, int32 NewIndex)
{
    // Update UI to reflect the new selection
}

void AMyCharacter::HandleProfileApplied()
{
    // Full profile was loaded -- refresh all UI elements
}
```

---

## Result

You now have a character that:

- Loads customization options from a data-driven database
- Swaps skin tones, hair styles, and other options at runtime via DMI
- Broadcasts events that your UI can bind to

---

## Next Steps

- Read the full [API Reference](api-reference.md) for all available functions and events.
- See [Configuration](configuration.md) for detailed database and race config setup.
- Follow the [Building a Character Creator](tutorials.md) tutorial for a complete UI integration walkthrough.
