---
title: Configuration - FWCustomizationSystem
---

# Configuration

This page explains how to set up your customization database, create race configurations, define customization options, and configure unlock requirements.

---

## Customization Database

The `UFWCustomizationDatabase` is the root asset that the component reads from. It holds references to all race configurations.

### Creating the Database

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWCustomizationDatabase` as the class.
3. Name it following your project convention (e.g., `DA_CustomizationDatabase`).

### Database Structure

```
DA_CustomizationDatabase
    |
    +-- DA_RaceConfig_Human
    +-- DA_RaceConfig_Dwarf
    +-- DA_RaceConfig_Orc
```

Add one `UFWRaceConfig` entry per race your game supports. The component resolves the correct config at runtime based on the character's `EFWRace` value.

!!! warning "One Config Per Race"
    The database expects exactly one `UFWRaceConfig` per `EFWRace` value. If you add two configs with the same race, the first one found will be used.

---

## Race Configuration

Each `UFWRaceConfig` defines the complete set of customization options available for a race.

### Creating a Race Config

1. Create a new Data Asset of type `FWRaceConfig`.
2. Set the **Race** enum to the target race (Human, Dwarf, or Orc).
3. Populate the option arrays with your customization option assets.

### Option Arrays

| Array | Option Type | Drives |
|---|---|---|
| `SkinTones` | `UFWSkinToneOption` | DI_Head, DI_Body textures |
| `EyeColors` | `UFWEyeColorOption` | DI_Eye textures |
| `HairStyles` | `UFWHairStyleOption` | Hair mesh + DI_Hair textures |
| `HairColors` | `UFWHairColorOption` | DI_Hair color parameter |
| `FacialHairOptions` | `UFWFacialHairOption` | Facial hair mesh + DI_Facials textures |
| `FaceOptions` | `UFWFaceOption` | DI_Head textures |
| `TattooOptions` | `UFWTattooOption` | DI_Head, DI_Body overlay textures |
| `ScarOptions` | `UFWScarOption` | DI_Head overlay textures |
| `TuskOptions` | `UFWTuskOption` | Tusk mesh attachment |
| `EarringOptions` | `UFWEarringOption` | Earring mesh attachment |
| `PiercingOptions` | `UFWPiercingOption` | Piercing mesh attachment |

!!! info "Empty Arrays Are Valid"
    Not every race needs every slot. Orcs might have tusk options but Humans would not. Leave arrays empty for slots that do not apply to a given race. The component gracefully skips empty slot arrays.

---

## Customization Options

### Texture-Based Options

Options that modify appearance through textures (skin tones, eye colors, faces, tattoos, scars) use the `FFWCustomizationTextureSet` struct.

#### FFWCustomizationTextureSet

| Field | Type | Description |
|---|---|---|
| `Diffuse` | `TSoftObjectPtr<UTexture2D>` | Base color / albedo texture |
| `Normal` | `TSoftObjectPtr<UTexture2D>` | Normal map |
| `MetallicRoughness` | `TSoftObjectPtr<UTexture2D>` | Packed metallic (R) and roughness (G) |
| `Mask` | `TSoftObjectPtr<UTexture2D>` | Blend mask for layered effects |

=== "Skin Tone Example"

    ```
    DA_SkinTone_Light
        Diffuse:            T_Skin_Light_D
        Normal:             T_Skin_Light_N
        MetallicRoughness:  T_Skin_Light_MR
        Mask:               T_Skin_Light_M
    ```

=== "Tattoo Example"

    ```
    DA_Tattoo_Tribal
        Diffuse:            T_Tattoo_Tribal_D
        Normal:             (none)
        MetallicRoughness:  (none)
        Mask:               T_Tattoo_Tribal_M
    ```

    Tattoos typically only need a diffuse texture and a mask for blending. Leave unused texture slots empty.

!!! tip "Soft Object Pointers"
    All texture references use `TSoftObjectPtr`, meaning they are loaded on demand. This prevents the customization database from pulling every texture into memory at startup. The component handles async loading when a selection changes.

---

### Mesh-Based Options

Options that swap geometry (hair styles, facial hair, tusks, earrings, piercings) reference mesh assets in addition to optional textures.

| Option Type | Mesh Field | Texture Set |
|---|---|---|
| `UFWHairStyleOption` | `Mesh` (skeletal or static) | Yes -- applied to DI_Hair |
| `UFWFacialHairOption` | `Mesh` (skeletal or static) | Yes -- applied to DI_Facials |
| `UFWTuskOption` | `Mesh` (static) | No |
| `UFWEarringOption` | `Mesh` (static) | No |
| `UFWPiercingOption` | `Mesh` (static) | No |

Accessory meshes (tusks, earrings, piercings) are purely geometric -- they use whatever material is assigned on the mesh asset directly, without DMI involvement.

---

### Color Options

`UFWHairColorOption` is unique in that it applies a color parameter rather than a full texture set:

```
DA_HairColor_Brown
    Color: (R=0.35, G=0.2, B=0.1, A=1.0)
```

The component sets a `HairColor` vector parameter on DI_Hair at runtime.

---

## Unlock Requirements

Every customization option can optionally define an `FFWUnlockRequirement` to gate access.

### FFWUnlockRequirement Fields

| Field | Type | Description |
|---|---|---|
| `UnlockType` | `EFWUnlockType` | Category of requirement (quest, achievement, currency, etc.) |
| `RequirementId` | `FName` | Identifier for the specific requirement (quest ID, achievement name) |
| `RequiredValue` | `int32` | Threshold value (e.g., currency amount, level requirement) |

### Example Configurations

=== "Quest Unlock"

    ```
    UnlockType:     Quest
    RequirementId:  "QST_Dragon_Slayer"
    RequiredValue:  1
    ```

    The option unlocks when the player has completed the specified quest.

=== "Currency Unlock"

    ```
    UnlockType:     Currency
    RequirementId:  "Gold"
    RequiredValue:  5000
    ```

    The option unlocks when the player has at least 5000 gold.

=== "No Requirement"

    Leave the `UnlockRequirement` at its default values (or set `UnlockType` to `None`) to make an option available immediately.

!!! note "Unlock Validation Is External"
    FWCustomizationSystem defines the requirement data but does not enforce it. Your game logic is responsible for checking whether a player meets the requirements before allowing selection. The component will apply any selection you pass to it regardless of unlock state.

---

## Recommended Folder Structure

Organize your customization assets in a clean hierarchy:

```
Content/
    Data/
        Customization/
            DA_CustomizationDatabase.uasset
            Human/
                DA_RaceConfig_Human.uasset
                SkinTones/
                    DA_SkinTone_Light.uasset
                    DA_SkinTone_Medium.uasset
                    DA_SkinTone_Dark.uasset
                HairStyles/
                    DA_Hair_Short.uasset
                    DA_Hair_Long.uasset
                EyeColors/
                    DA_Eye_Blue.uasset
                    DA_Eye_Brown.uasset
                ...
            Dwarf/
                DA_RaceConfig_Dwarf.uasset
                ...
            Orc/
                DA_RaceConfig_Orc.uasset
                TuskOptions/
                    DA_Tusk_Small.uasset
                    DA_Tusk_Large.uasset
                ...
    Textures/
        Customization/
            Human/
                T_Skin_Light_D.uasset
                T_Skin_Light_N.uasset
                ...
    Meshes/
        Customization/
            Hair/
                SM_Hair_Short.uasset
                SM_Hair_Long.uasset
            FacialHair/
                SM_Beard_Full.uasset
            ...
```

---

## Gender-Specific Configuration

Race configs apply to both genders by default. Gender-specific meshes and textures are handled within the option assets themselves -- each option can reference gender-specific variants internally.

If your game requires entirely different option sets per gender (e.g., different hair styles for male vs. female), create separate option arrays within your race config and filter them based on the active gender in your UI logic. The component's `GetGender()` function provides the current gender for this purpose.

---

## Performance Considerations

| Factor | Recommendation |
|---|---|
| **Texture count** | Keep option arrays under 32 entries per slot to limit memory pressure |
| **Texture resolution** | Use 1024x1024 for body/head diffuse; 512x512 for accessories |
| **Soft references** | Already used by default -- no manual async loading needed |
| **DMI parameter updates** | Batched internally per `RefreshVisuals` call -- avoid calling per-frame |
| **Profile replication** | ~8 bytes per character -- negligible bandwidth impact |
