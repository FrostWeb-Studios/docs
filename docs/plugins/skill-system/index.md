---
title: FWSkillSystem Plugin
description: Skill progression system with XP curves, leveling, milestones, combat XP integration, and skill-gated content for Unreal Engine 5.
tags:
  - plugin
  - skill
  - progression
  - RPG
---

# FWSkillSystem Plugin

A skill progression plugin for Unreal Engine 5 that provides 15 distinct skills across three categories, configurable XP curves, milestone rewards, combat XP distribution, and a skill requirement system for gating content behind skill levels.

---

## The 15 Skills

FWSkillSystem ships with 15 skills organized into three categories:

### Combat

Skills that advance through direct combat encounters.

| Skill | Description |
|---|---|
| Warfare | Melee combat proficiency |
| Marksmanship | Ranged combat proficiency |
| Magic | Magical combat proficiency |
| Bounty | Hunting and bounty contract proficiency |

### Gathering

Skills that advance through resource collection activities.

| Skill | Description |
|---|---|
| Woodcutting | Harvesting timber from trees |
| Mining | Extracting ore and minerals |
| Fishing | Catching fish and aquatic resources |
| Husbandry | Taming, breeding, and managing animals |

### Artisan

Skills that advance through crafting and construction activities.

| Skill | Description |
|---|---|
| Cooking | Preparing food and consumables |
| Woodworking | Crafting wooden items and furniture |
| Crafting | General-purpose item creation |
| Smithing | Forging metal weapons and armor |
| Alchemy | Brewing potions and transmuting materials |
| Construction | Building structures and fortifications |
| Artifice | Creating enchanted and magical items |

---

## Key Features

| Feature | Description |
|---|---|
| XP Curve System | Data-driven XP requirements per level with RS-style formula, per-skill overrides, and configurable grind/compression factors |
| Skill Leveling | Track XP, compute levels, and broadcast events on level-up |
| Combat XP Integration | Automatically distribute XP to combat skills based on damage dealt, with configurable XPPerDamage and KillBonusMultiplier |
| Milestone Rewards | Define rewards triggered at specific skill levels via milestone definitions and the milestone component |
| Skill-Gated Content | Gate abilities, items, quests, and areas behind skill requirements |
| Skill Database | Centralized data asset containing all skill definitions |
| GAS Integration | Works with the Gameplay Ability System for skill-gated ability activation and attribute integration |
| Blueprint Support | All progression, querying, and gating functions exposed to Blueprints |
| Project Settings | Global configuration via Project Settings including MaxSkillLevel, GrindFactor, CompressionFactor, and PreviousMaxLevel |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| `GameplayAbilities` | **Required** | Used for skill-gated ability activation and attribute integration |

!!! warning "Required Dependency"
    FWSkillSystem requires the Gameplay Ability System module. Your project must have `GameplayAbilities` enabled before activating this plugin.

---

## Architecture

```
UFWSkillProgressionComponent (per-actor)
 +-- XP API (AwardSkillXP, AwardCombatXP)
 +-- Query API (GetSkillLevel, GetSkillTotalXP, GetSkillLevelProgress, etc.)
 +-- Requirements (MeetsRequirement)
 +-- GAS Integration
 +-- Events (OnSkillXPGained, OnSkillLevelUp)

UFWSkillMilestoneComponent (per-actor)
 +-- Milestone tracking and reward distribution
 +-- Binds to OnSkillLevelUp automatically

Data Assets
 +-- UFWSkillDefinition (per-skill configuration)
 +-- UFWSkillDatabase (collection of all skill definitions)
 +-- UFWSkillXPCurve (XP-per-level curve with RS-style formula)
 +-- UFWSkillMilestoneDefinition (milestone reward definitions)

Configuration
 +-- UFWSkillSystemSettings (Project Settings)
 +-- MaxSkillLevel, GrindFactor, CompressionFactor, PreviousMaxLevel
 +-- XPPerDamage, KillBonusMultiplier (combat XP tuning)
```

---

## Quick Navigation

| Page | Description |
|---|---|
| [Installation](installation.md) | Enable the plugin and configure dependencies |
| [Quick Start](quick-start.md) | Get skill progression running in 10 minutes |
| [API Reference](api-reference.md) | Classes, methods, properties, structs, and enums |
| [Blueprints](blueprints.md) | Blueprint node reference for all exposed functions and events |
| [Configuration](configuration.md) | Set up SkillDatabase, XP curves, milestones, and project settings |
| [Tutorials](tutorials.md) | Building a Skill Progression UI |
| [FAQ](faq.md) | Common questions about XP formulas, max levels, and combat XP |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |

!!! warning "Dedicated Server"
    Skill XP mutations should only execute on the server or in a server-authoritative context. The `UFWSkillProgressionComponent` performs authority checks internally when used on replicated actors. Client-side calls to `AwardSkillXP` and `AwardCombatXP` are ignored unless the owning actor has authority.
