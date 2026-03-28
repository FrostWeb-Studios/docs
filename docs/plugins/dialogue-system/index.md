---
title: FWDialogueSystem Plugin
description: Data-driven dialogue tree system with conditions, actions, branching responses, and NPC conversation management for Unreal Engine 5.
tags:
  - plugin
  - dialogue
  - npc
  - rpg
---

# FWDialogueSystem Plugin

**Version:** 1.0 | **Module:** `FWDialogueSystem` | **Type:** Runtime | **No external dependencies**

A data-driven dialogue tree system for Unreal Engine 5 with conditions, actions, and NPC conversation support. FWDialogueSystem provides a complete framework for creating branching dialogues where player choices can be gated by game state and trigger gameplay effects.

---

## Feature Overview

| Feature | Description |
|---|---|
| Data-Driven Trees | All dialogue content is defined in `UFWDialogueTree` data assets |
| Branching Responses | Each node can present multiple player responses leading to different paths |
| Condition System | Gate responses behind game state checks (items, skills, quests, currency) |
| Action System | Trigger gameplay effects from dialogue (give items, start quests, open vendors, grant currency) |
| NPC Conversation Management | Full conversation lifecycle with interactor tracking and state management |
| Currency Integration | Built-in support for currency checks and grants within dialogue |
| Blueprint-Friendly API | Start, navigate, and end dialogues entirely from Blueprints |
| Extensible | Create custom conditions and actions by subclassing base classes |

---

## Core Concepts

### Dialogue Trees

A `UFWDialogueTree` is a data asset containing an array of `FFWDialogueNode` entries. Each node has a speaker name, dialogue text, optional entry actions, and an array of player responses. The node with `NodeId = 0` is the conversation entry point.

### Responses and Branching

Each `FFWDialogueResponse` contains response text, a `NextNodeId` to advance to, optional conditions that must pass for the response to appear, and optional actions that execute when selected. Setting `NextNodeId` to `-1` ends the dialogue.

### Condition Gating

Conditions are evaluated when the player views a node. Only responses whose conditions all pass are shown. If game state changes while a node is displayed, the available responses update on the next query.

### Action Execution

Actions fire at two points: **entry actions** execute when a node becomes active, and **response actions** execute when the player selects a response. This enables both passive effects (entering a room triggers an event) and player-driven effects (choosing to accept a quest).

---

## Quick Start

### 1. Enable the Plugin

Enable `FWDialogueSystem` in your `.uproject` file or Plugin Manager. No additional dependencies are required.

### 2. Create a Dialogue Tree

1. Right-click in the Content Browser.
2. Select **Miscellaneous > Data Asset**.
3. Choose `FWDialogueTree` as the class.
4. Add nodes with speaker names, dialogue text, and responses.

### 3. Add the Dialogue Component

=== "C++"

    ```cpp
    // Add to your NPC Actor
    DialogueComp = CreateDefaultSubobject<UFWDialogueComponent>(TEXT("Dialogue"));
    ```

=== "Blueprint"

    Add **FW Dialogue Component** to your NPC Actor.

### 4. Start a Conversation

```cpp
DialogueComp->StartDialogue(MyDialogueTree, PlayerCharacter);
```

### 5. Wire Up Your UI

```cpp
// Bind to node changes for UI updates
DialogueComp->OnNodeChanged.AddDynamic(this, &AMyHUD::HandleNodeChanged);

// Get filtered responses for display
TArray<FFWDialogueResponse> Responses = DialogueComp->GetAvailableResponses();

// When player clicks a response button
DialogueComp->SelectResponse(ChosenIndex);
```

---

## Built-In Conditions

| Condition | Description |
|---|---|
| **HasItem** | Checks if the interactor possesses a specific item |
| **HasCurrency** | Checks if the interactor has enough of a currency type |
| **QuestState** | Checks the state of a quest (not started, in progress, completed) |
| **SkillLevel** | Checks if the interactor meets a minimum skill level |

Create custom conditions by subclassing `UFWDialogueConditionBase` and overriding the `Evaluate` function.

## Built-In Actions

| Action | Description |
|---|---|
| **GiveItem** | Grants an item to the interactor |
| **GiveCurrency** | Grants currency to the interactor |
| **StartQuest** | Starts a quest for the interactor |
| **OpenVendor** | Opens the vendor/shop interface |

Create custom actions by subclassing `UFWDialogueActionBase` and overriding the `Execute` function.

---

## File Reference

| Page | Description |
|---|---|
| [Installation](installation.md) | Add the plugin to your project |
| [Quick Start](quick-start.md) | Get a basic NPC conversation running in 10 minutes |
| [API Reference](api-reference.md) | Full function, property, and event reference |
| [Blueprints](blueprints.md) | Blueprint integration patterns and examples |
| [Configuration](configuration.md) | Dialogue tree setup, conditions, and actions |
| [Tutorials](tutorials.md) | Step-by-step guide: Creating a Quest-Giving NPC |
| [FAQ](faq.md) | Common questions and troubleshooting |

---

## Compatibility

| Engine Version | Status |
|---|---|
| UE 5.4+ | Supported |
| UE 5.3 | Supported |
| UE 5.2 and below | Not tested |
