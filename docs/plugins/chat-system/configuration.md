---
title: Configuration - FWChatSystem
---

# Configuration

This guide covers how to configure chat history, reconnection behavior, presence updates, and channel settings.

---

## Chat Configuration (FFWChatConfig)

The `FFWChatConfig` struct is embedded in the `UFWChatStateComponent` and controls all configurable parameters. Edit these values in the Details panel of your Chat State Component or set them in C++.

### Message History

| Property | Type | Default | Range | Description |
|----------|------|---------|-------|-------------|
| `MaxHistoryPerChannel` | `int32` | `100` | 10 -- 1000 | Maximum messages stored per channel. Older messages are discarded when the limit is reached (ring buffer). |
| `MaxMessageLength` | `int32` | `500` | 1 -- 2000 | Maximum allowed message length. Messages exceeding this length are rejected with `EFWChatSendResult::InvalidMessage`. |

!!! tip "Memory Considerations"
    With 7 channels and 100 messages each, the state component stores up to 700 messages. Each `FFWChatMessage` is approximately 200-400 bytes, so total memory usage is well under 1 MB. Increase `MaxHistoryPerChannel` to 500 or more if you want players to scroll back further.

### Player Name Cache

| Property | Type | Default | Range | Description |
|----------|------|---------|-------|-------------|
| `NameCacheDuration` | `float` | `300.0` | 60+ | How long to cache player display names in seconds. After expiration, names are evicted on the next cleanup pass. |

The name cache prevents redundant lookups when the same player sends multiple messages. A 5-minute TTL (300 seconds) is appropriate for most games.

### Presence Updates

| Property | Type | Default | Range | Description |
|----------|------|---------|-------|-------------|
| `PresenceUpdateInterval` | `float` | `5.0` | 1.0+ | How often the transport sends presence updates (zone + position) to the server. |

!!! warning "Server Load"
    Lower intervals mean more frequent network traffic. For games with hundreds of concurrent players, 5-10 seconds is recommended. For smaller sessions, 1-2 seconds provides snappier presence updates.

### Reconnection

| Property | Type | Default | Range | Description |
|----------|------|---------|-------|-------------|
| `MaxReconnectAttempts` | `int32` | `10` | 0+ | Maximum reconnection attempts before entering the `Failed` state. Set to `0` to disable auto-reconnection. |
| `ReconnectBaseDelay` | `float` | `1.0` | 0.5+ | Base delay for exponential backoff reconnection in seconds. First retry happens after this delay. |
| `ReconnectMaxDelay` | `float` | `30.0` | 5.0+ | Maximum delay between reconnection attempts in seconds. Backoff stops increasing at this value. |

### Reconnection Backoff Formula

The delay between reconnection attempts follows exponential backoff:

```
Delay = min(ReconnectBaseDelay * 2^attempt, ReconnectMaxDelay)
```

With default values (base=1s, max=30s):

| Attempt | Delay |
|:-------:|:-----:|
| 1 | 1s |
| 2 | 2s |
| 3 | 4s |
| 4 | 8s |
| 5 | 16s |
| 6+ | 30s (capped) |

---

## Transport Component Configuration

The `UFWSocketIOChatTransportComponent` has its own editable properties in addition to those defined in `FFWChatConfig`.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `PresenceUpdateInterval` | `float` | -- | Presence update interval in seconds (overrides config if set) |
| `MaxReconnectAttempts` | `int32` | -- | Maximum reconnection attempts (overrides config if set) |

---

## Channel Configuration

### Default Channel

Set the default channel for messages without slash commands:

=== "Blueprint"

    ```
    Get Component by Class (Chat Router)
      |
      Set Default Channel
        Channel: EFWChatChannel::Party
    ```

=== "C++"

    ```cpp
    ChatRouter->SetDefaultChannel(EFWChatChannel::Party);
    ```

The default channel determines what happens when a player types text without a slash command. For most MMOs, `Local` is the appropriate default.

### Channel Colors

Use `FFWChatTypeUtils::GetChannelDefaultColor()` to retrieve the default hex color for each channel:

```cpp
FString Color = FFWChatTypeUtils::GetChannelDefaultColor(EFWChatChannel::Party);
// Returns hex color string for party chat
```

You can override these colors in your UI by mapping `EFWChatChannel` values to custom colors in your widget.

### Channel Display Names

Use `FFWChatTypeUtils::GetChannelDisplayName()` to get human-readable channel names:

```cpp
FString Name = FFWChatTypeUtils::GetChannelDisplayName(EFWChatChannel::Guild);
// Returns "Guild"
```

---

## Server Connection Settings

### Connecting to the Chat Server

The transport component accepts two connection methods:

=== "Token Response (Recommended)"

    ```cpp
    FFWChatTokenResponse Token;
    Token.Token = "eyJhbGciOiJIUzI1NiIs...";
    Token.ServerUrl = "wss://chat.example.com";
    Token.ExpiresAt = 1709290800;

    ChatTransport->ConnectWithTokenResponse(Token);
    ```

=== "Direct Connection"

    ```cpp
    ChatTransport->Connect(
        TEXT("wss://chat.example.com"),
        TEXT("eyJhbGciOiJIUzI1NiIs...")
    );
    ```

!!! info "Token Refresh"
    FWChatSystem does not automatically refresh expired tokens. Monitor `OnConnectionStateChanged` for the `Failed` state and request a new token from your API before reconnecting.

### Built-in Socket.IO Client

The transport component uses FWChatSystem's built-in lightweight Socket.IO v4 client, which is implemented on top of UE5's native WebSocket module. No external Socket.IO plugin is needed. The client is created and managed internally by the transport component.

---

## Guild Integration Configuration

### Using the IFWChatGuildProvider Interface

To receive guild events through the chat transport, implement `IFWChatGuildProvider` in your guild system:

```cpp
UCLASS()
class UMyGuildSubsystem : public UGameInstanceSubsystem, public IFWChatGuildProvider
{
    GENERATED_BODY()

public:
    virtual void OnGuildRosterReceived(const FString& GuildId, USIOJsonValue* RosterData) override;
    virtual void OnGuildUpdateReceived(USIOJsonValue* UpdateData) override;
    virtual void OnGuildInfoUpdated(const FString& GuildId, USIOJsonValue* InfoData) override;
    virtual void OnGuildChatReceived(const FString& GuildId, USIOJsonValue* MessageData) override;
};
```

Register the provider with the transport:

```cpp
ChatTransport->SetGuildProvider(GuildSubsystem);
```

### Using Delegates Instead

If you prefer not to implement the interface, bind to the transport's guild delegates:

```cpp
ChatTransport->OnGuildRosterReceived.AddDynamic(this, &AMyController::HandleGuildRoster);
ChatTransport->OnGuildUpdateReceived.AddDynamic(this, &AMyController::HandleGuildUpdate);
```

---

## UI Controller Configuration

### Implementing IFWChatUIController

For full multi-window chat support, implement `IFWChatUIController` on your PlayerController:

```cpp
UCLASS()
class AMyPlayerController : public APlayerController, public IFWChatUIController
{
    GENERATED_BODY()

public:
    // IFWChatUIController interface
    virtual void ActivateChatInput(EFWChatChannel Channel) override;
    virtual void DeactivateChatInput() override;
    virtual bool IsChatInputActive() const override;
    virtual void SubmitChatText(const FString& Text) override;
    virtual void SetActiveChatChannel(EFWChatChannel Channel) override;
    virtual EFWChatChannel GetActiveChatChannel() const override;
    virtual FFWChatWindowHandle RequestChatWindow(EFWChatChannel Channel,
        const FString& TargetName) override;
    virtual void CloseChatWindow(const FFWChatWindowHandle& Handle) override;
    virtual bool GetChatWindowInfo(const FFWChatWindowHandle& Handle,
        FFWChatWindowInfo& OutInfo) const override;
    virtual TArray<FFWChatWindowInfo> GetAllChatWindows() const override;
    virtual void SetFocusedChatWindow(const FFWChatWindowHandle& Handle) override;
    virtual FFWChatWindowHandle GetFocusedChatWindow() const override;
};
```

!!! info "Optional Interface"
    `IFWChatUIController` is optional. FWChatSystem works without it -- the Router and State components handle message routing and history independently of the UI layer. Implement this interface only when you need multi-window chat management and input mode switching.

---

## Recommended Configuration by Game Type

### Small Multiplayer (2-16 players)

| Setting | Value | Reason |
|---------|-------|--------|
| MaxHistoryPerChannel | 50 | Less history needed |
| PresenceUpdateInterval | 2.0 | Faster updates acceptable |
| MaxReconnectAttempts | 5 | Fewer retries needed |
| MaxMessageLength | 300 | Shorter messages typical |

### MMO (100+ players)

| Setting | Value | Reason |
|---------|-------|--------|
| MaxHistoryPerChannel | 200 | More scrollback for busy channels |
| PresenceUpdateInterval | 10.0 | Reduce server load |
| MaxReconnectAttempts | 15 | More resilience needed |
| MaxMessageLength | 500 | Standard MMO message length |
| ReconnectMaxDelay | 60.0 | Longer backoff for server issues |

### Competitive/Arena (2-10 players)

| Setting | Value | Reason |
|---------|-------|--------|
| MaxHistoryPerChannel | 30 | Minimal history needed |
| PresenceUpdateInterval | 1.0 | Fast updates for small groups |
| MaxReconnectAttempts | 3 | Quick fail for competitive |
| MaxMessageLength | 200 | Short tactical messages |
