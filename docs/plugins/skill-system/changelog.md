---
title: Changelog - FWSkillSystem
---

# Changelog

All notable changes to FWSkillSystem are documented on this page.

---

## v1.0 -- Initial Release

**Release Date:** 2026

### Features

- **15-skill progression system** organized into three categories: Combat (Warfare, Marksmanship, Magic, Bounty), Gathering (Woodcutting, Mining, Fishing, Husbandry), and Artisan (Cooking, Woodworking, Crafting, Smithing, Alchemy, Construction, Artifice).

- **UFWSkillProgressionComponent** -- Primary actor component for tracking XP and levels across all skills. Supports awarding skill-specific XP, distributing combat XP, loading saved data, querying levels and progress, and checking skill requirements.

- **Data-driven XP curves** via `UFWSkillXPCurve` data assets. Define any progression curve shape without code changes. Supports a global default curve with per-skill overrides.

- **Skill Database** (`UFWSkillDatabase`) -- Centralized data asset holding all `UFWSkillDefinition` entries. Configured in Project Settings for global access.

- **Skill Definitions** (`UFWSkillDefinition`) as `UPrimaryDataAsset` -- Per-skill configuration including display name, icon, category, max level, XP curve override, and milestones.

- **Milestone system** -- `UFWSkillMilestoneDefinition` and `UFWSkillMilestoneReward` for defining rewards at specific skill levels. `UFWSkillMilestoneComponent` tracks awarded milestones and triggers rewards on level-up.

- **Skill requirement system** -- `UFWSkillRequirement` base class for gating content behind skill levels. Evaluated via `MeetsRequirement` on the progression component. Designed for integration with FWInventorySystem, FWQuestSystem, and other systems.

- **Combat XP distribution** -- `AwardCombatXP` distributes XP across combat skills based on context, providing a single entry point for combat encounters.

- **Blueprint support** -- All progression functions, getters, and events exposed as `BlueprintCallable`, `BlueprintPure`, and `BlueprintAssignable`. Full Blueprint function library (`UFWSkillTypeLibrary`) for utility queries.

- **Events** -- `OnSkillXPGained` and `OnSkillLevelUp` multicast delegates for UI binding and gameplay reactions.

- **Project Settings** -- `UFWSkillSystemSettings` accessible via **Edit > Project Settings > Plugins > FW Skill System** for global configuration of skill database, default XP curve, and default max level.

- **Static accessor** -- `UFWSkillProgressionComponent::Get(Actor)` for convenient component lookup from any context.

- **Server authority** -- XP mutation functions perform authority checks on replicated actors, preventing unauthorized client-side modifications.

### Dependencies

- Requires `GameplayAbilities` plugin.

### Known Limitations

- `EFWSkillType` is a compile-time enum. Adding new skills requires a C++ change and recompilation.
- Combat XP distribution logic is internal to the progression component. Custom distribution requires subclassing.
- The plugin does not include built-in save/load persistence. Skill data serialization is delegated to the game's save system or backend API.
