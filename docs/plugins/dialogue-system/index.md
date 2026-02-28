---
title: FWDialogueSystem
---

# FWDialogueSystem

**Version 1.0** | **No external dependencies**

Data-driven dialogue tree system with conditions, actions, and NPC conversation support. FWDialogueSystem provides a complete framework for creating branching dialogues where player choices can be gated by game state and trigger gameplay effects.

---

## Key Features

- **Data-driven dialogue trees.** All dialogue content is defined in `UFWDialogueTree` data assets. Writers and designers can create and edit conversations without touching code.
- **Branching responses.** Each dialogue node can present multiple player responses, each leading to a different node in the tree. Build complex conversation flows with ease.
- **Condition system.** Gate responses behind game state checks -- item ownership, skill levels, quest progress, currency amounts. Conditions are modular and extensible.
- **Action system.** Trigger gameplay effects from dialogue -- give items, start quests, open vendor screens, grant currency. Actions are modular and extensible.
- **NPC conversation management.** The dialogue component tracks the current interactor, active node, and dialogue state, handling the full conversation lifecycle.
- **Blueprint-friendly API.** Start, navigate, and end dialogues entirely from Blueprints. All events are exposed for UI binding.

---

## Architecture Overview

```
UFWDialogueTree (DataAsset)
    |
    +-- FFWDialogueNode (array)
            |
            +-- NodeId, SpeakerName, DialogueText
            +-- EntryActions[] (fired when node is entered)
            +-- FFWDialogueResponse (array)
                    |
                    +-- ResponseText, NextNodeId
                    +-- Conditions[] (must pass for response to appear)
                    +-- Actions[] (fired when response is selected)

UFWDialogueComponent (on NPC Actor)
    |
    +-- StartDialogue(Tree, Interactor)
    +-- SelectResponse(Index)
    +-- EndDialogue()
    +-- Events: OnDialogueStarted, OnDialogueEnded, OnNodeChanged
```

The component drives the conversation state machine. Your UI reads the current node and available responses from the component, displays them, and calls `SelectResponse` when the player makes a choice.

---

## Quick Navigation

| Page | Description |
|---|---|
| [Installation](installation.md) | Add the plugin to your project |
| [Quick Start](quick-start.md) | Get a basic NPC conversation running in 10 minutes |
| [API Reference](api-reference.md) | Full C++ and Blueprint API documentation |
| [Blueprints](blueprints.md) | Blueprint integration patterns and examples |
| [Configuration](configuration.md) | Dialogue tree setup, conditions, and actions |
| [Tutorials](tutorials.md) | Step-by-step guide: Creating a Quest-Giving NPC |
| [FAQ](faq.md) | Common questions and troubleshooting |
| [Migration Guide](migration.md) | Upgrading between versions |
| [Changelog](changelog.md) | Version history and release notes |

---

## Condition and Action Classes

### Built-In Conditions

| Class | Description |
|---|---|
| `FWDlgCondition_HasItem` | Checks if the interactor possesses a specific item |
| `FWDlgCondition_SkillLevel` | Checks if the interactor meets a minimum skill level |
| `FWDlgCondition_QuestState` | Checks the state of a quest (not started, in progress, completed) |
| `FWDlgCondition_HasCurrency` | Checks if the interactor has enough of a currency type |

### Built-In Actions

| Class | Description |
|---|---|
| `FWDlgAction_OpenVendor` | Opens the vendor/shop interface |
| `FWDlgAction_GiveItem` | Grants an item to the interactor |
| `FWDlgAction_StartQuest` | Starts a quest for the interactor |
| `FWDlgAction_GiveCurrency` | Grants currency to the interactor |

Both conditions and actions are extensible. Create new subclasses of `UFWDialogueConditionBase` or `UFWDialogueActionBase` to add game-specific logic.
