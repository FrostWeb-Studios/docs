---
title: API Reference - FWChatSystem
---

# API Reference

Component, struct, enum, delegate, and interface documentation for FWChatSystem.

---

## Enums

### EFWChatChannel

Chat channel types for message routing.

| Value | Slash Command | Description |
|-------|:-------------:|-------------|
| `Local` | `/s` | Local/say chat, visible to nearby players |
| `Party` | `/p` | Party/group chat |
| `Guild` | `/g` | Guild chat |
| `Whisper` | `/w <name>` | Direct message to a specific player |
| `System` | -- | System messages (announcements, errors) |
| `Global` | `/gl` | Global/world chat |
| `Emote` | `/e` | Emote actions |

### EFWChatConnectionState

Connection state for the chat transport.

| Value | Description |
|-------|-------------|
| `Disconnected` | Not connected to the chat server |
| `Connecting` | Currently attempting to connect |
| `Connected` | Successfully connected and authenticated |
| `Reconnecting` | Connection lost, attempting to reconnect |
| `Failed` | Connection failed, not attempting to reconnect |

### EFWChatSendResult

Result of sending a chat message.

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

A chat message with all associated metadata.

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

**Utility Methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `GetFormattedTime` | `FString` | Returns a formatted time string |
| `IsSystemMessage` | `bool` | Returns `true` if this is a system message |
| `IsPrivateMessage` | `bool` | Returns `true` if this is a whisper |

### FFWChatConfig

Configuration for the chat system.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `MaxHistoryPerChannel` | `int32` | `100` | Maximum messages per channel ring buffer (10--1000) |
| `MaxMessageLength` | `int32` | `500` | Maximum message length allowed (1--2000) |
| `NameCacheDuration` | `float` | `300.0` | Player name cache TTL in seconds (min 60) |
| `PresenceUpdateInterval` | `float` | `5.0` | How often to send presence updates in seconds (min 1.0) |
| `MaxReconnectAttempts` | `int32` | `10` | Maximum reconnection attempts before giving up |
| `ReconnectBaseDelay` | `float` | `1.0` | Base delay for reconnection backoff in seconds (min 0.5) |
| `ReconnectMaxDelay` | `float` | `30.0` | Maximum reconnection delay in seconds (min 5.0) |

### FFWChatPresence

Player presence information.

| Field | Type | Description |
|-------|------|-------------|
| `PlayerId` | `FString` | Player ID |
| `DisplayName` | `FString` | Player display name |
| `ZoneId` | `FString` | Current zone ID |
| `Position` | `FVector` | World position |
| `bIsOnline` | `bool` | Whether the player is online |
| `LastUpdate` | `int64` | Last update timestamp |

### FFWChatPartyInfo

Party information from the chat server.

| Field | Type | Description |
|-------|------|-------------|
| `PartyId` | `FString` | Party unique ID |
| `MemberIds` | `TArray<FString>` | List of member player IDs |
| `MemberNames` | `TArray<FString>` | List of member display names (parallel to MemberIds) |
| `LeaderId` | `FString` | Party leader player ID |

### FFWChatTokenResponse

Token response used to connect to the chat server.

| Field | Type | Description |
|-------|------|-------------|
| `Token` | `FString` | Authentication token for the chat server |
| `ServerUrl` | `FString` | Chat server URL to connect to |
| `ExpiresAt` | `int64` | Token expiration timestamp (Unix seconds) |

### FFWChatWindowHandle

Thin handle identifying a chat window.

| Field | Type | Description |
|-------|------|-------------|
| `Name` | `FName` | The underlying FName identifier |

**Utility Methods:**

| Method | Returns | Description |
|--------|---------|-------------|
| `IsValid` | `bool` | Returns `true` if the handle is valid |
| `Invalid` | `FFWChatWindowHandle` | Static method returning an invalid handle |

### FFWChatWindowInfo

Descriptor for an open chat window (tab).

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

Static utility functions for chat types.

| Method | Returns | Description |
|--------|---------|-------------|
| `ChannelToEventType` | `FString` | Convert channel enum to Socket.IO event type string |
| `EventTypeToChannel` | `EFWChatChannel` | Convert Socket.IO event type string to channel enum |
| `GetChannelDisplayName` | `FString` | Get the human-readable name for a channel |
| `GetChannelDefaultColor` | `FString` | Get the default color for a channel (hex string) |

---

## Components

### UFWSocketIOChatTransportComponent

Socket.IO WebSocket transport layer. Handles connection, authentication, message sending/receiving, and presence updates.

#### Connection

| Method | Returns | Description |
|--------|---------|-------------|
| `Connect(ServerUrl, AuthToken)` | `void` | Connects to the chat server with the given URL and auth token |
| `ConnectWithTokenResponse(TokenResponse)` | `void` | Connects using an `FFWChatTokenResponse` |
| `Disconnect()` | `void` | Disconnects from the chat server |
| `GetConnectionState()` | `EFWChatConnectionState` | Returns the current connection state |
| `IsConnected()` | `bool` | Returns `true` if connected to the chat server |

#### Sending

| Method | Returns | Description |
|--------|---------|-------------|
| `SendMessage(Message)` | `EFWChatSendResult` | Sends an outgoing chat message |
| `SendLocalMessage(Body)` | `EFWChatSendResult` | Sends a local/say message |
| `SendGlobalMessage(Body)` | `EFWChatSendResult` | Sends a global/world message |
| `SendWhisper(TargetName, Body)` | `EFWChatSendResult` | Sends a whisper to a player |
| `SendPartyMessage(Body)` | `EFWChatSendResult` | Sends a party message |
| `SendGuildMessage(Body)` | `EFWChatSendResult` | Sends a guild message |
| `SendEmote(EmoteText)` | `EFWChatSendResult` | Sends an emote action |

#### Presence

| Method | Returns | Description |
|--------|---------|-------------|
| `UpdatePresence(ZoneId, Position)` | `void` | Updates the player's presence (zone and position) |
| `SyncParty(PartyId)` | `void` | Joins a party chat room |
| `LeaveParty()` | `void` | Leaves the current party chat |
| `SyncGuild(GuildId)` | `void` | Joins a guild chat room |
| `LeaveGuildChat()` | `void` | Leaves the current guild chat room |
| `OpenDmConversation(TargetPlayerId)` | `void` | Opens a DM conversation with a player |

#### Integration

| Method | Returns | Description |
|--------|---------|-------------|
| `SetChatStateComponent(StateComponent)` | `void` | Sets the chat state component for automatic message forwarding |
| `SetSocketIOClient(ClientComponent)` | `void` | Sets a pre-configured built-in Socket.IO client instead of creating one internally |
| `SetGuildProvider(Provider)` | `void` | Sets the guild provider interface for receiving guild updates (C++ only) |

#### Events

| Delegate | Signature | Description |
|----------|-----------|-------------|
| `OnConnectionStateChanged` | `(EFWChatConnectionState OldState, EFWChatConnectionState NewState)` | Connection state changed |
| `OnMessageReceived` | `(const FFWChatMessage& Message)` | Message received from server |
| `OnSystemNotice` | `(const FString& Code, const FString& Text)` | System notice received |
| `OnPartyUpdated` | `(const FFWChatPartyInfo& PartyInfo)` | Party info updated from server |
| `OnGuildRosterReceived` | `(const FString& GuildId, USIOJsonValue* RosterData)` | Guild roster data received |
| `OnGuildUpdateReceived` | `(USIOJsonValue* UpdateData)` | Guild update event received |
| `OnError` | `(const FString& Code, const FString& Message)` | Error from chat server |

---

### UFWChatStateComponent

Manages local chat state including message history, unread counts, player name cache, and party information. Purely local -- does not replicate.

#### Message History

| Method | Returns | Description |
|--------|---------|-------------|
| `AddMessage(Message)` | `void` | Adds a message to history (called by transport on arrival) |
| `GetMessagesForChannel(Channel)` | `TArray<FFWChatMessage>` | Returns messages for a channel, sorted oldest to newest |
| `GetAllMessages()` | `TArray<FFWChatMessage>` | Returns all messages across all channels, sorted by timestamp |
| `GetRecentMessages(Count)` | `TArray<FFWChatMessage>` | Returns the last `Count` messages (default 50) across all channels |
| `ClearChannelHistory(Channel)` | `void` | Clears message history for a specific channel |
| `ClearAllHistory()` | `void` | Clears all message history across all channels |

#### Unread Tracking

| Method | Returns | Description |
|--------|---------|-------------|
| `GetUnreadCount(Channel)` | `int32` | Returns unread message count for a channel |
| `GetTotalUnreadCount()` | `int32` | Returns total unread count across all channels |
| `MarkChannelAsRead(Channel)` | `void` | Marks all messages in a channel as read |
| `MarkAllAsRead()` | `void` | Marks all messages across all channels as read |

#### Player Cache

| Method | Returns | Description |
|--------|---------|-------------|
| `GetCachedPlayerName(PlayerId, OutDisplayName)` | `bool` | Returns `true` if name was found in cache |
| `CachePlayerName(PlayerId, DisplayName)` | `void` | Caches a player's display name with a configurable TTL |
| `ClearNameCache()` | `void` | Clears the entire player name cache |

#### Whisper Tracking

| Method | Returns | Description |
|--------|---------|-------------|
| `GetLastWhisperTarget()` | `FString` | Returns the last player who whispered us (for `/r` reply) |
| `GetLastWhisperSent()` | `FString` | Returns the last player we whispered to |
| `SetLastWhisperTarget(SenderName)` | `void` | Sets the last whisper target (called when receiving a whisper) |
| `SetLastWhisperSent(RecipientName)` | `void` | Sets the last whisper sent (called when sending a whisper) |

#### Party & Presence

| Method | Returns | Description |
|--------|---------|-------------|
| `GetPartyInfo()` | `const FFWChatPartyInfo&` | Returns the current party info |
| `IsInParty()` | `bool` | Returns `true` if the player is in a party |
| `UpdatePartyInfo(PartyInfo)` | `void` | Updates the party info (called by transport) |
| `ClearPartyInfo()` | `void` | Clears the party info (called when leaving a party) |
| `UpdatePresence(Presence)` | `void` | Updates a player's presence info |
| `GetPresence(PlayerId, OutPresence)` | `bool` | Returns `true` if presence was found for the player |
| `GetPlayersInZone(ZoneId)` | `TArray<FFWChatPresence>` | Returns presence info for all online players in a zone |
| `GetConfig()` | `const FFWChatConfig&` | Returns the chat configuration struct |

#### Events

| Delegate | Signature | Description |
|----------|-----------|-------------|
| `OnMessageReceived` | `(const FFWChatMessage& Message)` | New message added to history |
| `OnUnreadCountChanged` | `(EFWChatChannel Channel, int32 UnreadCount)` | Unread count changed |
| `OnPresenceUpdated` | `(const FFWChatPresence& Presence)` | Player presence updated |
| `OnPartyUpdated` | `(const FFWChatPartyInfo& PartyInfo)` | Party info changed |
| `OnLastWhisperTargetChanged` | `(const FString& TargetName)` | Last whisper target changed |

---

### UFWChatRouterComponent

Main chat entry point. Handles slash command parsing, message routing, input validation, and integration between transport and state components.

#### Input Processing

| Method | Returns | Description |
|--------|---------|-------------|
| `SubmitChatInput(RawInput, DefaultChannel)` | `EFWChatSendResult` | Submits raw chat input for processing. Parses slash commands and routes to the appropriate channel. `DefaultChannel` defaults to `Local`. |
| `SendMessage(Body, Channel, Target)` | `EFWChatSendResult` | Sends a message on a specific channel, bypassing command parsing. `Target` is optional (for whispers). |

#### Slash Commands

Built-in commands parsed by the router:

| Command | Method | Description |
|---------|--------|-------------|
| `/w <name> <msg>` | `Whisper(TargetName, Body)` | Send a whisper to a player |
| `/r <msg>` | `Reply(Body)` | Reply to the last whisper received |
| `/p <msg>` | `PartyChat(Body)` | Send a party message |
| `/g <msg>` | `GuildChat(Body)` | Send a guild message |
| `/s <msg>` | `Say(Body)` | Send a local/say message |
| `/gl <msg>` | `GlobalChat(Body)` | Send a global/world message |
| `/e <text>` | `Emote(EmoteText)` | Send an emote action |
| `/help` | -- | Displays help text for all commands |

All slash command methods return `EFWChatSendResult`.

#### Channel Management

| Method | Returns | Description |
|--------|---------|-------------|
| `GetDefaultChannel()` | `EFWChatChannel` | Returns the current default channel |
| `SetDefaultChannel(Channel)` | `void` | Sets the default channel for messages without a command |
| `AddLocalSystemMessage(Text)` | `void` | Adds a local system message (not sent to server) |
| `AddLocalErrorMessage(Text)` | `void` | Adds a local error message displayed in the System channel |
| `ParseSlashCommand(Input, OutCommand, OutArguments)` | `bool` | Static. Parses a slash command from input. Returns `true` if a command was found. |
| `GetHelpText()` | `FString` | Static. Returns help text for all available commands. |

#### Component Setup

| Method | Returns | Description |
|--------|---------|-------------|
| `SetTransportComponent(Transport)` | `void` | Sets the transport component |
| `SetStateComponent(State)` | `void` | Sets the state component |
| `GetTransportComponent()` | `UFWSocketIOChatTransportComponent*` | Returns the transport component |
| `GetStateComponent()` | `UFWChatStateComponent*` | Returns the state component |

#### Events

| Delegate | Signature | Description |
|----------|-----------|-------------|
| `OnHelpRequested` | `()` | `/help` command invoked |
| `OnMessageDisplay` | `(const FFWChatMessage& Message)` | Message ready for UI display |
| `OnInputSubmitted` | `(const FString& RawInput, EFWChatChannel Channel)` | Chat input submitted |
| `OnFocusChanged` | `(bool bIsFocused)` | Chat input focus changed |
| `OnActiveChannelChanged` | `(EFWChatChannel OldChannel, EFWChatChannel NewChannel)` | Active channel changed |
| `OnWindowRequested` | `(const FFWChatWindowInfo& WindowInfo)` | UI should create a chat window |
| `OnWindowClosed` | `(const FFWChatWindowHandle& Handle)` | UI should destroy a chat window |
| `OnFocusedWindowChanged` | `(const FFWChatWindowHandle& OldHandle, const FFWChatWindowHandle& NewHandle)` | Focused window changed |

---

## Interfaces

### IFWChatGuildProvider

Interface for external guild system integration with the chat transport. Implement this in your guild subsystem and pass it to the transport via `SetGuildProvider()`.

| Method | Description |
|--------|-------------|
| `OnGuildRosterReceived(GuildId, RosterData)` | Called when the guild roster is received from the server |
| `OnGuildUpdateReceived(UpdateData)` | Called on member join/leave, rank change, etc. |
| `OnGuildInfoUpdated(GuildId, InfoData)` | Called on name, MOTD, description changes |
| `OnGuildChatReceived(GuildId, MessageData)` | Called when a guild chat message is received |

### IFWChatUIController

Interface for UI-layer chat focus management and multi-window lifecycle. Implement on your PlayerController.

#### Input Focus

| Method | Returns | Description |
|--------|---------|-------------|
| `ActivateChatInput(Channel)` | `void` | Activates chat input, optionally setting the channel (default `Local`) |
| `DeactivateChatInput()` | `void` | Deactivates chat input |
| `IsChatInputActive()` | `bool` | Returns `true` if chat input is currently active |
| `SubmitChatText(Text)` | `void` | Submits chat text from the UI |

#### Channel Selection

| Method | Returns | Description |
|--------|---------|-------------|
| `SetActiveChatChannel(Channel)` | `void` | Sets the active chat channel |
| `GetActiveChatChannel()` | `EFWChatChannel` | Returns the currently active channel |

#### Window Management

| Method | Returns | Description |
|--------|---------|-------------|
| `RequestChatWindow(Channel, TargetName)` | `FFWChatWindowHandle` | Requests a new chat window for a channel. `TargetName` is optional (for whisper windows). |
| `CloseChatWindow(Handle)` | `void` | Closes a chat window |
| `GetChatWindowInfo(Handle, OutInfo)` | `bool` | Returns `true` if the window exists, fills `OutInfo` |
| `GetAllChatWindows()` | `TArray<FFWChatWindowInfo>` | Returns all open chat windows |
| `SetFocusedChatWindow(Handle)` | `void` | Sets the focused chat window |
| `GetFocusedChatWindow()` | `FFWChatWindowHandle` | Returns the currently focused chat window handle |
