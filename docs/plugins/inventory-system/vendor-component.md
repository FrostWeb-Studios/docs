---
title: UFWVendorComponent
description: NPC vendor component with buy, sell, buyback, and dynamic inventory management.
---

# UFWVendorComponent

**Header:** `FWVendorComponent.h` | **Parent:** `UActorComponent`

Provides NPC vendor functionality including a configurable shop inventory, buy/sell transactions with price calculation, a buyback buffer for recent sales, and currency validation. Placed on NPC actors that serve as merchants.

---

## Class Declaration

```cpp
UCLASS(ClassGroup = (FrostWeb), meta = (BlueprintSpawnableComponent))
class FWINVENTORYSYSTEM_API UFWVendorComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFWVendorComponent();

    // --- Transactions ---
    UFUNCTION(BlueprintCallable, Category = "Vendor")
    bool BuyItem(UFWInventoryComponent* BuyerInventory,
        int32 VendorSlotIndex, int32 Quantity = 1);

    UFUNCTION(BlueprintCallable, Category = "Vendor")
    bool SellItem(UFWInventoryComponent* SellerInventory,
        const FGuid& ItemInstanceId, int32 Quantity = 1);

    UFUNCTION(BlueprintCallable, Category = "Vendor")
    bool BuybackItem(UFWInventoryComponent* BuyerInventory,
        int32 BuybackSlotIndex);

    // --- Price Queries ---
    UFUNCTION(BlueprintPure, Category = "Vendor")
    int32 GetBuyPrice(int32 VendorSlotIndex) const;

    UFUNCTION(BlueprintPure, Category = "Vendor")
    int32 GetSellPrice(const FFWItemInstance& Item) const;

    // --- Vendor Inventory ---
    UFUNCTION(BlueprintPure, Category = "Vendor")
    const TArray<FFWItemInstance>& GetVendorInventory() const;

    UFUNCTION(BlueprintPure, Category = "Vendor")
    const TArray<FFWItemInstance>& GetBuybackInventory() const;

    UFUNCTION(BlueprintCallable, Category = "Vendor")
    void RefreshVendorInventory();

    // --- Configuration ---
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Vendor|Config")
    TArray<UFWItemDefinition*> StockDefinitions;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Vendor|Config")
    float BuyPriceMultiplier = 1.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Vendor|Config")
    float SellPriceMultiplier = 0.25f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Vendor|Config")
    int32 MaxBuybackSlots = 12;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Vendor|Config")
    bool bHasLimitedStock = false;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Vendor|Config",
        Meta = (EditCondition = "bHasLimitedStock"))
    float RestockIntervalSeconds = 3600.0f;

    // --- Delegates ---
    UPROPERTY(BlueprintAssignable, Category = "Vendor|Events")
    FOnItemPurchased OnItemPurchased;

    UPROPERTY(BlueprintAssignable, Category = "Vendor|Events")
    FOnItemSold OnItemSold;

    UPROPERTY(BlueprintAssignable, Category = "Vendor|Events")
    FOnTransactionFailed OnTransactionFailed;

    UPROPERTY(BlueprintAssignable, Category = "Vendor|Events")
    FOnVendorInventoryRefreshed OnVendorInventoryRefreshed;
};
```

---

## Functions

### BuyItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Vendor")
bool BuyItem(UFWInventoryComponent* BuyerInventory,
    int32 VendorSlotIndex, int32 Quantity = 1);
```

Purchases an item from the vendor. Deducts currency from the buyer's inventory and adds the purchased item.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `BuyerInventory` | `UFWInventoryComponent*` | | The buyer's inventory component. |
| `VendorSlotIndex` | `int32` | | Index in the vendor's stock list. |
| `Quantity` | `int32` | `1` | Number to purchase. |

**Returns:** `true` if the transaction succeeded.

**Validation:**

1. Vendor slot index is valid.
2. Buyer has enough currency.
3. Buyer has inventory space.
4. If `bHasLimitedStock`, vendor has sufficient quantity.

---

### SellItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Vendor")
bool SellItem(UFWInventoryComponent* SellerInventory,
    const FGuid& ItemInstanceId, int32 Quantity = 1);
```

Sells an item from the player's inventory to the vendor. Removes the item, adds currency, and places the item in the buyback buffer.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `SellerInventory` | `UFWInventoryComponent*` | | The seller's inventory component. |
| `ItemInstanceId` | `FGuid` | | Instance ID of the item to sell. |
| `Quantity` | `int32` | `1` | Number to sell from the stack. |

!!! warning "Bound Items"
    Items with `EFWItemBinding::BindOnPickup` or `EFWItemBinding::AccountBound` cannot be sold. The transaction will fail with `OnTransactionFailed`.

---

### BuybackItem

```cpp
UFUNCTION(BlueprintCallable, Category = "Vendor")
bool BuybackItem(UFWInventoryComponent* BuyerInventory,
    int32 BuybackSlotIndex);
```

Repurchases an item from the buyback buffer at the original sell price.

| Parameter | Type | Description |
|---|---|---|
| `BuyerInventory` | `UFWInventoryComponent*` | The buyer's inventory component. |
| `BuybackSlotIndex` | `int32` | Index in the buyback buffer. |

---

### Price Calculation

```cpp
UFUNCTION(BlueprintPure, Category = "Vendor")
int32 GetBuyPrice(int32 VendorSlotIndex) const;

UFUNCTION(BlueprintPure, Category = "Vendor")
int32 GetSellPrice(const FFWItemInstance& Item) const;
```

Prices are calculated based on the item definition's `BasePrice` and the vendor's multipliers:

- **Buy price:** `BasePrice * BuyPriceMultiplier * QualityMultiplier`
- **Sell price:** `BasePrice * SellPriceMultiplier * QualityMultiplier * DurabilityFactor`

Quality multipliers by `EFWItemQuality`:

| Quality | Multiplier |
|---|---|
| Poor | 0.5 |
| Common | 1.0 |
| Uncommon | 1.5 |
| Rare | 2.5 |
| Epic | 5.0 |
| Legendary | 10.0 |
| Mythic | 25.0 |
| Artifact | 50.0 |

---

## Configuration Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `StockDefinitions` | `TArray<UFWItemDefinition*>` | `[]` | Item definitions this vendor sells. Set in the editor. |
| `BuyPriceMultiplier` | `float` | `1.0` | Multiplier applied to base price for purchases. |
| `SellPriceMultiplier` | `float` | `0.25` | Multiplier applied to base price for sales. Players get 25% by default. |
| `MaxBuybackSlots` | `int32` | `12` | Maximum items in the buyback buffer. Oldest are evicted when full. |
| `bHasLimitedStock` | `bool` | `false` | Whether the vendor has finite stock quantities. |
| `RestockIntervalSeconds` | `float` | `3600.0` | Time between automatic restocks (only if `bHasLimitedStock`). |

---

## Delegates

### OnItemPurchased

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnItemPurchased,
    const FFWItemInstance&, Item,
    int32, Price,
    int32, Quantity);
```

### OnItemSold

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(
    FOnItemSold,
    const FFWItemInstance&, Item,
    int32, Price,
    int32, Quantity);
```

### OnTransactionFailed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnTransactionFailed,
    const FString&, Reason);
```

### OnVendorInventoryRefreshed

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnVendorInventoryRefreshed);
```

Fires when the vendor's stock is refreshed (restocked or manually refreshed).

---

## Usage Example

=== "C++"

    ```cpp
    void AMyNPCVendor::BeginPlay()
    {
        Super::BeginPlay();

        VendorComp = FindComponentByClass<UFWVendorComponent>();
        VendorComp->BuyPriceMultiplier = 1.2f;    // 20% markup
        VendorComp->SellPriceMultiplier = 0.30f;   // 30% return
        VendorComp->MaxBuybackSlots = 8;
    }

    void AMyNPCVendor::OnPlayerInteract(AMyCharacter* Player)
    {
        UFWInventoryComponent* PlayerInventory =
            Player->FindComponentByClass<UFWInventoryComponent>();

        // Open vendor UI with vendor and player inventory references
        UVendorWidget* Widget = CreateWidget<UVendorWidget>(
            Player->GetController<APlayerController>(), VendorWidgetClass);
        Widget->Initialize(VendorComp, PlayerInventory);
        Widget->AddToViewport();
    }
    ```

=== "Blueprint"

    1. Add **FW Vendor Component** to your NPC actor.
    2. In the Details panel, add item definitions to the **Stock Definitions** array.
    3. Set **Buy Price Multiplier** and **Sell Price Multiplier** as desired.
    4. On player interaction, open a vendor UI widget that references the vendor component.
    5. Bind **On Transaction Failed** to show error messages.
