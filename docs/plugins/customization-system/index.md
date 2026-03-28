---
title: FWCustomizationSystem
---

# FWCustomizationSystem

**No external dependencies**

Data-driven character customization with race/gender support, dynamic material instances, and unlock tracking. FWCustomizationSystem gives you a complete pipeline for building character creators and in-game appearance changes without writing material or mesh-switching logic by hand.

---

## Key Features

- **Race and gender support.** Define distinct body types, meshes, and texture sets per race/gender combination. Ships with Human, Dwarf, and Orc race presets.
- **Dynamic Material Instances (DMI).** Five material slots -- Head, Eye, Body, Hair, and Facials -- are created at runtime and driven by data. Swap skin tones, eye colors, hair colors, tattoos, and scars without duplicating materials.
- **Data-driven customization database.** All options are defined as DataAsset entries in a central `UFWCustomizationDatabase`. Artists add new hairstyles, face options, or tusk variants without touching code.
- **Compact replication profile.** The `FFWCustomizationProfile` struct packs a full appearance loadout into approximately 8 bytes, making it efficient for multiplayer replication.
- **Unlock tracking.** `UFWCustomizationUnlockSubsystem` manages unlock state with persistence, requirement checks, and integration hooks for quest/achievement/currency systems.
- **Blueprint-friendly API.** Every setter, getter, and event is exposed to Blueprints, so UI designers can build character creators entirely in UMG.

---

## Architecture Overview

```
UFWCustomizationDatabase (DataAsset)
    |
    +-- UFWRaceConfig (per race)
    |       |
    |       +-- Skin Tones, Eye Colors, Hair Styles, Hair Colors
    |       +-- Facial Hair, Faces, Tattoos, Scars
    |       +-- Tusks, Earrings, Piercings
    |
    +-- Used by UFWCustomizationComponent
    |       |
    |       +-- Profile Management (Get/Set/Apply)
    |       +-- Option Accessors (per-slot getters)
    |       +-- Creates DMIs (Head, Eye, Body, Hair, Facials)
    |       +-- Swaps meshes (Hair, Facial Hair, Tusks, Earrings, Piercings)
    |       +-- Broadcasts events (OnSlotChanged, OnProfileApplied, ...)
    |
    +-- UFWCustomizationUnlockSubsystem
            |
            +-- Unlock Checks and Unlocking
            +-- Integration Hooks (quest, achievement, currency)
            +-- Persistence
            +-- Events
```

The component reads from a `UFWCustomizationDatabase`, resolves the active `UFWRaceConfig` for the character's race, and applies the selected options by updating DMI parameters and swapping static/skeletal mesh components. The unlock subsystem manages which options are available based on player progression.

---

## Quick Navigation

| Page | Description |
|---|---|
| [Installation](installation.md) | Add the plugin to your project |
| [Quick Start](quick-start.md) | Get a character customization running in 10 minutes |
| [API Reference](api-reference.md) | Class, function, and property reference |
| [Blueprints](blueprints.md) | Blueprint integration patterns and examples |
| [Configuration](configuration.md) | Database setup, race configs, and customization options |
| [Tutorials](tutorials.md) | Step-by-step guide: Building a Character Creator |
| [FAQ](faq.md) | Common questions and troubleshooting |

---

## Supported Customization Slots

| Slot | Type | DMI Driven | Mesh Swap |
|---|---|---|---|
| Skin Tone | Texture set | Yes (Body, Head) | No |
| Eye Color | Texture set | Yes (Eye) | No |
| Hair Style | Mesh + texture | Yes (Hair) | Yes |
| Hair Color | Color parameter | Yes (Hair) | No |
| Facial Hair | Mesh + texture | Yes (Facials) | Yes |
| Face | Texture set | Yes (Head) | No |
| Tattoo | Texture overlay | Yes (Body, Head) | No |
| Scar | Texture overlay | Yes (Head) | No |
| Tusk | Mesh | No | Yes |
| Earring | Mesh | No | Yes |
| Piercing | Mesh | No | Yes |
