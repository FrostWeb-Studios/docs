---
title: API Reference - FWCustomizationSystem
---

# API Reference

Reference for all public classes, components, data assets, and types in FWCustomizationSystem.

---

## UFWCustomizationComponent

The core runtime component. Attach it to any Actor to give it data-driven character customization. Manages DMI creation, mesh swapping, profile state, and event broadcasting.

**Parent:** `UActorComponent`

### Profile Management

| Function | Return | Description |
|----------|--------|-------------|
| `SetDatabase(InDatabase)` | `void` | Assign the customization database. Must be called before other functions |
| `SetProfile(InProfile)` | `void` | Apply a complete customization profile. Triggers full visual refresh and broadcasts `OnProfileApplied` |
| `GetProfile()` | `FFWCustomizationProfile` | Returns the current profile as a compact struct (~8 bytes) |
| `SetRace(InRace)` | `void` | Change the character's race. Resolves a new race config and refreshes visuals |
| `SetGender(InGender)` | `void` | Change the character's gender. Triggers a visual refresh |
| `SetSlotSelection(Slot, Index)` | `void` | Set a single slot to the specified option index. Broadcasts `OnSlotChanged` |
| `RefreshVisuals()` | `void` | Force a complete visual refresh of all slots |
| `CreateDynamicMaterialInstances()` | `void` | Manually create or recreate all five DMI slots |

### Option Accessors

| Function | Return | Description |
|----------|--------|-------------|
| `GetRace()` | `EFWRace` | Current race |
| `GetGender()` | `EFWGender` | Current gender |
| `GetSlotSelection(Slot)` | `int32` | Currently selected index for a slot (-1 if none) |
| `GetCurrentSkinToneOption()` | `UFWSkinToneOption*` | Active skin tone option asset |
| `GetCurrentEyeColorOption()` | `UFWEyeColorOption*` | Active eye color option asset |
| `GetCurrentHairStyleOption()` | `UFWHairStyleOption*` | Active hair style option asset |
| `GetCurrentHairColorOption()` | `UFWHairColorOption*` | Active hair color option asset |
| `GetCurrentFacialHairOption()` | `UFWFacialHairOption*` | Active facial hair option asset |
| `GetCurrentFaceOption()` | `UFWFaceOption*` | Active face option asset |
| `GetCurrentTattooOption()` | `UFWTattooOption*` | Active tattoo option asset |
| `GetCurrentScarOption()` | `UFWScarOption*` | Active scar option asset |
| `GetCurrentTuskOption()` | `UFWTuskOption*` | Active tusk option asset |
| `GetCurrentEarringOption()` | `UFWEarringOption*` | Active earring option asset |
| `GetCurrentPiercingOption()` | `UFWPiercingOption*` | Active piercing option asset |
| `GetDatabase()` | `UFWCustomizationDatabase*` | The assigned customization database |
| `GetCurrentRaceConfig()` | `UFWRaceConfig*` | The resolved race config for the current race |

### DMI Access

| Property | Type | Description |
|----------|------|-------------|
| `DI_Head` | `UMaterialInstanceDynamic*` | Controls head textures: skin tone, face, scars, tattoos |
| `DI_Eye` | `UMaterialInstanceDynamic*` | Controls eye color textures |
| `DI_Body` | `UMaterialInstanceDynamic*` | Controls body textures: skin tone, tattoos |
| `DI_Hair` | `UMaterialInstanceDynamic*` | Controls hair textures and color |
| `DI_Facials` | `UMaterialInstanceDynamic*` | Controls facial hair textures |

!!! warning "Do Not Cache DMI Pointers"
    DMI references are invalidated whenever `CreateDynamicMaterialInstances` is called. Always query them from the component rather than storing long-lived pointers.

### Mesh Getters

| Function | Return | Description |
|----------|--------|-------------|
| `GetHairMesh()` | `UMeshComponent*` | Active hair mesh component |
| `GetFacialHairMesh()` | `UMeshComponent*` | Active facial hair mesh component |
| `GetTuskMesh()` | `UMeshComponent*` | Active tusk mesh component |
| `GetEarringMesh()` | `UMeshComponent*` | Active earring mesh component |
| `GetPiercingMesh()` | `UMeshComponent*` | Active piercing mesh component |

### Events

All events are `BlueprintAssignable` and can be bound in both C++ and Blueprints.

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnSlotChanged` | `Slot`, `NewIndex` | Fired when any single slot selection changes |
| `OnProfileApplied` | (none) | Fired when a complete profile is applied via `SetProfile` |
| `OnHairStyleChanged` | `NewOption` | Fired when the hair style changes |
| `OnFacialHairChanged` | `NewOption` | Fired when the facial hair changes |
| `OnFaceChanged` | `NewOption` | Fired when the face option changes |
| `OnTuskChanged` | `NewOption` | Fired when the tusk option changes |
| `OnEarringChanged` | `NewOption` | Fired when the earring option changes |
| `OnPiercingChanged` | `NewOption` | Fired when the piercing option changes |

---

## UFWCustomizationUnlockSubsystem

Game instance subsystem that manages unlock state for customization options. Provides requirement checks, unlocking, persistence, and integration hooks for external systems.

**Parent:** `UGameInstanceSubsystem`

### Unlock Checks

| Function | Return | Description |
|----------|--------|-------------|
| `IsOptionUnlocked(Option)` | `bool` | Check if a specific customization option is unlocked |
| `AreRequirementsMet(Requirements)` | `bool` | Check if all unlock requirements in an array are satisfied |
| `GetUnlockedOptionsForSlot(Slot, RaceConfig)` | `TArray<int32>` | Get indices of all unlocked options for a given slot and race |

### Unlocking

| Function | Return | Description |
|----------|--------|-------------|
| `UnlockOption(Option)` | `bool` | Unlock a specific customization option. Returns `false` if already unlocked |
| `UnlockOptionBySlotAndIndex(Slot, Index, RaceConfig)` | `bool` | Unlock an option by slot type and index within a race config |

### Integration Hooks

| Function | Description |
|----------|-------------|
| `OnQuestCompleted(QuestId)` | Notify the subsystem that a quest was completed. Checks and unlocks eligible options |
| `OnAchievementUnlocked(AchievementId)` | Notify that an achievement was earned |
| `OnCurrencyChanged(CurrencyType, NewAmount)` | Notify that a currency balance changed |

### Persistence

| Function | Description |
|----------|-------------|
| `SaveUnlockState()` | Persist current unlock state |
| `LoadUnlockState()` | Load previously saved unlock state |
| `ResetUnlockState()` | Clear all unlocked options and reset to defaults |

### Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnOptionUnlocked` | `Option` | Fired when a customization option is newly unlocked |
| `OnUnlockStateLoaded` | (none) | Fired when unlock state has been loaded from persistence |
| `OnUnlockStateReset` | (none) | Fired when unlock state has been reset |

---

## Data Assets

### UFWCustomizationDatabase

The root container that holds all race configurations. Assign to `UFWCustomizationComponent` via `SetDatabase()`.

**Parent:** `UPrimaryDataAsset`

| Property | Type | Description |
|----------|------|-------------|
| `RaceConfigs` | `TArray<UFWRaceConfig*>` | Array of race configuration assets |

---

### UFWRaceConfig

Defines all customization options available for a specific race. Each race (Human, Dwarf, Orc) should have its own config.

**Parent:** `UPrimaryDataAsset`

| Property | Type | Description |
|----------|------|-------------|
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

All option assets extend `UPrimaryDataAsset`. Each type defines the visual data for one selectable option.

| Asset Class | Key Properties | Description |
|---|---|---|
| `UFWSkinToneOption` | `TextureSet`, `UnlockRequirement` | A skin tone variant with full texture set |
| `UFWEyeColorOption` | `TextureSet`, `UnlockRequirement` | An eye color variant |
| `UFWHairStyleOption` | `Mesh`, `TextureSet`, `UnlockRequirement` | A hair mesh with associated textures |
| `UFWHairColorOption` | `Color (FLinearColor)`, `UnlockRequirement` | A hair color applied as a DMI parameter |
| `UFWFacialHairOption` | `Mesh`, `TextureSet`, `UnlockRequirement` | Facial hair mesh and textures |
| `UFWFaceOption` | `TextureSet`, `UnlockRequirement` | A face variant (applied to Head DMI) |
| `UFWTattooOption` | `TextureSet`, `UnlockRequirement` | Tattoo overlay textures |
| `UFWScarOption` | `TextureSet`, `UnlockRequirement` | Scar overlay textures |
| `UFWTuskOption` | `Mesh`, `UnlockRequirement` | Tusk mesh |
| `UFWEarringOption` | `Mesh`, `UnlockRequirement` | Earring mesh attachment |
| `UFWPiercingOption` | `Mesh`, `UnlockRequirement` | Piercing mesh attachment |

---

## Types

### EFWRace

| Value | Description |
|-------|-------------|
| `Human` | Human race |
| `Dwarf` | Dwarf race |
| `Orc` | Orc race |

### EFWGender

| Value | Description |
|-------|-------------|
| `Male` | Male gender |
| `Female` | Female gender |

### EFWCustomizationSlot

| Value | Description |
|-------|-------------|
| `SkinTone` | Skin tone slot |
| `EyeColor` | Eye color slot |
| `HairStyle` | Hair style slot |
| `HairColor` | Hair color slot |
| `FacialHair` | Facial hair slot |
| `Face` | Face slot |
| `Tattoo` | Tattoo slot |
| `Scar` | Scar slot |
| `Tusk` | Tusk slot |
| `Earring` | Earring slot |
| `Piercing` | Piercing slot |

### EFWUnlockType

Enum for the type of unlock requirement a customization option may have (quest completion, achievement, currency purchase, etc.).

### FFWCustomizationProfile

A compact struct (~8 bytes) that encodes the full customization state including race, gender, and all slot selections. Designed for efficient network replication. Use `SetProfile` / `GetProfile` to apply or read the complete state in one operation.

!!! info "Replication Size"
    The profile struct is intentionally compact so that it can be replicated cheaply as part of player state. Each slot index is packed into a few bits, since option arrays rarely exceed 16-32 entries.

### FFWCustomizationTextureSet

A bundle of textures used by DMI-driven customization options. Uses soft object pointers for async loading.

| Field | Type | Description |
|-------|------|-------------|
| `Diffuse` | `TSoftObjectPtr<UTexture2D>` | Diffuse/albedo texture |
| `Normal` | `TSoftObjectPtr<UTexture2D>` | Normal map texture |
| `MetallicRoughness` | `TSoftObjectPtr<UTexture2D>` | Metallic/roughness texture |
| `Mask` | `TSoftObjectPtr<UTexture2D>` | Mask texture |

### FFWUnlockRequirement

Defines what a player must achieve to unlock a customization option.

| Field | Type | Description |
|-------|------|-------------|
| `UnlockType` | `EFWUnlockType` | Type of requirement (quest, achievement, currency, etc.) |
| `RequirementId` | `FName` | Context-dependent identifier (quest ID, achievement ID, currency type) |
| `RequiredValue` | `int32` | Threshold value to satisfy the requirement |
