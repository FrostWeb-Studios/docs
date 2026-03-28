---
title: Quick Start - FWChatSystem
---

# Quick Start

Get a working in-game chat system running in under 10 minutes.

---

## Prerequisites

- FWChatSystem plugin [installed and enabled](installation.md)
- A running Socket.IO chat server to connect to (the plugin includes a built-in Socket.IO v4 client)
- A Player Controller class for your game

---

## Step 1: Add Chat Components to Your Player Controller

FWChatSystem uses three components that work together. Add all three to your Player Controller.

=== "Blueprint"

    1. Open your Player Controller Blueprint.
    2. Click **Add Component** and add:
        - `Socket.IO Chat Transport` (network layer)
        - `Chat State Component` (local state)
        - `Chat Router` (input routing)

=== "C++ (Constructor)"

    ```cpp
    #include "Components/FWSocketIOChatTransportComponent.h"
    #include "Components/FWChatStateComponent.h"
    #include "Components/FWChatRouterComponent.h"

    AMyPlayerController::AMyPlayerController()
    {
        ChatTransport = CreateDefaultSubobject<UFWSocketIOChatTransportComponent>(TEXT("ChatTransport"));
        ChatState = CreateDefaultSubobject<UFWChatStateComponent>(TEXT("ChatState"));
        ChatRouter = CreateDefaultSubobject<UFWChatRouterComponent>(TEXT("ChatRouter"));
    }
    ```

---

## Step 2: Wire the Components Together

The Router needs references to the Transport and State components.

=== "Blueprint (BeginPlay)"

    ```
    Event BeginPlay
      |
      Get Component by Class (Chat Router)
      |
      Set Transport Component -> Get Component by Class (Socket.IO Chat Transport)
      Set State Component -> Get Component by Class (Chat State Component)
    ```

=== "C++ (BeginPlay)"

    ```cpp
    void AMyPlayerController::BeginPlay()
    {
        Super::BeginPlay();

        ChatRouter->SetTransportComponent(ChatTransport);
        ChatRouter->SetStateComponent(ChatState);

        // Also connect transport to state for auto-updates
        ChatTransport->SetChatStateComponent(ChatState);
    }
    ```

!!! info "Auto-Update"
    When you call `SetChatStateComponent()` on the transport, incoming messages are automatically added to the state's history. Without this, you would need to manually listen to the transport's `OnMessageReceived` and call `ChatState->AddMessage()`.

---

## Step 3: Connect to the Chat Server

Obtain a chat token from your game API and connect.

=== "Blueprint"

    ```
    // After login/authentication:

    HTTP Request: POST /api/v1/chat/token
      |
      On Success:
        Parse JSON -> Make FFWChatTokenResponse
          Token: response.token
          ServerUrl: response.serverUrl
          ExpiresAt: response.expiresAt
        |
        Get Component by Class (Socket.IO Chat Transport)
        |
        Connect With Token Response (TokenResponse)
    ```

=== "C++"

    ```cpp
    void AMyPlayerController::ConnectToChat(const FFWChatTokenResponse& TokenResponse)
    {
        ChatTransport->ConnectWithTokenResponse(TokenResponse);
    }

    // Or connect directly with URL and token:
    void AMyPlayerController::ConnectToChat(const FString& ServerUrl, const FString& AuthToken)
    {
        ChatTransport->Connect(ServerUrl, AuthToken);
    }
    ```

!!! tip "Connection State"
    Monitor the connection state by binding to `OnConnectionStateChanged`:
    ```
    Bind Event to On Connection State Changed
      -> Custom Event (OldState, NewState)
           Switch on EFWChatConnectionState
             Connected -> Print "Chat connected!"
             Reconnecting -> Print "Reconnecting..."
             Failed -> Print "Chat connection failed"
    ```

---

## Step 4: Send a Chat Message

Use the Router to send messages. It handles slash command parsing automatically.

=== "Blueprint"

    ```
    // When the player submits text from your chat input widget:

    Get Component by Class (Chat Router)
      |
      Submit Chat Input
        Raw Input: "Hello, world!"
        Default Channel: Local
        |
        Return: EFWChatSendResult
          Switch:
            Success -> (message sent)
            Not Connected -> Show error "Not connected to chat"
            Rate Limited -> Show error "Sending too fast"
    ```

=== "C++"

    ```cpp
    void AMyPlayerController::OnChatInputSubmitted(const FString& Text)
    {
        EFWChatSendResult Result = ChatRouter->SubmitChatInput(Text);

        if (Result != EFWChatSendResult::Success)
        {
            ChatRouter->AddLocalErrorMessage(TEXT("Failed to send message."));
        }
    }
    ```

### Slash Commands

Players can use slash commands to target specific channels:

| Input | Behavior |
|-------|----------|
| `Hello` | Sends on the default channel (Local) |
| `/s Hello` | Sends on Local/Say channel |
| `/p Need heals!` | Sends on Party channel |
| `/g Anyone online?` | Sends on Guild channel |
| `/w PlayerName Hey!` | Whispers to PlayerName |
| `/r Got it` | Replies to the last whisper received |
| `/e dances` | Sends an emote action |
| `/help` | Displays available commands |

---

## Step 5: Display Incoming Messages

Listen for incoming messages to update your UI.

=== "Blueprint"

    ```
    Event BeginPlay
      |
      Get Component by Class (Chat Router)
      |
      Bind Event to On Message Display
        -> Custom Event "OnChatMessageDisplay"
             |
             Break FFWChatMessage
               Channel -> Get Channel Display Color
               Sender Display Name -> Format: "[Name]: "
               Body -> Append to chat log widget
    ```

=== "C++"

    ```cpp
    void AMyPlayerController::BeginPlay()
    {
        Super::BeginPlay();
        // ... (component wiring from Step 2) ...

        ChatRouter->OnMessageDisplay.AddDynamic(this, &AMyPlayerController::OnChatMessageDisplay);
    }

    void AMyPlayerController::OnChatMessageDisplay(const FFWChatMessage& Message)
    {
        FString Formatted = FString::Printf(TEXT("[%s] %s: %s"),
            *FFWChatTypeUtils::GetChannelDisplayName(Message.Channel),
            *Message.SenderDisplayName,
            *Message.Body);

        // Add to your chat UI widget
        AddChatLine(Formatted, FFWChatTypeUtils::GetChannelDefaultColor(Message.Channel));
    }
    ```

---

## Step 6: Test in PIE

1. Start your chat server.
2. Press **Play in Editor**.
3. Your player controller should automatically connect to the chat server (assuming you trigger the connection in BeginPlay or after login).
4. Type a message in your chat input -- it should appear in the chat log.
5. If running multiple PIE windows, messages should appear across all connected clients.

!!! warning "Chat Server Required"
    FWChatSystem requires a running Socket.IO chat server. Without one, the transport will enter the `Failed` connection state. Check the Output Log for connection errors.

---

## Quick Reference: Component Cheat Sheet

| Task | Component | Method |
|------|-----------|--------|
| Connect to server | Transport | `Connect()` or `ConnectWithTokenResponse()` |
| Disconnect | Transport | `Disconnect()` |
| Send chat (with command parsing) | Router | `SubmitChatInput()` |
| Send to specific channel | Router | `SendMessage()`, `Say()`, `PartyChat()`, `GuildChat()`, `Whisper()` |
| Reply to whisper | Router | `Reply()` |
| Add system message | Router | `AddLocalSystemMessage()` |
| Get message history | State | `GetMessagesForChannel()`, `GetAllMessages()` |
| Check unread count | State | `GetUnreadCount()`, `GetTotalUnreadCount()` |
| Mark as read | State | `MarkChannelAsRead()`, `MarkAllAsRead()` |
| Update presence | Transport | `UpdatePresence()` |
| Join party chat | Transport | `SyncParty()` |
| Leave party chat | Transport | `LeaveParty()` |
| Join guild chat | Transport | `SyncGuild()` |

---

## Next Steps

- [Configuration](configuration.md) -- Tune history limits, reconnection, and presence intervals
- [Tutorials](tutorials.md) -- Build a complete MMO chat UI with tabs and whisper windows
- [Blueprints](blueprints.md) -- Complete Blueprint node reference
- [API Reference](api-reference.md) -- Full C++ API documentation
