---
title: API Reference - FWDialogueSystem
---

# API Reference

Complete reference for all classes, functions, events, and types in FWDialogueSystem.

---

## UFWDialogueComponent

**Parent Class:** `UActorComponent`
**Specifiers:** `BlueprintSpawnableComponent`

The core runtime component that manages dialogue state. Attach it to an NPC Actor to give it conversation capabilities. It tracks the active dialogue tree, current node, interacting actor, and handles node transitions.

---

### BlueprintCallable Functions

#### StartDialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Dialogue")
void StartDialogue(UFWDialogueTree* Tree, AActor* Interactor);
```

Begins a conversation using the specified dialogue tree. Sets the current node to the tree's root node, stores the interactor reference, and broadcasts `OnDialogueStarted` followed by `OnNodeChanged`.

| Parameter | Type | Description |
|---|---|---|
| `Tree` | `UFWDialogueTree*` | The dialogue tree data asset to use |
| `Interactor` | `AActor*` | The actor initiating the conversation (typically the player character) |

!!! warning "Single Conversation"
    A dialogue component can only run one conversation at a time. Calling `StartDialogue` while a dialogue is already active will end the current dialogue first.

---

#### SelectResponse

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Dialogue")
void SelectResponse(int32 ResponseIndex);
```

Selects a player response by index from the current node's available responses. This executes any actions attached to the response, then advances to the next node specified by `NextNodeId`. If `NextNodeId` is `-1`, the dialogue ends automatically.

| Parameter | Type | Description |
|---|---|---|
| `ResponseIndex` | `int32` | Zero-based index into the array returned by `GetAvailableResponses` |

!!! note "Condition Filtering"
    The index refers to the filtered response list (responses whose conditions all pass), not the raw response array in the dialogue node. Always use the array from `GetAvailableResponses` to map UI buttons to indices.

---

#### EndDialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Dialogue")
void EndDialogue();
```

Ends the current conversation immediately. Clears the active tree, current node, and interactor reference. Broadcasts `OnDialogueEnded`.

---

### BlueprintPure Functions

#### GetCurrentNode

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Dialogue")
FFWDialogueNode GetCurrentNode() const;
```

Returns the current dialogue node, including speaker name, dialogue text, and the full response array (before condition filtering).

**Returns:** `FFWDialogueNode` -- The active node, or an empty node if no dialogue is active.

---

#### GetAvailableResponses

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Dialogue")
TArray<FFWDialogueResponse> GetAvailableResponses() const;
```

Returns only the responses from the current node whose conditions all evaluate to true for the current interactor. Use this to populate your UI response list.

**Returns:** `TArray<FFWDialogueResponse>` -- Filtered responses available to the current interactor.

!!! info "Condition Evaluation"
    Conditions are evaluated at the moment this function is called. If game state changes while the node is displayed (e.g., the player gains an item through another system), calling this function again will return an updated list.

---

#### IsDialogueActive

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Dialogue")
bool IsDialogueActive() const;
```

Returns whether a conversation is currently in progress.

**Returns:** `bool` -- `true` if a dialogue tree is loaded and the component is actively in a conversation.

---

#### GetCurrentInteractor

```cpp
UFUNCTION(BlueprintPure, Category = "FW|Dialogue")
AActor* GetCurrentInteractor() const;
```

Returns the actor that initiated the current conversation.

**Returns:** `AActor*` -- The interacting actor, or `nullptr` if no dialogue is active.

---

### Events (Delegates)

All events are `BlueprintAssignable` and can be bound in both C++ and Blueprints.

| Event | Signature | Description |
|---|---|---|
| `OnDialogueStarted` | `(UFWDialogueTree* Tree, AActor* Interactor)` | Fired when a conversation begins |
| `OnDialogueEnded` | `()` | Fired when a conversation ends (by reaching a terminal response or calling EndDialogue) |
| `OnNodeChanged` | `(const FFWDialogueNode& NewNode)` | Fired when the current node changes (including the initial node on StartDialogue) |

---

## UFWDialogueTree

**Parent Class:** `UPrimaryDataAsset`

The root data asset containing all dialogue nodes for a conversation. Each tree represents one complete conversation flow.

### Properties

| Property | Type | Description |
|---|---|---|
| `Nodes` | `TArray<FFWDialogueNode>` | All dialogue nodes in this tree |

The component looks up nodes by `NodeId`. Node order in the array does not matter -- nodes are resolved by their ID field.

!!! tip "Root Node"
    The node with `NodeId = 0` is used as the entry point when `StartDialogue` is called. Always ensure your tree has a node with ID 0.

---

## Condition Classes

All conditions extend `UFWDialogueConditionBase` and implement a single evaluation method:

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Dialogue|Condition")
virtual bool Evaluate(AActor* Interactor) const;
```

Returns `true` if the condition is met for the given interactor.

---

### UFWDialogueConditionBase

**Parent Class:** `UObject`

Abstract base class for all dialogue conditions. Subclass this to create custom conditions.

```cpp
// Custom condition example
UCLASS()
class UFWDlgCondition_PlayerLevel : public UFWDialogueConditionBase
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 MinimumLevel = 1;

    virtual bool Evaluate(AActor* Interactor) const override;
};
```

---

### FWDlgCondition_HasItem

Checks whether the interactor possesses a specific item.

| Property | Type | Description |
|---|---|---|
| `ItemId` | `FName` | The identifier of the required item |
| `RequiredCount` | `int32` | Minimum quantity the interactor must have (default: 1) |

---

### FWDlgCondition_SkillLevel

Checks whether the interactor has a minimum level in a specific skill.

| Property | Type | Description |
|---|---|---|
| `SkillId` | `FName` | The identifier of the skill to check |
| `MinLevel` | `int32` | Minimum skill level required |

---

### FWDlgCondition_QuestState

Checks the state of a quest for the interactor.

| Property | Type | Description |
|---|---|---|
| `QuestId` | `FName` | The identifier of the quest |
| `RequiredState` | `EFWQuestState` | The state the quest must be in (NotStarted, InProgress, Completed) |

---

### FWDlgCondition_HasCurrency

Checks whether the interactor has a minimum amount of a currency type.

| Property | Type | Description |
|---|---|---|
| `CurrencyType` | `FName` | The type of currency to check |
| `MinAmount` | `int32` | Minimum amount required |

---

## Action Classes

All actions extend `UFWDialogueActionBase` and implement a single execution method:

```cpp
UFUNCTION(BlueprintCallable, Category = "FW|Dialogue|Action")
virtual void Execute(AActor* Interactor) const;
```

Performs the action's effect on the given interactor.

---

### UFWDialogueActionBase

**Parent Class:** `UObject`

Abstract base class for all dialogue actions. Subclass this to create custom actions.

```cpp
// Custom action example
UCLASS()
class UFWDlgAction_PlayAnimation : public UFWDialogueActionBase
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    UAnimMontage* Montage;

    virtual void Execute(AActor* Interactor) const override;
};
```

---

### FWDlgAction_OpenVendor

Opens the vendor/shop interface for the interactor.

| Property | Type | Description |
|---|---|---|
| `VendorId` | `FName` | Identifier of the vendor inventory to display |

---

### FWDlgAction_GiveItem

Grants an item to the interactor.

| Property | Type | Description |
|---|---|---|
| `ItemId` | `FName` | The identifier of the item to give |
| `Quantity` | `int32` | Number of items to grant (default: 1) |

---

### FWDlgAction_StartQuest

Starts a quest for the interactor.

| Property | Type | Description |
|---|---|---|
| `QuestId` | `FName` | The identifier of the quest to start |

---

### FWDlgAction_GiveCurrency

Grants currency to the interactor.

| Property | Type | Description |
|---|---|---|
| `CurrencyType` | `FName` | The type of currency to grant |
| `Amount` | `int32` | Amount of currency to grant |

---

## Types

### FFWDialogueNode

```cpp
USTRUCT(BlueprintType)
struct FFWDialogueNode
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 NodeId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FText SpeakerName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, meta = (MultiLine = true))
    FText DialogueText;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Instanced)
    TArray<UFWDialogueActionBase*> EntryActions;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TArray<FFWDialogueResponse> Responses;
};
```

A single node in the dialogue tree. Contains the speaker, the text to display, optional entry actions that fire when the node is entered, and an array of player responses.

| Field | Type | Description |
|---|---|---|
| `NodeId` | `int32` | Unique identifier for this node within the tree |
| `SpeakerName` | `FText` | Display name of the character speaking |
| `DialogueText` | `FText` | The dialogue text to display (supports multiline) |
| `EntryActions` | `TArray<UFWDialogueActionBase*>` | Actions executed when this node becomes active |
| `Responses` | `TArray<FFWDialogueResponse>` | Player response options |

---

### FFWDialogueResponse

```cpp
USTRUCT(BlueprintType)
struct FFWDialogueResponse
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FText ResponseText;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 NextNodeId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Instanced)
    TArray<UFWDialogueConditionBase*> Conditions;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Instanced)
    TArray<UFWDialogueActionBase*> Actions;
};
```

A single player response option within a dialogue node.

| Field | Type | Description |
|---|---|---|
| `ResponseText` | `FText` | The text displayed on the response button |
| `NextNodeId` | `int32` | The NodeId to advance to, or `-1` to end the dialogue |
| `Conditions` | `TArray<UFWDialogueConditionBase*>` | All conditions must pass for this response to appear |
| `Actions` | `TArray<UFWDialogueActionBase*>` | Actions executed when the player selects this response |

!!! info "Condition Logic"
    Multiple conditions on a single response are evaluated with AND logic -- all must return `true` for the response to be available. For OR logic, create separate responses that lead to the same next node.
