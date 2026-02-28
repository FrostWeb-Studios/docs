---
title: FWChatSystem
---

# FWChatSystem

**Version 1.0** | MMO-ready chat system for Unreal Engine 5

FWChatSystem provides a complete in-game chat framework built for multiplayer and MMO games. It handles message routing across multiple channels, whisper/DM conversations, party and guild chat, player presence tracking, and slash command parsing -- all transported over Socket.IO WebSockets for real-time communication.

---

## Features

- **Multi-Channel Chat** -- Local (say), Party, Guild, Whisper, System, Global, and Emote channels with independent history and unread tracking.
- **Socket.IO Transport** -- WebSocket-based real-time messaging via the SocketIOClient plugin. Supports automatic reconnection with exponential backoff.
- **Slash Commands** -- Built-in command parser for `/w` (whisper), `/r` (reply), `/p` (party), `/g` (guild), `/s` (say), `/e` (emote), and `/help`.
- **State Management** -- Per-channel message history with configurable ring buffers, unread counts, and player name caching.
- **Presence System** -- Player zone and position tracking with periodic updates for "players nearby" and "friends online" features.
- **Party Integration** -- Party chat room management with sync/leave lifecycle.
- **Guild Integration** -- Guild chat with `IFWChatGuildProvider` interface for external guild system integration.
- **Multi-Window Support** -- `IFWChatUIController` interface for tabbed chat windows, whisper tabs, and focus management.
- **Input Mode Management** -- Automatic GameOnly/GameAndUI switching when chat input is activated/deactivated.

---

## Architecture

```
                     Player Controller
                           |
              +------------+------------+
              |            |            |
    UFWChatRouter   UFWChatState   UFWSocketIOChatTransport
    (Input/Routing) (History/Cache) (Network Layer)
              |            |            |
              +-----+------+            |
                    |                   |
              Slash Commands      Socket.IO Client
              Channel Routing     (SocketIOClient Plugin)
              Validation               |
                                  Chat Server
                                  (FrostWeb ChatServer)

    IFWChatUIController          IFWChatGuildProvider
    (UI Focus/Windows)           (Guild Integration)
```

### Component Responsibilities

| Component | Role |
|-----------|------|
| **UFWChatRouterComponent** | Entry point for all chat input. Parses slash commands, routes messages to channels, validates input, integrates transport and state. |
| **UFWChatStateComponent** | Local state management. Stores message history per channel, tracks unread counts, caches player names, holds party/presence info. |
| **UFWSocketIOChatTransportComponent** | Network layer. Connects to the chat server via Socket.IO, sends/receives messages, manages reconnection, handles presence updates. |

### Interfaces

| Interface | Purpose |
|-----------|---------|
| **IFWChatUIController** | Implemented by the PlayerController to manage chat input focus, active channel, and multi-window lifecycle. |
| **IFWChatGuildProvider** | Implemented by a guild system to receive real-time roster updates, guild info changes, and guild chat through the chat transport. |

---

## Key Concepts

### Channels

FWChatSystem supports seven built-in chat channels:

| Channel | Slash Command | Description |
|---------|:-------------:|-------------|
| Local | `/s` | Say chat, visible to nearby players in the same zone |
| Party | `/p` | Party/group chat, visible to party members only |
| Guild | `/g` | Guild chat, visible to guild members only |
| Whisper | `/w <name>` | Direct message to a specific player |
| System | -- | Server announcements, error messages, notifications |
| Global | -- | World/global chat, visible to all connected players |
| Emote | `/e` | Emote actions (e.g., "/e dances") |

### Message Flow

1. Player types text into the chat input.
2. `UFWChatRouterComponent::SubmitChatInput()` parses the text for slash commands.
3. The router determines the target channel and constructs an `FFWChatOutgoingMessage`.
4. The message is sent via `UFWSocketIOChatTransportComponent::SendMessage()`.
5. The transport serializes the message to JSON and emits it over Socket.IO.
6. The chat server broadcasts the message to appropriate recipients.
7. Recipients' transport components receive the message via Socket.IO event handlers.
8. The transport parses the JSON into an `FFWChatMessage` and forwards it to the state component.
9. The state component stores the message in history, updates unread counts, and broadcasts `OnMessageReceived`.
10. The UI displays the message.

### Authentication

The chat transport authenticates using JWT tokens obtained from the game's API:

1. Client requests a chat token from the game API endpoint (e.g., `POST /api/v1/chat/token`).
2. The API returns an `FFWChatTokenResponse` containing the JWT token, server URL, and expiration.
3. Client calls `ConnectWithTokenResponse()` to connect and authenticate in a single step.

---

## Plugin Dependencies

| Plugin | Required | Purpose |
|--------|:--------:|---------|
| SocketIOClient | Yes | Socket.IO WebSocket client for real-time communication |

---

## Module Dependencies

```csharp title="FWChatSystem.Build.cs"
// Public
Core, CoreUObject, Engine, SocketIOClient, SIOJson

// Private
NetCore
```

---

## Next Steps

<div class="grid cards" markdown>

- [Installation](installation.md) -- Enable the plugin and configure SocketIOClient
- [Quick Start](quick-start.md) -- Get chat running in under 10 minutes
- [API Reference](api-reference.md) -- Full C++ class and method documentation
- [Blueprints](blueprints.md) -- Blueprint node reference
- [Configuration](configuration.md) -- Chat config, reconnection, and history settings
- [Tutorials](tutorials.md) -- Build a complete MMO chat UI

</div>
