---
title: Quick Start - FWDialogueSystem
---

# Quick Start

Get a basic NPC conversation running in under 10 minutes.

---

## Overview

By the end of this guide you will have:

1. A dialogue tree data asset with branching responses
2. An NPC Blueprint with `UFWDialogueComponent` attached
3. A working conversation that the player can navigate

---

## Step 1 -- Create a Dialogue Tree

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWDialogueTree` as the class.
3. Name it `DA_Dialogue_Blacksmith`.

Open the asset and add dialogue nodes:

### Node 0 -- Greeting

| Field | Value |
|---|---|
| NodeId | `0` |
| SpeakerName | `Grunthor the Blacksmith` |
| DialogueText | `Welcome, traveler. My forge is the finest in the valley. What brings you here?` |
| EntryActions | (none) |

Add two responses:

| Response | ResponseText | NextNodeId |
|---|---|---|
| 0 | `I need a weapon repaired.` | `1` |
| 1 | `Just browsing. Farewell.` | `-1` |

!!! info "End-of-Dialogue Convention"
    A `NextNodeId` of `-1` signals the end of the conversation. When a response with `NextNodeId = -1` is selected, the component calls `EndDialogue` automatically.

### Node 1 -- Repair Branch

| Field | Value |
|---|---|
| NodeId | `1` |
| SpeakerName | `Grunthor the Blacksmith` |
| DialogueText | `Aye, I can fix that up for you. It will cost 50 gold. What say you?` |

Add two responses:

| Response | ResponseText | NextNodeId |
|---|---|---|
| 0 | `Here is the gold. Fix it up.` | `2` |
| 1 | `That is too expensive. Never mind.` | `-1` |

### Node 2 -- Completion

| Field | Value |
|---|---|
| NodeId | `2` |
| SpeakerName | `Grunthor the Blacksmith` |
| DialogueText | `Done. Good as new. Come back if you need anything else.` |

Add one response:

| Response | ResponseText | NextNodeId |
|---|---|---|
| 0 | `Thanks, Grunthor.` | `-1` |

---

## Step 2 -- Create the NPC Blueprint

1. Create a new Actor Blueprint (e.g., `BP_NPC_Blacksmith`).
2. Add a **Skeletal Mesh Component** or **Static Mesh Component** for the NPC's visual.
3. Click **Add Component** and search for `FWDialogue`.
4. Select **FW Dialogue Component**.

The dialogue component needs no configuration in the Details panel -- dialogue trees are passed at runtime when the conversation starts.

---

## Step 3 -- Start the Dialogue

When the player interacts with the NPC (via overlap, interaction key, etc.), start the dialogue:

=== "Blueprint"

    ```
    On Interact (Player)
        |
        +-- Get Dialogue Component
        |
        +-- Load DA_Dialogue_Blacksmith (Soft Object Reference)
        |
        +-- Start Dialogue
                Tree: DA_Dialogue_Blacksmith
                Interactor: Player Character
    ```

=== "C++"

    ```cpp
    void ANPC_Blacksmith::OnInteract(AActor* Interactor)
    {
        UFWDialogueTree* Tree = LoadObject<UFWDialogueTree>(
            nullptr, TEXT("/Game/Data/Dialogues/DA_Dialogue_Blacksmith"));

        DialogueComp->StartDialogue(Tree, Interactor);
    }
    ```

---

## Step 4 -- Display the Dialogue UI

Bind to the component's events to drive your UI:

```
Bind Event to On Node Changed
    |
    Event: ()
        |
        +-- CurrentNode = Get Current Node
        |
        +-- Set Speaker Name Text = CurrentNode.SpeakerName
        |
        +-- Set Dialogue Text = CurrentNode.DialogueText
        |
        +-- Responses = Get Available Responses
        |
        +-- For Each Response:
                +-- Create Response Button
                +-- Set Button Text = Response.ResponseText
```

When the player clicks a response button:

```
On Response Button Clicked (Index)
    |
    +-- Get Dialogue Component
    |
    +-- Select Response (Index)
```

The component advances to the next node and fires `OnNodeChanged` again, updating the UI.

---

## Step 5 -- Handle Dialogue End

```
Bind Event to On Dialogue Ended
    |
    Event: ()
        |
        +-- Hide Dialogue UI
        |
        +-- Re-enable Player Input
```

---

## Result

You now have:

- An NPC that speaks with branching dialogue
- A data-driven conversation defined entirely in a Data Asset
- UI events that update whenever the dialogue state changes

---

## Next Steps

- Add conditions to gate responses (see [Configuration](configuration.md)).
- Add actions to trigger gameplay effects from dialogue (see [Configuration](configuration.md)).
- Follow the [Creating a Quest-Giving NPC](tutorials.md) tutorial for a complete example with conditions and actions.
