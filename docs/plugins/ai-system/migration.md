---
title: Migration Guide - FWAISystem
---

# Migration Guide

---

## v1.0 (Current)

No migrations required for v1.0. This is the initial release of FWAISystem.

---

## General Upgrade Instructions

When upgrading FWAISystem to a future version, follow these steps:

### Before Upgrading

1. **Back up your project.** Create a full backup or ensure your changes are committed to version control before upgrading.
2. **Review the [Changelog](changelog.md)** for the target version. Pay attention to any breaking changes, deprecated APIs, or renamed classes.
3. **Check plugin dependencies.** If a new version requires a newer version of FWFactionSystem or FWGASSystem, upgrade those first.

### Upgrade Process

1. **Replace the plugin folder.** Delete the existing `Plugins/FWAISystem/` directory and copy the new version in its place.
2. **Regenerate project files.** Right-click your `.uproject` file and select "Generate Visual Studio project files" (or run `GenerateProjectFiles.bat`).
3. **Compile the project.** Build from your IDE or the Unreal Editor. Fix any compilation errors by consulting the changelog for API changes.
4. **Test in PIE.** Run your game in Play-in-Editor and verify that NPC spawning, threat, leashing, and scaling behave correctly.

### Common Migration Patterns

#### Renamed Classes or Methods

If a class or method is renamed in a future version, perform a project-wide find-and-replace:

1. Search for the old name across all `.h` and `.cpp` files.
2. Replace with the new name.
3. Update any Blueprint references by opening affected Blueprints and reconnecting broken nodes.

#### Deprecated APIs

Deprecated APIs will continue to work for at least one minor version after deprecation. They will emit compiler warnings with guidance on the replacement API. Address these warnings before the next major version.

#### Data Asset Changes

If `UFWNpcDefinition` gains new properties in a future version:

- Existing data assets will load with default values for new properties.
- No data migration is needed unless default values are inappropriate for your NPCs.
- Review new properties in the [Configuration](configuration.md) guide and update your data assets as needed.

#### Behavior Tree Node Changes

If behavior tree nodes gain new parameters:

- Existing behavior trees will load with default values for new parameters.
- Open each behavior tree in the editor to review and configure new parameters.

---

## Version Compatibility Matrix

| FWAISystem | Unreal Engine | FWFactionSystem | FWGASSystem |
|:----------:|:-------------:|:---------------:|:-----------:|
| 1.0 | 5.4+ | 1.0+ (optional) | 1.0+ (optional) |
