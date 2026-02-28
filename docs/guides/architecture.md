---
title: Architecture Overview
---

# Architecture Overview

All FrostWeb UE5 plugins share a consistent set of architectural patterns. Understanding these patterns once gives you a foundation for working with any plugin in the library.

---

## Component-Based Design

Every plugin exposes its runtime functionality through **UActorComponent** subclasses. This means you integrate a plugin by adding a component to an actor -- no custom base classes or deep inheritance hierarchies required.

```cpp
// Example: Adding the inventory component to your character
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "FrostWeb")
UFWInventoryComponent* InventoryComponent;
```

Components are designed to be attached to any actor that needs the functionality. A player character might carry `UFWInventoryComponent`, `UFWSkillComponent`, and `UFWFactionComponent` simultaneously, while an NPC might only use `UFWAIBehaviorComponent` and `UFWDialogueComponent`.

!!! info "Why Components?"
    The component model avoids the "god class" problem common in game frameworks. Each system owns its own state and logic on the component, and cross-system communication happens through delegates and interfaces rather than direct coupling.

---

## Data-Driven via DataAssets

Game content -- items, quests, abilities, dialogue trees, factions, skills -- is defined as **UPrimaryDataAsset** subclasses. These are standalone assets that live in your Content directory and can be edited in the Unreal Editor without touching code.

```cpp
// All FrostWeb data assets inherit from UPrimaryDataAsset
UCLASS(BlueprintType)
class UFWItemDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    FText DisplayName;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    int32 MaxStackSize = 1;

    // ...
};
```

This separation of data from logic means designers can create and tune content without recompiling. The Asset Manager handles loading and referencing these assets at runtime.

!!! tip "Asset Organization"
    We recommend organizing FrostWeb data assets under a dedicated folder structure:

    ```
    Content/
        Data/
            Items/
            Quests/
            Factions/
            Skills/
            Dialogue/
    ```

---

## Network Replication

All FrostWeb plugins are built for **dedicated server multiplayer** from the ground up. The replication strategy is consistent across every plugin:

### FFastArraySerializer

Replicated arrays (inventory slots, quest progress entries, faction standings) use `FFastArraySerializer` and `FFastArraySerializerItem` for bandwidth-efficient delta serialization. Only changed elements are sent over the network, not the entire array.

```cpp
USTRUCT()
struct FFWInventoryEntry : public FFastArraySerializerItem
{
    GENERATED_BODY()

    UPROPERTY()
    UFWItemDefinition* ItemDef;

    UPROPERTY()
    int32 StackCount;

    void PreReplicatedRemove(const FFWInventoryArray& Array);
    void PostReplicatedAdd(const FFWInventoryArray& Array);
    void PostReplicatedChange(const FFWInventoryArray& Array);
};
```

### Server-Authoritative RPCs

All gameplay-affecting mutations are **Server RPCs**. The server validates every request before applying it. Clients never directly modify replicated state.

```cpp
UFUNCTION(Server, Reliable)
void ServerRequestUseItem(int32 SlotIndex);
```

### Client RPCs for Feedback

Cosmetic feedback (UI notifications, sound cues, particle effects) is delivered through **Client RPCs** or **Multicast RPCs**, keeping the visual layer separate from the authoritative game state.

---

## Event-Driven via Delegates

Cross-system communication and UI binding rely on **BlueprintAssignable** multicast delegates. Across all 11 plugins, there are over 80 bindable events.

```cpp
// Declared on the component
UPROPERTY(BlueprintAssignable, Category = "FrostWeb|Inventory")
FOnInventoryChanged OnInventoryChanged;

UPROPERTY(BlueprintAssignable, Category = "FrostWeb|Inventory")
FOnItemEquipped OnItemEquipped;
```

```cpp
// Bound in your UI or gameplay code
InventoryComponent->OnInventoryChanged.AddDynamic(
    this, &UMyWidget::HandleInventoryChanged);
```

Delegates serve as the primary mechanism for:

- **UI updates** -- Widgets bind to component delegates and refresh when state changes.
- **Cross-plugin reactions** -- One plugin can react to events from another without a compile-time dependency (bind at runtime if the component exists).
- **Audio/VFX triggers** -- Play feedback in response to gameplay events without polluting the core logic.

---

## Cross-Plugin Integration Patterns

While each plugin functions independently, several plugins provide deeper integration when used together.

### FWAISystem and FWFactionSystem

The AI system reads faction standings from `UFWFactionComponent` on both the NPC and the player to determine hostility. When a player's reputation with a faction changes, nearby NPCs re-evaluate their threat assessment automatically.

### FWGuildSystem and FWChatSystem

When both plugins are active, guild creation automatically provisions a guild chat channel, and guild membership changes are reflected in chat channel access. This integration is optional -- FWGuildSystem functions fully without FWChatSystem.

### FWInventorySystem and FWSkillSystem

Item definitions can declare skill requirements (e.g., "requires Mining level 30 to use"). When FWSkillSystem is present, `UFWInventoryComponent` validates these requirements at equip time. Without FWSkillSystem, skill requirements are ignored.

### FWQuestSystem and Other Plugins

Quest task conditions can reference state from other plugins:

- "Kill 10 enemies of faction X" (integrates with FWAISystem and FWFactionSystem)
- "Craft 5 iron swords" (integrates with FWInventorySystem)
- "Reach Mining level 20" (integrates with FWSkillSystem)

These cross-references are resolved through soft class references and runtime checks, so missing plugins do not cause compile errors.

---

## Subsystem Pattern

Plugins that manage global state expose **UGameInstanceSubsystem** or **UWorldSubsystem** classes. These subsystems provide a single point of access for state that is not tied to any individual actor.

| Subsystem | Scope | Purpose |
|---|---|---|
| `UFWFactionSubsystem` | GameInstance | Manages faction definitions and global reputation rules |
| `UFWQuestStateSubsystem` | GameInstance | Tracks quest availability, cooldowns, and daily/weekly reset timers |

Subsystems are accessed through the standard Unreal pattern:

```cpp
UFWFactionSubsystem* FactionSub = GetGameInstance()->GetSubsystem<UFWFactionSubsystem>();
```

```cpp
// In Blueprints: Get Game Instance -> Get Subsystem (select class)
```

!!! note "Subsystem Lifecycle"
    GameInstance subsystems persist across level transitions, making them suitable for data that must survive map travel (faction definitions, quest cooldowns). Per-actor state (inventory contents, skill XP) lives on the component and is serialized/deserialized during level transfers by the owning system.

---

## Summary

| Pattern | Mechanism | Purpose |
|---|---|---|
| Component-based | `UActorComponent` subclasses | Attach functionality to any actor without inheritance |
| Data-driven | `UPrimaryDataAsset` subclasses | Define content in the editor, not in code |
| Replicated | `FFastArraySerializer`, Server/Client RPCs | Efficient, server-authoritative multiplayer |
| Event-driven | `BlueprintAssignable` delegates (80+) | Decouple systems, drive UI, trigger feedback |
| Cross-plugin | Soft references, runtime checks | Optional integration without compile-time coupling |
| Global state | `UGameInstanceSubsystem` / `UWorldSubsystem` | Centralized access to singleton data |

For details on any specific plugin's architecture, visit that plugin's documentation page from the [Plugins Overview](../plugins/index.md).
