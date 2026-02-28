---
title: Migration Guide - FWSkillSystem
---

# Migration Guide

Instructions for upgrading FWSkillSystem between versions.

---

## Current Version: 1.0

This is the initial release of FWSkillSystem. No migration steps are required.

---

## General Upgrade Instructions

When upgrading to a future version of FWSkillSystem, follow these steps:

### Before Upgrading

1. **Back up your project.** Create a full backup of your project directory before replacing plugin files.
2. **Review the changelog.** Read the [Changelog](changelog.md) for the target version to understand what changed, what was deprecated, and what breaking changes exist.
3. **Check deprecation warnings.** If the previous version emitted deprecation warnings during compilation, address them before upgrading. Deprecated APIs are typically removed in the next major version.

### Upgrade Procedure

1. **Close the Unreal Editor.**
2. **Replace the plugin folder.** Delete the existing `Plugins/FWSkillSystem/` directory and copy the new version in its place.
3. **Regenerate project files.** Right-click your `.uproject` file and select **Generate Visual Studio project files**.
4. **Compile the project.** Build from your IDE. Fix any compilation errors according to the changelog's migration notes.
5. **Open the editor and verify.** Launch the Unreal Editor and check the Output Log for warnings or errors related to FWSkillSystem.

### Data Asset Compatibility

FWSkillSystem data assets (`UFWSkillDatabase`, `UFWSkillDefinition`, `UFWSkillXPCurve`, `UFWSkillMilestoneDefinition`) use standard Unreal serialization. In most cases, data assets created in an earlier version will load in a newer version without modification. If a version introduces property renames or structural changes, the changelog will include specific migration instructions.

### Save Data Compatibility

If your game persists `FFWSkillData` to save files or a backend, check the changelog for any changes to the struct layout. Adding new fields to `FFWSkillData` in a future version will not break deserialization of old save data (new fields will use default values), but removing or renaming fields would require a data migration step.

!!! tip "Version Stamping"
    Consider storing a version number alongside your serialized skill data. This makes it straightforward to detect and migrate old save formats when upgrading the plugin.

---

## Version History

| Version | Status | Notes |
|---|---|---|
| 1.0 | Current | Initial release. No migration required. |
