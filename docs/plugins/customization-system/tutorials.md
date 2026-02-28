---
title: Tutorials - FWCustomizationSystem
---

# Tutorials

## Building a Character Creator

This tutorial walks you through building a complete character creator screen -- from setting up the customization database to creating a UMG widget that lets players pick their race, gender, skin tone, hair, and more, with real-time visual feedback via DMI materials.

---

### What You Will Build

A character creator screen with:

- Race and gender selection (dropdown or button group)
- Slot-based customization panels (skin tone, eye color, hair style, hair color, etc.)
- A 3D character preview that updates in real-time
- Save/load profile support

### Prerequisites

- FWCustomizationSystem installed and compiling (see [Installation](installation.md))
- A character Blueprint with a Skeletal Mesh Component
- Basic familiarity with UMG widget Blueprints

---

### Part 1 -- Setting Up the Customization Database

#### 1.1 Create the Database Asset

1. In the Content Browser, navigate to `Content/Data/Customization/`.
2. Right-click > **Miscellaneous > Data Asset** > select `FWCustomizationDatabase`.
3. Name it `DA_CustomizationDatabase`.

#### 1.2 Create a Race Config

1. In `Content/Data/Customization/Human/`, create a Data Asset of type `FWRaceConfig`.
2. Name it `DA_RaceConfig_Human`.
3. Open it and set **Race** to `Human`.

#### 1.3 Create Skin Tone Options

For each skin tone variant:

1. Create a Data Asset of type `FWSkinToneOption`.
2. Fill in the `FFWCustomizationTextureSet`:

    | Field | Asset |
    |---|---|
    | Diffuse | `T_Skin_Light_D` |
    | Normal | `T_Skin_Light_N` |
    | MetallicRoughness | `T_Skin_Light_MR` |
    | Mask | `T_Skin_Light_M` |

3. Repeat for each skin tone (Light, Medium, Dark, etc.).

#### 1.4 Create Hair Style Options

For each hairstyle:

1. Create a Data Asset of type `FWHairStyleOption`.
2. Set the **Mesh** field to the hair mesh asset.
3. Fill in the texture set for the hair material.

#### 1.5 Create Remaining Options

Repeat the process for eye colors, hair colors, facial hair, faces, tattoos, scars, and any race-specific options (tusks for Orcs, etc.).

#### 1.6 Wire Everything Together

1. Open `DA_RaceConfig_Human`.
2. Add all your option assets to the corresponding arrays (SkinTones, HairStyles, EyeColors, etc.).
3. Open `DA_CustomizationDatabase`.
4. Add `DA_RaceConfig_Human` to the **RaceConfigs** array.
5. Repeat for Dwarf and Orc race configs if needed.

!!! tip "Start Small"
    You do not need to populate every slot before testing. Start with 2-3 skin tones and 2-3 hair styles to verify the pipeline, then add more options incrementally.

---

### Part 2 -- Creating the Character Preview

#### 2.1 Set Up the Preview Actor

Create a Blueprint Actor that will serve as the character preview in your creator screen:

1. Create a new Actor Blueprint (e.g., `BP_CharacterPreview`).
2. Add a **Skeletal Mesh Component** as the root, using your character's base mesh.
3. Add a **FW Customization Component**.
4. In the Customization Component's Details panel, set **Database** to `DA_CustomizationDatabase`.

#### 2.2 Initialize in BeginPlay

In the preview Actor's Event Graph:

```
Event BeginPlay
    |
    +-- Customization Component > Set Race (Human)
    |
    +-- Customization Component > Set Gender (Male)
    |
    +-- Customization Component > Refresh Visuals
```

#### 2.3 Place in Scene for UI Capture

For a character creator screen, you typically render this Actor with a **Scene Capture Component 2D** to display it in your UMG widget, or place it in a dedicated level that the character creator screen loads.

---

### Part 3 -- Building the UI Widget

#### 3.1 Create the Widget Blueprint

1. Create a new Widget Blueprint (e.g., `WBP_CharacterCreator`).
2. Design the layout with these sections:

```
+------------------------------------------+
|  [Race Selector]    [Gender Selector]     |
+------------------------------------------+
|                                           |
|         3D Character Preview              |
|                                           |
+------------------------------------------+
|  Skin Tone:  [< ] Option Name [> ]       |
|  Eye Color:  [< ] Option Name [> ]       |
|  Hair Style: [< ] Option Name [> ]       |
|  Hair Color: [< ] Option Name [> ]       |
|  ...                                      |
+------------------------------------------+
|         [ Randomize ]  [ Confirm ]        |
+------------------------------------------+
```

#### 3.2 Store a Reference to the Component

In your widget Blueprint, create a variable:

- **CustomizationComp** -- Type: `FW Customization Component` (Object Reference)

Set this reference when the widget is initialized (passed from the character creator game mode or HUD).

#### 3.3 Implement Race Selection

```
On Race Button Clicked (e.g., "Human")
    |
    +-- CustomizationComp > Set Race (Human)
    |
    +-- CustomizationComp > Refresh Visuals
    |
    +-- Call "RefreshAllSlotPanels" (custom function)
```

When the race changes, the available options change entirely, so you must rebuild your slot panels to reflect the new race config's arrays.

#### 3.4 Implement Gender Selection

```
On Gender Button Clicked (e.g., "Female")
    |
    +-- CustomizationComp > Set Gender (Female)
    |
    +-- CustomizationComp > Refresh Visuals
```

#### 3.5 Implement Slot Navigation (Next/Previous)

For each customization slot, create next/previous buttons:

```
On "Next Skin Tone" Clicked
    |
    +-- CurrentIndex = CustomizationComp > Get Slot Selection (SkinTone)
    |
    +-- RaceConfig = CustomizationComp > Get Current Race Config
    |
    +-- MaxIndex = RaceConfig > SkinTones.Length - 1
    |
    +-- NewIndex = (CurrentIndex + 1)
    |       IF NewIndex > MaxIndex THEN NewIndex = 0
    |
    +-- CustomizationComp > Set Slot Selection (SkinTone, NewIndex)
    |
    +-- Update Slot Label Text
```

!!! info "Real-Time Updates"
    You do not need to call `RefreshVisuals` after `SetSlotSelection`. The component automatically updates the relevant DMI or mesh for that specific slot. `RefreshVisuals` is only needed after race or gender changes, which affect all slots simultaneously.

---

### Part 4 -- Real-Time Visual Updates with DMI Materials

This is where the power of the DMI-driven approach becomes apparent. When a player selects a new skin tone, the component does not swap the entire material -- it updates texture parameters on the existing DMI.

#### 4.1 How DMI Updates Work Internally

When you call `SetSlotSelection(SkinTone, 2)`:

1. The component looks up the `UFWSkinToneOption` at index 2 in the current race config.
2. It reads the `FFWCustomizationTextureSet` from that option.
3. It sets texture parameters on `DI_Head` and `DI_Body`:
    - `Diffuse` texture parameter
    - `Normal` texture parameter
    - `MetallicRoughness` texture parameter
    - `Mask` texture parameter

This happens within a single frame. The material does not flicker or reload.

#### 4.2 Material Setup Requirements

Your base materials (the ones assigned to your character's Skeletal Mesh) must expose the texture parameters that the DMI system expects. The component creates DMIs from these base materials and sets parameters by name.

Ensure your character materials have these **Texture Parameter** nodes:

| DMI Slot | Required Parameters |
|---|---|
| DI_Head | `Diffuse`, `Normal`, `MetallicRoughness`, `Mask` |
| DI_Eye | `Diffuse`, `Normal` |
| DI_Body | `Diffuse`, `Normal`, `MetallicRoughness`, `Mask` |
| DI_Hair | `Diffuse`, `Normal`, `HairColor` (Vector) |
| DI_Facials | `Diffuse`, `Normal` |

!!! warning "Parameter Name Matching"
    The parameter names in your material must match exactly what the component sets. If your material uses `BaseColor` instead of `Diffuse`, the DMI update will silently fail. Check your material parameter names carefully.

#### 4.3 Layered Effects (Tattoos and Scars)

Tattoos and scars work as overlay layers on the head and body DMIs. Your material must support blending these overlays using the **Mask** texture:

1. The base skin tone provides the foundation textures.
2. When a tattoo is selected, the component sets additional overlay parameters.
3. The mask texture controls where the overlay is visible.

This means your head and body materials need both base and overlay parameter slots.

---

### Part 5 -- Saving and Loading Profiles

#### 5.1 Saving the Profile

When the player clicks "Confirm":

```
On Confirm Button Clicked
    |
    +-- Profile = CustomizationComp > Get Profile
    |
    +-- SaveGame > Set CustomizationProfile = Profile
    |
    +-- Save Game to Slot
```

The `FFWCustomizationProfile` struct is compact (~8 bytes) and serializes cleanly with Unreal's SaveGame system.

#### 5.2 Loading the Profile

When spawning the character in-game:

```
Event BeginPlay
    |
    +-- Load SaveGame
    |
    +-- Profile = SaveGame > CustomizationProfile
    |
    +-- CustomizationComp > Set Database (DA_CustomizationDatabase)
    |
    +-- CustomizationComp > Set Profile (Profile)
```

`SetProfile` sets the race, gender, and all slot selections in one call, then triggers `RefreshVisuals` internally. The `OnProfileApplied` event fires when the visual update completes.

#### 5.3 Network Replication

For multiplayer games, replicate the `FFWCustomizationProfile` as part of your PlayerState:

```cpp
UPROPERTY(Replicated)
FFWCustomizationProfile CustomizationProfile;
```

When the replicated profile changes on a client, call `SetProfile` on that client's customization component to apply the appearance. At ~8 bytes per player, this adds negligible bandwidth overhead even with many players.

---

### Summary

| Step | What You Did |
|---|---|
| Part 1 | Created the data asset hierarchy: Database > Race Config > Options |
| Part 2 | Set up a preview Actor with the Customization Component |
| Part 3 | Built a UMG widget with race/gender/slot controls |
| Part 4 | Learned how DMI parameters drive real-time visual updates |
| Part 5 | Implemented save/load and network replication for profiles |

---

### Next Steps

- Add unlock requirement checks to gate premium customization options (see [Configuration](configuration.md#unlock-requirements)).
- Extend the UI with thumbnail previews for each option using render targets.
- Add animation support to the preview Actor (idle pose, turntable rotation).
