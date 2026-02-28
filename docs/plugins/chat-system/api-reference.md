---
title: API Reference - FWChatSystem
---

# API Reference

Complete C++ API documentation for all FWChatSystem classes, structs, enums, delegates, and interfaces.

---

## Enums

### EFWChatChannel

Chat channel types for message routing. Defined in `FWChatTypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWChatChannel : uint8
{
    Local,
    Party,
    Guild,
    Whisper,
    System,
    Global,
    Emote
};
```

| Value | Slash Command | Description |
|-------|:-------------:|-------------|
| `Local` | `/s` | Local/say chat, visible to nearby players |
| `Party` | `/p` | Party/group chat |
| `Guild` | `/g` | Guild chat |
| `Whisper` | `/w <name>` | Direct message to a specific player |
| `System` | -- | System messages (announcements, errors) |
| `Global` | -- | Global/world chat |
| `Emote` | `/e` | Emote actions |

### EFWChatConnectionState

Connection state for the chat transport. Defined in `FWChatTypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWChatConnectionState : uint8
{
    Disconnected,
    Connecting,
    Connected,
    Reconnecting,
    Failed
};
```

| Value | Description |
|-------|-------------|
| `Disconnected` | Not connected to the chat server |
| `Connecting` | Currently attempting to connect |
| `Connected` | Successfully connected and authenticated |
| `Reconnecting` | Connection lost, attempting to reconnect |
| `Failed` | Connection failed, not attempting to reconnect |

### EFWChatSendResult

Result of sending a chat message. Defined in `FWChatTypes.h`.

```cpp
UENUM(BlueprintType)
enum class EFWChatSendResult : uint8
{
    Success,
    NotConnected,
    RateLimited,
    TargetNotFound,
    InvalidMessage,
    NotInParty,
    NotInGuild,
    Muted,
    Failed
};
```

| Value | Description |
|-------|-------------|
| `Success` | Message sent successfully |
| `NotConnected` | Not connected to the chat server |
| `RateLimited` | Too many messages sent too quickly |
| `TargetNotFound` | Target player not found (whispers) |
| `InvalidMessage` | Invalid message content (empty or too long) |
| `NotInParty` | Not in a party (for party chat) |
| `NotInGuild` | Not in a guild (for guild chat) |
| `Muted` | Player is muted |
| `Failed` | General failure |

---

## Structs

### FFWChatMessage

A chat message with all associated metadata. Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatMessage
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    EFWChatChannel Channel = EFWChatChannel::Local;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString SenderId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString SenderDisplayName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString TargetId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString TargetDisplayName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString Body;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    int64 Timestamp = 0;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString ZoneId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FVector Position = FVector::ZeroVector;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    bool bIsOutgoing = false;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `Channel` | `EFWChatChannel` | The channel this message was sent on |
| `SenderId` | `FString` | Unique ID of the sender |
| `SenderDisplayName` | `FString` | Display name of the sender |
| `TargetId` | `FString` | Target player ID (for whispers) or empty |
| `TargetDisplayName` | `FString` | Target display name (for whispers) or empty |
| `Body` | `FString` | The message body/content |
| `Timestamp` | `int64` | Unix timestamp in milliseconds |
| `ZoneId` | `FString` | Zone ID where the message was sent (local chat) |
| `Position` | `FVector` | World position where message was sent (local chat) |
| `bIsOutgoing` | `bool` | Whether this is an outgoing message from the local player |

#### Utility Methods

```cpp
FString GetFormattedTime() const;
bool IsSystemMessage() const;
bool IsPrivateMessage() const;
```

### FFWChatOutgoingMessage

Outgoing message request to be sent via transport. Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatOutgoingMessage
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    EFWChatChannel Channel = EFWChatChannel::Local;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString Target;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString Body;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `Channel` | `EFWChatChannel` | The channel to send on |
| `Target` | `FString` | Target player name or ID (for whispers) |
| `Body` | `FString` | Message content |

### FFWChatPresence

Player presence information. Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatPresence
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString PlayerId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString DisplayName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString ZoneId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FVector Position = FVector::ZeroVector;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    bool bIsOnline = false;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    int64 LastUpdate = 0;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `PlayerId` | `FString` | Player ID |
| `DisplayName` | `FString` | Player display name |
| `ZoneId` | `FString` | Current zone ID |
| `Position` | `FVector` | World position |
| `bIsOnline` | `bool` | Whether the player is online |
| `LastUpdate` | `int64` | Last update timestamp |

### FFWChatPartyInfo

Party information from the chat server. Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatPartyInfo
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString PartyId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    TArray<FString> MemberIds;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    TArray<FString> MemberNames;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString LeaderId;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `PartyId` | `FString` | Party unique ID |
| `MemberIds` | `TArray<FString>` | List of member player IDs |
| `MemberNames` | `TArray<FString>` | List of member display names (parallel to MemberIds) |
| `LeaderId` | `FString` | Party leader player ID |

### FFWChatTokenResponse

Chat token response from the API. Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatTokenResponse
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString Token;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString ServerUrl;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    int64 ExpiresAt = 0;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `Token` | `FString` | JWT token for chat server authentication |
| `ServerUrl` | `FString` | Chat server URL to connect to |
| `ExpiresAt` | `int64` | Token expiration timestamp (Unix seconds) |

### FFWChatConfig

Configuration for the chat system. Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatConfig
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat", meta = (ClampMin = "10", ClampMax = "1000"))
    int32 MaxHistoryPerChannel = 100;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat", meta = (ClampMin = "1", ClampMax = "2000"))
    int32 MaxMessageLength = 500;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat", meta = (ClampMin = "60"))
    float NameCacheDuration = 300.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat", meta = (ClampMin = "1.0"))
    float PresenceUpdateInterval = 5.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat", meta = (ClampMin = "0"))
    int32 MaxReconnectAttempts = 10;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat", meta = (ClampMin = "0.5"))
    float ReconnectBaseDelay = 1.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat", meta = (ClampMin = "5.0"))
    float ReconnectMaxDelay = 30.0f;
};
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `MaxHistoryPerChannel` | `int32` | `100` | Maximum messages per channel ring buffer |
| `MaxMessageLength` | `int32` | `500` | Maximum message length allowed |
| `NameCacheDuration` | `float` | `300.0` | Player name cache TTL in seconds |
| `PresenceUpdateInterval` | `float` | `5.0` | How often to send presence updates (seconds) |
| `MaxReconnectAttempts` | `int32` | `10` | Maximum reconnection attempts before giving up |
| `ReconnectBaseDelay` | `float` | `1.0` | Base delay for reconnection backoff (seconds) |
| `ReconnectMaxDelay` | `float` | `30.0` | Maximum reconnection delay (seconds) |

### FFWChatWindowHandle

Thin handle identifying a chat window. Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatWindowHandle
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FName Name;

    bool IsValid() const;
    static FFWChatWindowHandle Invalid();
};
```

| Field | Type | Description |
|-------|------|-------------|
| `Name` | `FName` | The underlying FName identifier |

### FFWChatWindowInfo

Descriptor for an open chat window (tab). Defined in `FWChatTypes.h`.

```cpp
USTRUCT(BlueprintType)
struct FWCHATSYSTEM_API FFWChatWindowInfo
{
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FFWChatWindowHandle Handle;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    EFWChatChannel Channel = EFWChatChannel::Local;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString Title;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString WhisperTargetId;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Chat")
    FString WhisperTargetName;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Chat")
    bool bIsPrimary = false;
};
```

| Field | Type | Description |
|-------|------|-------------|
| `Handle` | `FFWChatWindowHandle` | Unique handle for this window |
| `Channel` | `EFWChatChannel` | The channel this window displays |
| `Title` | `FString` | Display title for the window/tab |
| `WhisperTargetId` | `FString` | Target player ID for whisper windows |
| `WhisperTargetName` | `FString` | Target player name for whisper windows |
| `bIsPrimary` | `bool` | Whether this is the primary (non-closeable) window |

---

## Utility Functions

### FFWChatTypeUtils

Static utility functions for chat types. Defined in `FWChatTypes.h`.

```cpp
struct FWCHATSYSTEM_API FFWChatTypeUtils
{
    static FString ChannelToEventType(EFWChatChannel Channel);
    static EFWChatChannel EventTypeToChannel(const FString& EventType);
    static FString GetChannelDisplayName(EFWChatChannel Channel);
    static FString GetChannelDefaultColor(EFWChatChannel Channel);
};
```

| Method | Description |
|--------|-------------|
| `ChannelToEventType` | Convert channel enum to Socket.IO event type string |
| `EventTypeToChannel` | Convert Socket.IO event type string to channel enum |
| `GetChannelDisplayName` | Get the human-readable name for a channel |
| `GetChannelDefaultColor` | Get the default color for a channel (hex string) |

---

## Delegates

### UFWChatStateComponent Delegates

| Delegate | Signature | Description |
|----------|-----------|-------------|
| `FOnChatMessageReceived` | `(const FFWChatMessage& Message)` | New message added to history |
| `FOnChatUnreadCountChanged` | `(EFWChatChannel Channel, int32 UnreadCount)` | Unread count changed |
| `FOnChatPresenceUpdated` | `(const FFWChatPresence& Presence)` | Player presence updated |
| `FOnChatPartyUpdated` | `(const FFWChatPartyInfo& PartyInfo)` | Party info changed |
| `FOnLastWhisperTargetChanged` | `(const FString& TargetName)` | Last whisper target changed |

### UFWChatRouterComponent Delegates

| Delegate | Signature | Description |
|----------|-----------|-------------|
| `FOnChatHelpRequested` | `()` | `/help` command invoked |
| `FOnChatMessageDisplay` | `(const FFWChatMessage& Message)` | Message ready for UI display |
| `FOnChatInputSubmitted` | `(const FString& RawInput, EFWChatChannel Channel)` | Chat input submitted |
| `FOnChatFocusChanged` | `(bool bIsFocused)` | Chat input focus changed |
| `FOnActiveChannelChanged` | `(EFWChatChannel OldChannel, EFWChatChannel NewChannel)` | Active channel changed |
| `FOnChatWindowRequested` | `(const FFWChatWindowInfo& WindowInfo)` | UI should create a chat window |
| `FOnChatWindowClosed` | `(const FFWChatWindowHandle& Handle)` | UI should destroy a chat window |
| `FOnFocusedWindowChanged` | `(const FFWChatWindowHandle& OldHandle, const FFWChatWindowHandle& NewHandle)` | Focused window changed |

### UFWSocketIOChatTransportComponent Delegates

| Delegate | Signature | Description |
|----------|-----------|-------------|
| `FOnChatConnectionStateChanged` | `(EFWChatConnectionState OldState, EFWChatConnectionState NewState)` | Connection state changed |
| `FOnChatTransportMessageReceived` | `(const FFWChatMessage& Message)` | Message received from server |
| `FOnChatSystemNotice` | `(const FString& Code, const FString& Text)` | System notice received |
| `FOnChatTransportPartyUpdated` | `(const FFWChatPartyInfo& PartyInfo)` | Party info updated from server |
| `FOnChatTransportGuildRosterReceived` | `(const FString& GuildId, USIOJsonValue* RosterData)` | Guild roster data received |
| `FOnChatTransportGuildUpdateReceived` | `(USIOJsonValue* UpdateData)` | Guild update event received |
| `FOnChatTransportError` | `(const FString& Code, const FString& Message)` | Error from chat server |

---

## Classes

### UFWChatStateComponent

`UActorComponent` | Header: `Components/FWChatStateComponent.h`

Manages local chat state including message history, unread counts, player name cache, and party information. Purely local -- does not replicate.

```cpp
UCLASS(ClassGroup = (Chat), meta = (BlueprintSpawnableComponent, DisplayName = "Chat State Component"))
class FWCHATSYSTEM_API UFWChatStateComponent : public UActorComponent
```

#### Message History

##### AddMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|History")
void AddMessage(const FFWChatMessage& Message);
```

Adds a message to the history. Called by the transport component when messages arrive.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Message` | `const FFWChatMessage&` | The message to add |

##### GetMessagesForChannel

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|History")
TArray<FFWChatMessage> GetMessagesForChannel(EFWChatChannel Channel) const;
```

**Returns:** Array of messages for the specified channel, sorted oldest to newest.

##### GetAllMessages

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|History")
TArray<FFWChatMessage> GetAllMessages() const;
```

**Returns:** Array of all messages across all channels, sorted by timestamp.

##### GetRecentMessages

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|History")
TArray<FFWChatMessage> GetRecentMessages(int32 Count = 50) const;
```

**Returns:** Array of the last `Count` messages across all channels.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Count` | `int32` | `50` | Maximum number of messages to return |

##### ClearChannelHistory

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|History")
void ClearChannelHistory(EFWChatChannel Channel);
```

Clears message history for a specific channel.

##### ClearAllHistory

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|History")
void ClearAllHistory();
```

Clears all message history across all channels.

#### Unread Counts

##### GetUnreadCount

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Unread")
int32 GetUnreadCount(EFWChatChannel Channel) const;
```

**Returns:** Number of unread messages for the specified channel.

##### GetTotalUnreadCount

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Unread")
int32 GetTotalUnreadCount() const;
```

**Returns:** Total unread count across all channels.

##### MarkChannelAsRead

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Unread")
void MarkChannelAsRead(EFWChatChannel Channel);
```

Marks all messages in a channel as read. Broadcasts `OnUnreadCountChanged`.

##### MarkAllAsRead

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Unread")
void MarkAllAsRead();
```

Marks all messages across all channels as read.

#### Player Name Cache

##### GetCachedPlayerName

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Players")
bool GetCachedPlayerName(const FString& PlayerId, FString& OutDisplayName) const;
```

**Returns:** `true` if the name was found in cache.

##### CachePlayerName

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Players")
void CachePlayerName(const FString& PlayerId, const FString& DisplayName);
```

Caches a player's display name with a TTL defined by `FFWChatConfig::NameCacheDuration`.

##### ClearNameCache

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Players")
void ClearNameCache();
```

Clears the entire player name cache.

#### Whisper State

##### GetLastWhisperTarget

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Whisper")
FString GetLastWhisperTarget() const;
```

**Returns:** The last player who whispered us (for `/r` reply).

##### GetLastWhisperSent

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Whisper")
FString GetLastWhisperSent() const;
```

**Returns:** The last player we whispered to.

##### SetLastWhisperTarget

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Whisper")
void SetLastWhisperTarget(const FString& SenderName);
```

Sets the last whisper target. Called when receiving a whisper.

##### SetLastWhisperSent

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Whisper")
void SetLastWhisperSent(const FString& RecipientName);
```

Sets the last whisper sent. Called when sending a whisper.

#### Party Info

##### GetPartyInfo

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Party")
const FFWChatPartyInfo& GetPartyInfo() const;
```

**Returns:** The current party info.

##### IsInParty

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Party")
bool IsInParty() const;
```

**Returns:** `true` if the player is currently in a party.

##### UpdatePartyInfo

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Party")
void UpdatePartyInfo(const FFWChatPartyInfo& PartyInfo);
```

Updates the party info. Called by transport when party data arrives.

##### ClearPartyInfo

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Party")
void ClearPartyInfo();
```

Clears the party info. Called when leaving a party.

#### Presence

##### UpdatePresence

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Presence")
void UpdatePresence(const FFWChatPresence& Presence);
```

Updates a player's presence info.

##### GetPresence

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Presence")
bool GetPresence(const FString& PlayerId, FFWChatPresence& OutPresence) const;
```

**Returns:** `true` if presence was found for the player.

##### GetPlayersInZone

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Presence")
TArray<FFWChatPresence> GetPlayersInZone(const FString& ZoneId) const;
```

**Returns:** Array of presence info for all online players in the specified zone.

#### Configuration

##### GetConfig

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Config")
const FFWChatConfig& GetConfig() const;
```

**Returns:** The chat configuration struct.

---

### UFWChatRouterComponent

`UActorComponent` | Header: `Components/FWChatRouterComponent.h`

Main chat entry point. Handles slash command parsing, message routing, input validation, and integration between transport and state components.

```cpp
UCLASS(ClassGroup = (Chat), meta = (BlueprintSpawnableComponent, DisplayName = "Chat Router"))
class FWCHATSYSTEM_API UFWChatRouterComponent : public UActorComponent
```

#### Component Setup

##### SetTransportComponent

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
void SetTransportComponent(UFWSocketIOChatTransportComponent* Transport);
```

##### SetStateComponent

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
void SetStateComponent(UFWChatStateComponent* State);
```

##### GetTransportComponent

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Router")
UFWSocketIOChatTransportComponent* GetTransportComponent() const;
```

##### GetStateComponent

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Router")
UFWChatStateComponent* GetStateComponent() const;
```

#### Input Processing

##### SubmitChatInput

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult SubmitChatInput(const FString& RawInput,
    EFWChatChannel DefaultChannel = EFWChatChannel::Local);
```

Submits raw chat input for processing and routing. Handles slash commands and routes to appropriate channel.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `RawInput` | `const FString&` | -- | The user's raw input text |
| `DefaultChannel` | `EFWChatChannel` | `Local` | Default channel if no command is specified |

**Returns:** `EFWChatSendResult` indicating success or failure reason.

##### SendMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult SendMessage(const FString& Body, EFWChatChannel Channel,
    const FString& Target = TEXT(""));
```

Sends a message on a specific channel, bypassing command parsing.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Body` | `const FString&` | -- | The message body |
| `Channel` | `EFWChatChannel` | -- | The channel to send on |
| `Target` | `const FString&` | `""` | Optional target for whispers |

#### Slash Commands

##### Whisper

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult Whisper(const FString& TargetName, const FString& Body);
```

Sends a whisper to a player.

##### Reply

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult Reply(const FString& Body);
```

Replies to the last whisper received.

##### PartyChat

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult PartyChat(const FString& Body);
```

Sends a party message.

##### GuildChat

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult GuildChat(const FString& Body);
```

Sends a guild message.

##### Say

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult Say(const FString& Body);
```

Sends a local/say message.

##### GlobalChat

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult GlobalChat(const FString& Body);
```

Sends a global/world message.

##### Emote

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
EFWChatSendResult Emote(const FString& EmoteText);
```

Sends an emote action.

#### Default Channel

##### GetDefaultChannel

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Router")
EFWChatChannel GetDefaultChannel() const;
```

**Returns:** The current default channel.

##### SetDefaultChannel

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
void SetDefaultChannel(EFWChatChannel Channel);
```

Sets the default channel for messages without a command.

#### System Messages

##### AddLocalSystemMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
void AddLocalSystemMessage(const FString& Text);
```

Adds a local system message (not sent to server). Useful for client-side notifications.

##### AddLocalErrorMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Router")
void AddLocalErrorMessage(const FString& Text);
```

Adds a local error message. Displayed in the System channel.

#### Command Parsing

##### ParseSlashCommand

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Router")
static bool ParseSlashCommand(const FString& Input, FString& OutCommand, FString& OutArguments);
```

Parses a slash command from input.

| Parameter | Type | Description |
|-----------|------|-------------|
| `Input` | `const FString&` | The raw input |
| `OutCommand` | `FString&` | The extracted command (without slash) |
| `OutArguments` | `FString&` | The remaining arguments |

**Returns:** `true` if a command was found.

##### GetHelpText

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Router")
static FString GetHelpText();
```

**Returns:** Help text for all available commands.

---

### UFWSocketIOChatTransportComponent

`UActorComponent` | Header: `Components/FWSocketIOChatTransportComponent.h`

Socket.IO WebSocket transport layer. Handles connection, authentication, message sending/receiving, and presence updates.

```cpp
UCLASS(ClassGroup = (Chat), meta = (BlueprintSpawnableComponent, DisplayName = "Socket.IO Chat Transport"))
class FWCHATSYSTEM_API UFWSocketIOChatTransportComponent : public UActorComponent
```

#### Connection Management

##### Connect

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void Connect(const FString& ServerUrl, const FString& AuthToken);
```

Connects to the chat server with the given token.

| Parameter | Type | Description |
|-----------|------|-------------|
| `ServerUrl` | `const FString&` | The WebSocket URL of the chat server |
| `AuthToken` | `const FString&` | The JWT token for authentication |

##### ConnectWithTokenResponse

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void ConnectWithTokenResponse(const FFWChatTokenResponse& TokenResponse);
```

Connects using a token response from the API.

##### Disconnect

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void Disconnect();
```

Disconnects from the chat server.

##### GetConnectionState

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Transport")
EFWChatConnectionState GetConnectionState() const;
```

**Returns:** The current connection state.

##### IsConnected

```cpp
UFUNCTION(BlueprintPure, Category = "Chat|Transport")
bool IsConnected() const;
```

**Returns:** `true` if connected to the chat server.

#### Message Sending

##### SendMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
EFWChatSendResult SendMessage(const FFWChatOutgoingMessage& Message);
```

Sends a chat message.

##### SendLocalMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
EFWChatSendResult SendLocalMessage(const FString& Body);
```

##### SendGlobalMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
EFWChatSendResult SendGlobalMessage(const FString& Body);
```

##### SendWhisper

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
EFWChatSendResult SendWhisper(const FString& TargetName, const FString& Body);
```

##### SendPartyMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
EFWChatSendResult SendPartyMessage(const FString& Body);
```

##### SendGuildMessage

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
EFWChatSendResult SendGuildMessage(const FString& Body);
```

##### SendEmote

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
EFWChatSendResult SendEmote(const FString& EmoteText);
```

#### Presence

##### UpdatePresence

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void UpdatePresence(const FString& ZoneId, const FVector& Position);
```

Updates the player's presence (zone and position).

#### Party

##### SyncParty

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void SyncParty(const FString& PartyId);
```

Joins a party chat room.

##### LeaveParty

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void LeaveParty();
```

Leaves the current party chat.

#### Direct Messages

##### OpenDmConversation

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void OpenDmConversation(const FString& TargetPlayerId);
```

Opens a DM conversation with a player.

#### Guild

##### SyncGuild

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void SyncGuild(const FString& GuildId);
```

Joins a guild chat room.

##### LeaveGuildChat

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void LeaveGuildChat();
```

Leaves the current guild chat room.

##### SetGuildProvider

```cpp
void SetGuildProvider(IFWChatGuildProvider* Provider);
```

Sets the guild provider interface for receiving guild updates. C++ only.

#### State Integration

##### SetChatStateComponent

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void SetChatStateComponent(UFWChatStateComponent* StateComponent);
```

Sets the chat state component for automatic message forwarding.

##### SetSocketIOClient

```cpp
UFUNCTION(BlueprintCallable, Category = "Chat|Transport")
void SetSocketIOClient(USocketIOClientComponent* ClientComponent);
```

Sets an external Socket.IO client component instead of creating one internally.

#### Configuration Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `PresenceUpdateInterval` | `float` | -- | Presence update interval in seconds |
| `MaxReconnectAttempts` | `int32` | -- | Maximum reconnection attempts |

---

## Interfaces

### IFWChatGuildProvider

Header: `IFWChatGuildProvider.h`

Interface for external guild system integration with the chat transport.

```cpp
class FWCHATSYSTEM_API IFWChatGuildProvider
{
public:
    virtual void OnGuildRosterReceived(const FString& GuildId, USIOJsonValue* RosterData) = 0;
    virtual void OnGuildUpdateReceived(USIOJsonValue* UpdateData) = 0;
    virtual void OnGuildInfoUpdated(const FString& GuildId, USIOJsonValue* InfoData) = 0;
    virtual void OnGuildChatReceived(const FString& GuildId, USIOJsonValue* MessageData) = 0;
};
```

| Method | Description |
|--------|-------------|
| `OnGuildRosterReceived` | Called when the guild roster is received from the server |
| `OnGuildUpdateReceived` | Called on member join/leave, rank change, etc. |
| `OnGuildInfoUpdated` | Called on name, MOTD, description changes |
| `OnGuildChatReceived` | Called when a guild chat message is received |

### IFWChatUIController

Header: `IFWChatUIController.h`

Interface for UI-layer chat focus management and multi-window lifecycle. Implement on your PlayerController.

```cpp
class FWCHATSYSTEM_API IFWChatUIController
{
public:
    // Input Focus
    virtual void ActivateChatInput(EFWChatChannel Channel = EFWChatChannel::Local) = 0;
    virtual void DeactivateChatInput() = 0;
    virtual bool IsChatInputActive() const = 0;
    virtual void SubmitChatText(const FString& Text) = 0;

    // Channel Selection
    virtual void SetActiveChatChannel(EFWChatChannel Channel) = 0;
    virtual EFWChatChannel GetActiveChatChannel() const = 0;

    // Window Management
    virtual FFWChatWindowHandle RequestChatWindow(EFWChatChannel Channel,
        const FString& TargetName = TEXT("")) = 0;
    virtual void CloseChatWindow(const FFWChatWindowHandle& Handle) = 0;
    virtual bool GetChatWindowInfo(const FFWChatWindowHandle& Handle,
        FFWChatWindowInfo& OutInfo) const = 0;
    virtual TArray<FFWChatWindowInfo> GetAllChatWindows() const = 0;
    virtual void SetFocusedChatWindow(const FFWChatWindowHandle& Handle) = 0;
    virtual FFWChatWindowHandle GetFocusedChatWindow() const = 0;
};
```
