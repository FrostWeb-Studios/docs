---
title: UFWCraftingComponent
description: Recipe-based crafting system component with material requirements, skill integration, and quality outcomes.
---

# UFWCraftingComponent

**Header:** `FWCraftingComponent.h` | **Parent:** `UActorComponent`

Provides recipe-based crafting functionality. Checks material requirements against the player's inventory, validates optional skill prerequisites (via FWSkillSystem), consumes materials, and produces output items.

---

## Class Declaration

```cpp
UCLASS(ClassGroup = (FrostWeb), meta = (BlueprintSpawnableComponent))
class FWINVENTORYSYSTEM_API UFWCraftingComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFWCraftingComponent();

    // --- Crafting Operations ---
    UFUNCTION(BlueprintCallable, Category = "Crafting")
    bool CraftItem(const UFWRecipeDefinition* Recipe, int32 Quantity = 1);

    UFUNCTION(BlueprintPure, Category = "Crafting")
    bool CanCraft(const UFWRecipeDefinition* Recipe,
        int32 Quantity = 1) const;

    UFUNCTION(BlueprintPure, Category = "Crafting")
    TArray<UFWRecipeDefinition*> GetAvailableRecipes() const;

    UFUNCTION(BlueprintPure, Category = "Crafting")
    TArray<UFWRecipeDefinition*> GetLearnedRecipes() const;

    // --- Recipe Management ---
    UFUNCTION(BlueprintCallable, Category = "Crafting")
    void LearnRecipe(const UFWRecipeDefinition* Recipe);

    UFUNCTION(BlueprintCallable, Category = "Crafting")
    void UnlearnRecipe(const UFWRecipeDefinition* Recipe);

    UFUNCTION(BlueprintPure, Category = "Crafting")
    bool KnowsRecipe(const UFWRecipeDefinition* Recipe) const;

    // --- Recipe Database ---
    UFUNCTION(BlueprintCallable, Category = "Crafting")
    void SetRecipeDatabase(UFWRecipeDatabase* Database);

    UFUNCTION(BlueprintPure, Category = "Crafting")
    UFWRecipeDatabase* GetRecipeDatabase() const;

    // --- Delegates ---
    UPROPERTY(BlueprintAssignable, Category = "Crafting|Events")
    FOnCraftingStarted OnCraftingStarted;

    UPROPERTY(BlueprintAssignable, Category = "Crafting|Events")
    FOnCraftingCompleted OnCraftingCompleted;

    UPROPERTY(BlueprintAssignable, Category = "Crafting|Events")
    FOnCraftingFailed OnCraftingFailed;

    UPROPERTY(BlueprintAssignable, Category = "Crafting|Events")
    FOnRecipeLearned OnRecipeLearned;
};
```

---

## Functions

### CraftItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Crafting")
bool CraftItem(const UFWRecipeDefinition* Recipe, int32 Quantity = 1);
```

Attempts to craft the specified recipe. Validates all requirements, consumes materials from the inventory, and adds the output item(s).

| Parameter | Type | Default | Description |
|---|---|---|---|
| `Recipe` | `UFWRecipeDefinition*` | | The recipe to craft. |
| `Quantity` | `int32` | `1` | Number of times to craft. Materials are consumed per craft. |

**Returns:** `true` if crafting succeeded.

**Validation steps (in order):**

1. Recipe is not null.
2. Player knows the recipe (or recipe does not require learning).
3. Player meets skill requirements (if FWSkillSystem is available).
4. Inventory contains sufficient materials for the requested quantity.
5. Inventory has room for the output item(s).

If any validation fails, `OnCraftingFailed` fires with a reason string.

---

### CanCraft

```cpp
UFUNCTION(BlueprintPure, Category = "Crafting")
bool CanCraft(const UFWRecipeDefinition* Recipe, int32 Quantity = 1) const;
```

Checks whether all requirements are met to craft the recipe without actually consuming materials. Use this to enable/disable craft buttons in UI.

---

### GetAvailableRecipes

```cpp
UFUNCTION(BlueprintPure, Category = "Crafting")
TArray<UFWRecipeDefinition*> GetAvailableRecipes() const;
```

Returns all recipes from the recipe database that the player currently has the materials and skills to craft. This filters the full database against current inventory and skill state.

---

### GetLearnedRecipes

```cpp
UFUNCTION(BlueprintPure, Category = "Crafting")
TArray<UFWRecipeDefinition*> GetLearnedRecipes() const;
```

Returns all recipes the player has learned (regardless of whether they have materials).

---

### LearnRecipe / UnlearnRecipe

```cpp
UFUNCTION(BlueprintCallable, Category = "Crafting")
void LearnRecipe(const UFWRecipeDefinition* Recipe);

UFUNCTION(BlueprintCallable, Category = "Crafting")
void UnlearnRecipe(const UFWRecipeDefinition* Recipe);
```

Adds or removes a recipe from the player's known recipes. Learned recipes persist via serialization.

---

## Crafting Flow

```
Player selects recipe in UI
    |
    v
CanCraft() check --> Disable craft button if false
    |
    v
CraftItem() called
    |
    +--> Validate recipe requirements
    |       |
    |       +--> Fail: OnCraftingFailed fires
    |
    +--> OnCraftingStarted fires
    |
    +--> Consume materials from inventory (RemoveItem per ingredient)
    |
    +--> Generate output item(s) with affixes (if applicable)
    |
    +--> AddItem to inventory
    |       |
    |       +--> Fail (full): Materials refunded, OnCraftingFailed fires
    |
    +--> OnCraftingCompleted fires
    |
    +--> Award skill XP (if FWSkillSystem available)
```

---

## Delegates

### OnCraftingStarted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnCraftingStarted,
    const UFWRecipeDefinition*, Recipe);
```

Fires when crafting begins, before materials are consumed. Use for UI animations or progress bars.

### OnCraftingCompleted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnCraftingCompleted,
    const UFWRecipeDefinition*, Recipe,
    const FFWItemInstance&, CraftedItem);
```

Fires when crafting completes successfully. `CraftedItem` is the generated item instance with any rolled affixes.

### OnCraftingFailed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnCraftingFailed,
    const UFWRecipeDefinition*, Recipe,
    const FString&, Reason);
```

Fires when crafting fails. `Reason` contains a human-readable failure description.

### OnRecipeLearned

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnRecipeLearned,
    const UFWRecipeDefinition*, Recipe);
```

Fires when a new recipe is learned.

---

## Usage Example

=== "C++"

    ```cpp
    void UCraftingWidget::OnCraftButtonClicked()
    {
        UFWCraftingComponent* Crafting =
            GetOwningPlayerPawn()->FindComponentByClass<UFWCraftingComponent>();

        if (Crafting && SelectedRecipe)
        {
            if (Crafting->CanCraft(SelectedRecipe, CraftQuantity))
            {
                Crafting->CraftItem(SelectedRecipe, CraftQuantity);
            }
        }
    }

    void UCraftingWidget::RefreshRecipeList()
    {
        UFWCraftingComponent* Crafting =
            GetOwningPlayerPawn()->FindComponentByClass<UFWCraftingComponent>();

        RecipeListView->ClearListItems();
        TArray<UFWRecipeDefinition*> Recipes = Crafting->GetLearnedRecipes();

        for (UFWRecipeDefinition* Recipe : Recipes)
        {
            URecipeEntry* Entry = CreateWidget<URecipeEntry>(
                this, RecipeEntryClass);
            Entry->SetRecipe(Recipe);
            Entry->SetCanCraft(Crafting->CanCraft(Recipe));
            RecipeListView->AddItem(Entry);
        }
    }
    ```

=== "Blueprint"

    1. Add **FW Crafting Component** to your character.
    2. Set the **Recipe Database** property to your recipe database asset.
    3. Call **Get Learned Recipes** to populate a recipe list widget.
    4. Use **Can Craft** to grey out unavailable recipes.
    5. Wire a craft button to call **Craft Item** with the selected recipe.
    6. Bind **On Crafting Completed** to show the crafted item result.
