---
title: Changelog - FWChatSystem
---

# Changelog

All notable changes to FWChatSystem are documented in this file.

---

## v1.0 -- Initial Release

### Components

- **UFWChatStateComponent** -- Local chat state management with per-channel message history ring buffers, unread count tracking, player name caching with configurable TTL, whisper reply target tracking, party info storage, and player presence caching.
- **UFWChatRouterComponent** -- Chat input entry point with slash command parsing (`/s`, `/p`, `/g`, `/w`, `/r`, `/e`, `/help`), channel-based message routing, input validation, default channel management, and local system/error message injection.
- **UFWSocketIOChatTransportComponent** -- Socket.IO WebSocket transport layer with a built-in lightweight Socket.IO v4 client using UE5's native WebSockets module, JWT authentication, automatic reconnection with exponential backoff, per-channel message sending (local, global, whisper, party, guild, emote), presence updates, party room sync/leave, guild room sync/leave, and DM conversations.

### Interfaces

- **IFWChatGuildProvider** -- C++ interface for external guild system integration. Receives guild roster data, update events, info changes, and guild chat messages through typed callbacks.
- **IFWChatUIController** -- C++ interface for UI-layer chat management. Provides chat input focus activation/deactivation, active channel selection, and multi-window lifecycle (create, close, focus, list windows).

### Types and Enums

- `EFWChatChannel` -- Local, Party, Guild, Whisper, System, Global, Emote
- `EFWChatConnectionState` -- Disconnected, Connecting, Connected, Reconnecting, Failed
- `EFWChatSendResult` -- Success, NotConnected, RateLimited, TargetNotFound, InvalidMessage, NotInParty, NotInGuild, Muted, Failed
- `FFWChatMessage` -- Full message struct with channel, sender/target IDs and names, body, timestamp, zone, position, and outgoing flag
- `FFWChatOutgoingMessage` -- Outgoing message request with channel, target, and body
- `FFWChatPresence` -- Player presence with ID, name, zone, position, online status, and last update
- `FFWChatPartyInfo` -- Party info with ID, member IDs/names, and leader ID
- `FFWChatTokenResponse` -- API token response with JWT token, server URL, and expiration
- `FFWChatConfig` -- Configuration struct with history limits, message length, name cache TTL, presence interval, and reconnection settings
- `FFWChatWindowHandle` -- FName-based thin handle for identifying chat windows
- `FFWChatWindowInfo` -- Window descriptor with handle, channel, title, whisper target, and primary flag

### Utility Functions

- `FFWChatTypeUtils::ChannelToEventType()` -- Convert channel enum to Socket.IO event string
- `FFWChatTypeUtils::EventTypeToChannel()` -- Convert Socket.IO event string to channel enum
- `FFWChatTypeUtils::GetChannelDisplayName()` -- Get human-readable channel name
- `FFWChatTypeUtils::GetChannelDefaultColor()` -- Get default color for a channel (hex)

### Delegates (UFWChatStateComponent)

- `FOnChatMessageReceived` -- New message added to history
- `FOnChatUnreadCountChanged` -- Unread count changed for a channel
- `FOnChatPresenceUpdated` -- Player presence updated
- `FOnChatPartyUpdated` -- Party info changed
- `FOnLastWhisperTargetChanged` -- Last whisper target changed

### Delegates (UFWChatRouterComponent)

- `FOnChatHelpRequested` -- Help command invoked
- `FOnChatMessageDisplay` -- Message ready for UI display
- `FOnChatInputSubmitted` -- Chat input submitted (before processing)
- `FOnChatFocusChanged` -- Chat input focus changed
- `FOnActiveChannelChanged` -- Active channel changed
- `FOnChatWindowRequested` -- UI should create a chat window
- `FOnChatWindowClosed` -- UI should destroy a chat window
- `FOnFocusedWindowChanged` -- Focused chat window changed

### Delegates (UFWSocketIOChatTransportComponent)

- `FOnChatConnectionStateChanged` -- Connection state transition
- `FOnChatTransportMessageReceived` -- Message received from server
- `FOnChatSystemNotice` -- System notice from server
- `FOnChatTransportPartyUpdated` -- Party info updated from server
- `FOnChatTransportGuildRosterReceived` -- Guild roster data received
- `FOnChatTransportGuildUpdateReceived` -- Guild update event received
- `FOnChatTransportError` -- Error from chat server

### Features

- Seven built-in chat channels with independent history and unread tracking
- Slash command parser with `/s`, `/p`, `/g`, `/w`, `/r`, `/e`, `/help`
- Whisper reply chain tracking (`/r` replies to last whisper sender)
- Per-channel message history with configurable ring buffer limits
- Player name caching with configurable TTL
- Party chat room sync/leave lifecycle
- Guild chat room sync/leave with IFWChatGuildProvider interface
- Player presence system with zone and position tracking
- Automatic reconnection with exponential backoff
- JWT-based authentication via token response
- Multi-window chat support via IFWChatUIController interface
- Input mode management (GameOnly/GameAndUI switching)
- Local system and error message injection
- Built-in lightweight Socket.IO v4 client using UE5's native WebSockets module (no external plugin required)
