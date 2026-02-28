---
title: API Reference - FWCustomizationSystem
---

# API Reference

Complete reference for all classes, functions, events, and types in FWCustomizationSystem.

---

## UFWCustomizationComponent

**Parent Class:** `UActorComponent`
**Specifiers:** `BlueprintSpawnableComponent`

The core runtime component. Attach it to any Actor to give it data-driven character customization. It manages DMI creation, mesh swapping, and profile state.

---

### BlueprintCallable Functions

#### SetDatabase

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Customization")
void SetDatabase(UFWCustomizationDatabase* InDatabase);
```

Assigns the customization database that this component reads options from. Must be called before any other customization functions.

| Parameter | Type | Description |
|---|---|---|
| `InDatabase` | `UFWCustomizationDatabase*` | The database asset containing all race configs and options |

---

#### SetProfile

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Customization")
void SetProfile(const FFWCustomizationProfile& InProfile);
```

Applies a complete customization profile to the character. This sets all slots at once and triggers a full visual refresh. Broadcasts `OnProfileApplied`.

| Parameter | Type | Description |
|---|---|---|
| `InProfile` | `const FFWCustomizationProfile&` | Compact profile struct (~8 bytes) containing all slot selections |

---

#### SetSlotSelection

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Customization")
void SetSlotSelection(EFWCustomizationSlot Slot, int32 Index);
```

Sets a single customization slot to the specified option index. Triggers visual update for that slot and broadcasts `OnSlotChanged`.

| Parameter | Type | Description |
|---|---|---|
| `Slot` | `EFWCustomizationSlot` | Which slot to change (SkinTone, HairStyle, etc.) |
| `Index` | `int32` | Zero-based index into the option array for the current race config |

---

#### SetRace

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Customization")
void SetRace(EFWRace InRace);
```

Changes the character's race. This resolves a new `UFWRaceConfig` from the database and triggers a full visual refresh, since available options change per race.

| Parameter | Type | Description |
|---|---|---|
| `InRace` | `EFWRace` | The target race (Human, Dwarf, Orc) |

---

#### SetGender

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Customization")
void SetGender(EFWGender InGender);
```

Changes the character's gender. Triggers a visual refresh to apply gender-specific meshes and textures.

| Parameter | Type | Description |
|---|---|---|
| `InGender` | `EFWGender` | The target gender (Male, Female) |

---

#### RefreshVisuals

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Customization")
void RefreshVisuals();
```

Forces a complete visual refresh of all slots. Recreates DMIs if needed and reapplies all current selections. Call this after bulk changes or when the owning mesh component changes.

---

#### CreateDynamicMaterialInstances

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Customization")
void CreateDynamicMaterialInstances();
```

Manually creates (or recreates) all five DMI slots (Head, Eye, Body, Hair, Facials). Normally called internally by `RefreshVisuals`, but can be called directly if you need to reinitialize materials after a mesh swap.

---

### BlueprintPure Functions

#### GetProfile

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
FFWCustomizationProfile GetProfile() const;
```

Returns the current customization profile as a compact struct suitable for replication or serialization.

**Returns:** `FFWCustomizationProfile` -- The current profile (~8 bytes).

---

#### GetSlotSelection

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
int32 GetSlotSelection(EFWCustomizationSlot Slot) const;
```

Returns the currently selected option index for a given slot.

| Parameter | Type | Description |
|---|---|---|
| `Slot` | `EFWCustomizationSlot` | The slot to query |

**Returns:** `int32` -- Zero-based index, or `-1` if the slot has no selection.

---

#### GetRace / GetGender

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
EFWRace GetRace() const;

UFUNCTION(BlueprintPure, Category = "FW|Customization")
EFWGender GetGender() const;
```

Return the character's current race and gender enum values.

---

#### GetCurrentSkinToneOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWSkinToneOption* GetCurrentSkinToneOption() const;
```

Returns the currently selected skin tone option asset, or `nullptr` if none is selected.

---

#### GetCurrentEyeColorOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWEyeColorOption* GetCurrentEyeColorOption() const;
```

Returns the currently selected eye color option asset.

---

#### GetCurrentHairStyleOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWHairStyleOption* GetCurrentHairStyleOption() const;
```

Returns the currently selected hair style option asset.

---

#### GetCurrentHairColorOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWHairColorOption* GetCurrentHairColorOption() const;
```

Returns the currently selected hair color option asset.

---

#### GetCurrentFacialHairOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWFacialHairOption* GetCurrentFacialHairOption() const;
```

Returns the currently selected facial hair option asset.

---

#### GetCurrentFaceOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWFaceOption* GetCurrentFaceOption() const;
```

Returns the currently selected face option asset.

---

#### GetCurrentTattooOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWTattooOption* GetCurrentTattooOption() const;
```

Returns the currently selected tattoo option asset.

---

#### GetCurrentScarOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWScarOption* GetCurrentScarOption() const;
```

Returns the currently selected scar option asset.

---

#### GetCurrentTuskOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWTuskOption* GetCurrentTuskOption() const;
```

Returns the currently selected tusk option asset (primarily used by Orc race).

---

#### GetCurrentEarringOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWEarringOption* GetCurrentEarringOption() const;
```

Returns the currently selected earring option asset.

---

#### GetCurrentPiercingOption

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWPiercingOption* GetCurrentPiercingOption() const;
```

Returns the currently selected piercing option asset.

---

#### Mesh Getters

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UMeshComponent* GetHairMesh() const;

UFUNCTION(BlueprintPure, Category = "FW|Customization")
UMeshComponent* GetFacialHairMesh() const;

UFUNCTION(BlueprintPure, Category = "FW|Customization")
UMeshComponent* GetTuskMesh() const;

UFUNCTION(BlueprintPure, Category = "FW|Customization")
UMeshComponent* GetEarringMesh() const;

UFUNCTION(BlueprintPure, Category = "FW|Customization")
UMeshComponent* GetPiercingMesh() const;
```

Return the mesh components currently being used for each attachment slot. Returns `nullptr` if the slot has no active mesh.

---

#### GetDatabase

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWCustomizationDatabase* GetDatabase() const;
```

Returns the assigned customization database.

---

#### GetCurrentRaceConfig

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Customization")
UFWRaceConfig* GetCurrentRaceConfig() const;
```

Returns the resolved `UFWRaceConfig` for the character's current race. This is the config actively providing customization options.

---

### Events (Delegates)

All events are `BlueprintAssignable` and can be bound in both C++ and Blueprints.

| Event | Signature | Description |
|---|---|---|
| `OnSlotChanged` | `(EFWCustomizationSlot Slot, int32 NewIndex)` | Fired when any single slot selection changes |
| `OnProfileApplied` | `()` | Fired when a complete profile is applied via `SetProfile` |
| `OnHairStyleChanged` | `(UFWHairStyleOption* NewOption)` | Fired when the hair style changes |
| `OnFacialHairChanged` | `(UFWFacialHairOption* NewOption)` | Fired when the facial hair changes |
| `OnFaceChanged` | `(UFWFaceOption* NewOption)` | Fired when the face option changes |
| `OnTuskChanged` | `(UFWTuskOption* NewOption)` | Fired when the tusk option changes |
| `OnEarringChanged` | `(UFWEarringOption* NewOption)` | Fired when the earring option changes |
| `OnPiercingChanged` | `(UFWPiercingOption* NewOption)` | Fired when the piercing option changes |

---

### DMI Properties

These are the Dynamic Material Instance references managed by the component. They are created automatically during `RefreshVisuals` or `CreateDynamicMaterialInstances`.

| Property | Type | Description |
|---|---|---|
| `DI_Head` | `UMaterialInstanceDynamic*` | Controls head textures: skin tone, face, scars, tattoos |
| `DI_Eye` | `UMaterialInstanceDynamic*` | Controls eye color textures |
| `DI_Body` | `UMaterialInstanceDynamic*` | Controls body textures: skin tone, tattoos |
| `DI_Hair` | `UMaterialInstanceDynamic*` | Controls hair textures and color |
| `DI_Facials` | `UMaterialInstanceDynamic*` | Controls facial hair textures |

!!! warning "Do Not Cache DMI Pointers"
    DMI references are invalidated whenever `CreateDynamicMaterialInstances` is called. Always query them from the component rather than storing long-lived pointers.

---

## Data Assets

### UFWCustomizationDatabase

**Parent Class:** `UPrimaryDataAsset`

The root container that holds all race configurations. Assign this to `UFWCustomizationComponent::SetDatabase()`.

| Property | Type | Description |
|---|---|---|
| `RaceConfigs` | `TArray<UFWRaceConfig*>` | Array of race configuration assets |

---

### UFWRaceConfig

**Parent Class:** `UPrimaryDataAsset`

Defines all customization options available for a specific race. Each race (Human, Dwarf, Orc) should have its own config.

| Property | Type | Description |
|---|---|---|
| `Race` | `EFWRace` | Which race this config applies to |
| `SkinTones` | `TArray<UFWSkinToneOption*>` | Available skin tone options |
| `EyeColors` | `TArray<UFWEyeColorOption*>` | Available eye color options |
| `HairStyles` | `TArray<UFWHairStyleOption*>` | Available hair styles (mesh + texture) |
| `HairColors` | `TArray<UFWHairColorOption*>` | Available hair color options |
| `FacialHairOptions` | `TArray<UFWFacialHairOption*>` | Available facial hair (mesh + texture) |
| `FaceOptions` | `TArray<UFWFaceOption*>` | Available face options |
| `TattooOptions` | `TArray<UFWTattooOption*>` | Available tattoo overlays |
| `ScarOptions` | `TArray<UFWScarOption*>` | Available scar overlays |
| `TuskOptions` | `TArray<UFWTuskOption*>` | Available tusk meshes |
| `EarringOptions` | `TArray<UFWEarringOption*>` | Available earring meshes |
| `PiercingOptions` | `TArray<UFWPiercingOption*>` | Available piercing meshes |

---

### Customization Option Assets

All option assets extend `UPrimaryDataAsset`. The following table lists each type and its key properties.

| Asset Class | Key Properties | Description |
|---|---|---|
| `UFWSkinToneOption` | `TextureSet (FFWCustomizationTextureSet)`, `UnlockRequirement` | A skin tone variant with full texture set |
| `UFWEyeColorOption` | `TextureSet`, `UnlockRequirement` | An eye color variant |
| `UFWHairStyleOption` | `Mesh`, `TextureSet`, `UnlockRequirement` | A hair mesh with associated textures |
| `UFWHairColorOption` | `Color (FLinearColor)`, `UnlockRequirement` | A hair color applied as a DMI parameter |
| `UFWFacialHairOption` | `Mesh`, `TextureSet`, `UnlockRequirement` | Facial hair mesh and textures |
| `UFWFaceOption` | `TextureSet`, `UnlockRequirement` | A face variant (applied to Head DMI) |
| `UFWTattooOption` | `TextureSet`, `UnlockRequirement` | Tattoo overlay textures |
| `UFWScarOption` | `TextureSet`, `UnlockRequirement` | Scar overlay textures |
| `UFWTuskOption` | `Mesh`, `UnlockRequirement` | Tusk mesh (Orc-specific) |
| `UFWEarringOption` | `Mesh`, `UnlockRequirement` | Earring mesh attachment |
| `UFWPiercingOption` | `Mesh`, `UnlockRequirement` | Piercing mesh attachment |
| `UFWTextureCustomizationOption` | `TextureSet`, `UnlockRequirement` | Generic base class for texture-based options |

---

## Types and Enums

### EFWRace

```cpp
UENUM(BlueprintType)
enum class EFWRace : uint8
{
    Human,
    Dwarf,
    Orc
};
```

---

### EFWGender

```cpp
UENUM(BlueprintType)
enum class EFWGender : uint8
{
    Male,
    Female
};
```

---

### EFWCustomizationSlot

```cpp
UENUM(BlueprintType)
enum class EFWCustomizationSlot : uint8
{
    SkinTone,
    EyeColor,
    HairStyle,
    HairColor,
    FacialHair,
    Face,
    Tattoo,
    Scar,
    Tusk,
    Earring,
    Piercing
};
```

---

### EFWMeshMaterialCategory

Enum identifying which material category a mesh belongs to, used internally for DMI assignment.

---

### EFWBodyMeshSlot

Enum identifying body mesh attachment points (head, torso, limbs) for mesh-swap operations.

---

### EFWUnlockType

Enum for the type of unlock requirement an option may have (quest, achievement, currency, etc.).

---

### FFWCustomizationTextureSet

```cpp
USTRUCT(BlueprintType)
struct FFWCustomizationTextureSet
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSoftObjectPtr<UTexture2D> Diffuse;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSoftObjectPtr<UTexture2D> Normal;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSoftObjectPtr<UTexture2D> MetallicRoughness;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSoftObjectPtr<UTexture2D> Mask;
};
```

A bundle of textures used by DMI-driven customization options. Soft object pointers allow async loading.

---

### FFWUnlockRequirement

```cpp
USTRUCT(BlueprintType)
struct FFWUnlockRequirement
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    EFWUnlockType UnlockType;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FName RequirementId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 RequiredValue;
};
```

Defines what a player must achieve to unlock a customization option. The `RequirementId` is context-dependent (quest ID, achievement ID, currency type, etc.).

---

### FFWCustomizationProfile

```cpp
USTRUCT(BlueprintType)
struct FFWCustomizationProfile
{
    GENERATED_BODY()

    // Packed slot selections -- approximately 8 bytes total
    // Contains Race, Gender, and index for each customization slot
};
```

A compact struct that encodes the full customization state. Designed for efficient network replication. Use `SetProfile` / `GetProfile` to apply or read the complete state in one operation.

!!! info "Replication Size"
    The profile struct is intentionally compact (~8 bytes) so that it can be replicated cheaply as part of player state. Each slot index is packed into a few bits, since option arrays rarely exceed 16-32 entries.
