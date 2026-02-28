---
title: API Reference - GAS System
---

# API Reference

Complete C++ reference for all public classes, structs, enums, and delegates in FWGASSystem.

---

## Core Ability Classes

### UFWAbilitySystemComponent

`#include "Abilities/FWAbilitySystemComponent.h"`

Extended Ability System Component designed for Blueprint attachment or GameFeature-based granting. Auto-grants abilities, attributes, effects, and ability sets on `InitAbilityActorInfo`.

**Extends:** `UAbilitySystemComponent` | **Specifiers:** `BlueprintSpawnableComponent`

#### Configurable Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `GrantedAbilities` | `TArray<FFWAbilityInputMapping>` | Empty | Abilities to grant on init, with optional input binding. |
| `GrantedAttributes` | `TArray<FFWAttributeSetDefinition>` | Empty | Attribute sets to grant, with optional DataTable init. |
| `GrantedEffects` | `TArray<TSubclassOf<UGameplayEffect>>` | Empty | Effects to apply on init (e.g., regen effects). |
| `GrantedAbilitySets` | `TArray<TSoftObjectPtr<UFWAbilitySet>>` | Empty | AbilitySet DataAssets to grant on init. |
| `bResetAbilitiesOnSpawn` | `bool` | `true` | Re-grant abilities on each `InitAbilityActorInfo` call. Set to `false` for Player State ASCs. |
| `bResetAttributesOnSpawn` | `bool` | `true` | Re-initialize attributes on each `InitAbilityActorInfo` call. |

#### Events

| Event | Signature | Description |
|-------|-----------|-------------|
| `OnInitAbilityActorInfo` | `FFWOnInitAbilityActorInfo` (no params) | Fired after `InitAbilityActorInfo`, once abilities and attributes have been granted. |

#### Key Methods

```cpp
// Grant an AbilitySet at runtime, returns a handle for later removal
bool GiveAbilitySet(const UFWAbilitySet* InAbilitySet, FFWAbilitySetHandle& OutHandle);

// Remove a previously granted AbilitySet
bool ClearAbilitySet(UPARAM(ref) FFWAbilitySetHandle& InAbilitySetHandle);

// Called automatically on InitAbilityActorInfo
virtual void GrantDefaultAbilitiesAndAttributes(AActor* InOwnerActor, AActor* InAvatarActor);
virtual void GrantDefaultAbilitySets(AActor* InOwnerActor, AActor* InAvatarActor);
```

!!! info "bResetAbilitiesOnSpawn"
    When set to `false` (recommended for Player State ASCs), abilities are granted only on the first `InitAbilityActorInfo` call. Subsequent calls (e.g., after respawn) skip re-granting. Do **not** set to `true` for Player State ASCs if using input binding -- it will cause duplicate bindings.

---

### UFWGameplayAbility

`#include "Abilities/FWGameplayAbility.h"`

Base gameplay ability class extending `UGameplayAbility` with effect container support and ability queue integration.

**Extends:** `UGameplayAbility`

#### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `bEnableAbilityQueue` | `bool` | `false` | Allows other abilities to be queued during this ability. Prefer using `AbilityQueueNotifyState` in montages for finer control. |
| `EffectContainerMap` | `TMap<FGameplayTag, FFWGameplayEffectContainer>` | Empty | Maps gameplay tags to effect containers for data-driven effect application. |

#### Key Methods

```cpp
// Create an effect container spec from a container definition
virtual FFWGameplayEffectContainerSpec MakeEffectContainerSpecFromContainer(
    const FFWGameplayEffectContainer& Container,
    const FGameplayEventData& EventData,
    int32 OverrideGameplayLevel = -1);

// Look up a container by tag and create its spec
virtual FFWGameplayEffectContainerSpec MakeEffectContainerSpec(
    FGameplayTag ContainerTag,
    const FGameplayEventData& EventData,
    int32 OverrideGameplayLevel = -1);

// Apply a previously created container spec
virtual TArray<FActiveGameplayEffectHandle> ApplyEffectContainerSpec(
    const FFWGameplayEffectContainerSpec& ContainerSpec);

// Shorthand: create and apply in one call
virtual TArray<FActiveGameplayEffectHandle> ApplyEffectContainer(
    FGameplayTag ContainerTag,
    const FGameplayEventData& EventData,
    int32 OverrideGameplayLevel = -1);
```

---

### UFWAbilitySet

`#include "Abilities/FWAbilitySet.h"`

DataAsset bundling abilities, attribute sets, effects, and owned tags for bulk granting to an ASC.

**Extends:** `UPrimaryDataAsset` | **Specifiers:** `BlueprintType`

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `GrantedAbilities` | `TArray<FFWGameFeatureAbilityMapping>` | Abilities with input binding configuration. |
| `GrantedAttributes` | `TArray<FFWGameFeatureAttributeSetMapping>` | Attribute sets with optional DataTable init. |
| `GrantedEffects` | `TArray<FFWGameFeatureGameplayEffectMapping>` | Effects to apply with level. |
| `OwnedTags` | `FGameplayTagContainer` | Tags to apply to the ASC when the set is granted. |

#### Key Methods

```cpp
// Grant to ASC, returns handle for later removal
bool GrantToAbilitySystem(UAbilitySystemComponent* InASC,
    FFWAbilitySetHandle& OutAbilitySetHandle,
    FText* OutErrorText = nullptr,
    const bool bShouldRegisterCoreDelegates = true) const;

// Grant to an actor (finds ASC automatically)
bool GrantToAbilitySystem(const AActor* InActor,
    FFWAbilitySetHandle& OutAbilitySetHandle,
    FText* OutErrorText = nullptr) const;

// Static: remove a previously granted set
static bool RemoveFromAbilitySystem(UAbilitySystemComponent* InASC,
    FFWAbilitySetHandle& InAbilitySetHandle,
    FText* OutErrorText = nullptr,
    const bool bShouldRegisterCoreDelegates = true);

// Check if any ability in this set has input binding
bool HasInputBinding() const;
```

---

### FFWAbilitySetHandle

Handle struct for tracking what was granted by an `UFWAbilitySet`.

| Field | Type | Description |
|-------|------|-------------|
| `Abilities` | `TArray<FGameplayAbilitySpecHandle>` | Granted ability handles. |
| `EffectHandles` | `TArray<FActiveGameplayEffectHandle>` | Active effect handles. |
| `Attributes` | `TArray<TObjectPtr<UAttributeSet>>` | Granted attribute set pointers. |
| `OwnedTags` | `FGameplayTagContainer` | Tags that were added. |
| `AbilitySetPathName` | `FString` | Source AbilitySet path (for debug). |

---

## Attribute Sets

### UFWAttributeSetBase

`#include "Abilities/Attributes/FWAttributeSetBase.h"`

Abstract base attribute set providing shared infrastructure: `PreAttributeChange`, `PostGameplayEffectExecute`, `AdjustAttributeForMaxChange`, and clamp value support.

**Extends:** `UAttributeSet` | **Specifiers:** `Abstract`

Provides the `ATTRIBUTE_ACCESSORS` macro for defining getter/setter/init helpers.

---

### UFWAttributeSet

`#include "Abilities/Attributes/FWAttributeSet.h"`

Concrete attribute set with common combat attributes. All gameplay attributes are replicated.

**Extends:** `UFWAttributeSetBase`

| Attribute | Type | Replicated | Description |
|-----------|------|------------|-------------|
| `Health` | `FGameplayAttributeData` | Yes | Current health. |
| `MaxHealth` | `FGameplayAttributeData` | Yes | Maximum health. |
| `HealthRegenRate` | `FGameplayAttributeData` | Yes | Health regen per period. |
| `Stamina` | `FGameplayAttributeData` | Yes | Current stamina. |
| `MaxStamina` | `FGameplayAttributeData` | Yes | Maximum stamina. |
| `StaminaRegenRate` | `FGameplayAttributeData` | Yes | Stamina regen per period. |
| `Mana` | `FGameplayAttributeData` | Yes | Current mana. |
| `MaxMana` | `FGameplayAttributeData` | Yes | Maximum mana. |
| `ManaRegenRate` | `FGameplayAttributeData` | Yes | Mana regen per period. |
| `Damage` | `FGameplayAttributeData` | No | Meta attribute for damage calculation. Server-only. |
| `StaminaDamage` | `FGameplayAttributeData` | No | Meta attribute for stamina damage. Server-only. |

---

## Components

### UFWGASCoreComponent

`#include "Components/FWGASCoreComponent.h"`

Abstraction layer over the ASC providing a unified Blueprint event surface. Handles ASC owner/avatar complexity, especially for Player State-based ASC setups.

**Extends:** `UActorComponent` | **Specifiers:** `BlueprintSpawnableComponent`

#### Attribute Events

| Event | Delegate | Parameters |
|-------|----------|------------|
| `OnHealthChange` | `FFWOnDefaultAttributeChange` | `DeltaValue`, `EventTags` |
| `OnStaminaChange` | `FFWOnDefaultAttributeChange` | `DeltaValue`, `EventTags` |
| `OnManaChange` | `FFWOnDefaultAttributeChange` | `DeltaValue`, `EventTags` |
| `OnFervorChange` | `FFWOnDefaultAttributeChange` | `DeltaValue`, `EventTags` |
| `OnAttributeChange` | `FFWOnAttributeChange` | `Attribute`, `DeltaValue`, `EventTags` |
| `OnDamage` | `FFWOnDamage` | `DamageAmount`, `SourceCharacter`, `DamageTags` |
| `OnDeath` | `FFWOnDeath` | (none) |

#### Ability Lifecycle Events

| Event | Delegate | Parameters |
|-------|----------|------------|
| `OnAbilityActivated` | `FFWOnAbilityActivated` | `Ability` |
| `OnAbilityEnded` | `FFWOnAbilityEnded` | `Ability` |
| `OnAbilityFailed` | `FFWOnAbilityFailed` | `Ability`, `ReasonTags` |
| `OnAbilityCommit` | `FFWOnAbilityCommit` | `Ability` |

#### Effect Events

| Event | Delegate | Parameters |
|-------|----------|------------|
| `OnGameplayEffectAdded` | `FFWOnGameplayEffectAdded` | `AssetTags`, `GrantedTags`, `ActiveHandle` |
| `OnGameplayEffectRemoved` | `FFWOnGameplayEffectRemoved` | `AssetTags`, `GrantedTags`, `ActiveHandle` |
| `OnGameplayEffectStackChange` | `FFWOnGameplayEffectStackChange` | `AssetTags`, `GrantedTags`, `ActiveHandle`, `NewStackCount`, `OldStackCount` |
| `OnGameplayEffectTimeChange` | `FFWOnGameplayEffectTimeChange` | `AssetTags`, `GrantedTags`, `ActiveHandle`, `NewStartTime`, `NewDuration` |

#### Other Events

| Event | Delegate | Parameters |
|-------|----------|------------|
| `OnGameplayTagChange` | `FFWOnGameplayTagStackChange` | `GameplayTag`, `NewTagCount` |
| `OnCooldownStart` | `FFWOnCooldownChanged` | `Ability`, `CooldownTags`, `TimeRemaining`, `Duration` |
| `OnCooldownEnd` | `FFWOnCooldownEnd` | `Ability`, `CooldownTag`, `Duration` |
| `OnPreAttributeChange` | `FFWOnPreAttributeChange` | `AttributeSet`, `Attribute`, `NewValue` |
| `OnPostGameplayEffectExecute` | `FFWOnPostGameplayEffectExecute` | `Attribute`, `SourceActor`, `TargetActor`, `SourceTags`, `Payload` |
| `OnInitAbilityActorInfo` | `FFWOnInitAbilityActorInfoCore` | (none) |

#### Key Methods

```cpp
// Attribute queries
float GetHealth() const;
float GetMaxHealth() const;
float GetStamina() const;
float GetMaxStamina() const;
float GetMana() const;
float GetMaxMana() const;
float GetAttributeValue(FGameplayAttribute Attribute) const;
float GetCurrentAttributeValue(FGameplayAttribute Attribute) const;
bool IsAlive() const;

// Attribute mutations
void SetAttributeValue(FGameplayAttribute Attribute, float NewValue);
void ClampAttributeValue(FGameplayAttribute Attribute, float MinValue, float MaxValue);
void AdjustAttributeForMaxChange(UFWAttributeSetBase* AttributeSet,
    const FGameplayAttribute AffectedAttributeProperty,
    const FGameplayAttribute MaxAttribute, float NewMaxValue);

// Ability management
void GrantAbility(TSubclassOf<UGameplayAbility> Ability, int32 Level = 1);
void ClearAbility(TSubclassOf<UGameplayAbility> Ability);
void ClearAbilities(TArray<TSubclassOf<UGameplayAbility>> Abilities);

// Ability activation
bool ActivateAbilityByClass(TSubclassOf<UGameplayAbility> AbilityClass,
    UFWGameplayAbility*& ActivatedAbility, bool bAllowRemoteActivation = true);
bool ActivateAbilityByTags(const FGameplayTagContainer AbilityTags,
    UFWGameplayAbility*& ActivatedAbility, const bool bAllowRemoteActivation = true);

// Ability queries
bool IsUsingAbilityByClass(TSubclassOf<UGameplayAbility> AbilityClass);
bool IsUsingAbilityByTags(FGameplayTagContainer AbilityTags);
TArray<UGameplayAbility*> GetActiveAbilitiesByClass(TSubclassOf<UGameplayAbility> AbilityToSearch) const;
TArray<UGameplayAbility*> GetActiveAbilitiesByTags(const FGameplayTagContainer GameplayTagContainer) const;

// Tag queries
bool HasAnyMatchingGameplayTags(const FGameplayTagContainer TagContainer) const;
bool HasMatchingGameplayTag(const FGameplayTag TagToCheck) const;

// Death
void Die();
```

---

### UFWComboManagerComponent

`#include "Components/FWComboManagerComponent.h"`

Manages combo ability activation and chaining. Works with `UFWComboWindowNotifyState` and `UFWTriggerComboNotify` anim notifies.

**Extends:** `UActorComponent` | **Specifiers:** `BlueprintSpawnableComponent`

#### Replicated Properties

| Property | Type | Description |
|----------|------|-------------|
| `ComboIndex` | `int32` | Current combo step (0-based). |
| `bComboWindowOpened` | `bool` | Whether the combo input window is active. |
| `bShouldTriggerCombo` | `bool` | Whether the next combo should activate. |
| `bRequestTriggerCombo` | `bool` | Whether a combo trigger was requested. |
| `bNextComboAbilityActivated` | `bool` | Whether the next combo ability has been activated. |

#### Key Methods

```cpp
void IncrementCombo();     // Increment combo counter (only if window open)
void ResetCombo();         // Reset combo counter to 0
void ActivateComboAbility(TSubclassOf<UGameplayAbility> AbilityClass,
    bool bAllowRemoteActivation = true);  // Activate a combo ability
UGameplayAbility* GetCurrentActiveComboAbility() const;
```

---

### UFWAbilityQueueComponent

`#include "Components/FWAbilityQueueComponent.h"`

Manages ability queuing so players can buffer the next ability during an active animation window.

**Extends:** `UActorComponent` | **Specifiers:** `BlueprintSpawnableComponent`

#### Key Methods

```cpp
void OpenAbilityQueue();
void CloseAbilityQueue();
void UpdateAllowedAbilitiesForAbilityQueue(TArray<TSubclassOf<UGameplayAbility>> AllowedAbilities);
void SetAllowAllAbilitiesForAbilityQueue(bool bAllowAllAbilities);
bool IsAbilityQueueOpened() const;
const UGameplayAbility* GetCurrentQueuedAbility() const;
```

---

### UFWAbilityInputBindingComponent

`#include "Components/FWAbilityInputBindingComponent.h"`

Bridges Enhanced Input with GAS ability activation. Maps `UInputAction` assets to `FGameplayAbilitySpecHandle`.

**Extends:** `UActorComponent` | **Specifiers:** `BlueprintSpawnableComponent`

#### Key Methods

```cpp
void SetInputBinding(UInputAction* InputAction, EFWAbilityTriggerEvent TriggerEvent,
    FGameplayAbilitySpecHandle AbilityHandle);
void ClearInputBinding(FGameplayAbilitySpecHandle AbilityHandle);
```

---

### UFWLinkAnimLayersComponent

`#include "Components/FWLinkAnimLayersComponent.h"`

Links animation layer interfaces to the owning character's anim instance at runtime. Used with the `UFWGameFeatureAction_AddAnimLayers` GameFeature action.

---

## Animation Notifies

### FWAbilityQueueNotifyState

Anim notify state that opens and closes the ability queue window during a montage. Add to montage sections where you want to accept queued ability input.

### FWComboWindowNotifyState

Anim notify state that opens and closes the combo input window during a montage. Determines when the player can chain into the next combo step.

### FWTriggerComboNotify

Anim notify that triggers the next combo ability in the chain. Place at the point in the montage where the next combo should begin.

### UFWNativeAnimInstance

Base anim instance class providing integration points for GAS-driven animation state.

---

## Modular Gameplay Actors

Pre-built base classes implementing `IGameFrameworkInitStateInterface` for modular game feature support.

| Class | Extends | Description |
|-------|---------|-------------|
| `AFWModularCharacter` | `ACharacter` | Character with modular init state. |
| `AFWModularPlayerState` | `APlayerState` | Player state that can own the ASC. |
| `AFWModularPlayerStateCharacter` | `ACharacter` | Character designed for Player State-owned ASC. |
| `AFWModularPlayerController` | `APlayerController` | Controller with modular init state. |
| `AFWModularActor` | `AActor` | Generic actor with modular init. |
| `AFWModularDefaultPawn` | `ADefaultPawn` | Default pawn with modular init. |
| `AFWModularGameMode` | `AGameModeBase` | GameMode with modular init. |
| `AFWModularGameState` | `AGameStateBase` | GameState with modular init. |
| `AFWModularPawn` | `APawn` | Pawn with modular init. |
| `AFWModularAIController` | `AAIController` | AI Controller with modular init. |

---

## GameFeature Actions

### UFWGameFeatureAction_AddAbilities

`#include "GameFeatures/Actions/FWGameFeatureAction_AddAbilities.h"`

Grants abilities, attribute sets, effects, and ability sets to actors of a specified class when a Game Feature is activated.

#### Configuration

| Property | Type | Description |
|----------|------|-------------|
| `AbilitiesList` | `TArray<FFWGameFeatureAbilitiesEntry>` | Per-actor-class list of abilities/attributes/effects/sets to grant. |

Each `FFWGameFeatureAbilitiesEntry` contains:

| Field | Type | Description |
|-------|------|-------------|
| `ActorClass` | `TSoftClassPtr<AActor>` | Target actor class to receive grants. |
| `GrantedAbilities` | `TArray<FFWGameFeatureAbilityMapping>` | Abilities with input binding. |
| `GrantedAttributes` | `TArray<FFWGameFeatureAttributeSetMapping>` | Attribute sets. |
| `GrantedEffects` | `TArray<FFWGameFeatureGameplayEffectMapping>` | Effects. |
| `GrantedAbilitySets` | `TArray<TSoftObjectPtr<UFWAbilitySet>>` | AbilitySet DataAssets. |

---

### UFWGameFeatureAction_AddAnimLayers

Adds animation layer interfaces to actors when a Game Feature is activated.

---

### UFWGameFeatureAction_AddInputMappingContext

Adds Input Mapping Contexts to local players when a Game Feature is activated.

---

## Types and Structs

### EFWAbilityTriggerEvent

```cpp
UENUM(BlueprintType)
enum class EFWAbilityTriggerEvent : uint8
{
    Started,     // Activate on Action Started (recommended)
    Triggered    // Activate on Action Triggered (use with caution)
};
```

!!! warning
    `Triggered` fires every frame while the input is held. Only use for Input Actions that fire once. For sustained inputs, use `Started` and manage the held state within the ability.

---

### FFWAbilityInputMapping

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Ability` | `TSubclassOf<UGameplayAbility>` | -- | Ability class to grant. |
| `Level` | `int32` | `1` | Ability level. |
| `InputAction` | `TObjectPtr<UInputAction>` | `nullptr` | Input action to bind (optional). |
| `TriggerEvent` | `EFWAbilityTriggerEvent` | `Started` | When to activate on input. |

---

### FFWAttributeSetDefinition

| Field | Type | Description |
|-------|------|-------------|
| `AttributeSet` | `TSubclassOf<UAttributeSet>` | Attribute set class to grant. |
| `InitializationData` | `TObjectPtr<UDataTable>` | DataTable for initial values (optional). |

---

### FFWGameplayEffectContainer

Data-driven effect container mapping a target type to a list of effects.

| Field | Type | Description |
|-------|------|-------------|
| `TargetType` | `TSubclassOf<UFWTargetType>` | How targets are resolved. |
| `TargetGameplayEffectClasses` | `TArray<TSubclassOf<UGameplayEffect>>` | Effects to apply. |
| `bUseSetByCallerMagnitude` | `bool` | Whether to use SetByCaller for magnitude. |
| `SetByCallerDataTag` | `FGameplayTag` | Tag for SetByCaller lookup. |
| `SetByCallerMagnitude` | `float` | Default magnitude value. |

---

### FFWGameplayEffectContainerSpec

Runtime-resolved version of `FFWGameplayEffectContainer`.

| Field | Type | Description |
|-------|------|-------------|
| `TargetData` | `FGameplayAbilityTargetDataHandle` | Resolved targets. |
| `TargetGameplayEffectSpecs` | `TArray<FGameplayEffectSpecHandle>` | Ready-to-apply effect specs. |

**Methods:** `HasValidEffects()`, `HasValidTargets()`, `AddTargets(HitResults, TargetActors)`

---

### FFWGameplayEffectExecuteData

Payload passed to `OnPostGameplayEffectExecute`.

| Field | Type | Description |
|-------|------|-------------|
| `AttributeSet` | `UAttributeSet*` | Source attribute set. |
| `AbilitySystemComponent` | `UAbilitySystemComponent*` | Owner ASC. |
| `DeltaValue` | `float` | Calculated change. |
| `ClampMinimumValue` | `float` | Min clamp value from config. |

---

## Blueprint Library

### UFWBlueprintFunctionLibrary

`#include "Abilities/FWBlueprintFunctionLibrary.h"`

Static Blueprint helpers for GAS operations. Available as nodes in any Blueprint graph.

### UFWTargetType

`#include "Abilities/FWTargetType.h"`

Base class for target type resolution in effect containers. Subclass to implement custom targeting logic (e.g., sphere trace, cone, projectile path).

---

## UI Widgets

| Class | Description |
|-------|-------------|
| `UFWUserWidget` | Base user widget with GAS integration. |
| `UFWUWHud` | HUD widget base for ability system display. |
| `UFWUWDebugAbilityQueue` | Debug widget showing ability queue state. |
| `UFWUWDebugComboWidget` | Debug widget showing combo system state. |
