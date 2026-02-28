---
title: Changelog - FWCustomizationSystem
---

# Changelog

All notable changes to FWCustomizationSystem are documented on this page.

The format follows [Keep a Changelog](https://keepachangelog.com/). Versions use [Semantic Versioning](https://semver.org/).

---

## [1.0.0] -- 2026-02-27

### Added

- **UFWCustomizationComponent** -- Core actor component with full Blueprint API for data-driven character customization.
- **Dynamic Material Instance (DMI) system** -- Five material slots (DI_Head, DI_Eye, DI_Body, DI_Hair, DI_Facials) created and managed at runtime.
- **Race and gender support** -- `EFWRace` (Human, Dwarf, Orc) and `EFWGender` (Male, Female) enums with per-race configuration resolution.
- **Customization database architecture** -- `UFWCustomizationDatabase` root asset with `UFWRaceConfig` entries per race.
- **11 customization option types** -- SkinTone, EyeColor, HairStyle, HairColor, FacialHair, Face, Tattoo, Scar, Tusk, Earring, Piercing.
- **FFWCustomizationTextureSet** -- Reusable texture bundle (Diffuse, Normal, MetallicRoughness, Mask) with soft object pointer references for on-demand loading.
- **FFWCustomizationProfile** -- Compact (~8 bytes) serializable struct for efficient network replication and SaveGame storage.
- **FFWUnlockRequirement** -- Data-driven unlock gating per option (quest, achievement, currency, etc.).
- **Mesh swap support** -- Automatic mesh component creation and destruction for Hair, FacialHair, Tusk, Earring, and Piercing slots.
- **Event delegates** -- OnSlotChanged, OnProfileApplied, OnHairStyleChanged, OnFacialHairChanged, OnFaceChanged, OnTuskChanged, OnEarringChanged, OnPiercingChanged.
- **Full Blueprint exposure** -- All functions marked BlueprintCallable or BlueprintPure, all structs and enums marked BlueprintType.
