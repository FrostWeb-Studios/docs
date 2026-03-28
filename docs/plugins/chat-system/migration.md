---
title: Migration Guide - FWChatSystem
---

# Migration Guide

---

## v1.0 (Current)

No migrations required for v1.0. This is the initial release of FWChatSystem.

---

## General Upgrade Instructions

When upgrading FWChatSystem to a future version, follow these steps:

### Before Upgrading

1. **Back up your project.** Create a full backup or ensure your changes are committed to version control before upgrading.
2. **Review the [Changelog](changelog.md)** for the target version. Pay attention to any breaking changes, deprecated APIs, or renamed classes.
3. **Check plugin dependencies.** Review the changelog for any changes to engine module requirements.

### Upgrade Process

1. **Replace the plugin folder.** Delete the existing `Plugins/FWChatSystem/` directory and copy the new version in its place.
2. **Regenerate project files.** Right-click your `.uproject` file and select "Generate Visual Studio project files" (or run `GenerateProjectFiles.bat`).
3. **Compile the project.** Build from your IDE or the Unreal Editor. Fix any compilation errors by consulting the changelog for API changes.
4. **Test chat functionality.** Verify connection, message sending/receiving, slash commands, party sync, and guild integration.

### Common Migration Patterns

#### Renamed Classes or Methods

If a class or method is renamed in a future version, perform a project-wide find-and-replace:

1. Search for the old name across all `.h` and `.cpp` files.
2. Replace with the new name.
3. Update any Blueprint references by opening affected Blueprints and reconnecting broken nodes.

#### Deprecated APIs

Deprecated APIs will continue to work for at least one minor version after deprecation. They will emit compiler warnings with guidance on the replacement API. Address these warnings before the next major version.

#### New Event Types

If new delegate types are added in a future version:

- Existing bindings will continue to work unchanged.
- New events are opt-in -- you only need to bind to them if your game needs the new functionality.
- Review the changelog for any events that replace or supersede existing ones.

#### FFWChatConfig Changes

If new configuration fields are added to `FFWChatConfig`:

- Existing data will load with default values for new fields.
- Review new fields in the [Configuration](configuration.md) guide and adjust as needed.

#### Socket.IO Protocol Changes

If the chat server protocol changes:

- Update your chat server to the new protocol version.
- The transport component handles protocol details -- client code changes are typically not needed.
- Review the changelog for any new required server-to-client or client-to-server events.

---

## Version Compatibility Matrix

| FWChatSystem | Unreal Engine |
|:------------:|:-------------:|
| 1.0 | 5.4+ |
