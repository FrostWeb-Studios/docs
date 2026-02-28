---
title: FAQ - GAS System
---

# Frequently Asked Questions

---

## Setup and Initialization

### I get crashes on ability activation. What is wrong?

The most common cause is forgetting to call `UAbilitySystemGlobals::Get().InitGlobalData()` during game initialization. Add this call to your `UGameInstance::Init()` override.

### Where should the ASC live -- on the Pawn or the Player State?

Both patterns are supported:

| Pattern | ASC Location | Use When |
|---------|--------------|----------|
| **Pawn-based** | On the Character/Pawn | Simple setup, abilities reset on death, no persistence needed. |
| **Player State-based** | On the Player State | Abilities and attributes persist across respawns. Recommended for multiplayer. |

For Player State-based ASC:

- Use `AFWModularPlayerState` (or subclass) to hold the ASC.
- Use `AFWModularPlayerStateCharacter` for the pawn.
- Set `bResetAbilitiesOnSpawn = false` and `bResetAttributesOnSpawn = false` on the ASC.

### When does InitAbilityActorInfo fire?

Multiple times during the actor's lifecycle:

1. Server: after component initialization.
2. Server: after owning actor replication (e.g., Possessed By for Player State).
3. Client: after component initialization.
4. Client: after owning actor replication (e.g., OnRep_PlayerState).

The `OnInitAbilityActorInfo` event on both the ASC and Core Component fires after each call. Design your initialization logic to be idempotent.

---

## Input Binding

### How does Enhanced Input binding work with GAS?

The flow is:

1. You configure `FFWAbilityInputMapping` with an `InputAction` and `TriggerEvent`.
2. When the ASC grants the ability, it finds the `UFWAbilityInputBindingComponent` on the avatar actor.
3. The binding component registers an Enhanced Input callback for the specified Input Action.
4. When the Input Action fires, it calls `AbilityLocalInputPressed()` on the ASC with the ability's `InputID`.
5. The ASC attempts to activate the bound ability.

### My input binding is not working. What should I check?

Checklist:

- [ ] The character has a `UFWAbilityInputBindingComponent`.
- [ ] The `InputAction` asset is valid and added to an active Input Mapping Context.
- [ ] The ability is actually granted (check with `showdebug abilitysystem`).
- [ ] The `TriggerEvent` matches the Input Action's trigger configuration.
- [ ] For Player State ASCs, `bResetAbilitiesOnSpawn` is `false` (prevents duplicate bindings).

### Can I change input bindings at runtime?

Yes. Call `SetInputBinding()` on the `UFWAbilityInputBindingComponent` with a new Input Action and the ability's `FGameplayAbilitySpecHandle`. Call `ClearInputBinding()` to remove a binding.

---

## Ability Sets

### What is the difference between GrantedAbilities on the ASC and an AbilitySet?

| Feature | ASC GrantedAbilities | AbilitySet DataAsset |
|---------|---------------------|---------------------|
| Configuration | Directly on the component | Separate DataAsset |
| Reusability | Per-component | Shared across actors |
| Runtime grant/remove | Manual | Handle-based lifecycle |
| Content separation | Tightly coupled | Loosely coupled |
| GameFeature support | Limited | Full support |

AbilitySets are recommended for most use cases. Use direct ASC granting only for simple prototyping or abilities that are truly per-actor.

### Can I grant the same AbilitySet multiple times?

The ASC tracks granted sets and prevents duplicate granting via `ShouldGrantAbilitySet()`. If the set is already granted, the method returns without re-granting.

### How do I remove an AbilitySet at runtime?

Use the handle returned from `GiveAbilitySet()`:

```cpp
FFWAbilitySetHandle CombatHandle;
ASC->GiveAbilitySet(CombatSet, CombatHandle);

// Later, to remove:
ASC->ClearAbilitySet(CombatHandle);
```

---

## Combo System

### How does the combo chain work?

The flow for a three-hit combo:

1. Player presses attack. ASC activates `GA_MeleeCombo`. Combo index is 0. Montage `AM_Slash1` plays.
2. During `AM_Slash1`, the `FWComboWindowNotifyState` opens the combo window.
3. Player presses attack during the open window. The `UFWComboManagerComponent` detects this and sets `bShouldTriggerCombo = true`.
4. The `FWTriggerComboNotify` fires, activating the next combo ability. Combo index increments to 1. `AM_Slash2` plays.
5. Repeat for the third hit.
6. After the final montage ends (or if the window closes without input), `ResetCombo()` is called.

### My combo is not chaining. What should I check?

Checklist:

- [ ] The character has a `UFWComboManagerComponent`.
- [ ] The montages have `FWComboWindowNotifyState` notify states at the correct positions.
- [ ] The montages have `FWTriggerComboNotify` instant notifies within the combo window.
- [ ] The `ComboMontages` array on the ability is populated with montages in order.
- [ ] The ability has `InstancingPolicy = InstancedPerActor` (required for combo state tracking).

### Can combos work in multiplayer?

Yes. The `UFWComboManagerComponent` uses replicated properties and Server/Multicast RPCs:

- `ComboIndex`, `bComboWindowOpened`, `bShouldTriggerCombo` are replicated.
- `ServerActivateComboAbility` and `MulticastActivateComboAbility` RPCs handle authority routing.
- `ServerSetComboIndex` and `MulticastSetComboIndex` RPCs synchronize the counter.

---

## Ability Queue

### When should I use ability queuing vs. combo chaining?

| System | Purpose | Example |
|--------|---------|---------|
| **Combo** | Chain the same ability into a multi-step sequence. | Three-hit melee combo. |
| **Queue** | Buffer a different ability to activate after the current one ends. | Queue a dodge roll during an attack animation. |

Both systems can be active simultaneously on the same character.

### How do I configure what abilities can be queued?

Use `UpdateAllowedAbilitiesForAbilityQueue()` on the `UFWAbilityQueueComponent`. Pass an array of allowed ability classes. Alternatively, call `SetAllowAllAbilitiesForAbilityQueue(true)` to allow any ability.

The `FWAbilityQueueNotifyState` in montages can also specify allowed abilities per animation window.

---

## Attributes

### How does the Damage meta attribute work?

`Damage` is a server-only, non-replicated attribute. The flow:

1. A `UGameplayEffect` modifies `FWAttributeSet.Damage` (e.g., +25).
2. `PostGameplayEffectExecute` fires on the server.
3. The attribute set reads the `Damage` value, subtracts it from `Health`, and resets `Damage` to 0.
4. If Health reaches 0, it fires the death event via `UFWGASCoreComponent::Die()`.
5. The Health change replicates to clients via `OnRep_Health`.

This pattern ensures damage calculations are server-authoritative while the visual result (health change) replicates efficiently.

### How do I add custom attributes?

Create a new class inheriting from `UFWAttributeSetBase`:

```cpp
UCLASS()
class UMyCustomAttributes : public UFWAttributeSetBase
{
    GENERATED_BODY()

public:
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_Luck)
    FGameplayAttributeData Luck;
    ATTRIBUTE_ACCESSORS(UMyCustomAttributes, Luck)

protected:
    UFUNCTION()
    void OnRep_Luck(const FGameplayAttributeData& OldValue);
};
```

Add it to the `GrantedAttributes` on the ASC or in an AbilitySet.

### Why are my attribute initial values zero?

Either:

1. You did not provide an `InitializationData` DataTable.
2. The DataTable row names do not match the attribute names.
3. The DataTable row structure is not `AttributeMetaData`.

---

## GameFeature Actions

### What happens when a GameFeature is deactivated?

The `UFWGameFeatureAction_AddAbilities` action tracks all granted abilities, attributes, effects, and ability sets per actor. When the feature deactivates, it calls the corresponding remove methods for each, cleanly tearing down everything that was granted.

### Can I use GameFeature actions with Player State ASCs?

Yes, but the `ActorClass` should be set to your Player State class (not the Pawn), since that is where the ASC lives.

---

## Performance

### Are there performance concerns with many ability grants?

The initial grant during `InitAbilityActorInfo` iterates through all `GrantedAbilities`, `GrantedAttributes`, `GrantedEffects`, and `GrantedAbilitySets`. For characters with dozens of abilities, this is still fast (sub-millisecond). AbilitySets are loaded asynchronously via soft references, so they do not block the main thread.

### Does the combo system add replication overhead?

The combo manager replicates five boolean/integer properties and uses reliable RPCs for activation. The overhead is minimal -- comparable to a single replicated actor component.
