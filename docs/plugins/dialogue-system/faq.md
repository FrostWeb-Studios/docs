---
title: FAQ - FWDialogueSystem
---

# Frequently Asked Questions

---

## General

### Does FWDialogueSystem require any other plugins?

No. FWDialogueSystem has no external dependencies. The built-in conditions and actions reference game concepts (items, quests, skills, currency) through their own evaluation/execution methods, which you implement to integrate with your game's systems.

### Can I use this plugin for non-NPC dialogue (e.g., intercom messages, environmental storytelling)?

Yes. The dialogue component does not assume anything about the owning Actor. Attach it to any Actor -- an NPC, a radio, a terminal, a sign. The `SpeakerName` field in each node can be set to anything.

### Does this plugin support Blueprint-only projects?

No. FWDialogueSystem contains C++ modules and requires a C++ project.

### Can I create conditions and actions in Blueprints?

Yes. Both `UFWDialogueConditionBase` and `UFWDialogueActionBase` can be subclassed in Blueprints. Override the `Evaluate` or `Execute` function respectively.

---

## Dialogue Trees

### How do I make the NPC say different things based on quest progress?

You have two approaches:

**Approach A -- Single tree with conditional responses.** Create one dialogue tree with a greeting node that has multiple responses, each gated by a `FWDlgCondition_QuestState` condition. The player only sees responses whose conditions pass. See the [tutorial](tutorials.md) for a complete example.

**Approach B -- Multiple trees.** Create separate dialogue trees for each quest state and select which tree to pass to `StartDialogue` based on external logic. This is simpler for NPCs with dramatically different conversations per state.

### What happens if no responses pass their conditions?

If all responses on a node have conditions and none of them pass, `GetAvailableResponses` returns an empty array. Your UI should handle this case -- typically by showing a "Continue" or "End Conversation" button that calls `EndDialogue`.

!!! tip "Always Include an Unconditional Response"
    As a best practice, include at least one response with no conditions on every node (such as "Farewell" or "Continue"). This ensures the player always has a way to exit or advance the conversation.

### Can a response loop back to an earlier node?

Yes. Set `NextNodeId` to any valid node ID in the tree, including nodes that appeared earlier in the conversation. This allows for:

- Returning to a main menu node after a sub-topic
- Repeating a node until the player makes a different choice
- Creating conversational loops

!!! warning "Infinite Loops"
    There is no built-in loop detection. If you create a cycle with no exit path (e.g., Node A leads to Node B which leads back to Node A, with no farewell option), the player will be stuck in the dialogue. Always provide an exit.

### Can I have multiple dialogue trees reference the same nodes?

No. Each `UFWDialogueTree` is a self-contained asset. Nodes are local to their tree and cannot be shared across trees. If you need shared dialogue segments, duplicate the nodes or create a shared subtree that multiple NPCs reference by using the same dialogue tree asset.

### What is the maximum number of nodes per tree?

There is no hard limit. Node lookup by ID uses a linear search through the array, so extremely large trees (hundreds of nodes) may have a minor lookup cost. For practical purposes, trees with up to 50-100 nodes perform without issue.

---

## Conditions

### How are multiple conditions on a single response evaluated?

With AND logic. All conditions must return `true` for the response to appear. If any condition returns `false`, the entire response is hidden.

### How do I implement OR logic for conditions?

Create separate responses that lead to the same `NextNodeId`. Each response has its own condition set. If either response's conditions pass, the player sees that path.

```
Response A: "I can pay in gold" (Condition: HasCurrency Gold >= 100) --> Node 5
Response B: "I can pay in gems" (Condition: HasItem Gems >= 3)      --> Node 5
```

### Can conditions modify game state?

No. The `Evaluate` method is `const` and should be side-effect-free. Use actions for state changes, not conditions.

### When are conditions evaluated?

Conditions are evaluated when `GetAvailableResponses` is called. This happens automatically when `OnNodeChanged` fires, and can be called again manually if you need to re-evaluate (e.g., if game state changed while the dialogue UI was open).

---

## Actions

### What happens if an action fails (e.g., inventory full)?

The dialogue still advances. Actions are fire-and-forget. If you need to validate before executing, add a corresponding condition to the response. For example, pair `FWDlgAction_GiveItem` with a custom condition that checks for available inventory space.

### Can I chain multiple actions on a single response?

Yes. Add multiple actions to a response's **Actions** array. They execute in array order, top to bottom.

### What is the difference between EntryActions and Response Actions?

- **EntryActions** fire when a node becomes the active node (i.e., it is now being displayed to the player). Use these for side effects that should happen regardless of which response the player picks.
- **Response Actions** fire when the player selects a specific response. Use these for effects tied to the player's choice.

### Can actions trigger dialogue events (e.g., start a new dialogue)?

Actions can call any game code, including starting a new dialogue. However, be careful about calling `StartDialogue` from within an action -- the current dialogue is still active at that point. The component will end the current dialogue first, which may cause unexpected event ordering.

---

## UI Integration

### How do I animate dialogue text (typewriter effect)?

The component provides the full text in `FFWDialogueNode::DialogueText`. Implement the typewriter effect in your UI widget by revealing characters over time. See the [Blueprints](blueprints.md#pattern-typewriter-text-effect) page for a pattern.

### Can I show speaker portraits?

The component provides `SpeakerName` as an `FText`. Map speaker names to portrait textures using a Data Table or a custom lookup function in your UI widget. The dialogue system itself does not manage portrait assets.

### How do I handle voice-over audio?

FWDialogueSystem does not include audio playback. Trigger audio from your UI or NPC Blueprint when `OnNodeChanged` fires. You can map `NodeId` values to audio assets using a Data Table or a custom mapping.

---

## Networking

### Is dialogue replicated across the network?

The dialogue component itself does not replicate. Dialogue is typically a client-side interaction between one player and one NPC. If you need other players to see that an NPC is "in conversation" (e.g., to prevent simultaneous interactions), implement a simple replicated busy flag on the NPC Actor.

### Can actions trigger server-side logic?

If your action subclass calls a server RPC, yes. The built-in actions execute locally. For server-authoritative games, create custom action subclasses that call your game's server functions.

---

## Performance

### Is there a performance cost to evaluating conditions?

Condition evaluation calls a virtual function per condition per response. For typical dialogue nodes (5-10 responses, 1-3 conditions each), the cost is negligible. Avoid placing expensive game queries (database lookups, pathfinding) inside condition evaluation.

### Are dialogue trees loaded into memory all at once?

Yes, when the tree asset is loaded. However, dialogue trees are small -- they contain only text and references to condition/action objects, not textures or meshes. A tree with 50 nodes is a few kilobytes at most.

---

## Troubleshooting

### The NPC starts talking but the UI does not appear.

Verify that you have bound to `OnDialogueStarted` and `OnNodeChanged` events. The component does not manage UI -- it only fires events. Your UI code must respond to those events.

### A response that should appear is missing.

Check its conditions. Call `GetAvailableResponses` in a debug context and log the result count. Then check each condition individually:

1. Is the condition class correct?
2. Are the condition properties (ItemId, QuestId, etc.) spelled exactly right?
3. Does the condition's `Evaluate` function return `true` for the current game state?

### SelectResponse does nothing or crashes.

Verify the index you are passing matches the `GetAvailableResponses` array, not the raw `Responses` array from `GetCurrentNode`. These arrays may differ if conditions filter some responses out.

### EntryActions fire twice.

This can happen if `StartDialogue` is called twice in rapid succession (e.g., double-click on interact). Guard your interaction logic with an `IsDialogueActive` check before calling `StartDialogue`.
