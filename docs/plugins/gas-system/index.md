---
title: FW GAS System
---

# FW GAS System

**Version 1.0** | **Category:** Gameplay | **Loading Phase:** PreDefault | **Dependencies:** GameplayAbilities, ModularGameplay, GameFeatures, EnhancedInput

An integrated Gameplay Ability System framework providing a complete foundation for ability-driven gameplay. FWGASSystem wraps Unreal's GAS with opinionated components for ability granting, input binding, combo systems, ability queuing, modular gameplay actors, and GameFeature actions -- reducing boilerplate and accelerating development.

---

## Key Features

- **Streamlined ASC setup.** `UFWAbilitySystemComponent` auto-grants abilities, attributes, effects, and ability sets from editor-configurable arrays. No manual initialization code required.
- **EnhancedInput binding.** `UFWAbilityInputBindingComponent` bridges GAS ability activation with UE5's Enhanced Input system. Bind abilities to Input Actions with configurable trigger events.
- **Combo system.** `UFWComboManagerComponent` with `FWComboWindowNotifyState` and `FWTriggerComboNotify` anim notifies enable montage-driven combo chains with network replication.
- **Ability queuing.** `UFWAbilityQueueComponent` with `FWAbilityQueueNotifyState` allows players to queue the next ability during an active animation window.
- **Data-driven ability sets.** `UFWAbilitySet` DataAssets bundle abilities, attributes, effects, and owned tags for bulk granting and removal with handle-based lifecycle management.
- **Modular gameplay actors.** Pre-built base classes (`AFWModularCharacter`, `AFWModularPlayerState`, etc.) with `IGameFrameworkInitStateInterface` support for modular game feature integration.
- **GameFeature actions.** `UFWGameFeatureAction_AddAbilities`, `UFWGameFeatureAction_AddAnimLayers`, `UFWGameFeatureAction_AddInputMappingContext` for clean feature-driven content injection.
- **Core component abstraction.** `UFWGASCoreComponent` provides a unified event surface for attribute changes, ability lifecycle, effect tracking, cooldowns, and death -- abstracting away ASC owner/avatar complexity.
- **Built-in attribute set.** `UFWAttributeSet` ships with Health, Stamina, Mana (each with Max and RegenRate), plus Damage and StaminaDamage meta attributes.
- **Effect containers.** `FFWGameplayEffectContainer` and `FFWGameplayEffectContainerSpec` simplify targeting and effect application patterns.

---

## Architecture Overview

```
UFWAbilitySystemComponent (extends UAbilitySystemComponent)
 +-- GrantedAbilities (FFWAbilityInputMapping[])
 +-- GrantedAttributes (FFWAttributeSetDefinition[])
 +-- GrantedEffects (TSubclassOf<UGameplayEffect>[])
 +-- GrantedAbilitySets (TSoftObjectPtr<UFWAbilitySet>[])
 +-- Auto-grants on InitAbilityActorInfo

UFWGASCoreComponent (per-actor abstraction)
 +-- Attribute change events (Health, Stamina, Mana, Fervor, generic)
 +-- Ability lifecycle events (Activated, Ended, Failed, Committed)
 +-- Effect tracking events (Added, Removed, Stack/Time Change)
 +-- Tag change events
 +-- Cooldown events
 +-- Death system

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

## Module Structure

| Directory | Contents |
|-----------|----------|
| `Source/FWGASSystem/Public/Abilities/` | ASC, GameplayAbility, AbilitySet, Types, TargetType, Blueprint Library |
| `Source/FWGASSystem/Public/Abilities/Attributes/` | AttributeSetBase, AttributeSet |
| `Source/FWGASSystem/Public/Components/` | GASCoreComponent, AbilityQueue, ComboManager, InputBinding, LinkAnimLayers |
| `Source/FWGASSystem/Public/Animations/` | Anim notifies (AbilityQueue, ComboWindow, TriggerCombo), NativeAnimInstance |
| `Source/FWGASSystem/Public/GameFeatures/` | GameFeature actions and types |
| `Source/FWGASSystem/Public/ModularGameplayActors/` | Modular base classes |
| `Source/FWGASSystem/Public/UI/` | Debug widgets (AbilityQueue, Combo, HUD), UserWidget base |
| `Source/FWGASSystem/Public/Core/Settings/` | Developer settings |
| `Source/FWGASSystem/Public/Subsystems/` | Console manager subsystem |

---

## Quick Links

| Page | Description |
|------|-------------|
| [Installation](installation.md) | Adding the plugin and its dependencies |
| [Quick Start](quick-start.md) | Get abilities working in under 10 minutes |
| [API Reference](api-reference.md) | Complete C++ class and struct reference |
| [Blueprints](blueprints.md) | Blueprint-exposed functions and events |
| [Configuration](configuration.md) | Developer settings and DataAsset configuration |
| [Tutorials](tutorials.md) | Step-by-step guide: Creating a Combat Ability with Combos |
| [FAQ](faq.md) | Common questions and troubleshooting |
| [Migration Guide](migration.md) | Upgrading between versions |
| [Changelog](changelog.md) | Version history |
