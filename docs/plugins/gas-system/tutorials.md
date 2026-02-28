---
title: Tutorials - GAS System
---

# Tutorial: Creating a Combat Ability with Combos

This tutorial walks through building a complete three-hit melee combo system. You will create an AbilitySet, implement a GameplayAbility subclass, configure combo windows via anim notifies, bind input with EnhancedInput, and optionally use GameFeature actions for modular content injection.

---

## Overview

By the end of this tutorial you will have:

- An AbilitySet DataAsset grouping combat abilities and attributes
- A melee ability subclass that plays montages and applies damage via effect containers
- A three-hit combo chain driven by anim notify states
- EnhancedInput binding so the player attacks with a single button
- GameFeature action configuration for clean content separation

---

## Step 1: Create the AbilitySet

Create a new `UFWAbilitySet` DataAsset at `/Game/Data/AbilitySets/AS_CombatMelee`.

### Granted Attributes

Add one entry:

| Attribute Set | Initialization Data |
|---------------|-------------------|
| `FWAttributeSet` | `DT_DefaultAttributes` (your DataTable) |

### Granted Abilities

Leave empty for now -- we will add the ability after creating it in Step 2.

### Granted Effects

Add your passive effects:

| Effect Type | Level |
|-------------|-------|
| `GE_HealthRegen` | `1.0` |
| `GE_StaminaRegen` | `1.0` |

### Owned Tags

Add `State.Combat.Ready` so the character is tagged as combat-capable when this set is active.

---

## Step 2: Create the Melee Ability

### Header

```cpp
#pragma once

#include "Abilities/FWGameplayAbility.h"
#include "GA_MeleeCombo.generated.h"

UCLASS()
class UGA_MeleeCombo : public UFWGameplayAbility
{
    GENERATED_BODY()

public:
    UGA_MeleeCombo();

    virtual void ActivateAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo,
        const FGameplayEventData* TriggerEventData) override;

    virtual void EndAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo,
        bool bReplicateEndAbility,
        bool bWasCancelled) override;

protected:
    /** Montages for each combo step */
    UPROPERTY(EditDefaultsOnly, Category = "Combo")
    TArray<TObjectPtr<UAnimMontage>> ComboMontages;

    /** Stamina cost tag for the effect container */
    UPROPERTY(EditDefaultsOnly, Category = "Combo")
    FGameplayTag StaminaCostTag;

    /** Gets the appropriate montage for the current combo index */
    UAnimMontage* GetMontageForComboIndex(int32 ComboIndex) const;
};
```

### Implementation

```cpp
#include "GA_MeleeCombo.h"
#include "AbilitySystemComponent.h"
#include "Components/FWComboManagerComponent.h"

UGA_MeleeCombo::UGA_MeleeCombo()
{
    InstancingPolicy = EGameplayAbilityInstancingPolicy::InstancedPerActor;
    NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;

    // Enable the ability queue so players can buffer inputs
    bEnableAbilityQueue = false; // We use notify states instead
}

void UGA_MeleeCombo::ActivateAbility(
    const FGameplayAbilitySpecHandle Handle,
    const FGameplayAbilityActorInfo* ActorInfo,
    const FGameplayAbilityActivationInfo ActivationInfo,
    const FGameplayEventData* TriggerEventData)
{
    if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
    {
        EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
        return;
    }

    // Get the combo manager to determine which montage to play
    AActor* AvatarActor = GetAvatarActorFromActorInfo();
    UFWComboManagerComponent* ComboComp = AvatarActor
        ? AvatarActor->FindComponentByClass<UFWComboManagerComponent>()
        : nullptr;

    int32 CurrentCombo = ComboComp ? ComboComp->ComboIndex : 0;
    UAnimMontage* Montage = GetMontageForComboIndex(CurrentCombo);

    if (!Montage)
    {
        EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
        return;
    }

    // Play the montage
    UAbilitySystemComponent* ASC = GetAbilitySystemComponentFromActorInfo();
    if (ASC)
    {
        ASC->PlayMontage(this, ActivationInfo, Montage, 1.0f);
    }

    // Increment the combo for the next activation
    if (ComboComp)
    {
        ComboComp->IncrementCombo();
    }
}

void UGA_MeleeCombo::EndAbility(
    const FGameplayAbilitySpecHandle Handle,
    const FGameplayAbilityActorInfo* ActorInfo,
    const FGameplayAbilityActivationInfo ActivationInfo,
    bool bReplicateEndAbility,
    bool bWasCancelled)
{
    Super::EndAbility(Handle, ActorInfo, ActivationInfo, bReplicateEndAbility, bWasCancelled);
}

UAnimMontage* UGA_MeleeCombo::GetMontageForComboIndex(int32 ComboIndex) const
{
    if (ComboMontages.IsValidIndex(ComboIndex))
    {
        return ComboMontages[ComboIndex];
    }
    return ComboMontages.IsEmpty() ? nullptr : ComboMontages[0];
}
```

---

## Step 3: Create the Attack Montages

Create three animation montages for the combo chain:

| Montage | Animation | Purpose |
|---------|-----------|---------|
| `AM_Combo_Slash1` | Horizontal slash | First hit |
| `AM_Combo_Slash2` | Vertical slash | Second hit |
| `AM_Combo_Slash3` | Heavy overhead | Third hit (finisher) |

### Add Anim Notify States

On each montage, add the following notify tracks:

#### Combo Window (FWComboWindowNotifyState)

This notify state defines when the player's input is accepted for chaining to the next combo step.

```
Montage Timeline:
|--Windup--|--Strike--|==Combo Window==|--Recovery--|
                      ^                ^
                      Start            End
```

Place the **FW Combo Window Notify State** to cover the window after the strike impact but before the recovery finishes. Typical placement: 40--70% of the montage duration.

#### Trigger Combo (FWTriggerComboNotify)

This instant notify triggers the transition to the next combo ability. Place it within the combo window, typically at the point where the animation allows a natural transition.

```
Montage Timeline:
|--Windup--|--Strike--|==Combo Window==|--Recovery--|
                          ^
                          Trigger Combo Notify
```

#### Ability Queue Window (FWAbilityQueueNotifyState)

Optionally add this to allow queuing of non-combo abilities (e.g., a dodge roll) during the montage.

```
Montage Timeline:
|--Windup--|--Strike--|==Combo Window==|==Queue Window==|--Recovery--|
```

!!! tip "Notify Placement"
    The combo window and ability queue window can overlap, but they serve different purposes:

    - **Combo Window** -- accepts the same ability input to chain the combo.
    - **Ability Queue Window** -- accepts a different ability input to queue for after this ability ends.

---

## Step 4: Configure the Effect Container

On your `GA_MeleeCombo` Blueprint (or in the CDO), configure the `EffectContainerMap`:

| Tag Key | Target Type | Effects |
|---------|-------------|---------|
| `Ability.Melee.Hit` | `TargetType_UseEventData` | `GE_MeleeDamage` |
| `Ability.Melee.Cost` | `TargetType_Self` | `GE_StaminaCost` |

In the ability's damage notification (e.g., from a hit trace or overlap event):

```cpp
void UGA_MeleeCombo::OnHitConfirmed(const FGameplayEventData& EventData)
{
    // Apply the damage container
    ApplyEffectContainer(
        FGameplayTag::RequestGameplayTag(FName("Ability.Melee.Hit")),
        EventData
    );
}
```

---

## Step 5: Set Up Input Binding

### Create the Input Action

1. Create a new `UInputAction` asset: `IA_MeleeAttack`.
2. Set **Value Type** to `Bool` (digital input).
3. Add it to your Input Mapping Context with the desired key binding (e.g., Left Mouse Button).

### Add to the AbilitySet

Go back to your `AS_CombatMelee` AbilitySet and add the ability:

| Ability Type | Level | Input Action | Trigger Event |
|-------------|-------|--------------|---------------|
| `GA_MeleeCombo` | `1` | `IA_MeleeAttack` | `Started` |

### Ensure Input Binding Component Exists

Your character must have a `UFWAbilityInputBindingComponent`. If using `AFWModularCharacter` or adding components manually, verify it is present:

```cpp
AbilityInputBinding = CreateDefaultSubobject<UFWAbilityInputBindingComponent>(
    TEXT("AbilityInputBinding"));
```

!!! info "How Input Binding Works"
    When the ASC grants an ability with an Input Action, it finds the `UFWAbilityInputBindingComponent` on the avatar actor and calls `SetInputBinding()`. This registers an Enhanced Input callback that calls `AbilityLocalInputPressed/Released` on the ASC when the Input Action fires.

---

## Step 6: Grant the AbilitySet

=== "Via ASC Property"

    On your character's `UFWAbilitySystemComponent`, add `AS_CombatMelee` to the **Granted Ability Sets** array. The set is automatically granted on `InitAbilityActorInfo`.

=== "Via GameFeature Action"

    In your GameFeature plugin's `GameFeatureData` asset:

    1. Add an action: **Add Abilities (FW GAS System)**.
    2. Configure an entry:
        - **Actor Class**: Your character class.
        - **Granted Ability Sets**: `AS_CombatMelee`.

    When the GameFeature activates, the combat set is granted. When deactivated, it is cleanly removed.

=== "Via C++ at Runtime"

    ```cpp
    void AMyCharacter::GrantCombatAbilities()
    {
        UFWAbilitySystemComponent* ASC = FindComponentByClass<UFWAbilitySystemComponent>();
        if (!ASC) return;

        // Load the AbilitySet
        UFWAbilitySet* CombatSet = LoadObject<UFWAbilitySet>(
            nullptr, TEXT("/Game/Data/AbilitySets/AS_CombatMelee.AS_CombatMelee"));

        if (CombatSet)
        {
            FFWAbilitySetHandle Handle;
            ASC->GiveAbilitySet(CombatSet, Handle);

            // Store handle to remove later if needed
            CombatSetHandle = Handle;
        }
    }
    ```

---

## Step 7: Wire Up the Combo Manager

Add a `UFWComboManagerComponent` to your character:

```cpp
ComboManager = CreateDefaultSubobject<UFWComboManagerComponent>(TEXT("ComboManager"));
```

The combo manager automatically:

1. **Tracks combo state** via replicated properties (`ComboIndex`, `bComboWindowOpened`).
2. **Responds to combo window notifies** -- when `FWComboWindowNotifyState` fires, it opens/closes the combo window.
3. **Handles combo activation** -- when the ASC receives input during an open combo window, it routes through the combo manager instead of creating a new ability activation.

### Combo Reset

The combo resets automatically when:

- The combo window closes without input.
- The ability ends or is cancelled.
- `ResetCombo()` is called explicitly.

---

## Step 8: Test the Combo Chain

1. Place your character in the level.
2. Ensure the Input Mapping Context with `IA_MeleeAttack` is active.
3. Press Play and press the attack button.
4. You should see `AM_Combo_Slash1` play.
5. During the combo window, press attack again to see `AM_Combo_Slash2`.
6. Press again during its combo window for `AM_Combo_Slash3`.
7. After the third hit, the combo resets to the beginning.

### Debug Tools

Add the debug widgets to verify combo state:

- `UFWUWDebugComboWidget` -- Shows combo index, window state, trigger state.
- `UFWUWDebugAbilityQueue` -- Shows queue state, allowed abilities, queued ability.

Use the console command `showdebug abilitysystem` to see granted abilities, active effects, and tags.

---

## Step 9: Add Damage Application

Create a gameplay effect for melee damage:

### GE_MeleeDamage

| Property | Value |
|----------|-------|
| Duration Policy | `Instant` |
| Modifiers[0].Attribute | `FWAttributeSet.Damage` |
| Modifiers[0].ModOp | `Additive` |
| Modifiers[0].Magnitude | `Scalable Float: 25.0` (or use SetByCaller) |

The `UFWAttributeSet::PostGameplayEffectExecute` automatically routes `Damage` meta attribute changes to reduce `Health`, fires `OnDamage` on the `UFWGASCoreComponent`, and triggers `OnDeath` when Health reaches zero.

---

## Step 10: GameFeature Action Setup (Optional)

For modular content, use GameFeature actions to inject the combat system.

### GameFeature Plugin Structure

```
GameFeatures/
  GF_CombatMelee/
    GF_CombatMelee.uplugin
    Content/
      GF_CombatMelee.uasset (GameFeatureData)
```

### GameFeatureData Configuration

Add three actions:

#### 1. Add Abilities

| Actor Class | Granted Ability Sets |
|-------------|---------------------|
| `AMyCharacter` | `AS_CombatMelee` |

#### 2. Add Anim Layers

Link your combat animation layer class to the character's anim instance.

#### 3. Add Input Mapping Context

| Input Mapping Context | Priority |
|----------------------|----------|
| `IMC_Combat` | `1` |

When `GF_CombatMelee` is activated (via Game Features subsystem), the character receives combat abilities, animations, and input bindings. Deactivating the feature cleanly removes everything.

!!! tip "Feature-Driven Design"
    Using GameFeature actions allows you to build combat systems, magic systems, and crafting systems as independent features that can be toggled at runtime. This is ideal for games with class switching, temporary power-ups, or modular content packs.
