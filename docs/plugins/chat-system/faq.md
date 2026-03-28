---
title: FAQ - FWChatSystem
---

# Frequently Asked Questions

---

## Connection and Transport

### What protocol does FWChatSystem use?

FWChatSystem uses Socket.IO over WebSockets for real-time bidirectional communication. Socket.IO handles transport negotiation, automatic reconnection, and event-based messaging. The plugin includes a built-in lightweight Socket.IO v4 client implemented on top of UE5's native WebSocket module -- no external plugin is required.

### What happens if the connection drops?

The transport component automatically enters the `Reconnecting` state and attempts to reconnect using exponential backoff. The delay starts at `ReconnectBaseDelay` (default 1 second) and doubles each attempt up to `ReconnectMaxDelay` (default 30 seconds). After `MaxReconnectAttempts` (default 10) failures, the component enters the `Failed` state and stops retrying.

### Can I manually trigger a reconnect after entering the Failed state?

Yes. Call `Connect()` or `ConnectWithTokenResponse()` again to restart the connection process. You may need to request a fresh token from your API first if the original token has expired.

### Does FWChatSystem handle token refresh?

No. FWChatSystem does not automatically refresh expired JWT tokens. Monitor the `OnConnectionStateChanged` delegate for the `Failed` state and implement token refresh logic in your game code before calling `Connect()` again.

### Can I use a custom Socket.IO client component?

FWChatSystem uses its own built-in Socket.IO v4 client by default. If you need to customize connection behavior, configure the transport component's connection settings (server URL, auth token, reconnection parameters) through its public API rather than replacing the client.

### Can I use FWChatSystem without a Socket.IO server?

The transport layer requires a Socket.IO server. However, you can use the Router and State components without the transport for local-only chat (e.g., single-player games with NPC dialogue in the chat log). Use `AddLocalSystemMessage()` and manually call `ChatState->AddMessage()` to populate the chat without a server connection.

---

## Channels and Messaging

### What channels are available?

Seven channels are built in: Local, Party, Guild, Whisper, System, Global, and Emote. Each channel has independent message history and unread tracking.

### Can I add custom channels?

The `EFWChatChannel` enum is defined in the plugin and cannot be extended without modifying plugin source. For most use cases, you can repurpose the Global channel or use the System channel with custom formatting. If you need additional channels, consider forking the plugin and adding new enum values.

### What slash commands are supported?

| Command | Description |
|---------|-------------|
| `/s <message>` | Send on Local/Say channel |
| `/p <message>` | Send on Party channel |
| `/g <message>` | Send on Guild channel |
| `/w <name> <message>` | Whisper to a player |
| `/r <message>` | Reply to last whisper received |
| `/e <action>` | Send an emote (e.g., `/e dances`) |
| `/help` | Show available commands |

### How does /r (reply) work?

When you receive a whisper, the state component's `LastWhisperTarget` is automatically set to the sender's display name. When you type `/r <message>`, the router sends a whisper to that stored target. The target updates every time you receive a new whisper from a different player.

### How does /w (whisper) determine the target?

The whisper command expects the format `/w <name> <message>`. The router takes the first word after `/w` as the target name and the rest as the message body. For names with spaces, this currently requires quoting or a unique first-name match on the server side.

### What is the difference between the Router's SendMessage and the Transport's SendMessage?

- **Router's `SendMessage()`** -- Sends a message through the full pipeline: validates the input, checks connection state, forwards to transport, and updates state.
- **Transport's `SendMessage()`** -- Sends a raw `FFWChatOutgoingMessage` directly over Socket.IO without Router-level validation.

For most use cases, use the Router. Use the Transport directly only for programmatic messages that bypass slash command parsing.

### Are messages stored persistently?

No. FWChatSystem stores messages in memory only. When the game session ends, message history is lost. For persistent chat history, implement server-side message storage and load recent messages on connect.

---

## Party and Guild

### How do I sync party chat?

When a player joins a party:

1. Call `ChatTransport->SyncParty(PartyId)` to join the party's Socket.IO room.
2. Call `ChatState->UpdatePartyInfo(Info)` to update the local party state.

When a player leaves a party:

1. Call `ChatTransport->LeaveParty()` to leave the Socket.IO room.
2. Call `ChatState->ClearPartyInfo()` to clear the local state.

### What if the player is not in a party and tries /p?

`SubmitChatInput()` returns `EFWChatSendResult::NotInParty`. The router checks `ChatState->IsInParty()` before sending. Display an appropriate error message to the player.

### How does guild chat integration work?

Two options:

1. **IFWChatGuildProvider interface** -- Implement this in your guild system to receive roster updates, guild info changes, and chat messages through typed callbacks.
2. **Delegates** -- Bind to `OnGuildRosterReceived` and `OnGuildUpdateReceived` on the transport component for raw JSON events.

Call `ChatTransport->SyncGuild(GuildId)` when the player logs in with a guild, and `LeaveGuildChat()` when they leave the guild.

---

## Presence

### What data does presence include?

Each `FFWChatPresence` entry contains: player ID, display name, zone ID, world position, online status, and a last-update timestamp.

### How often is presence updated?

The transport sends presence updates to the server at the interval defined by `PresenceUpdateInterval` (default 5 seconds). You can update the zone and position by calling `ChatTransport->UpdatePresence(ZoneId, Position)`.

### Can I get all players in my zone?

Yes. Call `ChatState->GetPlayersInZone(ZoneId)` to get an array of `FFWChatPresence` for all online players in the specified zone. This relies on the server broadcasting presence data to clients.

### Does presence affect local chat range?

FWChatSystem transmits position data with local chat messages but does not enforce range filtering on the client. Range-based visibility should be implemented on the chat server (i.e., the server only delivers local messages to players within a configured radius).

---

## UI and Input

### How do I switch between game input and chat input?

Implement the `IFWChatUIController` interface on your PlayerController. The `ActivateChatInput()` method should switch to `GameAndUI` input mode and focus the chat input widget. `DeactivateChatInput()` should switch back to `GameOnly` mode.

### How do multi-window/tabbed chat windows work?

The `IFWChatUIController` interface provides window lifecycle management:

- `RequestChatWindow()` -- Creates or reuses a window for a channel (e.g., opens a whisper tab)
- `CloseChatWindow()` -- Closes a window (cannot close the primary window)
- `SetFocusedChatWindow()` -- Switches input focus to a specific window
- `GetAllChatWindows()` -- Returns all open windows for tab bar rendering

The Router broadcasts `OnChatWindowRequested` and `OnChatWindowClosed` events that your UI widgets can listen to for creating/destroying tab widgets.

### Can I use FWChatSystem without implementing IFWChatUIController?

Yes. The interface is optional. Without it, you still get full message routing, history, and state management. You just manage input focus and UI lifecycle manually.

---

## Performance

### How much memory does message history use?

With default settings (100 messages per channel, 7 channels), the state component stores up to 700 messages. Each `FFWChatMessage` is approximately 200-400 bytes, so total memory usage is well under 1 MB.

### What is the network overhead of presence updates?

Each presence update is a small JSON payload (approximately 100-200 bytes) sent at the configured interval (default 5 seconds). For 100 concurrent players, this is approximately 2-4 KB/s of aggregate upstream traffic from all clients. The server fans out presence to relevant clients (typically only those in the same zone).

### Does FWChatSystem replicate any data?

No. All chat state is client-local. The transport communicates with the chat server over Socket.IO, not through Unreal's replication system. This keeps the replication budget clean for gameplay-critical data.

---

## Troubleshooting

### Messages are not appearing

1. Verify the transport is in the `Connected` state (`GetConnectionState()`).
2. Verify the state component is connected to the transport (`SetChatStateComponent()`).
3. Verify the router is connected to both state and transport.
4. Check the Output Log for Socket.IO connection errors.
5. Verify your chat server is running and accepting connections.

### Whisper target not found

The `/w` command sends the target name to the server. If the server cannot find a player with that display name, the transport receives an error event. Bind to `OnError` on the transport to display the error to the player.

### Messages appear duplicated

If you are binding to both `OnMessageDisplay` (on the Router) and `OnMessageReceived` (on the State component), you will see each message twice. Use only `OnMessageDisplay` for UI rendering.

### Connection keeps failing

1. Verify the chat server URL is correct and accessible.
2. Verify the JWT token is valid and not expired.
3. Check for CORS or firewall issues if the server is on a different domain.
4. Check the Output Log for specific Socket.IO error messages.
