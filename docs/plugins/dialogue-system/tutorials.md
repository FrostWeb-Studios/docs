---
title: Tutorials - FWDialogueSystem
---

# Tutorials

## Creating a Quest-Giving NPC

This tutorial walks you through building a complete quest-giving NPC -- from creating a dialogue tree with condition checks (quest state, item requirements) to triggering quest start and item rewards through dialogue actions.

---

### What You Will Build

An NPC named "Elder Miraya" who:

- Greets the player and offers a quest to retrieve a stolen relic
- Checks if the player already has the quest or has completed it
- Requires the player to have the relic item before accepting it
- Starts the quest, gives a reward, and changes dialogue after completion

### Prerequisites

- FWDialogueSystem installed and compiling (see [Installation](installation.md))
- Basic understanding of dialogue trees (see [Quick Start](quick-start.md))
- An NPC Blueprint with a Dialogue Component

---

### Part 1 -- Designing the Conversation Flow

Before creating data assets, plan the conversation on paper:

```
[Node 0: Greeting - branches based on quest state]
    |
    +-- "What quest do you have?" (Condition: Quest NOT started)
    |       --> Node 1: Quest Offer
    |
    +-- "I found the relic." (Condition: Quest InProgress AND HasItem)
    |       --> Node 3: Turn In
    |
    +-- "I am still searching." (Condition: Quest InProgress AND NOT HasItem)
    |       --> Node 2: Reminder
    |
    +-- "Hello again, Elder." (Condition: Quest Completed)
    |       --> Node 4: Post-Quest
    |
    +-- "Farewell."
            --> -1 (end)

[Node 1: Quest Offer]
    +-- "I accept." (Action: StartQuest) --> Node 5: Accepted
    +-- "Not right now." --> -1

[Node 2: Reminder]
    +-- "I will keep looking." --> -1

[Node 3: Turn In]
    +-- "Here it is." (Action: GiveItem to NPC, GiveCurrency reward) --> Node 6: Reward
    +-- "Not yet, I want to hold onto it." --> -1

[Node 4: Post-Quest]
    +-- "Farewell, Elder." --> -1

[Node 5: Accepted]
    +-- "I will return with the relic." --> -1

[Node 6: Reward]
    +-- "Thank you, Elder." --> -1
```

---

### Part 2 -- Creating the Dialogue Tree Asset

1. In the Content Browser, navigate to `Content/Data/Dialogues/ElderMiraya/`.
2. Create a Data Asset of type `FWDialogueTree`.
3. Name it `DA_Dialogue_ElderMiraya`.

Now add each node:

#### Node 0 -- Greeting

| Field | Value |
|---|---|
| NodeId | `0` |
| SpeakerName | `Elder Miraya` |
| DialogueText | `Ah, a visitor. These are troubled times. The ancient relic of our village has been stolen by bandits in the northern caves.` |
| EntryActions | (none) |

Add five responses:

---

**Response 0 -- Offer the quest (player has not started it)**

| Field | Value |
|---|---|
| ResponseText | `What can I do to help?` |
| NextNodeId | `1` |

Add one condition:

| Condition | Class | Properties |
|---|---|---|
| [0] | `FWDlgCondition_QuestState` | QuestId: `QST_StolenRelic`, RequiredState: `NotStarted` |

---

**Response 1 -- Turn in (player has the relic)**

| Field | Value |
|---|---|
| ResponseText | `I recovered the relic from the caves.` |
| NextNodeId | `3` |

Add two conditions (both must pass):

| Condition | Class | Properties |
|---|---|---|
| [0] | `FWDlgCondition_QuestState` | QuestId: `QST_StolenRelic`, RequiredState: `InProgress` |
| [1] | `FWDlgCondition_HasItem` | ItemId: `ITEM_AncientRelic`, RequiredCount: `1` |

---

**Response 2 -- Still searching (quest active, no relic)**

| Field | Value |
|---|---|
| ResponseText | `I am still searching for the relic.` |
| NextNodeId | `2` |

Add one condition:

| Condition | Class | Properties |
|---|---|---|
| [0] | `FWDlgCondition_QuestState` | QuestId: `QST_StolenRelic`, RequiredState: `InProgress` |

!!! info "Condition Evaluation Order"
    Response 1 and Response 2 both check for `InProgress`. Response 1 additionally checks for the item. Since both responses are always evaluated independently, they will correctly show the right option: Response 1 only appears when the player has the item, and Response 2 appears whenever the quest is in progress. If the player has the item, both responses will appear -- which is fine, as the player can choose either path.

    If you want Response 2 to only appear when the player does NOT have the item, you would need a custom `FWDlgCondition_NotHasItem` condition (see [Configuration](configuration.md#creating-custom-conditions) for how to create custom conditions).

---

**Response 3 -- Post-quest greeting**

| Field | Value |
|---|---|
| ResponseText | `Good to see you, Elder. How are things?` |
| NextNodeId | `4` |

Add one condition:

| Condition | Class | Properties |
|---|---|---|
| [0] | `FWDlgCondition_QuestState` | QuestId: `QST_StolenRelic`, RequiredState: `Completed` |

---

**Response 4 -- Farewell (always available)**

| Field | Value |
|---|---|
| ResponseText | `Farewell.` |
| NextNodeId | `-1` |

No conditions. This response is always available as a way to exit the conversation.

---

#### Node 1 -- Quest Offer

| Field | Value |
|---|---|
| NodeId | `1` |
| SpeakerName | `Elder Miraya` |
| DialogueText | `Travel to the northern caves and retrieve the relic from the bandits. Be careful -- they are dangerous folk. I will reward you handsomely for its return.` |

**Response 0 -- Accept**

| Field | Value |
|---|---|
| ResponseText | `I accept. I will bring back the relic.` |
| NextNodeId | `5` |

Add one action:

| Action | Class | Properties |
|---|---|---|
| [0] | `FWDlgAction_StartQuest` | QuestId: `QST_StolenRelic` |

**Response 1 -- Decline**

| Field | Value |
|---|---|
| ResponseText | `I am not ready for that. Perhaps later.` |
| NextNodeId | `-1` |

---

#### Node 2 -- Reminder

| Field | Value |
|---|---|
| NodeId | `2` |
| SpeakerName | `Elder Miraya` |
| DialogueText | `The caves lie to the north. Please hurry -- every day without the relic weakens our village's protection.` |

**Response 0:**

| Field | Value |
|---|---|
| ResponseText | `I will not rest until I find it.` |
| NextNodeId | `-1` |

---

#### Node 3 -- Turn In

| Field | Value |
|---|---|
| NodeId | `3` |
| SpeakerName | `Elder Miraya` |
| DialogueText | `You found it! The relic... I can feel its power. You have saved our village. Please, accept this reward.` |

**Response 0 -- Complete the quest**

| Field | Value |
|---|---|
| ResponseText | `Thank you, Elder. It was a difficult journey.` |
| NextNodeId | `6` |

Add two actions:

| Action | Class | Properties |
|---|---|---|
| [0] | `FWDlgAction_GiveItem` | ItemId: `ITEM_EldersBlessingAmulet`, Quantity: `1` |
| [1] | `FWDlgAction_GiveCurrency` | CurrencyType: `Gold`, Amount: `500` |

!!! note "Quest Completion"
    The quest system handles marking the quest as completed when the turn-in conditions are met. The dialogue actions here handle the reward side. Your quest system and dialogue system work together but remain decoupled.

---

#### Node 4 -- Post-Quest

| Field | Value |
|---|---|
| NodeId | `4` |
| SpeakerName | `Elder Miraya` |
| DialogueText | `The village thrives once more, thanks to you. You are always welcome here.` |

**Response 0:**

| Field | Value |
|---|---|
| ResponseText | `Take care, Elder.` |
| NextNodeId | `-1` |

---

#### Node 5 -- Quest Accepted

| Field | Value |
|---|---|
| NodeId | `5` |
| SpeakerName | `Elder Miraya` |
| DialogueText | `May the spirits guide your path. Return to me when you have the relic.` |

**Response 0:**

| Field | Value |
|---|---|
| ResponseText | `I will return soon.` |
| NextNodeId | `-1` |

---

#### Node 6 -- Reward Given

| Field | Value |
|---|---|
| NodeId | `6` |
| SpeakerName | `Elder Miraya` |
| DialogueText | `Here -- the Elder's Blessing Amulet and 500 gold. You have earned every coin. Our village is in your debt.` |

**Response 0:**

| Field | Value |
|---|---|
| ResponseText | `Farewell, Elder Miraya.` |
| NextNodeId | `-1` |

---

### Part 3 -- Setting Up the NPC Blueprint

#### 3.1 Create the NPC Actor

1. Create a new Actor Blueprint: `BP_NPC_ElderMiraya`.
2. Add a **Skeletal Mesh Component** with the Elder's mesh.
3. Add a **FW Dialogue Component**.
4. Add a **Sphere Collision** component for interaction range.

#### 3.2 Implement the Interaction

In the Event Graph:

```
Event BeginPlay
    |
    +-- Sphere Collision > On Component Begin Overlap
            |
            +-- Cast to Player Character
            |
            +-- Store Player Reference
            |
            +-- Show "Talk to Elder Miraya" prompt

Sphere Collision > On Component End Overlap
    |
    +-- Hide Prompt

On Interact Input (while in range)
    |
    +-- Dialogue Component > Is Dialogue Active?
    |
    +-- Branch:
            +-- False:
            |       Dialogue Component > Start Dialogue
            |           Tree: DA_Dialogue_ElderMiraya
            |           Interactor: Stored Player Reference
            |
            +-- True:
                    (do nothing -- already talking)
```

#### 3.3 Bind UI Events

```
Event BeginPlay
    |
    +-- Dialogue Component > Bind Event to On Dialogue Started
    |       --> Show Dialogue Widget
    |
    +-- Dialogue Component > Bind Event to On Node Changed
    |       --> Update Dialogue Widget (speaker, text, responses)
    |
    +-- Dialogue Component > Bind Event to On Dialogue Ended
            --> Hide Dialogue Widget
```

---

### Part 4 -- Building the Dialogue UI Widget

Create `WBP_DialoguePanel` with this layout:

```
+--------------------------------------------+
|  [Portrait]  Elder Miraya                   |
+--------------------------------------------+
|                                             |
|  "Ah, a visitor. These are troubled         |
|   times..."                                 |
|                                             |
+--------------------------------------------+
|  > What can I do to help?                   |
|  > Farewell.                                |
+--------------------------------------------+
```

#### Key Implementation Details

**Populating responses:**

```
On Node Changed (NewNode)
    |
    +-- Set Speaker Text = NewNode.SpeakerName
    +-- Set Body Text = NewNode.DialogueText
    +-- Clear Response List
    |
    +-- Responses = Dialogue Component > Get Available Responses
    |
    +-- For Each Response (with Index):
            +-- Create Response Button Widget
            +-- Set Text = Response.ResponseText
            +-- Bind On Clicked:
                    Dialogue Component > Select Response (Index)
```

!!! warning "Use GetAvailableResponses, Not Raw Responses"
    Always call `GetAvailableResponses` to get the filtered list. The raw `Responses` array from `GetCurrentNode` includes responses whose conditions have not passed. Using the raw array will show options the player should not see and will cause index mismatches with `SelectResponse`.

---

### Part 5 -- Testing the Full Flow

Test each conversation path:

| Test Case | Initial State | Expected Behavior |
|---|---|---|
| First visit | Quest not started | Shows "What can I do to help?" and "Farewell" |
| Accept quest | After selecting accept | StartQuest action fires, sees acceptance dialogue |
| Return without relic | Quest in progress, no item | Shows "I am still searching" response |
| Return with relic | Quest in progress, has item | Shows "I recovered the relic" response |
| Turn in relic | Select turn-in response | GiveItem and GiveCurrency actions fire |
| Post-quest visit | Quest completed | Shows post-quest greeting |

!!! tip "Testing Conditions"
    Use console commands or cheat functions to grant items and change quest states during testing. This lets you quickly verify all dialogue branches without playing through the full game loop.

---

### Summary

| Step | What You Built |
|---|---|
| Part 1 | Designed the conversation flow on paper |
| Part 2 | Created the dialogue tree data asset with 7 nodes, conditions, and actions |
| Part 3 | Set up the NPC Blueprint with interaction and event binding |
| Part 4 | Built the dialogue UI widget driven by component events |
| Part 5 | Tested all conversation branches |

---

### Next Steps

- Add more NPCs with their own dialogue trees.
- Create custom conditions for game-specific checks (player level, faction reputation, time of day).
- Create custom actions for game-specific effects (play cinematic, teleport player, change weather).
- Chain multiple quests through the same NPC by using different dialogue trees per quest state.
