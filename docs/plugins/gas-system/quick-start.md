---
title: Quick Start - GAS System
---

# Quick Start

This guide gets you from zero to a working ability system in under 10 minutes. By the end, you will have a character with health and stamina attributes, a basic melee ability, and input binding through Enhanced Input.

---

## 1. Set Up Your Character

Use one of the provided modular base classes, or add the components to your existing character.

=== "Using Modular Character"

    Inherit your character from `AFWModularCharacter`:

    ```cpp
    #pragma once
    #include "ModularGameplayActors/FWModularCharacter.h"
    #include "MyCharacter.generated.h"

    UCLASS()
    class AMyCharacter : public AFWModularCharacter
    {
        GENERATED_BODY()
    };
    ```

=== "Using Existing Character"

    Add the required components to your existing character:

    ```cpp
    AMyCharacter::AMyCharacter()
    {
        // Ability System Component
        AbilitySystemComponent = CreateDefaultSubobject<UFWAbilitySystemComponent>(
            TEXT("AbilitySystemComponent"));

        // Core Component (event bridge)
        GASCoreComponent = CreateDefaultSubobject<UFWGASCoreComponent>(
            TEXT("GASCoreComponent"));

        // Input Binding (for EnhancedInput integration)
        AbilityInputBinding = CreateDefaultSubobject<UFWAbilityInputBindingComponent>(
            TEXT("AbilityInputBinding"));
    }
    ```

---

## 2. Configure Attributes

On your character Blueprint (or in the C++ `GrantedAttributes` array on the ASC), add the default attribute set:

1. Select the **FW Ability System Component** in the Components panel.
2. Under **Granted Attributes**, add an entry:
    - **Attribute Set:** `FWAttributeSet`
    - **Initialization Data:** (optional) Point to a DataTable with initial values.

If you do not provide a DataTable, attributes start at `0.0`. To set initial values via a DataTable:

1. Create a DataTable with Row Structure `AttributeMetaData`.
2. Add rows for each attribute (e.g., `FWAttributeSet.Health`, `FWAttributeSet.MaxHealth`).
3. Set `BaseValue` for each row.

**Example DataTable rows:**

| Row Name | Attribute | BaseValue |
|----------|-----------|-----------|
| Health | `FWAttributeSet.Health` | `100.0` |
| MaxHealth | `FWAttributeSet.MaxHealth` | `100.0` |
| Stamina | `FWAttributeSet.Stamina` | `50.0` |
| MaxStamina | `FWAttributeSet.MaxStamina` | `50.0` |
| HealthRegenRate | `FWAttributeSet.HealthRegenRate` | `1.0` |
| StaminaRegenRate | `FWAttributeSet.StaminaRegenRate` | `2.0` |

---

## 3. Create a Gameplay Ability

Create a simple melee attack ability:

```cpp
#pragma once
#include "Abilities/FWGameplayAbility.h"
#include "GA_MeleeAttack.generated.h"

UCLASS()
class UGA_MeleeAttack : public UFWGameplayAbility
{
    GENERATED_BODY()

public:
    UGA_MeleeAttack();

    virtual void ActivateAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo,
        const FGameplayEventData* TriggerEventData) override;
};
```

```cpp
#include "GA_MeleeAttack.h"

UGA_MeleeAttack::UGA_MeleeAttack()
{
    InstancingPolicy = EGameplayAbilityInstancingPolicy::InstancedPerActor;
    NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UGA_MeleeAttack::ActivateAbility(
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

    // Play a montage, apply effects, etc.
    // EndAbility when done
}
```

---

## 4. Grant the Ability with Input Binding

On the **FW Ability System Component**, add an entry to **Granted Abilities**:

| Property | Value |
|----------|-------|
| Ability | `GA_MeleeAttack` |
| Level | `1` |
| Input Action | Your `IA_Attack` Input Action asset |
| Trigger Event | `Started` |

The ASC will automatically grant this ability and bind it to the specified Input Action when `InitAbilityActorInfo` is called.

!!! info "Input Action Setup"
    Create an `UInputAction` asset (e.g., `IA_Attack`) and add it to your Input Mapping Context. The `UFWAbilityInputBindingComponent` handles the bridge between Enhanced Input and GAS ability activation.

---

## 5. Add the Core Component for Events

Add a `UFWGASCoreComponent` to your character (or it may already be present if using `AFWModularCharacter`). This component provides Blueprint-assignable events for common GAS scenarios:

In your character Blueprint's Event Graph, bind to:

- **On Health Change** -- Update health bar UI.
- **On Stamina Change** -- Update stamina bar UI.
- **On Ability Activated** -- Play activation effects.
- **On Death** -- Trigger death animation.

---

## 6. Test It

1. Place your character in a level.
2. Ensure an Input Mapping Context with `IA_Attack` is active.
3. Press Play and press the attack input.
4. The `GA_MeleeAttack` ability should activate.

!!! tip "Debug Commands"
    Use `showdebug abilitysystem` in the console to see granted abilities, active effects, and attribute values in real time.

---

## Next Steps

- [Configuration](configuration.md) -- Learn about AbilitySet DataAssets and developer settings.
- [API Reference](api-reference.md) -- Full documentation of all classes.
- [Tutorials](tutorials.md) -- Build a complete combat ability with combo chains.
