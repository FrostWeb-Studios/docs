---
title: Data Assets
description: Reference for UFWItemDefinition, UFWItemSetDefinition, UFWPerkDefinition, UFWRecipeDefinition, and UFWRecipeDatabase data assets.
---

# Data Assets

The FWInventorySystem uses Unreal Engine's `PrimaryDataAsset` system for item definitions, set bonuses, perks, and crafting recipes. Data assets are created in the Content Browser and referenced at runtime via soft object pointers.

---

## UFWItemDefinition

**Header:** `FWItemDefinition.h` | **Parent:** `UPrimaryDataAsset`

The base data asset that defines an item's properties, stats, visual representation, and affix pool. Every item instance references a single item definition.

```cpp
UCLASS(BlueprintType)
class FWINVENTORYSYSTEM_API UFWItemDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    // --- Identity ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Identity")
    FName ItemId;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Identity")
    FText DisplayName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Identity",
        meta = (MultiLine = "true"))
    FText Description;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Identity")
    TSoftObjectPtr<UTexture2D> Icon;

    // --- Classification ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Classification")
    EFWItemType ItemType;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Classification",
        meta = (EditCondition = "ItemType == EFWItemType::Equipment"))
    EFWEquipmentType EquipmentType;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Classification")
    EFWItemQuality Quality;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Classification")
    EFWItemBinding Binding;

    // --- Stacking ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Stacking")
    bool bIsStackable = false;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Stacking",
        meta = (EditCondition = "bIsStackable"))
    int32 MaxStackSize = 999;

    // --- Item Level ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Level")
    int32 RequiredLevel = 1;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Level")
    int32 ItemLevel = 1;

    // --- Base Stats (Equipment only) ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Stats",
        meta = (EditCondition = "ItemType == EFWItemType::Equipment"))
    TMap<FGameplayTag, float> BaseStats;

    // --- Durability ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Durability")
    bool bHasDurability = false;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Durability",
        meta = (EditCondition = "bHasDurability"))
    float MaxDurability = 100.0f;

    // --- Economy ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Economy")
    int32 BasePrice = 0;

    // --- Affix Pool ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Affixes")
    TArray<FFWAffixRule> AffixPool;

    // --- Visuals ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Visuals",
        meta = (EditCondition = "ItemType == EFWItemType::Equipment"))
    TSoftObjectPtr<USkeletalMesh> EquipmentMesh;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Visuals",
        meta = (EditCondition = "ItemType == EFWItemType::Equipment"))
    TSoftObjectPtr<UStaticMesh> WorldMesh;

    // --- Set Membership ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Set")
    TSoftObjectPtr<UFWItemSetDefinition> ItemSet;

    // --- Perk ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Perk")
    TSoftObjectPtr<UFWPerkDefinition> IntrinsicPerk;

    // --- Tags ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Item|Tags")
    FGameplayTagContainer ItemTags;
};
```

### Creating an Item Definition

1. In the Content Browser, right-click and select **Miscellaneous > Data Asset**.
2. Choose `FWItemDefinition` as the parent class.
3. Name it with a `DA_` prefix (e.g., `DA_IronSword`).
4. Configure all properties in the Details panel.

!!! tip "Asset Organization"
    Organize item definitions in folders by type:
    ```
    Content/
    └── Items/
        ├── Weapons/
        │   ├── DA_IronSword.uasset
        │   └── DA_SteelAxe.uasset
        ├── Armor/
        │   ├── DA_LeatherHelmet.uasset
        │   └── DA_ChainChest.uasset
        ├── Consumables/
        │   └── DA_HealthPotion.uasset
        └── Materials/
            └── DA_IronOre.uasset
    ```

---

## UFWItemSetDefinition

**Header:** `FWItemSetDefinition.h` | **Parent:** `UPrimaryDataAsset`

Defines an item set with tiered bonuses that activate when a player equips a certain number of set pieces.

```cpp
UCLASS(BlueprintType)
class FWINVENTORYSYSTEM_API UFWItemSetDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Set")
    FText SetName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Set")
    FText SetDescription;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Set")
    TArray<TSoftObjectPtr<UFWItemDefinition>> SetPieces;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Set")
    TArray<FFWSetBonusTier> BonusTiers;
};
```

### FFWSetBonusTier

```cpp
USTRUCT(BlueprintType)
struct FFWSetBonusTier
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Set")
    int32 RequiredPieces;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Set")
    FText BonusDescription;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Set")
    TMap<FGameplayTag, float> BonusStats;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Set")
    TSubclassOf<UGameplayEffect> BonusEffect;
};
```

**Example set definition:**

| Pieces | Bonus |
|---|---|
| 2 | +10% Critical Chance |
| 4 | +25 Strength, +15% Attack Speed |
| 6 | Unique proc effect (Gameplay Effect) |

---

## UFWPerkDefinition

**Header:** `FWPerkDefinition.h` | **Parent:** `UPrimaryDataAsset`

Defines a passive perk that can be intrinsic to an item or granted by other game systems.

```cpp
UCLASS(BlueprintType)
class FWINVENTORYSYSTEM_API UFWPerkDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Perk")
    FName PerkId;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Perk")
    FText PerkName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Perk")
    FText PerkDescription;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Perk")
    TSoftObjectPtr<UTexture2D> PerkIcon;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Perk")
    TSubclassOf<UGameplayEffect> PerkEffect;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Perk")
    TMap<FGameplayTag, float> PerkStats;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Perk")
    FGameplayTagContainer PerkTags;
};
```

---

## UFWRecipeDefinition

**Header:** `FWRecipeDefinition.h` | **Parent:** `UPrimaryDataAsset`

Defines a crafting recipe with ingredients, output, skill requirements, and crafting time.

```cpp
UCLASS(BlueprintType)
class FWINVENTORYSYSTEM_API UFWRecipeDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe")
    FName RecipeId;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe")
    FText RecipeName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe")
    FText RecipeDescription;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe")
    TSoftObjectPtr<UTexture2D> RecipeIcon;

    // --- Ingredients ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Ingredients")
    TArray<FFWRecipeIngredient> Ingredients;

    // --- Output ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Output")
    TSoftObjectPtr<UFWItemDefinition> OutputItem;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Output")
    int32 OutputQuantity = 1;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Output")
    EFWItemQuality OutputQuality = EFWItemQuality::Common;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Output")
    bool bGenerateAffixes = false;

    // --- Requirements ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Requirements")
    bool bRequiresLearning = true;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Requirements")
    FGameplayTag RequiredSkillTag;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Requirements")
    int32 RequiredSkillLevel = 0;

    // --- Crafting ---
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Crafting")
    float CraftingTimeSeconds = 0.0f;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipe|Crafting")
    int32 SkillXPReward = 0;
};
```

---

## UFWRecipeDatabase

**Header:** `FWRecipeDatabase.h` | **Parent:** `UPrimaryDataAsset`

A collection asset that aggregates recipes for use by the crafting component.

```cpp
UCLASS(BlueprintType)
class FWINVENTORYSYSTEM_API UFWRecipeDatabase : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Recipes")
    TArray<TSoftObjectPtr<UFWRecipeDefinition>> Recipes;

    UFUNCTION(BlueprintPure, Category = "Recipes")
    TArray<UFWRecipeDefinition*> GetAllRecipes() const;

    UFUNCTION(BlueprintPure, Category = "Recipes")
    UFWRecipeDefinition* FindRecipeById(FName RecipeId) const;

    UFUNCTION(BlueprintPure, Category = "Recipes")
    TArray<UFWRecipeDefinition*> FindRecipesByOutput(
        const UFWItemDefinition* OutputItem) const;

    UFUNCTION(BlueprintPure, Category = "Recipes")
    TArray<UFWRecipeDefinition*> FindRecipesBySkill(
        FGameplayTag SkillTag) const;
};
```

### Creating a Recipe Database

1. Create a `FWRecipeDatabase` data asset in the Content Browser.
2. Add references to all `FWRecipeDefinition` assets.
3. Assign the database to the crafting component's `RecipeDatabase` property.

```
Content/
└── Crafting/
    ├── DB_AllRecipes.uasset           // Recipe database
    ├── Recipes/
    │   ├── DA_Recipe_IronSword.uasset
    │   ├── DA_Recipe_HealthPotion.uasset
    │   └── DA_Recipe_LeatherHelmet.uasset
    └── ...
```

---

## Asset Reference Patterns

### Soft References

All cross-asset references use `TSoftObjectPtr` to avoid hard dependencies and allow streaming:

```cpp
// In code, load on demand
if (UFWItemDefinition* Def = ItemSet->SetPieces[0].LoadSynchronous())
{
    // Use the definition
}

// Async load
FStreamableManager& Streamable = UAssetManager::GetStreamableManager();
Streamable.RequestAsyncLoad(ItemDef.ToSoftObjectPath(),
    FStreamableDelegate::CreateLambda([this]()
    {
        // Asset loaded
    }));
```

### Asset Manager Registration

Register item definitions with the Asset Manager for efficient discovery:

```ini
; DefaultEngine.ini
[/Script/Engine.AssetManagerSettings]
+PrimaryAssetTypesToScan=(PrimaryAssetType="FWItemDefinition",AssetBaseClass="/Script/FWInventorySystem.FWItemDefinition",bHasBlueprintClasses=false,Directories=((Path="/Game/Items")))
+PrimaryAssetTypesToScan=(PrimaryAssetType="FWRecipeDefinition",AssetBaseClass="/Script/FWInventorySystem.FWRecipeDefinition",bHasBlueprintClasses=false,Directories=((Path="/Game/Crafting/Recipes")))
```
