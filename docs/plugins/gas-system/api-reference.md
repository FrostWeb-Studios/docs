---
title: API Reference - GAS System
---

# API Reference

Reference for all public classes, components, data assets, and types in FWGASSystem.

---

## UFWAbilitySystemComponent

Extended Ability System Component with startup grants, ability sets, and configurable replication. Auto-grants abilities, attributes, effects, and ability sets on `InitAbilityActorInfo`.

**Parent:** `UAbilitySystemComponent`

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `DefaultAbilities` | `TArray<AbilityInputMapping>` | Empty | Abilities to grant on init, with optional input binding |
| `DefaultAttributes` | `TArray<AttributeSetDefinition>` | Empty | Attribute sets to grant, with optional DataTable init |
| `StartupEffects` | `TArray<TSubclassOf<UGameplayEffect>>` | Empty | Effects to apply on init (e.g., regen effects) |
| `GrantedAbilitySets` | `TArray<TSoftObjectPtr<UFWAbilitySet>>` | Empty | AbilitySet DataAssets to grant on init |
| `ReplicationMode` | `EGameplayEffectReplicationMode` | `Mixed` | GAS replication mode |
| `bResetAbilitiesOnSpawn` | `bool` | `true` | Re-grant abilities on each init. Set `false` for Player State ASCs |
| `bResetAttributesOnSpawn` | `bool` | `true` | Re-initialize attributes on each init |

### Functions

| Function | Return | Description |
|----------|--------|-------------|
| `GiveAbilitySet(AbilitySet, OutHandle)` | `bool` | Grant an AbilitySet at runtime; returns a handle for later removal |
| `ClearAbilitySet(InHandle)` | `bool` | Remove a previously granted AbilitySet by handle |

### Events

| Event | Description |
|-------|-------------|
| `OnInitAbilityActorInfo` | Fired after initialization, once abilities and attributes have been granted |

!!! info "Player State ASCs"
    When using a Player State-owned ASC, set `bResetAbilitiesOnSpawn` to `false`. Abilities are granted only on the first init call. Subsequent calls (e.g., after respawn) skip re-granting. Do **not** set to `true` if using input binding -- it will cause duplicate bindings.

---

## UFWGASCoreComponent

Abstraction layer over the ASC providing a unified Blueprint event surface. Handles ASC owner/avatar complexity, especially for Player State-based ASC setups.

**Parent:** `UActorComponent`

### Attribute Query Functions

| Function | Return | Description |
|----------|--------|-------------|
| `GetHealth()` | `float` | Current health value |
| `GetMaxHealth()` | `float` | Maximum health value |
| `GetStamina()` | `float` | Current stamina value |
| `GetMaxStamina()` | `float` | Maximum stamina value |
| `GetMana()` | `float` | Current mana value |
| `GetMaxMana()` | `float` | Maximum mana value |
| `GetAttributeValue(Attribute)` | `float` | Get any attribute's base value |
| `GetCurrentAttributeValue(Attribute)` | `float` | Get any attribute's current (modified) value |
| `IsAlive()` | `bool` | Whether the actor is alive (Health > 0) |

### Attribute Modification Functions

| Function | Description |
|----------|-------------|
| `SetAttributeValue(Attribute, NewValue)` | Set an attribute's base value directly |
| `ClampAttributeValue(Attribute, Min, Max)` | Clamp an attribute between min and max |
| `AdjustAttributeForMaxChange(AttributeSet, AffectedAttribute, MaxAttribute, NewMaxValue)` | Proportionally adjust a current value when its max changes |

### Ability Management Functions

| Function | Return | Description |
|----------|--------|-------------|
| `GrantAbility(AbilityClass, Level)` | `void` | Grant an ability at the specified level |
| `ClearAbility(AbilityClass)` | `void` | Remove a granted ability |
| `ClearAbilities(AbilityClasses)` | `void` | Remove multiple granted abilities |
| `ActivateAbilityByClass(AbilityClass, OutAbility, bAllowRemote)` | `bool` | Activate an ability by class |
| `ActivateAbilityByTags(Tags, OutAbility, bAllowRemote)` | `bool` | Activate an ability by gameplay tags |
| `IsUsingAbilityByClass(AbilityClass)` | `bool` | Check if an ability of this class is active |
| `IsUsingAbilityByTags(Tags)` | `bool` | Check if an ability matching these tags is active |
| `GetActiveAbilitiesByClass(AbilityClass)` | `TArray<UGameplayAbility*>` | Get all active instances of an ability class |
| `GetActiveAbilitiesByTags(Tags)` | `TArray<UGameplayAbility*>` | Get active abilities matching tags |
| `HasMatchingGameplayTag(Tag)` | `bool` | Check for a specific gameplay tag |
| `HasAnyMatchingGameplayTags(Tags)` | `bool` | Check for any matching gameplay tags |
| `Die()` | `void` | Trigger the death sequence |

### Attribute Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnHealthChange` | `DeltaValue`, `EventTags` | Health attribute changed |
| `OnStaminaChange` | `DeltaValue`, `EventTags` | Stamina attribute changed |
| `OnManaChange` | `DeltaValue`, `EventTags` | Mana attribute changed |
| `OnFervorChange` | `DeltaValue`, `EventTags` | Fervor attribute changed |
| `OnAttributeChange` | `Attribute`, `DeltaValue`, `EventTags` | Any attribute changed (generic) |
| `OnDamage` | `DamageAmount`, `SourceCharacter`, `DamageTags` | Damage was dealt |
| `OnDeath` | (none) | Actor died |

### Ability Lifecycle Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnAbilityActivated` | `Ability` | An ability was activated |
| `OnAbilityEnded` | `Ability` | An ability ended |
| `OnAbilityFailed` | `Ability`, `ReasonTags` | An ability failed to activate |
| `OnAbilityCommit` | `Ability` | An ability committed (cost/cooldown applied) |

### Effect Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnGameplayEffectAdded` | `AssetTags`, `GrantedTags`, `ActiveHandle` | An effect was applied |
| `OnGameplayEffectRemoved` | `AssetTags`, `GrantedTags`, `ActiveHandle` | An effect was removed |
| `OnGameplayEffectStackChange` | `AssetTags`, `GrantedTags`, `ActiveHandle`, `NewStackCount`, `OldStackCount` | Effect stack count changed |
| `OnGameplayEffectTimeChange` | `AssetTags`, `GrantedTags`, `ActiveHandle`, `NewStartTime`, `NewDuration` | Effect duration changed |

### Other Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `OnGameplayTagChange` | `GameplayTag`, `NewTagCount` | A gameplay tag count changed |
| `OnCooldownStart` | `Ability`, `CooldownTags`, `TimeRemaining`, `Duration` | A cooldown started |
| `OnCooldownEnd` | `Ability`, `CooldownTag`, `Duration` | A cooldown ended |
| `OnPreAttributeChange` | `AttributeSet`, `Attribute`, `NewValue` | Attribute about to change (pre-modification) |
| `OnPostGameplayEffectExecute` | `Attribute`, `SourceActor`, `TargetActor`, `SourceTags`, `Payload` | Post effect execution |
| `OnInitAbilityActorInfo` | (none) | ASC initialization complete |

---

## UFWAbilityInputBindingComponent

Bridges Enhanced Input with GAS ability activation. Maps Input Action assets to ability spec handles.

**Parent:** `UActorComponent`

### Functions

| Function | Description |
|----------|-------------|
| `SetInputBinding(InputAction, TriggerEvent, AbilityHandle)` | Bind an Input Action to an ability |
| `ClearInputBinding(AbilityHandle)` | Remove an input binding for an ability |

---

## UFWAbilityQueueComponent

Manages ability queuing so players can buffer the next ability during an active animation window.

**Parent:** `UActorComponent`

### Functions

| Function | Return | Description |
|----------|--------|-------------|
| `OpenAbilityQueue()` | `void` | Open the queue window (accepts input) |
| `CloseAbilityQueue()` | `void` | Close the queue window |
| `UpdateAllowedAbilitiesForAbilityQueue(AllowedAbilities)` | `void` | Set which abilities can be queued |
| `SetAllowAllAbilitiesForAbilityQueue(bAllowAll)` | `void` | Allow or restrict all abilities in the queue |
| `IsAbilityQueueOpened()` | `bool` | Whether the queue is currently accepting input |
| `GetCurrentQueuedAbility()` | `UGameplayAbility*` | The currently queued ability, if any |

---

## UFWComboManagerComponent

Manages combo ability activation and chaining. Works with combo window and trigger combo anim notifies in montages.

**Parent:** `UActorComponent`

### Replicated Properties

| Property | Type | Description |
|----------|------|-------------|
| `ComboIndex` | `int32` | Current combo step (0-based) |
| `bComboWindowOpened` | `bool` | Whether the combo input window is active |
| `bShouldTriggerCombo` | `bool` | Whether the next combo should activate |
| `bRequestTriggerCombo` | `bool` | Whether a combo trigger was requested |
| `bNextComboAbilityActivated` | `bool` | Whether the next combo ability has been activated |

### Functions

| Function | Return | Description |
|----------|--------|-------------|
| `IncrementCombo()` | `void` | Increment combo counter (only if window open) |
| `ResetCombo()` | `void` | Reset combo counter to 0 |
| `ActivateComboAbility(AbilityClass, bAllowRemote)` | `void` | Activate a combo ability |
| `GetCurrentActiveComboAbility()` | `UGameplayAbility*` | Get the currently active combo ability |

---

## UFWLinkAnimLayersComponent

Links animation layer interfaces to the owning character's anim instance at runtime. Used with the AddAnimLayers GameFeature action.

**Parent:** `UActorComponent`

---

## Data Assets

### UFWAbilitySet

DataAsset bundling abilities, attribute sets, effects, and owned tags for bulk granting to an ASC.

**Parent:** `UPrimaryDataAsset`

| Property | Type | Description |
|----------|------|-------------|
| `GrantedAbilities` | `TArray<AbilityMapping>` | Abilities with input binding configuration |
| `GrantedAttributes` | `TArray<AttributeSetMapping>` | Attribute sets with optional DataTable init |
| `GrantedEffects` | `TArray<EffectMapping>` | Effects to apply with level |
| `OwnedTags` | `FGameplayTagContainer` | Tags applied to the ASC when the set is granted |

| Function | Return | Description |
|----------|--------|-------------|
| `GrantToAbilitySystem(ASC, OutHandle)` | `bool` | Grant this set to an ASC; returns a handle for removal |
| `GrantToAbilitySystem(Actor, OutHandle)` | `bool` | Grant to an actor (finds ASC automatically) |
| `RemoveFromAbilitySystem(ASC, Handle)` | `bool` | Static: remove a previously granted set |
| `HasInputBinding()` | `bool` | Check if any ability in this set has input binding |

---

### UFWGameplayAbility

Base gameplay ability class with effect container support and ability queue integration.

**Parent:** `UGameplayAbility`

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `bEnableAbilityQueue` | `bool` | `false` | Allows other abilities to be queued during this ability |
| `EffectContainerMap` | `TMap<FGameplayTag, EffectContainer>` | Empty | Maps gameplay tags to effect containers |

| Function | Return | Description |
|----------|--------|-------------|
| `MakeEffectContainerSpec(Tag, EventData, Level)` | `EffectContainerSpec` | Look up a container by tag and create its spec |
| `ApplyEffectContainerSpec(Spec)` | `TArray<ActiveEffectHandle>` | Apply a previously created container spec |
| `ApplyEffectContainer(Tag, EventData, Level)` | `TArray<ActiveEffectHandle>` | Create and apply an effect container in one call |

---

## Attribute Sets

### UFWAttributeSet

Concrete attribute set with common combat attributes. All gameplay attributes are replicated.

**Parent:** `UFWAttributeSetBase`

| Attribute | Replicated | Description |
|-----------|------------|-------------|
| `Health` | Yes | Current health |
| `MaxHealth` | Yes | Maximum health |
| `HealthRegenRate` | Yes | Health regen per period |
| `Stamina` | Yes | Current stamina |
| `MaxStamina` | Yes | Maximum stamina |
| `StaminaRegenRate` | Yes | Stamina regen per period |
| `Mana` | Yes | Current mana |
| `MaxMana` | Yes | Maximum mana |
| `ManaRegenRate` | Yes | Mana regen per period |
| `Damage` | No | Meta attribute for damage calculation (server-only) |
| `StaminaDamage` | No | Meta attribute for stamina damage (server-only) |

### UFWAttributeSetBase

Abstract base attribute set providing shared infrastructure: `PreAttributeChange`, `PostGameplayEffectExecute`, `AdjustAttributeForMaxChange`, and clamp value support. Extend this class to create custom attribute sets.

**Parent:** `UAttributeSet` (Abstract)

---

## Modular Actor Base Classes

Pre-built base classes implementing `IGameFrameworkInitStateInterface` for modular game feature support.

| Class | Parent | Description |
|-------|--------|-------------|
| `AFWModularCharacter` | `ACharacter` | Character with modular init state |
| `AFWModularPlayerState` | `APlayerState` | Player state that can own the ASC |
| `AFWModularPlayerStateCharacter` | `ACharacter` | Character designed for Player State-owned ASC |
| `AFWModularPlayerController` | `APlayerController` | Controller with modular init state |
| `AFWModularActor` | `AActor` | Generic actor with modular init |
| `AFWModularDefaultPawn` | `ADefaultPawn` | Default pawn with modular init |
| `AFWModularGameMode` | `AGameModeBase` | GameMode with modular init |
| `AFWModularGameState` | `AGameStateBase` | GameState with modular init |
| `AFWModularPawn` | `APawn` | Pawn with modular init |
| `AFWModularAIController` | `AAIController` | AI Controller with modular init |
| `AFWModularHUD` | `AHUD` | HUD with modular init |
| `AFWModularPlayerStatePlayerController` | `APlayerController` | Controller for Player State-owned ASC setups |

---

## GameFeature Actions

### AddAbilities

Grants abilities, attribute sets, effects, and ability sets to actors of a specified class when a Game Feature is activated.

**Configuration per actor class:**

| Field | Type | Description |
|-------|------|-------------|
| `ActorClass` | `TSoftClassPtr<AActor>` | Target actor class to receive grants |
| `GrantedAbilities` | `TArray<AbilityMapping>` | Abilities with input binding |
| `GrantedAttributes` | `TArray<AttributeSetMapping>` | Attribute sets |
| `GrantedEffects` | `TArray<EffectMapping>` | Effects |
| `GrantedAbilitySets` | `TArray<TSoftObjectPtr<UFWAbilitySet>>` | AbilitySet DataAssets |

### AddAnimLayers

Adds animation layer interfaces to actors when a Game Feature is activated.

### AddInputMappingContext

Adds Input Mapping Contexts to local players when a Game Feature is activated.

---

## Animation Notifies

| Notify | Type | Description |
|--------|------|-------------|
| `FWAbilityQueueNotifyState` | Notify State | Opens/closes the ability queue window during a montage section |
| `FWComboWindowNotifyState` | Notify State | Opens/closes the combo input window during a montage section |
| `FWTriggerComboNotify` | Notify | Triggers the next combo ability at the notify point in the montage |

### UFWNativeAnimInstance

Base anim instance class providing integration points for GAS-driven animation state. Use as the parent class for animation Blueprints that need to read GAS data.

---

## Types

### EFWAbilityTriggerEvent

| Value | Description |
|-------|-------------|
| `Started` | Activate on input action started (recommended) |
| `Triggered` | Activate on input action triggered (fires every frame while held -- use with caution) |

!!! warning
    `Triggered` fires every frame while the input is held. Only use for Input Actions that fire once. For sustained inputs, use `Started` and manage the held state within the ability.

### FFWAbilityInputMapping

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Ability` | `TSubclassOf<UGameplayAbility>` | -- | Ability class to grant |
| `Level` | `int32` | `1` | Ability level |
| `InputAction` | `UInputAction*` | `nullptr` | Input action to bind (optional) |
| `TriggerEvent` | `EFWAbilityTriggerEvent` | `Started` | When to activate on input |

### FFWAttributeSetDefinition

| Field | Type | Description |
|-------|------|-------------|
| `AttributeSet` | `TSubclassOf<UAttributeSet>` | Attribute set class to grant |
| `InitializationData` | `UDataTable*` | DataTable for initial values (optional) |

### FFWGameplayEffectContainer

Data-driven effect container mapping a target type to a list of effects.

| Field | Type | Description |
|-------|------|-------------|
| `TargetType` | `TSubclassOf<UFWTargetType>` | How targets are resolved |
| `TargetGameplayEffectClasses` | `TArray<TSubclassOf<UGameplayEffect>>` | Effects to apply |
| `bUseSetByCallerMagnitude` | `bool` | Whether to use SetByCaller for magnitude |
| `SetByCallerDataTag` | `FGameplayTag` | Tag for SetByCaller lookup |
| `SetByCallerMagnitude` | `float` | Default magnitude value |

### FFWAbilitySetHandle

Handle struct for tracking what was granted by a `UFWAbilitySet`. Pass this to removal functions to cleanly undo a grant.

---

## Target Types

### UFWTargetType

Base class for target type resolution in effect containers. Subclass to implement custom targeting logic (e.g., sphere trace, cone, projectile path).

---

## UI Widgets

| Class | Description |
|-------|-------------|
| `UFWUserWidget` | Base user widget with GAS integration |
| `UFWUWHud` | HUD widget base for ability system display |
| `UFWUWDebugAbilityQueue` | Debug widget showing ability queue state |
| `UFWUWDebugComboWidget` | Debug widget showing combo system state |

---

## Blueprint Function Library

`UFWBlueprintFunctionLibrary` provides static Blueprint helpers for common GAS operations. Available as nodes in any Blueprint graph.

---

## Configuration

### UFWGASDeveloperSettings

Project-wide GAS developer settings. Accessible via **Project Settings > Plugins > FW GAS System**. Use this to configure default behaviors and attribute clamping values across your project.
