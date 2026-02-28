---
title: "Tutorial: Creating a Loot System"
description: Step-by-step guide to implementing a loot system with item definitions, affix pools, equipment slots, vendor NPCs, and inventory serialization.
---

# Tutorial: Creating a Loot System

This tutorial walks through implementing a complete loot system using the FWInventorySystem plugin. You will define items with affix pools, configure equipment slots, create vendor NPCs, and implement inventory save/load with JSON serialization.

---

## Prerequisites

- Unreal Engine 5.3 or later
- FWInventorySystem plugin enabled
- GameplayAbilities plugin enabled
- A character class with an `AbilitySystemComponent`
- Gameplay tags configured (see [Configuration](configuration.md))

---

## Step 1: Define Item Definitions

### Creating a Weapon Definition

Create a data asset for a weapon with an affix pool.

=== "Editor"

    1. Right-click in the Content Browser: **Miscellaneous > Data Asset**.
    2. Select `FWItemDefinition` as the class. Name it `DA_IronSword`.
    3. Configure the following properties:

    | Property | Value |
    |---|---|
    | Item Id | `iron_sword` |
    | Display Name | `Iron Sword` |
    | Description | `A sturdy iron blade.` |
    | Item Type | `Equipment` |
    | Equipment Type | `MeleeWeapon` |
    | Quality | `Common` |
    | Binding | `BindOnEquip` |
    | Required Level | `1` |
    | Item Level | `5` |
    | Has Durability | `true` |
    | Max Durability | `100` |
    | Base Price | `50` |

    4. Under **Base Stats**, add:
        - `Stat.Strength` = `10.0`
        - `Stat.CriticalChance` = `2.0`

=== "C++"

    ```cpp
    // Item definitions are created as data assets in the editor.
    // Reference them via soft pointers in your loot tables:
    UPROPERTY(EditAnywhere, Category = "Loot")
    TSoftObjectPtr<UFWItemDefinition> IronSwordDef;
    ```

### Configuring the Affix Pool

Add affix rules to the item definition's `AffixPool` array. These define what random stats can roll on the item.

In the `DA_IronSword` data asset, expand **Affix Pool** and add the following entries:

| Stat Tag | Min | Max | Percentage | Weight | Required Level | Exclusion Tags |
|---|---|---|---|---|---|---|
| `Stat.Strength` | 3.0 | 15.0 | No | 10.0 | 1 | -- |
| `Stat.CriticalChance` | 1.0 | 5.0 | Yes | 8.0 | 1 | -- |
| `Stat.CriticalDamage` | 5.0 | 20.0 | Yes | 6.0 | 5 | -- |
| `Stat.AttackSpeed` | 2.0 | 8.0 | Yes | 7.0 | 1 | `Stat.MoveSpeed` |
| `Stat.MoveSpeed` | 1.0 | 5.0 | Yes | 4.0 | 1 | `Stat.AttackSpeed` |
| `Stat.Vitality` | 5.0 | 20.0 | No | 5.0 | 3 | -- |

!!! info "How Affix Rolling Works"
    When an item is generated:

    1. The maximum number of affixes is determined by `EFWItemQuality` (Common = 0, Uncommon = 1, Rare = 2, etc.).
    2. Eligible affixes are filtered by `RequiredItemLevel`.
    3. An affix is selected using weighted random selection.
    4. The value is rolled uniformly between `MinValue` and `MaxValue`.
    5. Exclusion tags are checked -- if a selected affix's `StatTag` appears in another affix's `ExclusionTags`, both cannot coexist.
    6. Repeat until the affix count is reached.

### Creating More Item Types

Create additional data assets for different item types:

**DA_HealthPotion (Consumable):**

| Property | Value |
|---|---|
| Item Type | `Consumable` |
| Is Stackable | `true` |
| Max Stack Size | `20` |
| Base Price | `10` |

**DA_IronOre (Material):**

| Property | Value |
|---|---|
| Item Type | `Material` |
| Is Stackable | `true` |
| Max Stack Size | `999` |
| Base Price | `2` |

---

## Step 2: Set Up Inventory and Equipment Components

Add the inventory system components to your character.

### Header File

```cpp
// MyCharacter.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "AbilitySystemInterface.h"
#include "FWInventoryComponent.h"
#include "FWEquipmentComponent.h"
#include "FWCraftingComponent.h"
#include "MyCharacter.generated.h"

UCLASS()
class MYGAME_API AMyCharacter : public ACharacter, public IAbilitySystemInterface
{
    GENERATED_BODY()

public:
    AMyCharacter();

    virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;

protected:
    virtual void BeginPlay() override;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Inventory")
    TObjectPtr<UFWInventoryComponent> Inventory;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Inventory")
    TObjectPtr<UFWEquipmentComponent> Equipment;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Inventory")
    TObjectPtr<UFWCraftingComponent> Crafting;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Abilities")
    TObjectPtr<UAbilitySystemComponent> AbilitySystem;

private:
    UFUNCTION()
    void HandleItemAdded(const FFWItemInstance& Item, int32 SlotIndex);

    UFUNCTION()
    void HandleEquipped(EFWEquipmentType Slot, const FFWItemInstance& Item);

    UFUNCTION()
    void HandleInventoryFull(const FFWItemInstance& RejectedItem);
};
```

### Source File

```cpp
// MyCharacter.cpp
#include "MyCharacter.h"

AMyCharacter::AMyCharacter()
{
    bReplicates = true;

    AbilitySystem = CreateDefaultSubobject<UAbilitySystemComponent>(
        TEXT("AbilitySystem"));
    Inventory = CreateDefaultSubobject<UFWInventoryComponent>(
        TEXT("Inventory"));
    Equipment = CreateDefaultSubobject<UFWEquipmentComponent>(
        TEXT("Equipment"));
    Crafting = CreateDefaultSubobject<UFWCraftingComponent>(
        TEXT("Crafting"));
}

UAbilitySystemComponent* AMyCharacter::GetAbilitySystemComponent() const
{
    return AbilitySystem;
}

void AMyCharacter::BeginPlay()
{
    Super::BeginPlay();

    Inventory->OnItemAdded.AddDynamic(
        this, &AMyCharacter::HandleItemAdded);
    Inventory->OnInventoryFull.AddDynamic(
        this, &AMyCharacter::HandleInventoryFull);
    Equipment->OnItemEquipped.AddDynamic(
        this, &AMyCharacter::HandleEquipped);
}

void AMyCharacter::HandleItemAdded(
    const FFWItemInstance& Item, int32 SlotIndex)
{
    UE_LOG(LogInventory, Log, TEXT("Picked up: %s (x%d) in slot %d"),
        *Item.Definition->DisplayName.ToString(),
        Item.Quantity, SlotIndex);
}

void AMyCharacter::HandleEquipped(
    EFWEquipmentType Slot, const FFWItemInstance& Item)
{
    UE_LOG(LogInventory, Log, TEXT("Equipped %s in %s slot"),
        *Item.Definition->DisplayName.ToString(),
        *UEnum::GetValueAsString(Slot));
}

void AMyCharacter::HandleInventoryFull(const FFWItemInstance& RejectedItem)
{
    // Show "inventory full" notification to player
    UE_LOG(LogInventory, Warning, TEXT("Inventory full! Cannot pick up %s"),
        *RejectedItem.Definition->DisplayName.ToString());
}
```

---

## Step 3: Implement a Loot Drop System

Create a loot table that generates item instances with random affixes.

```cpp
// LootGenerator.h
UCLASS(BlueprintType, Blueprintable)
class MYGAME_API ULootGenerator : public UObject
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Loot")
    static TArray<FFWItemInstance> GenerateLoot(
        const TArray<FFWLootEntry>& LootTable,
        int32 PlayerLevel,
        int32 DropCount);

    UFUNCTION(BlueprintCallable, Category = "Loot")
    static FFWItemInstance GenerateItemInstance(
        UFWItemDefinition* Definition,
        EFWItemQuality Quality,
        int32 ItemLevel);

private:
    static TArray<FFWAffixInstance> RollAffixes(
        const TArray<FFWAffixRule>& AffixPool,
        int32 MaxAffixes,
        int32 ItemLevel);
};
```

### Affix Rolling Implementation

```cpp
// LootGenerator.cpp
FFWItemInstance ULootGenerator::GenerateItemInstance(
    UFWItemDefinition* Definition,
    EFWItemQuality Quality,
    int32 ItemLevel)
{
    FFWItemInstance Instance;
    Instance.InstanceId = FGuid::NewGuid();
    Instance.Definition = Definition;
    Instance.Quantity = 1;
    Instance.ItemLevel = ItemLevel;
    Instance.AcquiredAt = FDateTime::UtcNow();

    // Set durability from definition
    if (Definition->bHasDurability)
    {
        Instance.Durability = Definition->MaxDurability;
    }

    // Determine max affixes based on quality
    static const int32 QualityAffixCounts[] =
        { 0, 0, 1, 2, 3, 4, 5, 6 };  // Poor through Artifact
    int32 MaxAffixes = QualityAffixCounts[static_cast<uint8>(Quality)];

    // Roll affixes from the item's affix pool
    if (MaxAffixes > 0 && Definition->AffixPool.Num() > 0)
    {
        Instance.Affixes = RollAffixes(
            Definition->AffixPool, MaxAffixes, ItemLevel);
    }

    return Instance;
}

TArray<FFWAffixInstance> ULootGenerator::RollAffixes(
    const TArray<FFWAffixRule>& AffixPool,
    int32 MaxAffixes,
    int32 ItemLevel)
{
    TArray<FFWAffixInstance> Result;
    TArray<FFWAffixRule> EligiblePool;
    FGameplayTagContainer UsedExclusions;

    // Filter by item level requirement
    for (const FFWAffixRule& Rule : AffixPool)
    {
        if (Rule.RequiredItemLevel <= ItemLevel)
        {
            EligiblePool.Add(Rule);
        }
    }

    for (int32 i = 0; i < MaxAffixes && EligiblePool.Num() > 0; ++i)
    {
        // Calculate total weight
        float TotalWeight = 0.0f;
        for (const FFWAffixRule& Rule : EligiblePool)
        {
            TotalWeight += Rule.Weight;
        }

        // Weighted random selection
        float Roll = FMath::FRandRange(0.0f, TotalWeight);
        float Accumulated = 0.0f;
        int32 SelectedIndex = -1;

        for (int32 j = 0; j < EligiblePool.Num(); ++j)
        {
            Accumulated += EligiblePool[j].Weight;
            if (Roll <= Accumulated)
            {
                SelectedIndex = j;
                break;
            }
        }

        if (SelectedIndex < 0) continue;

        const FFWAffixRule& Selected = EligiblePool[SelectedIndex];

        // Roll value within range, scaled by item level
        float ScaleFactor = 1.0f + (ItemLevel * 0.02f);
        float Value = FMath::FRandRange(
            Selected.MinValue * ScaleFactor,
            Selected.MaxValue * ScaleFactor);

        // Create affix instance
        FFWAffixInstance Affix;
        Affix.StatTag = Selected.StatTag;
        Affix.Value = FMath::RoundToFloat(Value * 10.0f) / 10.0f;
        Affix.bIsPercentage = Selected.bIsPercentage;
        Result.Add(Affix);

        // Add exclusion tags and remove conflicting entries
        UsedExclusions.AppendTags(Selected.ExclusionTags);
        EligiblePool.RemoveAt(SelectedIndex);

        // Remove entries excluded by the selected affix
        EligiblePool.RemoveAll([&UsedExclusions](const FFWAffixRule& Rule)
        {
            return UsedExclusions.HasTag(Rule.StatTag);
        });
    }

    return Result;
}
```

---

## Step 4: Set Up Equipment Slots

### Configuring Slots

In `DefaultGame.ini`, define which equipment slots are active and their counts:

```ini
[/Script/FWInventorySystem.FWInventorySystemSettings]
DefaultInventoryCapacity=40
```

### Equipping Items from Inventory

```cpp
void AMyCharacter::TryEquipItem(const FGuid& InstanceId)
{
    const FFWItemInstance* Item = Inventory->FindItemByDefinition(nullptr);
    // Use the EquipItem shortcut which handles both inventory removal
    // and equipment slot placement
    if (Inventory->EquipItem(InstanceId))
    {
        UE_LOG(LogInventory, Log, TEXT("Item equipped successfully."));
    }
}
```

### Equipment UI

```cpp
void UEquipmentWidget::RefreshSlots()
{
    auto* Equip = GetCharacter()->FindComponentByClass<UFWEquipmentComponent>();

    TArray<EFWEquipmentType> SlotTypes = Equip->GetAllSlotTypes();
    for (EFWEquipmentType Slot : SlotTypes)
    {
        UEquipmentSlotWidget* SlotWidget = SlotWidgets.FindRef(Slot);
        if (!SlotWidget) continue;

        const FFWItemInstance* Equipped = Equip->GetEquippedItem(Slot);
        if (Equipped && Equipped->IsValid())
        {
            SlotWidget->SetItem(*Equipped);

            // Display affixes
            FString AffixText;
            for (const FFWAffixInstance& Affix : Equipped->Affixes)
            {
                AffixText += FString::Printf(TEXT("+%.1f%s %s\n"),
                    Affix.Value,
                    Affix.bIsPercentage ? TEXT("%") : TEXT(""),
                    *Affix.StatTag.ToString());
            }
            SlotWidget->SetAffixText(AffixText);
        }
        else
        {
            SlotWidget->ClearItem();
        }
    }
}
```

---

## Step 5: Create a Vendor NPC

### Vendor Actor Setup

```cpp
// MyVendorNPC.h
UCLASS()
class MYGAME_API AMyVendorNPC : public ACharacter
{
    GENERATED_BODY()

public:
    AMyVendorNPC();

    UFUNCTION(BlueprintCallable, Category = "Vendor")
    void OnPlayerInteract(AMyCharacter* Player);

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Vendor")
    TObjectPtr<UFWVendorComponent> VendorComp;
};

// MyVendorNPC.cpp
AMyVendorNPC::AMyVendorNPC()
{
    VendorComp = CreateDefaultSubobject<UFWVendorComponent>(TEXT("Vendor"));
    VendorComp->BuyPriceMultiplier = 1.0f;
    VendorComp->SellPriceMultiplier = 0.25f;
    VendorComp->MaxBuybackSlots = 12;
    VendorComp->bHasLimitedStock = false;
}
```

### Configuring Vendor Stock

In the editor, select the vendor actor and configure its `UFWVendorComponent`:

1. Expand **Stock Definitions** in the Details panel.
2. Add references to item definitions the vendor should sell.
3. Set **Buy Price Multiplier** (e.g., `1.2` for 20% markup).
4. Set **Sell Price Multiplier** (e.g., `0.3` for 30% return).

### Vendor Transaction Flow

```cpp
void UVendorWidget::OnBuyButtonClicked(int32 SlotIndex)
{
    int32 Price = VendorComp->GetBuyPrice(SlotIndex);

    // Check player currency
    if (PlayerInventory->GetItemCount(CurrencyDefinition) < Price)
    {
        ShowNotification(TEXT("Not enough gold."));
        return;
    }

    if (VendorComp->BuyItem(PlayerInventory, SlotIndex))
    {
        PlayPurchaseSound();
        RefreshUI();
    }
}

void UVendorWidget::OnSellButtonClicked(const FGuid& ItemId)
{
    if (VendorComp->SellItem(PlayerInventory, ItemId))
    {
        PlaySellSound();
        RefreshUI();
    }
}

void UVendorWidget::OnBuybackClicked(int32 BuybackSlot)
{
    if (VendorComp->BuybackItem(PlayerInventory, BuybackSlot))
    {
        PlayPurchaseSound();
        RefreshUI();
    }
}
```

---

## Step 6: Implement Inventory Save/Load

The `UFWInventorySerializationLibrary` provides JSON serialization for persisting inventory state to a database.

### Saving Inventory

```cpp
void AMyCharacter::SaveInventory()
{
    // Serialize inventory to JSON
    FString InventoryJson =
        UFWInventorySerializationLibrary::SerializeInventory(Inventory);
    FString EquipmentJson =
        UFWInventorySerializationLibrary::SerializeEquipment(Equipment);

    // Send to backend API for database storage
    FHttpModule& Http = FHttpModule::Get();
    TSharedRef<IHttpRequest> Request = Http.CreateRequest();

    Request->SetURL(TEXT("https://ayndora.frostweb.dev/api/v1/inventory/save"));
    Request->SetVerb(TEXT("POST"));
    Request->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"),
        FString::Printf(TEXT("Bearer %s"), *AuthToken));

    TSharedPtr<FJsonObject> Body = MakeShareable(new FJsonObject());
    Body->SetStringField(TEXT("inventory"), InventoryJson);
    Body->SetStringField(TEXT("equipment"), EquipmentJson);

    FString BodyString;
    TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&BodyString);
    FJsonSerializer::Serialize(Body.ToSharedRef(), Writer);

    Request->SetContentAsString(BodyString);
    Request->ProcessRequest();
}
```

### Loading Inventory

```cpp
void AMyCharacter::LoadInventory(const FString& InventoryJson,
    const FString& EquipmentJson)
{
    // Validate data before loading
    if (!UFWInventorySerializationLibrary::ValidateSerializedData(InventoryJson))
    {
        UE_LOG(LogInventory, Error, TEXT("Invalid inventory data."));
        return;
    }

    if (UFWInventorySerializationLibrary::DeserializeInventory(
            Inventory, InventoryJson))
    {
        UE_LOG(LogInventory, Log, TEXT("Inventory loaded: %d items"),
            Inventory->GetUsedSlots());
    }

    if (UFWInventorySerializationLibrary::DeserializeEquipment(
            Equipment, EquipmentJson))
    {
        UE_LOG(LogInventory, Log, TEXT("Equipment loaded."));
    }
}
```

### JSON Format

The serialized JSON format for a single item instance:

```json
{
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "definitionPath": "/Game/Items/Weapons/DA_IronSword",
    "quantity": 1,
    "durability": 85.5,
    "itemLevel": 12,
    "augmentState": "Augmented_2",
    "augmentLevel": 2,
    "boundToPlayerId": "player_123",
    "acquiredAt": "2026-01-15T10:30:00Z",
    "affixes": [
        {
            "statTag": "Stat.Strength",
            "value": 12.5,
            "isPercentage": false
        },
        {
            "statTag": "Stat.CriticalChance",
            "value": 3.2,
            "isPercentage": true
        }
    ]
}
```

The full inventory serialization wraps items in an array with metadata:

```json
{
    "version": 1,
    "capacity": 40,
    "items": [
        { "slot": 0, "item": { ... } },
        { "slot": 3, "item": { ... } }
    ]
}
```

---

## Step 7: Loot Drop Integration

Tie the loot system to enemy deaths.

```cpp
void AMyEnemy::OnDeath()
{
    // Generate loot based on enemy level and loot table
    TArray<FFWItemInstance> Drops = ULootGenerator::GenerateLoot(
        LootTable, EnemyLevel, FMath::RandRange(1, 3));

    // Spawn loot actors in the world
    for (const FFWItemInstance& Drop : Drops)
    {
        FVector SpawnLoc = GetActorLocation()
            + FMath::VRand() * FMath::FRandRange(50.0f, 150.0f);

        AWorldItem* WorldItem = GetWorld()->SpawnActor<AWorldItem>(
            WorldItemClass, SpawnLoc, FRotator::ZeroRotator);
        WorldItem->SetItemInstance(Drop);
    }
}
```

---

## Complete Setup Checklist

- [ ] GameplayAbilities plugin enabled
- [ ] FWInventorySystem plugin enabled
- [ ] Gameplay tags configured for stats
- [ ] Item definitions created as data assets
- [ ] Affix pools configured on equipment definitions
- [ ] Inventory and Equipment components added to character
- [ ] AbilitySystemComponent on character (for stat application)
- [ ] Delegates bound in `BeginPlay`
- [ ] Loot generator with weighted affix rolling
- [ ] Equipment UI showing slots and affix details
- [ ] Vendor NPC with stock configuration
- [ ] Buy/sell/buyback UI flow
- [ ] Inventory serialization to JSON
- [ ] Backend API integration for save/load
- [ ] Crafting component with recipe database (optional)
- [ ] Item set definitions with tiered bonuses (optional)

---

## Next Steps

- Review the [Data Assets](data-assets.md) page for set bonuses and perk definitions.
- See the [Equipment Component](equipment-component.md) documentation for stat application details.
- Read the [Events and Delegates](events-delegates.md) reference for all available events.
- Check [Configuration](configuration.md) for augmentation settings and scaling options.
