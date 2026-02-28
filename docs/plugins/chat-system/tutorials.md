---
title: Tutorials - FWChatSystem
---

# Tutorials

## Building an MMO Chat UI

This tutorial walks through building a complete MMO chat interface with:

- Connecting to the chat server and handling authentication
- Sending and receiving messages across all channels
- Slash command support with whisper replies
- Party chat integration with automatic room sync
- A tabbed chat window with unread indicators

---

### Prerequisites

- FWChatSystem [installed and enabled](installation.md)
- A Player Controller with the three chat components added ([Quick Start](quick-start.md) Steps 1-2 completed)
- A running Socket.IO chat server
- Basic UMG widget experience

---

### Step 1: Create the Chat Widget

Create a UMG widget called `WBP_ChatPanel` with the following hierarchy:

```
WBP_ChatPanel (User Widget)
  VerticalBox
    |
    +-- HorizontalBox [Channel Tabs]
    |     +-- Button "Local" (TabLocal)
    |     +-- Button "Party" (TabParty)
    |     +-- Button "Guild" (TabGuild)
    |     +-- Button "Global" (TabGlobal)
    |
    +-- ScrollBox [Message Log] (ChatLog)
    |     +-- (populated dynamically with text blocks)
    |
    +-- HorizontalBox [Input Area]
          +-- TextBlock [Channel Label] (ChannelLabel)
          +-- EditableTextBox (ChatInput)
          +-- Button "Send" (SendButton)
```

---

### Step 2: Connect to the Chat Server

In your Player Controller, connect to the chat server after the player logs in.

=== "Blueprint"

    ```
    Custom Event: OnLoginComplete (Token, ServerUrl)
      |
      Make FFWChatTokenResponse
        Token: Token
        Server Url: ServerUrl
        Expires At: 0  // Server handles expiration
      |
      Get Component by Class (Socket.IO Chat Transport)
      |
      Connect With Token Response (TokenResponse)
      |
      Bind Event to On Connection State Changed
        -> Custom Event "OnChatConnectionChanged"
             Switch on EFWChatConnectionState
               Connected:
                 Print "Chat connected"
                 // Sync party if in one
                 Get Component by Class (Chat State)
                 Is In Party?
                   True -> Get Party Info -> Sync Party (PartyId)
               Reconnecting:
                 Add Local System Message "Reconnecting to chat..."
               Failed:
                 Add Local Error Message "Chat connection failed"
    ```

=== "C++"

    ```cpp
    void AMyPlayerController::OnLoginComplete(const FString& Token, const FString& ServerUrl)
    {
        FFWChatTokenResponse TokenResponse;
        TokenResponse.Token = Token;
        TokenResponse.ServerUrl = ServerUrl;

        ChatTransport->ConnectWithTokenResponse(TokenResponse);
        ChatTransport->OnConnectionStateChanged.AddDynamic(
            this, &AMyPlayerController::OnChatConnectionChanged);
    }

    void AMyPlayerController::OnChatConnectionChanged(
        EFWChatConnectionState OldState, EFWChatConnectionState NewState)
    {
        switch (NewState)
        {
        case EFWChatConnectionState::Connected:
            UE_LOG(LogChat, Log, TEXT("Chat connected"));
            if (ChatState->IsInParty())
            {
                ChatTransport->SyncParty(ChatState->GetPartyInfo().PartyId);
            }
            break;
        case EFWChatConnectionState::Reconnecting:
            ChatRouter->AddLocalSystemMessage(TEXT("Reconnecting to chat..."));
            break;
        case EFWChatConnectionState::Failed:
            ChatRouter->AddLocalErrorMessage(TEXT("Chat connection failed."));
            break;
        default:
            break;
        }
    }
    ```

---

### Step 3: Display Incoming Messages

Bind to the Router's `OnMessageDisplay` event to add messages to your chat log.

=== "Blueprint"

    ```
    Event Construct (WBP_ChatPanel)
      |
      Get Owning Player -> Get Component by Class (Chat Router)
      |
      Bind Event to On Message Display
        -> Custom Event "DisplayMessage"
             |
             Create Widget: WBP_ChatLine
               Channel: Message.Channel
               SenderName: Message.SenderDisplayName
               Body: Message.Body
               Timestamp: Message.GetFormattedTime()
               Color: FFWChatTypeUtils::GetChannelDefaultColor(Channel)
             |
             Chat Log ScrollBox -> Add Child (ChatLine)
             Chat Log ScrollBox -> Scroll to End
    ```

=== "C++"

    ```cpp
    void UMyChatWidget::NativeConstruct()
    {
        Super::NativeConstruct();

        APlayerController* PC = GetOwningPlayer();
        if (UFWChatRouterComponent* Router = PC->FindComponentByClass<UFWChatRouterComponent>())
        {
            Router->OnMessageDisplay.AddDynamic(this, &UMyChatWidget::OnMessageDisplay);
        }
    }

    void UMyChatWidget::OnMessageDisplay(const FFWChatMessage& Message)
    {
        FString ChannelName = FFWChatTypeUtils::GetChannelDisplayName(Message.Channel);
        FString ColorHex = FFWChatTypeUtils::GetChannelDefaultColor(Message.Channel);

        FString Formatted;
        if (Message.Channel == EFWChatChannel::Whisper)
        {
            if (Message.bIsOutgoing)
                Formatted = FString::Printf(TEXT("[To %s]: %s"), *Message.TargetDisplayName, *Message.Body);
            else
                Formatted = FString::Printf(TEXT("[From %s]: %s"), *Message.SenderDisplayName, *Message.Body);
        }
        else if (Message.Channel == EFWChatChannel::Emote)
        {
            Formatted = FString::Printf(TEXT("* %s %s"), *Message.SenderDisplayName, *Message.Body);
        }
        else if (Message.Channel == EFWChatChannel::System)
        {
            Formatted = FString::Printf(TEXT("[System] %s"), *Message.Body);
        }
        else
        {
            Formatted = FString::Printf(TEXT("[%s] %s: %s"), *ChannelName, *Message.SenderDisplayName, *Message.Body);
        }

        AddChatLine(Formatted, ColorHex);
    }
    ```

---

### Step 4: Handle Chat Input

Wire up the chat input box and send button.

=== "Blueprint"

    ```
    // On SendButton Clicked or ChatInput Committed (On Text Committed):

    Custom Event: OnSendClicked
      |
      ChatInput -> Get Text
      |
      Branch: Is Empty?
        True -> Return
        False ->
          Get Owning Player -> Get Component by Class (Chat Router)
          |
          Submit Chat Input
            Raw Input: InputText
            Default Channel: CurrentActiveChannel
          |
          Return Value -> Switch on EFWChatSendResult
            Success: ChatInput -> Set Text ""
            Not Connected: Add Local Error Message "Not connected to chat"
            Not In Party: Add Local Error Message "You are not in a party"
            Not In Guild: Add Local Error Message "You are not in a guild"
            Rate Limited: Add Local Error Message "Sending too fast"
            default: Add Local Error Message "Failed to send message"
    ```

=== "C++"

    ```cpp
    void UMyChatWidget::OnSendClicked()
    {
        FString InputText = ChatInput->GetText().ToString();
        if (InputText.IsEmpty()) return;

        APlayerController* PC = GetOwningPlayer();
        UFWChatRouterComponent* Router = PC->FindComponentByClass<UFWChatRouterComponent>();

        EFWChatSendResult Result = Router->SubmitChatInput(InputText, ActiveChannel);

        if (Result == EFWChatSendResult::Success)
        {
            ChatInput->SetText(FText::GetEmpty());
        }
        else
        {
            FString Error;
            switch (Result)
            {
            case EFWChatSendResult::NotConnected: Error = TEXT("Not connected to chat"); break;
            case EFWChatSendResult::NotInParty:   Error = TEXT("You are not in a party"); break;
            case EFWChatSendResult::NotInGuild:    Error = TEXT("You are not in a guild"); break;
            case EFWChatSendResult::RateLimited:   Error = TEXT("Sending too fast"); break;
            default:                               Error = TEXT("Failed to send message"); break;
            }
            Router->AddLocalErrorMessage(Error);
        }
    }
    ```

---

### Step 5: Implement Channel Tabs

Add tab switching for different chat channels.

```
// On TabLocal Clicked:
Set Active Channel -> EFWChatChannel::Local
Refresh Chat Log with Local messages

// On TabParty Clicked:
Set Active Channel -> EFWChatChannel::Party
Refresh Chat Log with Party messages
Mark Channel As Read (Party)

// On TabGuild Clicked:
Set Active Channel -> EFWChatChannel::Guild
Refresh Chat Log with Guild messages
Mark Channel As Read (Guild)
```

When switching tabs:

1. Update the `ChannelLabel` text to show the active channel name.
2. Call `ChatState->GetMessagesForChannel(Channel)` to populate the log with that channel's history.
3. Call `ChatState->MarkChannelAsRead(Channel)` to clear the unread indicator.
4. Update `ChatRouter->SetDefaultChannel(Channel)` so unslashed messages go to the visible channel.

### Unread Indicators

Bind to `OnUnreadCountChanged` to show notification badges on tabs:

```
Bind Event to On Unread Count Changed
  -> Custom Event (Channel, UnreadCount)
       Switch on Channel
         Party -> TabParty Badge -> Set Text (UnreadCount)
                  TabParty Badge -> Set Visibility (UnreadCount > 0 ? Visible : Hidden)
         Guild -> TabGuild Badge -> Set Text (UnreadCount)
                  ...
```

---

### Step 6: Whisper Handling

Handle incoming whispers and the `/r` reply mechanic.

#### Receiving Whispers

When a whisper arrives, the state component automatically updates the last whisper target. You can display a notification:

```
Bind Event to On Last Whisper Target Changed
  -> Custom Event (TargetName)
       Add Local System Message: "Whisper from " + TargetName + ". Type /r to reply."
```

#### Sending Whispers

Players can whisper using:

- `/w PlayerName message` -- Sends a whisper to PlayerName
- `/r message` -- Replies to the last person who whispered them

Both are handled automatically by `SubmitChatInput()`. No additional code is needed.

#### Whisper Tab (Optional)

For a dedicated whisper tab, listen for whisper messages and open a new tab:

```cpp
void UMyChatWidget::OnMessageDisplay(const FFWChatMessage& Message)
{
    if (Message.Channel == EFWChatChannel::Whisper && !Message.bIsOutgoing)
    {
        // Open or focus a whisper tab for this sender
        OpenWhisperTab(Message.SenderDisplayName, Message.SenderId);
    }

    // ... normal message display ...
}
```

---

### Step 7: Party Chat Integration

When the player joins or leaves a party, sync with the chat transport.

=== "On Party Joined"

    ```
    Custom Event: OnPartyJoined (PartyId)
      |
      Get Component by Class (Socket.IO Chat Transport)
      |
      Sync Party (PartyId)
      |
      Add Local System Message: "Joined party chat. Use /p to chat."
    ```

=== "On Party Left"

    ```
    Custom Event: OnPartyLeft
      |
      Get Component by Class (Socket.IO Chat Transport)
      |
      Leave Party
      |
      Get Component by Class (Chat State)
      |
      Clear Party Info
      |
      Add Local System Message: "Left party chat."
    ```

Listen for party updates to show member changes:

```
Bind Event to On Party Updated (on Chat State Component)
  -> Custom Event (PartyInfo)
       For Each Member Name in PartyInfo.MemberNames
         // Update party member list UI
```

---

### Step 8: Update Presence

Send presence updates so other players can see your zone and position.

```cpp
// In your PlayerController Tick or on a timer:
void AMyPlayerController::UpdateChatPresence()
{
    if (ChatTransport && ChatTransport->IsConnected())
    {
        APawn* Pawn = GetPawn();
        if (Pawn)
        {
            ChatTransport->UpdatePresence(CurrentZoneId, Pawn->GetActorLocation());
        }
    }
}
```

!!! tip "Automatic Updates"
    The transport component handles timing internally based on `PresenceUpdateInterval`. You only need to call `UpdatePresence()` when the zone changes. Position updates are sent automatically on the configured interval.

---

### Step 9: Test the Complete Chat System

1. Start your chat server.
2. Launch two PIE windows (or a listen server + client).
3. Verify the following:

- [x] Both clients connect to the chat server
- [x] Local messages appear for both clients
- [x] `/w <name> message` delivers whispers to the correct client
- [x] `/r message` replies to the last whisper sender
- [x] `/p message` sends party chat (after joining a party)
- [x] `/e dances` sends an emote visible to nearby players
- [x] Channel tabs switch the displayed history
- [x] Unread badges appear on inactive tabs when messages arrive
- [x] Reconnection works after temporarily stopping the server
- [x] Presence updates show players in the correct zone

!!! success "Congratulations"
    You now have a production-quality MMO chat UI with multi-channel support, whisper handling, party integration, and presence tracking. Customize the styling, add guild chat support, and implement the `IFWChatUIController` interface for multi-window management.

---

## Next Steps

- [Configuration](configuration.md) -- Fine-tune history, reconnection, and presence settings
- [API Reference](api-reference.md) -- Detailed method documentation
- [FAQ](faq.md) -- Common questions and troubleshooting
