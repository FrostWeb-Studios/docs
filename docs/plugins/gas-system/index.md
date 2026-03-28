---
title: FW GAS System
---

# FW GAS System

**Category:** Gameplay | **Loading Phase:** PreDefault | **Dependencies:** GameplayAbilities, ModularGameplay, GameFeatures, EnhancedInput

An integrated Gameplay Ability System framework providing a complete foundation for ability-driven gameplay. FWGASSystem wraps Unreal's GAS with opinionated components for ability granting, input binding, combo systems, ability queuing, modular gameplay actors, and GameFeature actions -- reducing boilerplate and accelerating development.

---

## Key Features

- **Streamlined ASC setup.** `UFWAbilitySystemComponent` auto-grants abilities, attributes, effects, and ability sets from editor-configurable arrays. No manual initialization code required.
- **EnhancedInput binding.** `UFWAbilityInputBindingComponent` bridges GAS ability activation with UE5's Enhanced Input system. Bind abilities to Input Actions with configurable trigger events.
- **Combo system.** `UFWComboManagerComponent` with combo window and trigger anim notifies enables montage-driven combo chains with network replication.
- **Ability queuing.** `UFWAbilityQueueComponent` allows players to queue the next ability during an active animation window.
- **Data-driven ability sets.** `UFWAbilitySet` DataAssets bundle abilities, attributes, effects, and owned tags for bulk granting and removal with handle-based lifecycle management.
- **Modular gameplay actors.** 12 pre-built base classes with `IGameFrameworkInitStateInterface` support for modular game feature integration.
- **GameFeature actions.** Actions for adding abilities, animation layers, and input mapping contexts through the Game Features framework.
- **Core component abstraction.** `UFWGASCoreComponent` provides a unified event surface with 20+ events for attribute changes, ability lifecycle, effect tracking, cooldowns, and death -- abstracting away ASC owner/avatar complexity.
- **Built-in attribute set.** `UFWAttributeSet` ships with Health, Stamina, Mana (each with Max and RegenRate), plus Damage and StaminaDamage meta attributes.
- **Effect containers.** Simplified targeting and effect application patterns via data-driven effect containers.
- **Developer settings.** `UFWGASDeveloperSettings` for project-wide GAS configuration.

---

## Architecture Overview

```
UFWAbilitySystemComponent (extends UAbilitySystemComponent)
 +-- DefaultAbilities, DefaultAttributes, StartupEffects
 +-- GrantedAbilitySets
 +-- ReplicationMode configuration
 +-- Auto-grants on InitAbilityActorInfo

UFWGASCoreComponent (per-actor abstraction)
 +-- Attribute queries (GetHealth, GetMaxHealth, GetStamina, etc.)
 +-- Ability management (Grant, Clear, Activate)
 +-- Attribute modification (Set, Clamp, Adjust)
 +-- 20+ events (Attribute, Ability, Effect, Tag, Cooldown, Death)

UFWComboManagerComponent
 +-- ComboIndex (replicated)
 +-- Combo window state (replicated)
 +-- Server/Multicast RPCs for combo activation

UFWAbilityQueueComponent
 +-- Queue state management
 +-- Allowed abilities filtering

UFWAbilityInputBindingComponent
 +-- InputAction -> AbilitySpecHandle mapping
 +-- EnhancedInput integration

UFWAbilitySet (DataAsset)
 +-- Abilities + Input bindings
 +-- Attribute Sets + Init data
 +-- Effects
 +-- Owned Tags

GameFeature Actions
 +-- AddAbilities, AddAnimLayers, AddInputMappingContext
```

---

## Quick Links

| Page | Description |
|------|-------------|
| [Installation](installation.md) | Adding the plugin and its dependencies |
| [Quick Start](quick-start.md) | Get abilities working in under 10 minutes |
| [API Reference](api-reference.md) | Class, function, and property reference |
| [Blueprints](blueprints.md) | Blueprint-exposed functions and events |
| [Configuration](configuration.md) | Developer settings and DataAsset configuration |
| [Tutorials](tutorials.md) | Step-by-step guide: Creating a Combat Ability with Combos |
| [FAQ](faq.md) | Common questions and troubleshooting |
