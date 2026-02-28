---
title: Blueprints - FWCustomizationSystem
---

# Blueprint Integration

This page covers how to use FWCustomizationSystem entirely from Blueprints, including component setup, event binding, and common UI patterns.

---

## Adding the Component

1. Open your character Blueprint (or any Actor Blueprint that has a Skeletal Mesh).
2. In the Components panel, click **Add** and search for `FWCustomization`.
3. Select **FW Customization Component**.
4. In the Details panel, assign your `UFWCustomizationDatabase` asset to the **Database** property.

!!! tip "Component Placement"
    The Customization Component does not need to be attached to a specific mesh. It finds the owning Actor's mesh components automatically during `RefreshVisuals`. However, the Actor must have at least one Skeletal Mesh Component for DMI creation to work.

---

## Initial Setup in BeginPlay

Create the following setup logic in your character's **Event BeginPlay**:

```
Event BeginPlay
    |
    +-- Get Customization Component
    |
    +-- Set Database (DA_CustomizationDatabase)
    |
    +-- Set Race (Human)
    |
    +-- Set Gender (Male)
    |
    +-- Refresh Visuals
```

This initializes the component with a default appearance. In a real game, you would load the race, gender, and profile from saved data instead of hardcoding defaults.

---

## Changing Customization Slots

### Using SetSlotSelection

The primary way to change appearance is through `SetSlotSelection`:

```
On Button Clicked (Skin Tone 3)
    |
    +-- Get Customization Component
    |
    +-- Set Slot Selection
            Slot: SkinTone
            Index: 2
```

The component handles all DMI updates and mesh swaps internally. No manual material calls are needed.

### Using Specific Setters

For race and gender changes, use the dedicated functions:

```
On Race Dropdown Changed
    |
    +-- Get Customization Component
    |
    +-- Set Race (selected race enum)
    |
    +-- Refresh Visuals
```

!!! warning "Refresh After Race/Gender Change"
    When you change race or gender, the available options change entirely (different meshes, textures, option counts). Always call **Refresh Visuals** after changing race or gender to ensure the character updates completely.

---

## Binding to Events

### OnSlotChanged

Fires whenever a single customization slot changes. Use this to update individual UI elements:

```
Bind Event to On Slot Changed
    |
    Event: (Slot, NewIndex)
        |
        +-- Switch on Slot
                |
                +-- SkinTone: Update skin tone preview
                +-- HairStyle: Update hair style preview
                +-- EyeColor: Update eye color preview
                ...
```

### OnProfileApplied

Fires when an entire profile is loaded at once. Use this to refresh all UI elements:

```
Bind Event to On Profile Applied
    |
    Event: ()
        |
        +-- Refresh All UI Panels
```

### Mesh-Specific Events

For slots that involve mesh swaps, dedicated events provide the new option asset directly:

| Event | Payload | Use Case |
|---|---|---|
| `OnHairStyleChanged` | `UFWHairStyleOption*` | Update hair preview widget |
| `OnFacialHairChanged` | `UFWFacialHairOption*` | Update facial hair preview |
| `OnFaceChanged` | `UFWFaceOption*` | Update face preview |
| `OnTuskChanged` | `UFWTuskOption*` | Update tusk preview (Orc UI) |
| `OnEarringChanged` | `UFWEarringOption*` | Update earring preview |
| `OnPiercingChanged` | `UFWPiercingOption*` | Update piercing preview |

---

## Querying Current State

### Getting the Current Selection

Use the pure getter functions to query state without side effects:

```
Get Customization Component
    |
    +-- Get Current Skin Tone Option --> returns UFWSkinToneOption*
    +-- Get Current Hair Style Option --> returns UFWHairStyleOption*
    +-- Get Slot Selection (SkinTone) --> returns int32 index
    +-- Get Race                      --> returns EFWRace
    +-- Get Gender                    --> returns EFWGender
```

### Getting Available Options

To populate a UI list with available options, query the current race config:

```
Get Customization Component
    |
    +-- Get Current Race Config
            |
            +-- Access SkinTones array --> iterate for UI list
            +-- Access HairStyles array --> iterate for UI list
            +-- Access EyeColors array --> iterate for UI list
```

---

## Common Blueprint Patterns

### Pattern: Cycle Through Options

For "next/previous" buttons that cycle through a slot's options:

```
On Next Button Clicked
    |
    +-- Get Customization Component
    |
    +-- CurrentIndex = Get Slot Selection (HairStyle)
    |
    +-- Get Current Race Config
    |       |
    |       +-- MaxIndex = HairStyles.Length - 1
    |
    +-- NewIndex = (CurrentIndex + 1) % (MaxIndex + 1)
    |
    +-- Set Slot Selection (HairStyle, NewIndex)
```

### Pattern: Randomize Appearance

```
On Randomize Button Clicked
    |
    +-- Get Current Race Config
    |
    +-- For Each Slot:
    |       +-- RandomIndex = Random Integer in Range (0, OptionCount - 1)
    |       +-- Set Slot Selection (Slot, RandomIndex)
```

### Pattern: Save and Load Profile

```
-- Save --
Get Customization Component
    +-- GetProfile --> FFWCustomizationProfile
    +-- Serialize to SaveGame struct

-- Load --
Deserialize from SaveGame struct --> FFWCustomizationProfile
    +-- Get Customization Component
    +-- SetProfile (loaded profile)
```

!!! info "Profile Serialization"
    `FFWCustomizationProfile` is a `BlueprintType` struct, so it works directly with Unreal's SaveGame system. You can store it in a `USaveGame` subclass property without any manual serialization.

---

## Blueprint Node Reference

### Callable Nodes

| Node | Category | Description |
|---|---|---|
| Set Database | FW Customization | Assigns the data source |
| Set Profile | FW Customization | Applies a full profile |
| Set Slot Selection | FW Customization | Changes one slot |
| Set Race | FW Customization | Changes character race |
| Set Gender | FW Customization | Changes character gender |
| Refresh Visuals | FW Customization | Full visual rebuild |
| Create Dynamic Material Instances | FW Customization | Manual DMI recreation |

### Pure Nodes

| Node | Category | Returns |
|---|---|---|
| Get Profile | FW Customization | FFWCustomizationProfile |
| Get Slot Selection | FW Customization | int32 |
| Get Race | FW Customization | EFWRace |
| Get Gender | FW Customization | EFWGender |
| Get Current Skin Tone Option | FW Customization | UFWSkinToneOption* |
| Get Current Eye Color Option | FW Customization | UFWEyeColorOption* |
| Get Current Hair Style Option | FW Customization | UFWHairStyleOption* |
| Get Current Hair Color Option | FW Customization | UFWHairColorOption* |
| Get Current Facial Hair Option | FW Customization | UFWFacialHairOption* |
| Get Current Face Option | FW Customization | UFWFaceOption* |
| Get Current Tattoo Option | FW Customization | UFWTattooOption* |
| Get Current Scar Option | FW Customization | UFWScarOption* |
| Get Current Tusk Option | FW Customization | UFWTuskOption* |
| Get Current Earring Option | FW Customization | UFWEarringOption* |
| Get Current Piercing Option | FW Customization | UFWPiercingOption* |
| Get Hair Mesh | FW Customization | UMeshComponent* |
| Get Facial Hair Mesh | FW Customization | UMeshComponent* |
| Get Tusk Mesh | FW Customization | UMeshComponent* |
| Get Earring Mesh | FW Customization | UMeshComponent* |
| Get Piercing Mesh | FW Customization | UMeshComponent* |
| Get Database | FW Customization | UFWCustomizationDatabase* |
| Get Current Race Config | FW Customization | UFWRaceConfig* |
