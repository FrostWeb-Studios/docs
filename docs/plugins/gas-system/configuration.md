---
title: Configuration - GAS System
---

# Configuration

FWGASSystem is configured through **component properties**, **AbilitySet DataAssets**, **Developer Settings**, and **GameFeature actions**.

---

## Ability System Component Configuration

The `UFWAbilitySystemComponent` is the central configuration point. Set these properties in the Blueprint Details panel or in C++ constructors.

### Granted Abilities

Each entry is an `FFWAbilityInputMapping`:

| Field | Type | Notes |
|-------|------|-------|
| `Ability` | `TSubclassOf<UGameplayAbility>` | The ability class to grant. |
| `Level` | `int32` | Defaults to 1. |
| `InputAction` | `UInputAction*` | Leave null for abilities that are not input-triggered. |
| `TriggerEvent` | `EFWAbilityTriggerEvent` | `Started` (default) or `Triggered`. Only visible when InputAction is set. |

### Granted Attributes

Each entry is an `FFWAttributeSetDefinition`:

| Field | Type | Notes |
|-------|------|-------|
| `AttributeSet` | `TSubclassOf<UAttributeSet>` | The attribute set class. |
| `InitializationData` | `UDataTable*` | Row structure must be `AttributeMetaData`. Optional. |

!!! tip "Initialization DataTable Format"
    Create a DataTable with the `AttributeMetaData` row structure. Each row name should be the attribute name (e.g., `Health`). Set `BaseValue` to the desired initial value.

### Granted Effects

A simple array of `TSubclassOf<UGameplayEffect>`. These are applied once during `InitAbilityActorInfo`. Common uses:

- Passive regeneration effects (Health/Stamina/Mana regen)
- Base stat modifiers
- Starting buffs

### Granted Ability Sets

Soft references to `UFWAbilitySet` DataAssets. Using AbilitySets is the recommended approach for organizing abilities into logical groups.

### Spawn Behavior

| Property | Default | Description |
|----------|---------|-------------|
| `bResetAbilitiesOnSpawn` | `true` | Re-grant abilities on each `InitAbilityActorInfo`. |
| `bResetAttributesOnSpawn` | `true` | Re-initialize attribute values on each `InitAbilityActorInfo`. |

!!! warning "Player State ASC"
    When the ASC lives on a Player State (using `AFWModularPlayerState`):

    - Set `bResetAbilitiesOnSpawn` to `false` to prevent re-granting abilities on respawn.
    - Set `bResetAttributesOnSpawn` to `false` to preserve attribute values across respawns.
    - Do **not** set `bResetAbilitiesOnSpawn` to `true` if using input binding -- it causes duplicate bindings.

---

## AbilitySet DataAssets

`UFWAbilitySet` is a `UPrimaryDataAsset` that bundles related abilities, attributes, effects, and tags.

### Creating an AbilitySet

1. Right-click in the Content Browser.
2. Select **Miscellaneous > Data Asset**.
3. Choose `FWAbilitySet` as the class.
4. Name it descriptively (e.g., `AS_CombatMelee`, `AS_MagicFire`, `AS_PassiveRegen`).

### AbilitySet Properties

#### Granted Abilities

Each entry is an `FFWGameFeatureAbilityMapping`:

| Field | Type | Description |
|-------|------|-------------|
| `AbilityType` | `TSubclassOf<UGameplayAbility>` | Ability class. |
| `Level` | `int32` | Ability level. |
| `InputAction` | `UInputAction*` | Input binding (optional). |
| `TriggerEvent` | `EFWAbilityTriggerEvent` | Input trigger type. |

#### Granted Attributes

Each entry is an `FFWGameFeatureAttributeSetMapping`:

| Field | Type | Description |
|-------|------|-------------|
| `AttributeSet` | `TSubclassOf<UAttributeSet>` | Attribute set class. |
| `InitializationData` | `UDataTable*` | Initialization DataTable (optional). |

#### Granted Effects

Each entry is an `FFWGameFeatureGameplayEffectMapping`:

| Field | Type | Description |
|-------|------|-------------|
| `EffectType` | `TSubclassOf<UGameplayEffect>` | Effect class. |
| `Level` | `float` | Effect level. |

#### Owned Tags

A `FGameplayTagContainer` applied to the ASC when the set is granted and removed when the set is cleared.

### AbilitySet Lifecycle

```
Grant:
  AbilitySet.GrantToAbilitySystem(ASC, OutHandle)
    -> Creates abilities, attributes, effects
    -> Applies owned tags
    -> Returns FFWAbilitySetHandle

Remove:
  UFWAbilitySet::RemoveFromAbilitySystem(ASC, Handle)
    -> Removes abilities by handle
    -> Removes effects by handle
    -> Removes attributes
    -> Removes owned tags
    -> Invalidates handle
```

!!! info "Runtime Granting"
    AbilitySets can be granted and removed at runtime, making them ideal for equipment-based ability systems, skill trees, or class switching. The handle-based lifecycle ensures clean teardown.

---

## Effect Container Configuration

Configure effect containers on `UFWGameplayAbility` subclasses via the `EffectContainerMap`.

### EffectContainerMap

A `TMap<FGameplayTag, FFWGameplayEffectContainer>`. Each entry maps a tag key to a container:

| Field | Description |
|-------|-------------|
| **Key** | A `FGameplayTag` used to look up the container (e.g., `Ability.Attack.Hit`). |
| **TargetType** | A `UFWTargetType` subclass defining how targets are resolved. |
| **TargetGameplayEffectClasses** | Array of `UGameplayEffect` classes to apply to resolved targets. |
| **bUseSetByCallerMagnitude** | Whether to pass a magnitude via SetByCaller. |
| **SetByCallerDataTag** | The tag used for SetByCaller lookup. |
| **SetByCallerMagnitude** | The default magnitude value. |

**Usage in ability code:**

```cpp
// Look up container by tag and apply
ApplyEffectContainer(
    FGameplayTag::RequestGameplayTag(FName("Ability.Attack.Hit")),
    EventData
);
```

---

## GameFeature Action Configuration

### Add Abilities Action

Configure in a GameFeature plugin's `GameFeatureData` asset:

1. Add an action of type **Add Abilities (FW GAS System)**.
2. For each entry in `AbilitiesList`:
    - Set `ActorClass` to the target actor (e.g., your character class).
    - Add abilities, attributes, effects, and ability sets.

When the GameFeature activates, these are automatically granted to matching actors. When deactivated, they are cleanly removed.

### Add Anim Layers Action

Links animation layer interfaces to characters when the GameFeature activates. Configure the anim class containing the layer implementations.

### Add Input Mapping Context Action

Adds an Input Mapping Context to local players. Configure the IMC asset and priority.

---

## Developer Settings

Navigate to **Project Settings > Game > FW GAS System** (if developer settings are registered).

Settings exposed through `UFWGASDeveloperSettings`:

| Setting | Description |
|---------|-------------|
| Project-specific GAS configuration | Varies by project setup |

---

## Attribute Set Configuration

### Default Attribute Set (UFWAttributeSet)

The built-in attribute set includes:

| Category | Attributes |
|----------|-----------|
| **Health** | Health, MaxHealth, HealthRegenRate |
| **Stamina** | Stamina, MaxStamina, StaminaRegenRate |
| **Mana** | Mana, MaxMana, ManaRegenRate |
| **Meta (server-only)** | Damage, StaminaDamage |

### Creating Custom Attribute Sets

Inherit from `UFWAttributeSetBase` to get:

- Automatic `PreAttributeChange` / `PostGameplayEffectExecute` routing to the `UFWGASCoreComponent`.
- `AdjustAttributeForMaxChange` helper for proportional attribute scaling.
- `GetClampMinimumValueFor` virtual for per-attribute minimum values.
- `ATTRIBUTE_ACCESSORS` macro for boilerplate-free getter/setter definitions.

```cpp
UCLASS()
class UMyCraftingAttributeSet : public UFWAttributeSetBase
{
    GENERATED_BODY()

public:
    UPROPERTY(BlueprintReadOnly, Category = "Crafting", ReplicatedUsing = OnRep_CraftingSkill)
    FGameplayAttributeData CraftingSkill;
    ATTRIBUTE_ACCESSORS(UMyCraftingAttributeSet, CraftingSkill)

protected:
    UFUNCTION()
    void OnRep_CraftingSkill(const FGameplayAttributeData& OldValue);
};
```

---

## Component Placement Patterns

### Pawn-Based ASC

The ASC lives on the character (pawn). Simplest setup.

```
AMyCharacter
 +-- UFWAbilitySystemComponent
 +-- UFWGASCoreComponent
 +-- UFWAbilityInputBindingComponent
 +-- UFWComboManagerComponent (optional)
 +-- UFWAbilityQueueComponent (optional)
```

- Set `bResetAbilitiesOnSpawn = true` (default).
- ASC is destroyed/recreated on pawn destruction.

### Player State-Based ASC

The ASC lives on the Player State, persisting across pawn respawns.

```
AMyPlayerState (extends AFWModularPlayerState)
 +-- UFWAbilitySystemComponent
     bResetAbilitiesOnSpawn = false
     bResetAttributesOnSpawn = false

AMyCharacter (extends AFWModularPlayerStateCharacter)
 +-- UFWGASCoreComponent
 +-- UFWAbilityInputBindingComponent
 +-- UFWComboManagerComponent (optional)
 +-- UFWAbilityQueueComponent (optional)
```

- Attributes and abilities persist across respawns.
- `InitAbilityActorInfo` is called on possession, re-linking the ASC to the new pawn.
