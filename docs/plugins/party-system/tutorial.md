---
title: Tutorial - Implementing Multiplayer Parties
description: Step-by-step guide to implementing a complete multiplayer party system with join codes, invitations, seamless travel, and chat integration.
---

# Tutorial: Implementing Multiplayer Parties

This tutorial walks through building a complete multiplayer party system from scratch. You will implement party creation, join codes, invitation workflows, seamless travel persistence, and party chat integration with FWChatSystem.

---

## Prerequisites

Before starting, ensure you have:

- [x] FWPartySystem installed and enabled ([Installation](installation.md))
- [x] A multiplayer project with a Player Controller class
- [x] OnlineSubsystem configured (Null is fine for development)
- [x] FWChatSystem installed (for Part 5 -- optional)

---

## Part 1: Creating a Party

### Step 1 -- Set Up the Player Controller

Add the Party Manager Component and bind the core events:

```cpp
// APartyPlayerController.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "FWPartyManagerComponent.h"
#include "Types/FWPartyTypes.h"
#include "PartyPlayerController.generated.h"

UCLASS()
class APartyPlayerController : public APlayerController
{
    GENERATED_BODY()

public:
    APartyPlayerController();

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Party")
    TObjectPtr<UFWPartyManagerComponent> PartyManager;

    /** Called from UI when the player clicks "Create Party". */
    UFUNCTION(BlueprintCallable, Category = "Party")
    void RequestCreateParty();

protected:
    virtual void BeginPlay() override;

private:
    UFUNCTION()
    void HandlePartyCreated(const FFWPartyInfo& PartyInfo);

    UFUNCTION()
    void HandlePartyDisbanded();
};
```

```cpp
// APartyPlayerController.cpp
#include "PartyPlayerController.h"

APartyPlayerController::APartyPlayerController()
{
    PartyManager = CreateDefaultSubobject<UFWPartyManagerComponent>(TEXT("PartyManager"));
}

void APartyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    PartyManager->OnPartyCreated.AddDynamic(this, &APartyPlayerController::HandlePartyCreated);
    PartyManager->OnPartyDisbanded.AddDynamic(this, &APartyPlayerController::HandlePartyDisbanded);
}

void APartyPlayerController::RequestCreateParty()
{
    if (PartyManager->IsInParty())
    {
        UE_LOG(LogTemp, Warning, TEXT("Already in a party. Leave first."));
        return;
    }
    PartyManager->CreateParty();
}

void APartyPlayerController::HandlePartyCreated(const FFWPartyInfo& PartyInfo)
{
    UE_LOG(LogTemp, Log, TEXT("Party created! Code: %s"), *PartyInfo.JoinCode);
    // Pass PartyInfo.JoinCode to your UI widget for display
}

void APartyPlayerController::HandlePartyDisbanded()
{
    UE_LOG(LogTemp, Log, TEXT("Party disbanded."));
    // Clear party UI
}
```

### Step 2 -- Initialize the Beacon Host in Your GameMode

The server must spawn the beacon host for party state management:

```cpp
// APartyGameMode.cpp
#include "PartyGameMode.h"
#include "Beacons/FWPartyBeaconHost.h"

void APartyGameMode::InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage)
{
    Super::InitGame(MapName, Options, ErrorMessage);

    // Spawn the party beacon host on the server
    FActorSpawnParameters SpawnParams;
    SpawnParams.Owner = this;
    PartyBeaconHost = GetWorld()->SpawnActor<AFWPartyBeaconHost>(SpawnParams);

    if (PartyBeaconHost && PartyBeaconHost->InitHost())
    {
        UE_LOG(LogTemp, Log, TEXT("Party beacon host initialized on port %d"),
            PartyBeaconHost->GetBeaconPort());
    }
}
```

!!! tip "Listen Server"
    On a listen server, the host player's `UFWPartyManagerComponent` automatically detects the local beacon host and connects without going through the network stack.

---

## Part 2: Sharing Join Codes

### Step 3 -- Display the Join Code

When `OnPartyCreated` fires, the `FFWPartyInfo` contains the generated `JoinCode`. Pass this to your UI:

```cpp
// In your UMG Widget (C++ backing)
void UPartyWidget::DisplayJoinCode(const FString& JoinCode)
{
    if (JoinCodeText)
    {
        JoinCodeText->SetText(FText::FromString(JoinCode));
    }
}
```

=== "Blueprint"

    1. On your Party Widget, create a **Text Block** named `JoinCodeText`.
    2. When the Player Controller's **On Party Created** fires, call **Set Text** on the text block with the `JoinCode` from the `PartyInfo` struct.
    3. Consider adding a **Copy to Clipboard** button using `FPlatformApplicationMisc::ClipboardCopy`.

### Step 4 -- Implement Join by Code

Add a text input field and join button to your UI:

```cpp
void APartyPlayerController::RequestJoinParty(const FString& JoinCode)
{
    if (PartyManager->IsInParty())
    {
        UE_LOG(LogTemp, Warning, TEXT("Already in a party."));
        return;
    }

    PartyManager->OnJoinPartyResult.AddDynamic(this, &APartyPlayerController::HandleJoinResult);
    PartyManager->OnLocalPlayerJoinedParty.AddDynamic(this, &APartyPlayerController::HandleLocalJoined);
    PartyManager->JoinPartyByCode(JoinCode);
}

void APartyPlayerController::HandleJoinResult(EFWPartyJoinResult Result)
{
    if (Result != EFWPartyJoinResult::Success)
    {
        // Show error to the user
        FText ErrorMsg = FFWPartyTypeUtils::GetJoinResultMessage(Result);
        ShowNotification(ErrorMsg);
    }
}

void APartyPlayerController::HandleLocalJoined(const FFWPartyInfo& PartyInfo)
{
    UE_LOG(LogTemp, Log, TEXT("Joined party: %s (%d members)"),
        *PartyInfo.PartyId, PartyInfo.Members.Num());
    // Refresh party roster UI
    RefreshPartyRoster(PartyInfo);
}
```

---

## Part 3: Handling Invitations

### Step 5 -- Send Invitations

```cpp
void APartyPlayerController::RequestInvitePlayer(const FUniqueNetIdRepl& TargetId)
{
    if (!PartyManager->IsInParty())
    {
        UE_LOG(LogTemp, Warning, TEXT("Must be in a party to invite."));
        return;
    }
    PartyManager->InvitePlayer(TargetId);
}
```

### Step 6 -- Receive and Respond to Invitations

```cpp
void APartyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    // ... existing bindings ...

    PartyManager->OnInvitationReceived.AddDynamic(
        this, &APartyPlayerController::HandleInvitationReceived);
    PartyManager->OnInvitationExpired.AddDynamic(
        this, &APartyPlayerController::HandleInvitationExpired);
}

void APartyPlayerController::HandleInvitationReceived(const FFWPartyInvitation& Invitation)
{
    // Show an invitation popup in your UI
    UE_LOG(LogTemp, Log, TEXT("Invitation from %s for party %s (expires: %s)"),
        *Invitation.SenderDisplayName,
        *Invitation.PartyId,
        *Invitation.ExpiresAt.ToString());

    // Store the invitation for UI display
    PendingInvitations.Add(Invitation);
    ShowInvitationPopup(Invitation);
}

void APartyPlayerController::AcceptPendingInvitation(const FString& InvitationId)
{
    PartyManager->AcceptInvitation(InvitationId);
}

void APartyPlayerController::DeclinePendingInvitation(const FString& InvitationId)
{
    PartyManager->DeclineInvitation(InvitationId);
    PendingInvitations.RemoveAll([&](const FFWPartyInvitation& Inv)
    {
        return Inv.InvitationId == InvitationId;
    });
}

void APartyPlayerController::HandleInvitationExpired(const FString& InvitationId)
{
    PendingInvitations.RemoveAll([&](const FFWPartyInvitation& Inv)
    {
        return Inv.InvitationId == InvitationId;
    });
    // Remove from UI
    DismissInvitationPopup(InvitationId);
}
```

!!! note "Invitation Expiry"
    Invitations expire server-side after the configured `InvitationExpirySeconds` (default 120 seconds). The server sends `OnInvitationExpired` to the target client when the timer fires, regardless of whether the client's local timer has already dismissed the popup.

---

## Part 4: Persisting Across Seamless Travel

### Step 7 -- Configure Seamless Travel

FWPartySystem persists automatically during seamless travel because the beacon host is not part of the traveling world. However, your GameMode must be configured for seamless travel:

```cpp
// APartyGameMode constructor
APartyGameMode::APartyGameMode()
{
    bUseSeamlessTravel = true;
}
```

### Step 8 -- Handle Post-Travel State Refresh

After seamless travel, the `UFWPartyManagerComponent` re-queries state from the beacon client. Bind `OnPartyUpdated` to refresh your UI:

```cpp
PartyManager->OnPartyUpdated.AddDynamic(this, &APartyPlayerController::HandlePartyUpdated);

void APartyPlayerController::HandlePartyUpdated(const FFWPartyInfo& PartyInfo)
{
    // This fires after seamless travel with the current party state
    RefreshPartyRoster(PartyInfo);
    UE_LOG(LogTemp, Log, TEXT("Party state refreshed after travel: %d members"),
        PartyInfo.Members.Num());
}
```

### Step 9 -- Handle Member Disconnection During Travel

During travel, some members may temporarily disconnect. The beacon host marks them offline and starts the grace period:

```cpp
PartyManager->OnMemberLeft.AddDynamic(this, &APartyPlayerController::HandleMemberLeft);

void APartyPlayerController::HandleMemberLeft(const FFWPartyMemberInfo& MemberInfo)
{
    UE_LOG(LogTemp, Log, TEXT("%s left the party."), *MemberInfo.DisplayName);
    RefreshPartyRoster(PartyManager->GetCurrentPartyInfo());
}
```

!!! tip "Grace Period"
    The default disconnect grace period is 60 seconds. During this time, the member appears in the roster with `bIsOnline = false`. If they reconnect within the grace period, they rejoin seamlessly. Configure this in `FFWPartySettings::DisconnectGracePeriod`.

---

## Part 5: Integrating Party Chat with FWChatSystem

### Step 10 -- Enable Chat Integration

With FWChatSystem installed, party chat works automatically. When a party is created, FWPartySystem creates a private chat channel. Verify it is working:

```cpp
PartyManager->OnPartyCreated.AddDynamic(this, &APartyPlayerController::HandlePartyCreated);

void APartyPlayerController::HandlePartyCreated(const FFWPartyInfo& PartyInfo)
{
    // The party chat channel name follows the pattern: "party_{PartyId}"
    FString ChatChannelName = FString::Printf(TEXT("party_%s"), *PartyInfo.PartyId);
    UE_LOG(LogTemp, Log, TEXT("Party chat channel: %s"), *ChatChannelName);
}
```

### Step 11 -- Send Messages to Party Chat

Use FWChatSystem's API to send messages to the party channel:

```cpp
// Assuming you have a reference to the FWChatSystem component
void APartyPlayerController::SendPartyChatMessage(const FString& Message)
{
    if (!PartyManager->IsInParty()) return;

    const FFWPartyInfo& PartyInfo = PartyManager->GetCurrentPartyInfo();
    FString ChannelName = FString::Printf(TEXT("party_%s"), *PartyInfo.PartyId);

    // Use FWChatSystem to send the message
    ChatManager->SendMessage(ChannelName, Message);
}
```

### Step 12 -- Handle Member Chat Channel Sync

When members join or leave the party, they are automatically added to or removed from the chat channel:

```
Party Member Joins:
    OnMemberJoined fires -> FWPartySystem adds member to chat channel
    -> Member sees existing chat history (if configured)

Party Member Leaves:
    OnMemberLeft fires -> FWPartySystem removes member from chat channel
    -> Member can no longer see or send party messages
```

!!! info "Chat Without FWChatSystem"
    If FWChatSystem is not installed, all chat integration is silently skipped. You can implement your own chat bridge by listening to party events and managing channels manually.

---

## Complete Example: Party UI Widget

Here is a complete Blueprint-compatible UMG widget pattern:

```cpp
// UPartyRosterWidget.h
UCLASS()
class UPartyRosterWidget : public UUserWidget
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Party")
    void Initialize(UFWPartyManagerComponent* InPartyManager);

protected:
    virtual void NativeDestruct() override;

private:
    UPROPERTY()
    TObjectPtr<UFWPartyManagerComponent> PartyManager;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UVerticalBox> MemberList;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UTextBlock> JoinCodeText;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UTextBlock> MemberCountText;

    void RefreshRoster(const FFWPartyInfo& PartyInfo);

    UFUNCTION()
    void HandlePartyUpdated(const FFWPartyInfo& PartyInfo);

    UFUNCTION()
    void HandleMemberJoined(const FFWPartyMemberInfo& MemberInfo);

    UFUNCTION()
    void HandleMemberLeft(const FFWPartyMemberInfo& MemberInfo);

    UFUNCTION()
    void HandleLocalLeft();
};
```

```cpp
// UPartyRosterWidget.cpp
void UPartyRosterWidget::Initialize(UFWPartyManagerComponent* InPartyManager)
{
    PartyManager = InPartyManager;

    PartyManager->OnPartyUpdated.AddDynamic(this, &UPartyRosterWidget::HandlePartyUpdated);
    PartyManager->OnMemberJoined.AddDynamic(this, &UPartyRosterWidget::HandleMemberJoined);
    PartyManager->OnMemberLeft.AddDynamic(this, &UPartyRosterWidget::HandleMemberLeft);
    PartyManager->OnLocalPlayerLeftParty.AddDynamic(this, &UPartyRosterWidget::HandleLocalLeft);

    if (PartyManager->IsInParty())
    {
        RefreshRoster(PartyManager->GetCurrentPartyInfo());
    }
}

void UPartyRosterWidget::RefreshRoster(const FFWPartyInfo& PartyInfo)
{
    JoinCodeText->SetText(FText::FromString(PartyInfo.JoinCode));
    MemberCountText->SetText(FText::Format(
        LOCTEXT("MemberCount", "{0}/{1}"),
        FText::AsNumber(PartyInfo.Members.Num()),
        FText::AsNumber(PartyInfo.MaxMembers)));

    MemberList->ClearChildren();
    for (const FFWPartyMemberInfo& Member : PartyInfo.Members)
    {
        // Create a member entry widget for each member
        UPartyMemberEntry* Entry = CreateWidget<UPartyMemberEntry>(this, MemberEntryClass);
        Entry->SetMemberInfo(Member);
        MemberList->AddChildToVerticalBox(Entry);
    }
}

void UPartyRosterWidget::HandlePartyUpdated(const FFWPartyInfo& PartyInfo)
{
    RefreshRoster(PartyInfo);
}

void UPartyRosterWidget::HandleMemberJoined(const FFWPartyMemberInfo& MemberInfo)
{
    RefreshRoster(PartyManager->GetCurrentPartyInfo());
}

void UPartyRosterWidget::HandleMemberLeft(const FFWPartyMemberInfo& MemberInfo)
{
    RefreshRoster(PartyManager->GetCurrentPartyInfo());
}

void UPartyRosterWidget::HandleLocalLeft()
{
    SetVisibility(ESlateVisibility::Collapsed);
}
```

---

## Summary

You have implemented a complete multiplayer party system with:

- **Party creation** with automatic join code generation
- **Join by code** for frictionless party joining
- **Invitation workflow** with accept, decline, and expiration
- **Seamless travel persistence** with automatic state refresh
- **Party chat integration** via FWChatSystem

---

## Next Steps

- See [Configuration](configuration.md) to tune party size limits, invitation expiry, and disconnect grace periods.
- See [Beacon Architecture](beacon-architecture.md) for advanced server topology options.
- See [Events and Delegates](events-delegates.md) for the complete event reference.
