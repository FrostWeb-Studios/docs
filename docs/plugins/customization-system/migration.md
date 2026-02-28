---
title: Migration Guide - FWCustomizationSystem
---

# Migration Guide

This guide covers breaking changes and upgrade steps between versions of FWCustomizationSystem.

---

## Version Compatibility

| Plugin Version | UE Version | Notes |
|---|---|---|
| 1.0 | 5.4+ | Initial release |

---

## Current Version: 1.0

This is the initial release of FWCustomizationSystem. There are no prior versions to migrate from.

---

## Future Migration Notes

When upgrading to future versions, this page will document:

- **Breaking API changes** -- renamed or removed functions, changed signatures
- **Data asset changes** -- new required fields, struct layout changes
- **Profile format changes** -- if `FFWCustomizationProfile` packing changes, saved profiles may need conversion
- **Enum changes** -- new values added to `EFWRace`, `EFWGender`, or `EFWCustomizationSlot`

---

## General Upgrade Procedure

When a new version is released, follow these steps:

### Step 1 -- Back Up

Before upgrading, back up your project or ensure all changes are committed to version control.

### Step 2 -- Replace Plugin Files

Delete the existing `Plugins/FWCustomizationSystem/` folder and replace it with the new version.

!!! warning "Do Not Merge"
    Always do a clean replacement of the plugin folder. Do not attempt to merge old and new plugin files, as internal implementation files may have changed significantly.

### Step 3 -- Regenerate Project Files

Right-click your `.uproject` file and select **Generate Visual Studio project files**.

### Step 4 -- Compile and Fix Errors

Build your project. If the new version includes breaking API changes, the compiler will flag them. Refer to the version-specific migration notes above for the exact changes and how to update your code.

### Step 5 -- Verify Data Assets

Open your `UFWCustomizationDatabase` and `UFWRaceConfig` assets in the editor. Check for any new required fields that may have been added. The editor will show default values for new fields, but you should review them to ensure correct behavior.

### Step 6 -- Test Profiles

If the `FFWCustomizationProfile` format changed, load existing saved profiles and verify they apply correctly. The changelog will note if profile migration is needed.

---

## Deprecation Policy

- Deprecated functions will be marked with `UE_DEPRECATED` and will continue to compile with warnings for at least one minor version.
- Removed functions will be listed in the migration guide with their replacements.
- Data asset changes that require re-saving assets will be called out explicitly.
