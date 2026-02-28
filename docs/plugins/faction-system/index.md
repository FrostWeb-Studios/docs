---
title: FW Faction System
---

# FW Faction System

**Version 1.0** | **Category:** Gameplay | **Loading Phase:** Default | **Dependencies:** None

A dynamic faction and reputation system with AI integration, replication, and configurable persistence. FWFactionSystem provides a complete framework for managing faction identities, inter-faction relationships, personal reputation, and attitude resolution -- all with built-in network replication and optional save/load support.

---

## Key Features

- **Five-level attitude model.** Allied, Friendly, Neutral, Unfriendly, and Hostile -- mapped automatically to Unreal's three-state `ETeamAttitude` for seamless AI Perception integration.
- **Global reputation matrix.** Define default relationships between factions via DataAssets. The subsystem maintains a runtime reputation matrix that can be modified during gameplay.
- **Personal reputation overrides.** Per-actor reputation scores override the global matrix, enabling individual standing with factions (e.g., a player who has earned favor with an otherwise hostile faction).
- **Reputation propagation.** Configure rules so that reputation changes ripple to allied factions with configurable falloff multipliers.
- **Network replication.** Faction identity replicates to all clients. Personal reputation replicates to the owning client only. Server-authoritative RPCs enforce all mutations.
- **Pluggable persistence.** Abstract `UFWFactionPersistenceHandler` base class lets you implement save/load for any backend (REST API, SaveGame, cloud storage). Point to your subclass in Project Settings.
- **AI-ready.** Static `SolveTeamAttitude()` integrates directly with `IGenericTeamAgentInterface`, so AI controllers automatically respect faction relationships.
- **Debug tooling.** Console commands and an optional debug HUD (non-shipping builds only) for inspecting the reputation matrix at runtime.

---

## Architecture Overview

```
UFWFactionSubsystem (World Subsystem)
 +-- Faction Definitions (DataAssets via AssetManager)
 +-- Reputation Matrix (global scores)
 +-- Attitude Cache (derived from scores + thresholds)
 +-- Component Registry (tracks all active faction components)
 +-- Persistence Handler (optional, from project settings)

UFWFactionComponent (per-actor)
 +-- CurrentFactionId (replicated to all)
 +-- PersonalReputations (replicated to owner)
 +-- Events: OnFactionChanged, OnReputationChanged, OnAttitudeChanged

UFWFactionDefinition (DataAsset)
 +-- Identity (FactionId, DisplayName, TeamId, Color, Icon)
 +-- Classification (Tags, bPlayerJoinable, bHidden)
 +-- Reputation Thresholds
 +-- Default Relationships
 +-- Propagation Rules
```

---

## Module Structure

| Directory | Contents |
|-----------|----------|
| `Source/FWFactionSystem/Public/Components/` | `UFWFactionComponent` |
| `Source/FWFactionSystem/Public/Subsystems/` | `UFWFactionSubsystem` |
| `Source/FWFactionSystem/Public/DataAssets/` | `UFWFactionDefinition` |
| `Source/FWFactionSystem/Public/` | Types, Settings, Persistence, Blueprint Library, Module |
| `Source/FWFactionSystem/Public/Debug/` | Console commands, Debug HUD |

---

## Quick Links

| Page | Description |
|------|-------------|
| [Installation](installation.md) | Adding the plugin to your project |
| [Quick Start](quick-start.md) | Get factions working in under 10 minutes |
| [API Reference](api-reference.md) | Complete C++ class and struct reference |
| [Blueprints](blueprints.md) | Blueprint-exposed functions and events |
| [Configuration](configuration.md) | Project settings and DataAsset configuration |
| [Tutorials](tutorials.md) | Step-by-step guide: Setting Up a Faction War |
| [FAQ](faq.md) | Common questions and troubleshooting |
| [Migration Guide](migration.md) | Upgrading between versions |
| [Changelog](changelog.md) | Version history |
