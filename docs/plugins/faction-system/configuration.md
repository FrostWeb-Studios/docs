---
title: Configuration - Faction System
---

# Configuration

FWFactionSystem is configured through three mechanisms: **Project Settings**, **Faction Definition DataAssets**, and **AssetManager registration**.

---

## Project Settings

Navigate to **Project Settings > Game > Faction System**.

### Persistence Handler Class

| Setting | Type | Default |
|---------|------|---------|
| Persistence Handler Class | `TSoftClassPtr<UFWFactionPersistenceHandler>` | Empty (disabled) |

Points to a subclass of `UFWFactionPersistenceHandler` that implements save/load for your backend. Leave empty to disable persistence entirely (faction data will be runtime-only).

!!! info "Creating a Persistence Handler"
    1. Create a new C++ or Blueprint class inheriting from `UFWFactionPersistenceHandler`.
    2. Override `SavePlayerFactionData` and `LoadPlayerFactionData`.
    3. Set the class in Project Settings.
    4. The subsystem instantiates it automatically on world initialization.

    ```cpp
    UCLASS()
    class UMyFactionPersistence : public UFWFactionPersistenceHandler
    {
        GENERATED_BODY()

    public:
        virtual void SavePlayerFactionData(
            const FString& PlayerId,
            const FFWFactionPlayerData& Data,
            FOnFactionSaveComplete OnComplete) override
        {
            // Your API call here
            // Call OnComplete when done
            OnComplete.ExecuteIfBound(EFWFactionOperationResult::Success);
        }

        virtual void LoadPlayerFactionData(
            const FString& PlayerId,
            FOnFactionLoadComplete OnComplete) override
        {
            // Your API call here
            FFWFactionPlayerData Data;
            Data.PrimaryFactionId = TEXT("Ironclad");
            OnComplete.ExecuteIfBound(EFWFactionOperationResult::Success, Data);
        }
    };
    ```

---

## AssetManager Registration

FWFactionSystem relies on the AssetManager to discover faction definitions at startup. This must be configured once per project.

Navigate to **Project Settings > Game > Asset Manager** and add:

| Field | Value |
|-------|-------|
| Primary Asset Type | `FactionDef` |
| Asset Base Class | `FWFactionDefinition` |
| Directories | Your faction content directory (e.g., `/Game/Data/Factions/`) |
| Rules > Apply Recursively | `true` |
| Rules > Cook Rule | `Always Cook` |

!!! warning "Packaging"
    Ensure `Cook Rule` is set to `Always Cook` so faction definitions are included in packaged builds. Without this, the subsystem will find zero factions in a cooked game.

---

## Faction Definition DataAssets

Each faction is defined by a `UFWFactionDefinition` DataAsset. Create them in the Content Browser via **Miscellaneous > Data Asset > FWFactionDefinition**.

### Identity Section

| Property | Type | Description | Notes |
|----------|------|-------------|-------|
| `FactionId` | `FName` | Unique string identifier. | Used in all code/API calls. Cannot be changed after deployment without migration. |
| `DisplayName` | `FText` | Localized name shown in UI. | Supports localization via string tables. |
| `Description` | `FText` | Lore/flavor text. | Multiline field. |
| `Icon` | `TSoftObjectPtr<UTexture2D>` | Faction icon. | Soft reference for async loading. |
| `FactionColor` | `FLinearColor` | Representative color. | Used by debug HUD and available for UI. |
| `TeamId` | `uint8` | Maps to `FGenericTeamId`. | Must be unique per faction. Range 0--254. 255 = NoTeam. |

### Classification Section

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `FactionTags` | `FGameplayTagContainer` | Empty | Tags for categorizing factions (e.g., `Faction.Type.Military`, `Faction.Alignment.Evil`). |
| `bPlayerJoinable` | `bool` | `false` | If true, players can join this faction through gameplay mechanics. |
| `bHidden` | `bool` | `false` | If true, faction is excluded from UI faction listings. Useful for system factions (e.g., `Wildlife`). |

### Reputation Thresholds

Each faction can define custom thresholds for mapping reputation scores to attitudes. If not customized, the subsystem uses system-wide defaults.

| Threshold | Default Value | Resulting Attitude |
|-----------|---------------|-------------------|
| Allied | >= `75.0` | Allied |
| Friendly | >= `25.0` | Friendly |
| *(Neutral zone)* | >= `-25.0` | Neutral |
| Unfriendly | < `-25.0` | Unfriendly |
| Hostile | < `-75.0` | Hostile |

```
Score:  -100 ----[-75]---- -25 ----[25]---- 75 ----[100]
         Hostile | Unfriendly | Neutral | Friendly | Allied
```

!!! tip "Custom Thresholds per Faction"
    A diplomatic faction might use narrower hostile thresholds (`HostileThreshold = -90`) making it harder to become hostile with them. A militant faction might use wider hostile thresholds (`HostileThreshold = -50`) making hostility easier to trigger.

### Relationships Section

#### Default Relationships

An array of `FFWFactionRelationship` entries defining the starting reputation toward other factions. These are loaded into the global reputation matrix when the subsystem initializes.

| Field | Description |
|-------|-------------|
| `OtherFaction` | `FPrimaryAssetId` pointing to another faction definition. |
| `BaseReputation` | Starting reputation score. |

**Example Configuration:**

| This Faction | Other Faction | Base Reputation | Starting Attitude |
|-------------|---------------|-----------------|-------------------|
| Ironclad | Shadow | -100.0 | Hostile |
| Ironclad | Merchants | 50.0 | Friendly |
| Ironclad | Wildlife | -30.0 | Unfriendly |

#### Propagation Rules

An array of `FFWReputationPropagationRule` entries defining how reputation changes ripple to allied factions.

| Field | Description |
|-------|-------------|
| `TargetFaction` | The allied faction receiving propagated changes. |
| `FalloffMultiplier` | Multiplier on the original delta (0.0--1.0). |

**Example:**

If the player gains +20 reputation with Ironclad, and Ironclad has a propagation rule to Merchants with a `FalloffMultiplier` of `0.5`, then the player also gains +10 reputation with Merchants.

```
Player kills Shadow NPC → +20 Ironclad rep
  Propagation: Ironclad → Merchants (0.5x) → +10 Merchants rep
  Propagation: Ironclad → Royal Guard (0.25x) → +5 Royal Guard rep
```

!!! warning "Circular Propagation"
    The subsystem does not guard against circular propagation rules. If Faction A propagates to B and B propagates to A, you can create infinite loops. Design your propagation graph as a DAG (directed acyclic graph).

---

## Debug Configuration (Non-Shipping Only)

### Console Commands

| Command | Description |
|---------|-------------|
| `FWFaction.ListFactions` | Lists all loaded faction definitions. |
| `FWFaction.ShowMatrix` | Prints the current reputation matrix. |
| `FWFaction.SetRep <Source> <Target> <Value>` | Sets a reputation value in the matrix. |
| `FWFaction.ToggleHUD` | Toggles the debug HUD overlay. |

### Debug HUD

The debug HUD displays the reputation matrix as a color-coded grid overlay. Toggle with `FWFaction.ToggleHUD`. Only available in non-shipping builds.
