---
title: FW Faction System
description: Dynamic faction and reputation system with AI integration, replication, and configurable persistence for Unreal Engine 5.
tags:
  - plugin
  - faction
  - reputation
  - AI
---

# FW Faction System

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
- **Blueprint Library.** Common faction queries exposed as static Blueprint functions via `UFWFactionBlueprintLibrary`.
- **Debug tooling.** Console commands and an optional debug HUD (non-shipping builds only) for inspecting the reputation matrix at runtime.

---

## Architecture

```
UFWFactionSubsystem (World Subsystem)
 +-- Definition Access (GetFactionDefinition, GetAllFactions, GetFactionByTeamId)
 +-- Reputation Matrix (global scores, attitude resolution)
 +-- Attitude Cache (derived from scores + thresholds)
 +-- Component Registry (tracks all active faction components)
 +-- Persistence Handler (optional, from project settings)
 +-- Events (OnFactionWarDeclared, OnFactionAllianceFormed, OnGlobalAttitudeChanged)

UFWFactionComponent (per-actor)
 +-- Faction Identity (CurrentFactionId, replicated to all)
 +-- Personal Reputation (replicated to owning client only)
 +-- Server RPCs (Server_SetFaction, Server_ModifyPersonalReputation)
 +-- Events (OnFactionChanged, OnReputationChanged, OnAttitudeChanged)

UFWFactionDefinition (DataAsset)
 +-- Identity (FactionId, DisplayName, TeamId, Color, Icon)
 +-- Classification (Tags, bPlayerJoinable, bHidden)
 +-- Reputation Thresholds
 +-- Default Relationships
 +-- Propagation Rules

Persistence
 +-- UFWFactionPersistenceHandler (abstract base, implement for your backend)

Debug Tools
 +-- Console commands for reputation matrix inspection
 +-- Debug HUD (non-shipping builds only)
```

---

## Quick Navigation

| Page | Description |
|------|-------------|
| [Installation](installation.md) | Adding the plugin to your project |
| [Quick Start](quick-start.md) | Get factions working in under 10 minutes |
| [API Reference](api-reference.md) | Classes, methods, properties, structs, and enums |
| [Blueprints](blueprints.md) | Blueprint-exposed functions and events |
| [Configuration](configuration.md) | Project settings and DataAsset configuration |
| [Tutorials](tutorials.md) | Step-by-step guide: Setting Up a Faction War |
| [FAQ](faq.md) | Common questions and troubleshooting |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |

!!! warning "Dedicated Server"
    Faction mutations are server-authoritative. All reputation and faction changes must originate from the server. Client-side RPCs are validated before execution.
