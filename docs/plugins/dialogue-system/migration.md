---
title: Migration Guide - FWDialogueSystem
---

# Migration Guide

This guide covers breaking changes and upgrade steps between versions of FWDialogueSystem.

---

## Version Compatibility

| Plugin Version | UE Version | Notes |
|---|---|---|
| 1.0 | 5.4+ | Initial release |

---

## Current Version: 1.0

This is the initial release of FWDialogueSystem. There are no prior versions to migrate from.

---

## Future Migration Notes

When upgrading to future versions, this page will document:

- **Breaking API changes** -- renamed or removed functions, changed signatures
- **Data asset changes** -- new required fields on `FFWDialogueNode` or `FFWDialogueResponse`
- **Condition/Action interface changes** -- if `Evaluate` or `Execute` signatures change
- **Node ID resolution changes** -- if the node lookup mechanism changes

---

## General Upgrade Procedure

When a new version is released, follow these steps:

### Step 1 -- Back Up

Before upgrading, back up your project or ensure all changes are committed to version control. Pay special attention to dialogue tree data assets, as struct changes may require re-saving.

### Step 2 -- Replace Plugin Files

Delete the existing `Plugins/FWDialogueSystem/` folder and replace it with the new version.

!!! warning "Do Not Merge"
    Always do a clean replacement of the plugin folder. Do not attempt to merge old and new plugin files.

### Step 3 -- Regenerate Project Files

Right-click your `.uproject` file and select **Generate Visual Studio project files**.

### Step 4 -- Compile and Fix Errors

Build your project. If the new version includes breaking API changes, the compiler will flag them. Refer to the version-specific migration notes above.

### Step 5 -- Verify Custom Conditions and Actions

If you have created custom subclasses of `UFWDialogueConditionBase` or `UFWDialogueActionBase`, verify that the base class interface has not changed. Compile errors in your subclasses indicate that `Evaluate` or `Execute` signatures were modified.

### Step 6 -- Re-save Dialogue Trees

If the `FFWDialogueNode` or `FFWDialogueResponse` struct layouts changed, you may need to open and re-save your dialogue tree data assets. The Unreal Editor will apply default values for any new fields.

!!! tip "Batch Re-save"
    Use **File > Save All** or the **Asset Actions > Resave Packages** command to batch-process dialogue tree assets after a struct change.

### Step 7 -- Test All Dialogue Paths

After upgrading, play through each NPC's dialogue to verify:

- Conditions still evaluate correctly
- Actions still fire as expected
- Node transitions follow the correct flow
- UI events still trigger properly

---

## Deprecation Policy

- Deprecated functions will be marked with `UE_DEPRECATED` and will continue to compile with warnings for at least one minor version.
- Removed functions will be listed in the migration guide with their replacements.
- Struct field additions are non-breaking (new fields get default values). Struct field removals or type changes are breaking and will be called out explicitly.
- Condition and action base class changes are always treated as breaking changes and will include migration instructions.
