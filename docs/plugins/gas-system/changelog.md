---
title: Changelog - GAS System
---

# Changelog

All notable changes to FWGASSystem are documented here. This project follows [Semantic Versioning](https://semver.org/).

---

## [1.0.0] - 2025-01-01

### Added

- **UFWAbilitySystemComponent** -- Extended ASC with auto-granting of abilities, attributes, effects, and ability sets.
    - `GrantedAbilities` (`FFWAbilityInputMapping` array) with EnhancedInput binding.
    - `GrantedAttributes` (`FFWAttributeSetDefinition` array) with DataTable initialization.
    - `GrantedEffects` (startup gameplay effects).
    - `GrantedAbilitySets` (soft-referenced AbilitySet DataAssets).
    - `bResetAbilitiesOnSpawn`, `bResetAttributesOnSpawn` for controlling re-granting behavior.
    - `GiveAbilitySet`, `ClearAbilitySet` for runtime set management.
    - `OnInitAbilityActorInfo` event.
    - Combo-aware `AbilityLocalInputPressed` override.

- **UFWGameplayAbility** -- Base ability class with effect container support.
    - `EffectContainerMap` for data-driven effect application.
    - `MakeEffectContainerSpec`, `MakeEffectContainerSpecFromContainer`, `ApplyEffectContainerSpec`, `ApplyEffectContainer`.
    - `bEnableAbilityQueue` for ability queue integration.

- **UFWAbilitySet** -- DataAsset bundling abilities, attributes, effects, and owned tags.
    - `GrantToAbilitySystem` (ASC and Actor overloads).
    - `RemoveFromAbilitySystem` (static, handle-based).
    - `FFWAbilitySetHandle` for lifecycle management.

- **UFWAttributeSetBase** -- Abstract base attribute set with shared infrastructure.
    - `AdjustAttributeForMaxChange`, `GetClampMinimumValueFor`, `GetExecutionDataFromMod`.
    - `ATTRIBUTE_ACCESSORS` macro.

- **UFWAttributeSet** -- Concrete attribute set with Health, MaxHealth, HealthRegenRate, Stamina, MaxStamina, StaminaRegenRate, Mana, MaxMana, ManaRegenRate, Damage, StaminaDamage.
    - All gameplay attributes replicated with `OnRep_` handlers.
    - Meta attributes (Damage, StaminaDamage) server-only.

- **UFWGASCoreComponent** -- Unified event surface for ability system activity.
    - Attribute events: OnHealthChange, OnStaminaChange, OnManaChange, OnFervorChange, OnAttributeChange, OnDamage, OnDeath.
    - Ability events: OnAbilityActivated, OnAbilityEnded, OnAbilityFailed, OnAbilityCommit.
    - Effect events: OnGameplayEffectAdded, OnGameplayEffectRemoved, OnGameplayEffectStackChange, OnGameplayEffectTimeChange.
    - Cooldown events: OnCooldownStart, OnCooldownEnd.
    - Tag events: OnGameplayTagChange.
    - Advanced events: OnPreAttributeChange, OnPostGameplayEffectExecute, OnInitAbilityActorInfo.
    - Attribute accessors: GetHealth, GetMaxHealth, GetStamina, GetMaxStamina, GetMana, GetMaxMana, IsAlive.
    - Ability management: GrantAbility, ClearAbility, ClearAbilities, ActivateAbilityByClass, ActivateAbilityByTags.
    - Die() function triggering death pipeline.

- **UFWComboManagerComponent** -- Combo chain management with network replication.
    - Replicated properties: ComboIndex, bComboWindowOpened, bShouldTriggerCombo, bRequestTriggerCombo, bNextComboAbilityActivated.
    - IncrementCombo, ResetCombo, ActivateComboAbility.
    - Server/Multicast RPCs for combo activation and index synchronization.

- **UFWAbilityQueueComponent** -- Ability queuing during animation windows.
    - OpenAbilityQueue, CloseAbilityQueue.
    - UpdateAllowedAbilitiesForAbilityQueue, SetAllowAllAbilitiesForAbilityQueue.

- **UFWAbilityInputBindingComponent** -- Enhanced Input to GAS binding.
    - SetInputBinding, ClearInputBinding.
    - Automatic binding setup during ability granting.

- **UFWLinkAnimLayersComponent** -- Animation layer linking.

- **Animation Notifies:**
    - `FWAbilityQueueNotifyState` -- opens/closes ability queue window.
    - `FWComboWindowNotifyState` -- opens/closes combo input window.
    - `FWTriggerComboNotify` -- triggers next combo step.
    - `UFWNativeAnimInstance` -- base anim instance with GAS integration.

- **Modular Gameplay Actors:**
    - `AFWModularCharacter`, `AFWModularPlayerState`, `AFWModularPlayerStateCharacter`, `AFWModularPlayerController`, `AFWModularActor`, `AFWModularDefaultPawn`, `AFWModularGameMode`, `AFWModularGameState`, `AFWModularPawn`, `AFWModularAIController`.

- **GameFeature Actions:**
    - `UFWGameFeatureAction_AddAbilities` -- grants abilities, attributes, effects, and ability sets to actors.
    - `UFWGameFeatureAction_AddAnimLayers` -- links anim layer interfaces.
    - `UFWGameFeatureAction_AddInputMappingContext` -- adds input mapping contexts.

- **Types:**
    - `EFWAbilityTriggerEvent` (Started, Triggered).
    - `FFWAbilityInputMapping`, `FFWAttributeSetDefinition`, `FFWMappedAbility`.
    - `FFWGameplayEffectContainer`, `FFWGameplayEffectContainerSpec`.
    - `FFWGameplayEffectExecuteData`, `FFWAttributeSetExecutionData`.
    - `FFWGameFeatureAbilityMapping`, `FFWGameFeatureAttributeSetMapping`, `FFWGameFeatureGameplayEffectMapping`.
    - `FFWGameFeatureAbilitiesEntry`.

- **Utilities:**
    - `UFWBlueprintFunctionLibrary` -- static Blueprint helpers.
    - `UFWTargetType` -- base class for effect container targeting.

- **UI Widgets:**
    - `UFWUserWidget`, `UFWUWHud`, `UFWUWDebugAbilityQueue`, `UFWUWDebugComboWidget`.

- **Subsystems:**
    - `UFWConsoleManagerSubsystem` -- console command management.

- **Settings:**
    - `UFWGASDeveloperSettings` -- project-wide GAS configuration.
