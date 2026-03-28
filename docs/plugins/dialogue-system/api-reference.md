---
title: API Reference - FWDialogueSystem
description: Complete function, property, event, and type reference for FWDialogueSystem.
---

# API Reference

Complete reference for all classes, functions, events, and types in FWDialogueSystem.

---

## UFWDialogueComponent

**Parent:** `UActorComponent`

The core runtime component that manages dialogue state. Attach it to an NPC Actor to give it conversation capabilities. It tracks the active dialogue tree, current node, interacting actor, and handles node transitions.

### Functions

| Function | Parameters | Return | Description |
|---|---|---|---|
| `StartDialogue` | `Tree, Interactor` | `void` | Begins a conversation using the specified dialogue tree. Sets the current node to the root node (NodeId 0), stores the interactor reference, and broadcasts `OnDialogueStarted` followed by `OnNodeChanged`. |
| `SelectResponse` | `ResponseIndex` | `void` | Selects a player response by index from the filtered response list. Executes response actions, then advances to the next node. If `NextNodeId` is `-1`, the dialogue ends. |
| `EndDialogue` | -- | `void` | Ends the current conversation immediately. Clears the active tree, current node, and interactor reference. |
| `GetCurrentNode` | -- | `FFWDialogueNode` | Returns the current dialogue node (speaker, text, responses). |
| `GetAvailableResponses` | -- | `Array of Responses` | Returns only responses whose conditions all pass for the current interactor. Use this to populate UI. |
| `IsDialogueActive` | -- | `bool` | Whether a conversation is currently in progress. |
| `GetCurrentInteractor` | -- | `Actor` | The actor that initiated the current conversation (null if inactive). |

!!! warning "Single Conversation"
    A dialogue component can only run one conversation at a time. Calling `StartDialogue` while a dialogue is already active will end the current dialogue first.

!!! note "Response Indexing"
    The `ResponseIndex` in `SelectResponse` refers to the filtered response list returned by `GetAvailableResponses`, not the raw response array on the node. Always use `GetAvailableResponses` to map UI buttons to indices.

### Properties

| Property | Type | Description |
|---|---|---|
| `DefaultTree` | `UFWDialogueTree` | Optional default dialogue tree to use when no tree is specified |

### Events

All events are `BlueprintAssignable` and can be bound in both C++ and Blueprints.

| Delegate | Signature | Description |
|---|---|---|
| `OnDialogueStarted` | `(Tree, Interactor)` | Fired when a conversation begins |
| `OnDialogueEnded` | `()` | Fired when a conversation ends |
| `OnNodeChanged` | `(NewNode)` | Fired when the current node changes (including the initial node) |

---

## Data Assets

### UFWDialogueTree

**Parent:** `UPrimaryDataAsset`

The root data asset containing all dialogue nodes for a conversation. Each tree represents one complete conversation flow.

| Property | Type | Description |
|---|---|---|
| `Nodes` | `Array of FFWDialogueNode` | All dialogue nodes in this tree |

!!! tip "Root Node"
    The node with `NodeId = 0` is used as the entry point when `StartDialogue` is called. Always ensure your tree has a node with ID 0.

---

## Core Types

### FFWDialogueNode

A single node in the dialogue tree.

| Field | Type | Description |
|---|---|---|
| `NodeId` | `int32` | Unique identifier for this node within the tree |
| `SpeakerName` | `FText` | Display name of the speaking character |
| `DialogueText` | `FText` | The dialogue text to display (supports multiline) |
| `EntryActions` | `Array of Actions` | Actions executed when this node becomes active |
| `Responses` | `Array of FFWDialogueResponse` | Player response options |

### FFWDialogueResponse

A single player response option within a dialogue node.

| Field | Type | Description |
|---|---|---|
| `ResponseText` | `FText` | The text displayed on the response button |
| `NextNodeId` | `int32` | The NodeId to advance to, or `-1` to end the dialogue |
| `Conditions` | `Array of Conditions` | All conditions must pass for this response to appear |
| `Actions` | `Array of Actions` | Actions executed when the player selects this response |

!!! info "Condition Logic"
    Multiple conditions on a single response use AND logic -- all must return true. For OR logic, create separate responses that lead to the same next node.

---

## Condition Classes

All conditions extend `UFWDialogueConditionBase` and implement a single evaluation method that returns `true` if the condition is met.

### HasItem

Checks whether the interactor possesses a specific item.

| Property | Type | Description |
|---|---|---|
| `ItemId` | `FName` | The identifier of the required item |
| `RequiredCount` | `int32` | Minimum quantity required (default: 1) |

### HasCurrency

Checks whether the interactor has a minimum amount of a currency type.

| Property | Type | Description |
|---|---|---|
| `CurrencyType` | `FName` | The type of currency to check |
| `MinAmount` | `int32` | Minimum amount required |

### QuestState

Checks the state of a quest for the interactor.

| Property | Type | Description |
|---|---|---|
| `QuestId` | `FName` | The identifier of the quest |
| `RequiredState` | `EFWQuestState` | Required state (NotStarted, InProgress, Completed) |

### SkillLevel

Checks whether the interactor meets a minimum skill level.

| Property | Type | Description |
|---|---|---|
| `SkillId` | `FName` | The identifier of the skill to check |
| `MinLevel` | `int32` | Minimum skill level required |

### Custom Conditions

Create custom conditions by subclassing `UFWDialogueConditionBase` and overriding the `Evaluate` function in C++ or Blueprints.

---

## Action Classes

All actions extend `UFWDialogueActionBase` and implement a single execution method that applies the action's effect.

### GiveItem

Grants an item to the interactor.

| Property | Type | Description |
|---|---|---|
| `ItemId` | `FName` | The identifier of the item to give |
| `Quantity` | `int32` | Number of items to grant (default: 1) |

### GiveCurrency

Grants currency to the interactor.

| Property | Type | Description |
|---|---|---|
| `CurrencyType` | `FName` | The type of currency to grant |
| `Amount` | `int32` | Amount of currency to grant |

### StartQuest

Starts a quest for the interactor.

| Property | Type | Description |
|---|---|---|
| `QuestId` | `FName` | The identifier of the quest to start |

### OpenVendor

Opens the vendor/shop interface for the interactor.

| Property | Type | Description |
|---|---|---|
| `VendorId` | `FName` | Identifier of the vendor inventory to display |

### Custom Actions

Create custom actions by subclassing `UFWDialogueActionBase` and overriding the `Execute` function in C++ or Blueprints.

---

## Currency Integration

FWDialogueSystem integrates with your game's currency system through the built-in `HasCurrency` condition and `GiveCurrency` action. These use a `CurrencyType` identifier (FName) that maps to your project's currency types. Implement the `IFWCurrencyProvider` interface on your player character or controller to connect these to your currency backend.
