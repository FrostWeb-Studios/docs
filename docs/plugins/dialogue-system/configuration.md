---
title: Configuration - FWDialogueSystem
---

# Configuration

This page explains how to build dialogue trees, configure conditions and actions, and structure your dialogue data assets.

---

## Dialogue Tree Structure

A `UFWDialogueTree` is a flat array of `FFWDialogueNode` entries. Nodes reference each other by `NodeId`, forming a directed graph.

### Node Graph Example

```
[Node 0: Greeting]
    |
    +-- Response: "Tell me about the quest" --> Node 1
    +-- Response: "Open your shop"         --> Node 2 (action: OpenVendor)
    +-- Response: "Goodbye"                --> -1 (end)

[Node 1: Quest Info]
    |
    +-- Response: "I accept the quest"     --> -1 (action: StartQuest)
    +-- Response: "Not interested"         --> Node 0 (loop back)

[Node 2: Vendor]
    |
    (entry action: OpenVendor)
    +-- Response: "Thanks"                 --> -1 (end)
```

---

## Creating a Dialogue Tree

### Step 1 -- Create the Data Asset

1. Right-click in the Content Browser > **Miscellaneous > Data Asset**.
2. Select `FWDialogueTree` as the class.
3. Name it with a descriptive prefix (e.g., `DA_Dialogue_Blacksmith`).

### Step 2 -- Add Nodes

Open the asset and add entries to the **Nodes** array. Each node requires:

| Field | Required | Description |
|---|---|---|
| `NodeId` | Yes | Unique integer ID. Node 0 is the entry point. |
| `SpeakerName` | Yes | Who is speaking (displayed in UI) |
| `DialogueText` | Yes | The dialogue line (supports FText localization) |
| `EntryActions` | No | Actions executed when this node becomes active |
| `Responses` | Yes | At least one response for the player |

### Step 3 -- Add Responses

Each response requires:

| Field | Required | Description |
|---|---|---|
| `ResponseText` | Yes | Button text shown to the player |
| `NextNodeId` | Yes | Where to go next (-1 = end dialogue) |
| `Conditions` | No | All must pass for the response to appear |
| `Actions` | No | Executed when the player selects this response |

!!! warning "Orphan Nodes"
    Nodes that are not reachable from any response's `NextNodeId` will never be displayed. Use this intentionally for conditional branches, but audit your trees to avoid accidentally orphaned content.

---

## Conditions

Conditions gate whether a response appears in the player's available choices. They are instanced UObject subclasses configured directly on the response.

### Adding a Condition

1. In a response's **Conditions** array, click the **+** button.
2. Select the condition class from the dropdown.
3. Configure the condition's properties in the expanded panel.

### Built-In Condition Classes

=== "HasItem"

    **Class:** `FWDlgCondition_HasItem`

    Checks if the interactor has a specific item in their inventory.

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `ItemId` | `FName` | (none) | Item identifier to check for |
    | `RequiredCount` | `int32` | 1 | Minimum quantity needed |

    **Example:** Only show "I have the dragon scale" if the player possesses at least 1 item with ID `ITEM_DragonScale`.

=== "SkillLevel"

    **Class:** `FWDlgCondition_SkillLevel`

    Checks if the interactor has a minimum skill level.

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `SkillId` | `FName` | (none) | Skill identifier to check |
    | `MinLevel` | `int32` | 1 | Minimum level required |

    **Example:** Only show "I can forge that for you" if the player's Smithing skill is at least level 50.

=== "QuestState"

    **Class:** `FWDlgCondition_QuestState`

    Checks whether a quest is in a specific state.

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `QuestId` | `FName` | (none) | Quest identifier to check |
    | `RequiredState` | `EFWQuestState` | NotStarted | Expected state |

    **Example:** Only show "I completed your task" if quest `QST_DragonSlayer` is in `Completed` state.

=== "HasCurrency"

    **Class:** `FWDlgCondition_HasCurrency`

    Checks if the interactor has enough of a currency type.

    | Property | Type | Default | Description |
    |---|---|---|---|
    | `CurrencyType` | `FName` | (none) | Currency identifier |
    | `MinAmount` | `int32` | 0 | Minimum amount required |

    **Example:** Only show "I will pay 100 gold" if the player has at least 100 of currency type `Gold`.

### Multiple Conditions

When a response has multiple conditions, all of them must evaluate to `true` (AND logic). The response is hidden if any single condition fails.

```
Response: "I have the materials and the skill"
    Conditions:
        [0] HasItem: ITEM_DragonScale, Count >= 1
        [1] SkillLevel: Smithing >= 50
```

For OR logic (show the response if either condition passes), create two separate responses that lead to the same next node:

```
Response A: "I found the scale already" (Condition: HasItem)     --> Node 5
Response B: "I know where to find one" (Condition: QuestState)   --> Node 5
```

---

## Actions

Actions trigger gameplay effects during dialogue. They can be attached to:

- **Entry Actions** on a node -- executed when the node becomes active.
- **Response Actions** on a response -- executed when the player selects that response.

### Adding an Action

1. In a node's **EntryActions** array or a response's **Actions** array, click **+**.
2. Select the action class from the dropdown.
3. Configure the action's properties.

### Built-In Action Classes

=== "OpenVendor"

    **Class:** `FWDlgAction_OpenVendor`

    Opens the vendor/shop interface.

    | Property | Type | Description |
    |---|---|---|
    | `VendorId` | `FName` | Identifier of the vendor inventory to display |

    Typically used as a response action on a "Show me your wares" dialogue option.

=== "GiveItem"

    **Class:** `FWDlgAction_GiveItem`

    Grants an item to the interactor's inventory.

    | Property | Type | Description |
    |---|---|---|
    | `ItemId` | `FName` | The item to give |
    | `Quantity` | `int32` | Number of items to grant |

    Used for quest rewards, NPC gifts, or trade outcomes.

=== "StartQuest"

    **Class:** `FWDlgAction_StartQuest`

    Starts a quest for the interactor.

    | Property | Type | Description |
    |---|---|---|
    | `QuestId` | `FName` | The quest to start |

    Typically attached to a response like "I accept your quest."

=== "GiveCurrency"

    **Class:** `FWDlgAction_GiveCurrency`

    Grants currency to the interactor.

    | Property | Type | Description |
    |---|---|---|
    | `CurrencyType` | `FName` | The type of currency |
    | `Amount` | `int32` | Amount to grant |

    Used for payment, rewards, or bounties.

### Action Execution Order

1. When a node is entered, **EntryActions** execute in array order.
2. When a response is selected, the response's **Actions** execute in array order.
3. After response actions execute, the component advances to the next node.

!!! info "Actions Are Fire-and-Forget"
    Actions execute synchronously and do not block dialogue progression. If an action fails (e.g., inventory full when giving an item), the dialogue still advances. Implement validation logic in your action subclasses if you need to handle failure cases.

---

## Creating Custom Conditions

To add game-specific conditions, create a new C++ or Blueprint class extending `UFWDialogueConditionBase`:

### C++ Example

```cpp
UCLASS(EditInlineNew, DefaultToInstanced)
class UFWDlgCondition_PlayerLevel : public UFWDialogueConditionBase
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Condition")
    int32 MinimumLevel = 1;

    virtual bool Evaluate(AActor* Interactor) const override
    {
        // Your game-specific level check
        if (AMyPlayerCharacter* PC = Cast<AMyPlayerCharacter>(Interactor))
        {
            return PC->GetLevel() >= MinimumLevel;
        }
        return false;
    }
};
```

### Blueprint Example

1. Create a new Blueprint class with parent `FWDialogueConditionBase`.
2. Override the **Evaluate** function.
3. Add variables for your condition parameters (exposed to the editor with `EditAnywhere`).

!!! tip "EditInlineNew"
    Use the `EditInlineNew` and `DefaultToInstanced` class specifiers so your condition can be created inline within the dialogue tree editor, without requiring a separate asset.

---

## Creating Custom Actions

Same pattern as conditions, but extend `UFWDialogueActionBase`:

```cpp
UCLASS(EditInlineNew, DefaultToInstanced)
class UFWDlgAction_PlayCameraShake : public UFWDialogueActionBase
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Action")
    TSubclassOf<UCameraShakeBase> ShakeClass;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Action")
    float Scale = 1.0f;

    virtual void Execute(AActor* Interactor) const override
    {
        if (APlayerController* PC = Cast<APlayerController>(
                Cast<APawn>(Interactor)->GetController()))
        {
            PC->ClientStartCameraShake(ShakeClass, Scale);
        }
    }
};
```

---

## Localization

All text fields (`SpeakerName`, `DialogueText`, `ResponseText`) use `FText`, which fully supports Unreal's localization pipeline. To localize your dialogue:

1. Set up your project's localization targets in **Edit > Project Settings > Localization**.
2. Gather text from dialogue tree data assets using the **Localization Dashboard**.
3. Translate strings using the `.po` file workflow.

!!! note "FText Keys"
    Each `FText` in a data asset automatically generates a stable key based on the asset path and property name. Renaming or moving data assets will invalidate translation keys, requiring re-mapping in your localization files.

---

## Recommended Folder Structure

```
Content/
    Data/
        Dialogues/
            Blacksmith/
                DA_Dialogue_Blacksmith_Intro.uasset
                DA_Dialogue_Blacksmith_Quest.uasset
                DA_Dialogue_Blacksmith_PostQuest.uasset
            QuestGiver/
                DA_Dialogue_QuestGiver_Main.uasset
            Merchant/
                DA_Dialogue_Merchant.uasset
```

Group dialogue trees by NPC or by quest line, depending on which organizational style fits your project.
