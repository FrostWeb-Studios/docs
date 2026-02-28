---
title: Blueprints - FWDialogueSystem
---

# Blueprint Integration

This page covers how to use FWDialogueSystem entirely from Blueprints, including component setup, event binding, UI patterns, and common dialogue workflows.

---

## Adding the Component

1. Open your NPC Actor Blueprint.
2. In the Components panel, click **Add** and search for `FWDialogue`.
3. Select **FW Dialogue Component**.

The component has no required configuration in the Details panel. Dialogue trees are passed at runtime when starting a conversation.

!!! tip "One Component Per NPC"
    Each NPC that can have conversations needs its own Dialogue Component. The component tracks conversation state for one dialogue at a time.

---

## Starting a Dialogue

### From an Interaction Event

The most common pattern: the player presses an interact key near an NPC.

```
On Interact (Player Character)
    |
    +-- Get Dialogue Component
    |
    +-- Start Dialogue
            Tree: DA_Dialogue_QuestGiver (soft reference)
            Interactor: Player Character
    |
    +-- Show Dialogue UI Widget
    |
    +-- Disable Player Movement Input
```

### From an Overlap Event

For dialogue that triggers automatically when the player enters an area:

```
On Actor Begin Overlap (Other Actor)
    |
    +-- Cast to Player Character
    |       (if fails, return)
    |
    +-- Is Dialogue Active?
    |       (if true, return -- prevent double-start)
    |
    +-- Start Dialogue (Tree, Player Character)
```

---

## Driving the UI from Events

### OnNodeChanged -- Update Display

This is the primary event for updating your dialogue UI:

```
Bind Event to On Node Changed
    |
    Event: (NewNode)
        |
        +-- Set Speaker Name Text = NewNode.SpeakerName
        |
        +-- Set Dialogue Body Text = NewNode.DialogueText
        |
        +-- Clear Response Button Container
        |
        +-- Available = Get Available Responses
        |
        +-- For Each Response (with index):
                |
                +-- Create WBP_ResponseButton widget
                +-- Set Button Text = Response.ResponseText
                +-- Set Button Index = Loop Index
                +-- Add to Response Container
```

### OnDialogueStarted -- Show UI

```
Bind Event to On Dialogue Started
    |
    Event: (Tree, Interactor)
        |
        +-- Show Dialogue Widget
        +-- Set Input Mode UI Only
```

### OnDialogueEnded -- Hide UI

```
Bind Event to On Dialogue Ended
    |
    Event: ()
        |
        +-- Hide Dialogue Widget
        +-- Set Input Mode Game Only
        +-- Re-enable Player Movement
```

---

## Selecting a Response

When the player clicks a response button, call `SelectResponse` with the button's index:

```
WBP_ResponseButton -- On Clicked
    |
    +-- Get Owning NPC's Dialogue Component
    |
    +-- Select Response (ButtonIndex)
```

The component handles everything from here:

1. Executes the response's actions (give item, start quest, etc.)
2. Advances to the next node (or ends the dialogue if `NextNodeId = -1`)
3. Fires `OnNodeChanged` (which updates the UI) or `OnDialogueEnded`

---

## Querying Dialogue State

### Check If Dialogue Is Active

Use this before allowing interactions or to show/hide UI indicators:

```
Get Dialogue Component
    |
    +-- Is Dialogue Active --> Boolean
```

### Get the Current Interactor

Useful for NPCs that need to face or animate toward the speaking player:

```
Get Dialogue Component
    |
    +-- Get Current Interactor --> AActor*
    |
    +-- Find Look At Rotation (NPC -> Interactor)
    |
    +-- Set Actor Rotation
```

### Get the Current Node Directly

For cases where you need to read node data outside of the `OnNodeChanged` event:

```
Get Dialogue Component
    |
    +-- Get Current Node --> FFWDialogueNode
    |
    +-- Break FFWDialogueNode
            +-- NodeId
            +-- SpeakerName
            +-- DialogueText
            +-- EntryActions
            +-- Responses
```

---

## Common Blueprint Patterns

### Pattern: NPC With Multiple Dialogue Trees

An NPC that says different things based on quest progress:

```
On Interact (Player Character)
    |
    +-- Check Quest State for "QST_Blacksmith_Intro"
    |
    +-- Branch:
            |
            +-- Not Started:
            |       Start Dialogue (DA_Dialogue_Blacksmith_Intro)
            |
            +-- In Progress:
            |       Start Dialogue (DA_Dialogue_Blacksmith_InProgress)
            |
            +-- Completed:
                    Start Dialogue (DA_Dialogue_Blacksmith_PostQuest)
```

### Pattern: Typewriter Text Effect

Animate dialogue text character by character:

```
On Node Changed (NewNode)
    |
    +-- FullText = NewNode.DialogueText
    |
    +-- DisplayedText = ""
    |
    +-- CharIndex = 0
    |
    +-- Set Timer by Event (0.03s, Looping)
            |
            +-- DisplayedText += FullText[CharIndex]
            +-- CharIndex++
            +-- Set Dialogue Text Widget = DisplayedText
            +-- If CharIndex >= FullText.Length:
                    Clear Timer
                    Show Response Buttons
```

!!! tip "Skip Button"
    Allow the player to click to skip the typewriter effect and show the full text immediately. This is a standard UX expectation for dialogue systems.

### Pattern: Speaker Portrait

If your dialogue nodes include speaker information, display a portrait alongside the text:

```
On Node Changed (NewNode)
    |
    +-- SpeakerName = NewNode.SpeakerName
    |
    +-- Look Up Portrait in Data Table (SpeakerName -> Portrait Texture)
    |
    +-- Set Portrait Image Widget = Portrait Texture
```

### Pattern: Continue Button (No Responses)

For narrative nodes where the NPC speaks and the player just clicks "Continue":

```
On Node Changed (NewNode)
    |
    +-- Responses = Get Available Responses
    |
    +-- If Responses is Empty:
    |       Show "Continue" button that calls EndDialogue
    |
    +-- Else:
            Show response buttons as normal
```

### Pattern: Dialogue Log / History

Keep a scrollable history of the conversation:

```
On Node Changed (NewNode)
    |
    +-- Create Log Entry Widget
    |       Speaker: NewNode.SpeakerName
    |       Text: NewNode.DialogueText
    |
    +-- Add to Scroll Box
    |
    +-- Scroll to Bottom
```

---

## Blueprint Node Reference

### Callable Nodes

| Node | Category | Description |
|---|---|---|
| Start Dialogue | FW Dialogue | Begins a conversation with a dialogue tree |
| Select Response | FW Dialogue | Chooses a player response by index |
| End Dialogue | FW Dialogue | Ends the active conversation immediately |

### Pure Nodes

| Node | Category | Returns |
|---|---|---|
| Get Current Node | FW Dialogue | FFWDialogueNode |
| Get Available Responses | FW Dialogue | TArray\<FFWDialogueResponse\> |
| Is Dialogue Active | FW Dialogue | bool |
| Get Current Interactor | FW Dialogue | AActor* |

### Event Nodes

| Event | Category | Payload |
|---|---|---|
| On Dialogue Started | FW Dialogue | UFWDialogueTree*, AActor* |
| On Dialogue Ended | FW Dialogue | (none) |
| On Node Changed | FW Dialogue | FFWDialogueNode |
