---
title: Plugins Overview
---

# Plugins Overview

FrostWeb provides 11 Unreal Engine 5 plugins, each targeting a specific game system. All plugins are designed to work independently or together as part of a larger game framework.

---

## Plugin Catalog

| Plugin | Description | Dependencies |
|---|---|---|
| [FWAISystem](ai-system/) | MMO-style NPC behavior: threat tables, leashing, patrol, spawning, instance scaling | FWFactionSystem; optionally FWGASSystem |
| [FWChatSystem](chat-system/) | Socket.IO chat with channels, party/guild chat, whisper, and presence | SocketIOClient |
| [FWCustomizationSystem](customization-system/) | Character customization: race/gender, skin tones, hair, tattoos, DMI materials | None |
| [FWDialogueSystem](dialogue-system/) | Data-driven dialogue trees with conditions, actions, and NPC conversations | None |
| [FWFactionSystem](faction-system/) | Dynamic faction reputation with AI integration and configurable persistence | None |
| [FWGASSystem](gas-system/) | Integrated Gameplay Ability System framework with combos, queuing, and modular gameplay | GameplayAbilities, ModularGameplay, GameFeatures, EnhancedInput |
| [FWGuildSystem](guild-system/) | Full guild management: ranks, permissions, invitations, HTTP API integration | Optionally FWChatSystem |
| [FWInventorySystem](inventory-system/) | Inventory, equipment, crafting, vendors with RNG affixes and augmentation | GameplayAbilities; optionally FWSkillSystem |
| [FWPartySystem](party-system/) | Beacon-based party system with invitations, join codes, and level transfer persistence | OnlineSubsystem, SocketIOClient |
| [FWQuestSystem](quest-system/) | Modular quest system: tasks, conditions, rewards, party support, daily/weekly resets | GameplayAbilities |
| [FWSkillSystem](skill-system/) | Skill progression system: 15 skills, XP curves, milestones, skill-gated content | GameplayAbilities |

---

## Modular Architecture

Every FrostWeb plugin follows the same architectural principles:

- **Self-contained modules.** Each plugin ships as a standalone Unreal plugin with its own `.uplugin` descriptor, source modules, and content. You install only what you need.

- **No forced coupling.** Plugins that reference other plugins do so through optional interfaces or soft dependencies. Disabling a plugin that another plugin optionally integrates with will not cause compile failures -- the integration code is gated by module availability checks.

- **Consistent API surface.** All plugins expose their functionality through `UActorComponent` subclasses, `UPrimaryDataAsset` data definitions, and `BlueprintAssignable` delegates. If you know how to use one FrostWeb plugin, you know the patterns for all of them.

- **Shared replication strategy.** Network replication across all plugins uses `FFastArraySerializer` for efficient array property sync, server-authoritative RPCs for mutations, and client RPCs for cosmetic feedback. No plugin relies on Unreal's automatic property replication for gameplay-critical state.

For a deeper look at these patterns, see the [Architecture Overview](../guides/architecture.md).
