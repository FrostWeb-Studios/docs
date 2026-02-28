---
title: Quick Start - FWPartySystem
description: Get a working party system running in under 10 minutes.
---

# Quick Start

Get a working party system running in under 10 minutes with party creation, join codes, and member events.

---

## Overview

By the end of this guide you will have:

1. A Player Controller with `UFWPartyManagerComponent` attached
2. Party creation with automatic join code generation
3. A second player joining via join code
4. Event handling for member joins and leaves

---

## Step 1 -- Add the Component to Your Player Controller

### In C++

```cpp
// MyPlayerController.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "FWPartyManagerComponent.h"
#include "MyPlayerController.generated.h"

UCLASS()
class AMyPlayerController : public APlayerController
{
    GENERATED_BODY()

public:
    AMyPlayerController();

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Party")
    TObjectPtr<UFWPartyManagerComponent> PartyManager;

protected:
    virtual void BeginPlay() override;

private:
    UFUNCTION()
    void HandlePartyCreated(const FFWPartyInfo& PartyInfo);

    UFUNCTION()
    void HandleMemberJoined(const FFWPartyMemberInfo& MemberInfo);

    UFUNCTION()
    void HandleMemberLeft(const FFWPartyMemberInfo& MemberInfo);
};
```

```cpp
// MyPlayerController.cpp
#include "MyPlayerController.h"

AMyPlayerController::AMyPlayerController()
{
    PartyManager = CreateDefaultSubobject<UFWPartyManagerComponent>(TEXT("PartyManager"));
}

void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    PartyManager->OnPartyCreated.AddDynamic(this, &AMyPlayerController::HandlePartyCreated);
    PartyManager->OnMemberJoined.AddDynamic(this, &AMyPlayerController::HandleMemberJoined);
    PartyManager->OnMemberLeft.AddDynamic(this, &AMyPlayerController::HandleMemberLeft);
}
```

### In Blueprints

1. Open your Player Controller Blueprint.
2. Click **Add Component** and search for `FWPartyManager`.
3. Select **FW Party Manager Component**.

---

## Step 2 -- Create a Party

=== "C++"

    ```cpp
    // Call from a UI button handler or console command
    PartyManager->CreateParty();
    ```

=== "Blueprint"

    Call **Create Party** on the Party Manager Component node.

Handle the result:

```cpp
void AMyPlayerController::HandlePartyCreated(const FFWPartyInfo& PartyInfo)
{
    UE_LOG(LogTemp, Log, TEXT("Party created! ID: %s, Code: %s"),
        *PartyInfo.PartyId, *PartyInfo.JoinCode);

    // Display the join code in your UI so other players can join
}
```

---

## Step 3 -- Join a Party by Code

On a second player's controller:

=== "C++"

    ```cpp
    // Player enters the join code from the party leader's UI
    PartyManager->JoinPartyByCode(TEXT("ABC123"));

    PartyManager->OnJoinPartyResult.AddDynamic(
        this, &AMyPlayerController::HandleJoinResult);
    ```

=== "Blueprint"

    Call **Join Party By Code** with the code string, then bind **On Join Party Result** to handle success or failure.

```cpp
void AMyPlayerController::HandleJoinResult(EFWPartyJoinResult Result)
{
    switch (Result)
    {
        case EFWPartyJoinResult::Success:
            UE_LOG(LogTemp, Log, TEXT("Successfully joined party!"));
            break;
        case EFWPartyJoinResult::PartyFull:
            UE_LOG(LogTemp, Warning, TEXT("Party is full."));
            break;
        case EFWPartyJoinResult::InvalidCode:
            UE_LOG(LogTemp, Warning, TEXT("Invalid join code."));
            break;
        case EFWPartyJoinResult::InviteOnly:
            UE_LOG(LogTemp, Warning, TEXT("Party is invite-only."));
            break;
    }
}
```

---

## Step 4 -- Handle Member Events

```cpp
void AMyPlayerController::HandleMemberJoined(const FFWPartyMemberInfo& MemberInfo)
{
    UE_LOG(LogTemp, Log, TEXT("%s joined the party."), *MemberInfo.DisplayName);
    // Update party UI roster
}

void AMyPlayerController::HandleMemberLeft(const FFWPartyMemberInfo& MemberInfo)
{
    UE_LOG(LogTemp, Log, TEXT("%s left the party."), *MemberInfo.DisplayName);
    // Update party UI roster
}
```

---

## Step 5 -- Leave or Disband

=== "C++"

    ```cpp
    // Members leave
    PartyManager->LeaveParty();

    // Leader disbands (automatically called when leader leaves with no other members,
    // or can be triggered explicitly)
    ```

=== "Blueprint"

    Call **Leave Party** on the Party Manager Component. If the leaving player is the leader and other members remain, leadership automatically transfers to the longest-tenured member.

---

## Result

You now have a party system that:

- Creates parties with unique join codes
- Allows players to join via code entry
- Broadcasts member join/leave events for UI updates
- Handles leader migration automatically

---

## Next Steps

- Read the [Party Manager Component](party-manager-component.md) reference for the full API.
- See [Beacon Architecture](beacon-architecture.md) to understand server-side state management.
- Follow the [Tutorial](tutorial.md) for a complete implementation including invitations, seamless travel, and chat integration.
