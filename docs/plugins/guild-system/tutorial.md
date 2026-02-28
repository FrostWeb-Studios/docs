---
title: "Tutorial: Setting Up a Guild System"
description: Step-by-step guide to implementing a guild system including creation, joining, rank hierarchies, permissions, invitations, and guild chat integration.
---

# Tutorial: Setting Up a Guild System

This tutorial walks through implementing a complete guild system in your UE5 project using the FWGuildSystem plugin. By the end, you will have guild creation, member management, a custom rank hierarchy with permissions, invitation handling, and optional guild chat integration via FWChatSystem.

---

## Prerequisites

- Unreal Engine 5.3 or later
- FWGuildSystem plugin enabled in your project
- A working player controller or player state class
- (Optional) FWChatSystem plugin for guild chat

---

## Step 1: Add Guild Components to Your Player Controller

The guild system requires three components on the actor that represents your player. Typically this is your custom Player Controller.

### Header File

```cpp
// MyPlayerController.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "FWGuildManagerComponent.h"
#include "FWGuildStateComponent.h"
#include "FWGuildChatIntegrationComponent.h"
#include "MyPlayerController.generated.h"

UCLASS()
class MYGAME_API AMyPlayerController : public APlayerController
{
    GENERATED_BODY()

public:
    AMyPlayerController();

protected:
    virtual void BeginPlay() override;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Guild")
    TObjectPtr<UFWGuildManagerComponent> GuildManager;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Guild")
    TObjectPtr<UFWGuildStateComponent> GuildState;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Guild")
    TObjectPtr<UFWGuildChatIntegrationComponent> GuildChat;

private:
    // Event handlers
    UFUNCTION()
    void HandleGuildCreated(EFWGuildOperationResult Result,
        const FFWGuildDetails& Details);

    UFUNCTION()
    void HandleGuildError(EFWGuildOperationResult Result,
        const FString& Message);

    UFUNCTION()
    void HandleInvitationReceived(const FFWGuildInvitation& Invitation);
};
```

### Source File

```cpp
// MyPlayerController.cpp
#include "MyPlayerController.h"

AMyPlayerController::AMyPlayerController()
{
    GuildManager = CreateDefaultSubobject<UFWGuildManagerComponent>(
        TEXT("GuildManager"));
    GuildState = CreateDefaultSubobject<UFWGuildStateComponent>(
        TEXT("GuildState"));
    GuildChat = CreateDefaultSubobject<UFWGuildChatIntegrationComponent>(
        TEXT("GuildChat"));
}

void AMyPlayerController::BeginPlay()
{
    Super::BeginPlay();

    // Bind delegates
    GuildManager->OnGuildCreated.AddDynamic(
        this, &AMyPlayerController::HandleGuildCreated);
    GuildManager->OnGuildOperationFailed.AddDynamic(
        this, &AMyPlayerController::HandleGuildError);
    GuildState->OnInvitationReceived.AddDynamic(
        this, &AMyPlayerController::HandleInvitationReceived);
}
```

!!! info "Blueprint Alternative"
    You can also add these components in the Blueprint editor by clicking **Add Component** in your Player Controller Blueprint and searching for the guild components.

---

## Step 2: Creating a Guild

Implement the guild creation flow. This is typically triggered from a UI button.

```cpp
void AMyPlayerController::CreateNewGuild(const FString& Name,
    const FString& Description)
{
    // Check if already in a guild
    if (GuildState->IsInGuild())
    {
        UE_LOG(LogGuild, Warning,
            TEXT("Cannot create guild: already in a guild."));
        return;
    }

    GuildManager->CreateGuild(Name, Description);
}

void AMyPlayerController::HandleGuildCreated(
    EFWGuildOperationResult Result,
    const FFWGuildDetails& Details)
{
    if (Result == EFWGuildOperationResult::Success)
    {
        UE_LOG(LogGuild, Log,
            TEXT("Guild '%s' created successfully with %d default ranks."),
            *Details.Info.Name, Details.Ranks.Num());

        // The guild state component is automatically populated.
        // Refresh your guild UI here.
    }
}

void AMyPlayerController::HandleGuildError(
    EFWGuildOperationResult Result,
    const FString& Message)
{
    // Display error to player
    FText ErrorText = FFWGuildTypeUtils::GetResultMessage(Result);
    ShowErrorNotification(ErrorText);
}
```

---

## Step 3: Configuring a Rank Hierarchy

After creating a guild, the leader can customize the rank hierarchy. Here is how to set up a 5-rank structure.

```cpp
void AMyPlayerController::SetupCustomRanks()
{
    if (!GuildState->HasPermission(EFWGuildPermission::EditRanks))
    {
        return;
    }

    TArray<FFWGuildRank> CustomRanks;

    // Rank 0: Guild Leader - all permissions
    FFWGuildRank Leader;
    Leader.RankId = TEXT("leader");
    Leader.Name = TEXT("Guild Leader");
    Leader.Priority = 0;
    Leader.Permissions = 0xFF; // All permissions
    CustomRanks.Add(Leader);

    // Rank 1: Commander - management permissions
    FFWGuildRank Commander;
    Commander.RankId = TEXT("commander");
    Commander.Name = TEXT("Commander");
    Commander.Priority = 1;
    Commander.Permissions =
        static_cast<uint8>(EFWGuildPermission::Invite) |
        static_cast<uint8>(EFWGuildPermission::Kick) |
        static_cast<uint8>(EFWGuildPermission::Promote) |
        static_cast<uint8>(EFWGuildPermission::Demote) |
        static_cast<uint8>(EFWGuildPermission::EditInfo) |
        static_cast<uint8>(EFWGuildPermission::ViewAuditLog);
    CustomRanks.Add(Commander);

    // Rank 2: Officer - moderate permissions
    FFWGuildRank Officer;
    Officer.RankId = TEXT("officer");
    Officer.Name = TEXT("Officer");
    Officer.Priority = 2;
    Officer.Permissions =
        static_cast<uint8>(EFWGuildPermission::Invite) |
        static_cast<uint8>(EFWGuildPermission::Kick) |
        static_cast<uint8>(EFWGuildPermission::ViewAuditLog);
    CustomRanks.Add(Officer);

    // Rank 3: Veteran - limited permissions
    FFWGuildRank Veteran;
    Veteran.RankId = TEXT("veteran");
    Veteran.Name = TEXT("Veteran");
    Veteran.Priority = 3;
    Veteran.Permissions =
        static_cast<uint8>(EFWGuildPermission::Invite);
    CustomRanks.Add(Veteran);

    // Rank 4: Recruit - no permissions
    FFWGuildRank Recruit;
    Recruit.RankId = TEXT("recruit");
    Recruit.Name = TEXT("Recruit");
    Recruit.Priority = 4;
    Recruit.Permissions = 0;
    CustomRanks.Add(Recruit);

    GuildManager->EditGuildRanks(CustomRanks);
}
```

The following table summarizes the rank hierarchy:

| Priority | Rank | Invite | Kick | Promote | Demote | EditInfo | EditRanks | Disband | AuditLog |
|---|---|---|---|---|---|---|---|---|---|
| 0 | Guild Leader | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| 1 | Commander | Yes | Yes | Yes | Yes | Yes | -- | -- | Yes |
| 2 | Officer | Yes | Yes | -- | -- | -- | -- | -- | Yes |
| 3 | Veteran | Yes | -- | -- | -- | -- | -- | -- | -- |
| 4 | Recruit | -- | -- | -- | -- | -- | -- | -- | -- |

---

## Step 4: Handling Invitations

### Sending Invitations

```cpp
void AMyPlayerController::InviteToGuild(const FString& TargetPlayerId)
{
    if (!GuildState->IsInGuild())
    {
        UE_LOG(LogGuild, Warning, TEXT("Not in a guild."));
        return;
    }

    if (!GuildState->HasPermission(EFWGuildPermission::Invite))
    {
        UE_LOG(LogGuild, Warning, TEXT("No invite permission."));
        return;
    }

    GuildManager->InvitePlayer(TargetPlayerId);
}
```

### Receiving and Responding to Invitations

```cpp
void AMyPlayerController::HandleInvitationReceived(
    const FFWGuildInvitation& Invitation)
{
    UE_LOG(LogGuild, Log,
        TEXT("Received guild invitation from '%s' to join '%s'. Expires at %s."),
        *Invitation.InvitedByDisplayName,
        *Invitation.GuildName,
        *Invitation.ExpiresAt.ToString());

    // Show invitation popup in UI
    ShowInvitationPopup(Invitation);
}
```

### Invitation UI Widget

=== "C++"

    ```cpp
    void UInvitationWidget::ShowInvitation(const FFWGuildInvitation& Invitation)
    {
        GuildNameText->SetText(FText::FromString(Invitation.GuildName));
        InviterText->SetText(FText::FromString(
            FString::Printf(TEXT("Invited by %s"),
                *Invitation.InvitedByDisplayName)));

        // Calculate time remaining
        FTimespan Remaining = Invitation.ExpiresAt - FDateTime::UtcNow();
        ExpiryText->SetText(FText::FromString(
            FString::Printf(TEXT("Expires in %d hours"),
                FMath::Max(0, (int32)Remaining.GetTotalHours()))));

        CachedInvitationId = Invitation.InvitationId;
        SetVisibility(ESlateVisibility::Visible);
    }
    ```

=== "Blueprint"

    1. Create a widget with text fields for guild name, inviter name, and expiry time.
    2. Add Accept and Decline buttons.
    3. On the **On Invitation Received** event, call `ShowInvitation` with the invitation data.
    4. Wire Accept to a custom "Accept Invitation" function on the player controller.
    5. Wire Decline to a custom "Decline Invitation" function.

---

## Step 5: Member Management

### Promoting and Demoting Members

```cpp
void AMyPlayerController::PromoteGuildMember(const FString& MemberId)
{
    // Validate we can act on this member
    const FFWGuildMember* TargetMember = GuildState->FindMemberById(MemberId);
    if (!TargetMember)
    {
        return;
    }

    // Check rank hierarchy (cannot promote to own rank or above)
    const FFWGuildRank& MyRank = GuildState->GetMyRank();
    if (!FFWGuildTypeUtils::CanActOnRank(MyRank, TargetMember->Rank))
    {
        ShowErrorNotification(
            NSLOCTEXT("Guild", "CannotActOnRank",
                "You cannot modify members of equal or higher rank."));
        return;
    }

    GuildManager->PromoteMember(MemberId);
}
```

### Building a Member Roster

```cpp
void UGuildRosterWidget::BuildRoster()
{
    MemberListView->ClearListItems();

    if (!GuildState->IsInGuild())
    {
        return;
    }

    const TArray<FFWGuildMember>& Members = GuildState->GetMembers();

    // Sort by rank priority, then by name
    TArray<FFWGuildMember> SortedMembers = Members;
    SortedMembers.Sort([](const FFWGuildMember& A, const FFWGuildMember& B)
    {
        if (A.Rank.Priority != B.Rank.Priority)
            return A.Rank.Priority < B.Rank.Priority;
        return A.DisplayName < B.DisplayName;
    });

    for (const FFWGuildMember& Member : SortedMembers)
    {
        UGuildMemberEntry* Entry = CreateWidget<UGuildMemberEntry>(
            this, MemberEntryClass);
        Entry->SetMemberData(Member);

        // Show action buttons based on permissions
        Entry->SetCanKick(
            GuildState->HasPermission(EFWGuildPermission::Kick) &&
            FFWGuildTypeUtils::CanActOnRank(
                GuildState->GetMyRank(), Member.Rank));
        Entry->SetCanPromote(
            GuildState->HasPermission(EFWGuildPermission::Promote) &&
            FFWGuildTypeUtils::CanActOnRank(
                GuildState->GetMyRank(), Member.Rank));
        Entry->SetCanDemote(
            GuildState->HasPermission(EFWGuildPermission::Demote) &&
            FFWGuildTypeUtils::CanActOnRank(
                GuildState->GetMyRank(), Member.Rank));

        MemberListView->AddItem(Entry);
    }
}
```

---

## Step 6: Integrating Guild Chat

If FWChatSystem is available, the `UFWGuildChatIntegrationComponent` automatically manages a guild chat channel.

### Verifying Chat Availability

```cpp
void AMyPlayerController::SetupGuildChat()
{
    if (GuildChat && GuildChat->IsChatSystemAvailable())
    {
        GuildChat->OnGuildChatMessageReceived.AddDynamic(
            this, &AMyPlayerController::OnGuildChatMessage);

        UE_LOG(LogGuild, Log, TEXT("Guild chat integration active."));
    }
    else
    {
        UE_LOG(LogGuild, Log,
            TEXT("FWChatSystem not available, guild chat disabled."));
    }
}
```

### Sending and Receiving Guild Messages

```cpp
void AMyPlayerController::SendGuildChatMessage(const FString& Message)
{
    if (GuildChat && GuildChat->IsChatSystemAvailable()
        && GuildState->IsInGuild())
    {
        GuildChat->SendGuildMessage(Message);
    }
}

void AMyPlayerController::OnGuildChatMessage(
    const FString& SenderId, const FString& Message)
{
    // Find sender display name from guild roster
    const FFWGuildMember* Sender = GuildState->FindMemberById(SenderId);
    FString SenderName = Sender ? Sender->DisplayName : TEXT("Unknown");

    // Add to chat UI
    GuildChatWidget->AddMessage(SenderName, Message);
}
```

### Chat Channel Lifecycle

The chat integration component handles channel lifecycle automatically:

| Guild Event | Chat Channel Action |
|---|---|
| Guild created | Channel `guild_{GuildId}` created; leader added |
| Player joins guild | Player added to channel |
| Player leaves guild | Player removed from channel |
| Guild disbanded | Channel destroyed |

!!! tip "Custom Channel Prefix"
    Change the channel prefix if your game uses multiple guild-like systems:
    ```cpp
    GuildChat->SetChatChannelPrefix(TEXT("clan"));
    // Channel becomes: clan_{GuildId}
    ```

---

## Step 7: Searching for Guilds

Implement a guild browser for players looking to join a guild.

```cpp
void UGuildBrowserWidget::SearchForGuilds(const FString& SearchTerm)
{
    auto* GuildManager = GetOwningPlayerController()
        ->FindComponentByClass<UFWGuildManagerComponent>();

    GuildManager->OnGuildSearchCompleted.AddDynamic(
        this, &UGuildBrowserWidget::HandleSearchResults);

    GuildManager->SearchGuilds(SearchTerm, CurrentPage, ResultsPerPage);
}

void UGuildBrowserWidget::HandleSearchResults(
    const FFWGuildSearchResult& Result)
{
    ResultListView->ClearListItems();

    for (const FFWGuildInfo& Guild : Result.Guilds)
    {
        UGuildSearchEntry* Entry = CreateWidget<UGuildSearchEntry>(
            this, SearchEntryClass);
        Entry->SetGuildInfo(Guild);
        ResultListView->AddItem(Entry);
    }

    PageText->SetText(FText::FromString(
        FString::Printf(TEXT("Page %d of %d"),
            Result.CurrentPage + 1, Result.TotalPages)));

    PrevButton->SetIsEnabled(Result.CurrentPage > 0);
    NextButton->SetIsEnabled(
        Result.CurrentPage < Result.TotalPages - 1);
}
```

---

## Step 8: Viewing the Audit Log

The audit log provides accountability for all guild operations.

```cpp
void UGuildAuditLogWidget::LoadAuditLog()
{
    auto* GuildManager = GetGuildManager();
    auto* GuildState = GetGuildState();

    if (!GuildState->HasPermission(EFWGuildPermission::ViewAuditLog))
    {
        ShowAccessDenied();
        return;
    }

    GuildManager->OnAuditLogReceived.AddDynamic(
        this, &UGuildAuditLogWidget::HandleAuditLog);
    GuildManager->ViewAuditLog(CurrentPage, EntriesPerPage);
}

void UGuildAuditLogWidget::HandleAuditLog(
    const TArray<FFWGuildAuditLogEntry>& Entries,
    int32 TotalEntries, int32 Page)
{
    LogListView->ClearListItems();

    for (const FFWGuildAuditLogEntry& Entry : Entries)
    {
        UAuditLogRow* Row = CreateWidget<UAuditLogRow>(
            this, AuditLogRowClass);

        Row->SetTimestamp(Entry.Timestamp);
        Row->SetActor(Entry.ActorDisplayName);
        Row->SetAction(FFWGuildTypeUtils::UpdateTypeToString(
            Entry.ActionType));
        Row->SetTarget(Entry.TargetDisplayName);
        Row->SetDescription(Entry.Description);

        LogListView->AddItem(Row);
    }
}
```

---

## Complete Setup Checklist

- [ ] FWGuildSystem plugin enabled in project
- [ ] Guild components added to Player Controller (Manager, State, Chat Integration)
- [ ] Delegates bound in `BeginPlay`
- [ ] Guild creation UI wired to `CreateGuild()`
- [ ] Invitation send/receive/accept/decline flow implemented
- [ ] Member roster UI with promote/demote/kick buttons
- [ ] Permission-based UI element visibility
- [ ] Rank editor UI for guild leaders
- [ ] Guild search/browser UI
- [ ] Audit log viewer UI
- [ ] (Optional) Guild chat panel wired to chat integration component
- [ ] Backend API URL configured in `DefaultGame.ini`
- [ ] Error handling delegate bound (`OnGuildOperationFailed`)

---

## Next Steps

- Read the [Permissions Guide](permissions.md) for advanced rank hierarchy patterns.
- See the [Events and Delegates](events-delegates.md) reference for all available event bindings.
- Review the [Configuration](configuration.md) page for tuning limits and timeouts.
