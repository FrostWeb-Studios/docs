---
title: Migration Guide - GAS System
---

# Migration Guide

This page documents breaking changes and migration steps between versions of FWGASSystem.

---

## Version 1.0 (Current)

This is the initial release. No migration steps are required.

---

## Engine Version Compatibility

FWGASSystem includes engine version checks for API differences between UE versions:

```cpp
#if ENGINE_MAJOR_VERSION == 5 && ENGINE_MINOR_VERSION >= 3
    virtual EDataValidationResult IsDataValid(class FDataValidationContext& Context) const override;
#else
    virtual EDataValidationResult IsDataValid(FDataValidationContext& Context) override;
#endif
```

When upgrading your project's engine version, the plugin handles these differences automatically. No manual changes are required.

---

## Migrating from Raw GAS to FWGASSystem

If you have an existing project using raw Gameplay Ability System and want to adopt FWGASSystem, follow these steps:

### Step 1: Replace the AbilitySystemComponent

=== "Before"

    ```cpp
    AbilitySystem = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("ASC"));
    ```

=== "After"

    ```cpp
    AbilitySystem = CreateDefaultSubobject<UFWAbilitySystemComponent>(TEXT("ASC"));
    ```

Move your manual ability granting code into the `GrantedAbilities` array on the component, or into AbilitySet DataAssets.

### Step 2: Add the Core Component

Add `UFWGASCoreComponent` to your character. Replace any manual attribute change delegates with the core component's events:

=== "Before"

    ```cpp
    ASC->GetGameplayAttributeValueChangeDelegate(
        UFWAttributeSet::GetHealthAttribute())
        .AddUObject(this, &AMyChar::OnHealthChanged);
    ```

=== "After"

    ```cpp
    // In Blueprint: bind to OnHealthChange on the GAS Core Component
    // Or in C++:
    GASCoreComponent->OnHealthChange.AddDynamic(this, &AMyChar::HandleHealthChange);
    ```

### Step 3: Replace Manual Input Binding

If you were manually binding Input Actions to ability activation:

=== "Before"

    ```cpp
    void AMyChar::OnAttackPressed()
    {
        ASC->TryActivateAbilityByClass(UGA_Attack::StaticClass());
    }
    ```

=== "After"

    Add `UFWAbilityInputBindingComponent` to your character and configure the `InputAction` field on the ability's `FFWAbilityInputMapping` entry. Input binding is handled automatically.

### Step 4: Convert to AbilitySets (Optional)

Move groups of related abilities from per-component arrays to `UFWAbilitySet` DataAssets for better organization and reuse.

### Step 5: Adopt Modular Base Classes (Optional)

Replace your character/controller/game mode base classes with the FW modular variants to enable GameFeature action support.

---

## Future Migration Notes

When upgrading between major versions, this page will document:

- **Breaking API changes** -- renamed or removed functions, changed signatures.
- **Component changes** -- new required components, deprecated components.
- **AbilitySet schema changes** -- new fields, deprecated properties.
- **Replication changes** -- modifications to replicated properties.
- **Step-by-step upgrade instructions** with before/after code examples.

!!! tip "Before Upgrading"
    Always follow these steps when upgrading the plugin:

    1. Back up your project (source control recommended).
    2. Read the [Changelog](changelog.md) for the target version.
    3. Review this migration guide for breaking changes.
    4. Update the plugin files.
    5. Regenerate project files (right-click `.uproject` > Generate Project Files).
    6. Build and resolve any compile errors using this guide.
    7. Test ability granting, input binding, combos, and replication in a PIE session.
