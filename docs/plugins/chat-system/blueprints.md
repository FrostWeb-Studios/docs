---
title: Blueprints - FWChatSystem
---

# Blueprint Reference

Complete reference for all FWChatSystem Blueprint-exposed functions and event dispatchers.

---

## UFWChatStateComponent

Manages local chat state. Add to your Player Controller.

### Message History

**Category:** `Chat|History`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Add Message** | Callable | -- | Add a message to history |
| **Get Messages For Channel** | Pure | `TArray<FFWChatMessage>` | All messages for a channel |
| **Get All Messages** | Pure | `TArray<FFWChatMessage>` | All messages across channels |
| **Get Recent Messages** | Pure | `TArray<FFWChatMessage>` | Last N messages across channels |
| **Clear Channel History** | Callable | -- | Clear a specific channel |
| **Clear All History** | Callable | -- | Clear all channels |

##### Add Message

```
Add Message
  Target: Chat State Component (self)
  Message: FFWChatMessage (struct)
```

##### Get Messages For Channel

```
Get Messages For Channel
  Target: Chat State Component (self)
  Channel: EFWChatChannel
  Return: Array of FFWChatMessage
```

##### Get Recent Messages

```
Get Recent Messages
  Target: Chat State Component (self)
  Count: Integer (default: 50)
  Return: Array of FFWChatMessage
```

### Unread Counts

**Category:** `Chat|Unread`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Unread Count** | Pure | Integer | Unread count for a channel |
| **Get Total Unread Count** | Pure | Integer | Total unread across all channels |
| **Mark Channel As Read** | Callable | -- | Mark a channel as read |
| **Mark All As Read** | Callable | -- | Mark all channels as read |

### Player Name Cache

**Category:** `Chat|Players`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Cached Player Name** | Pure | Boolean + String | Look up a cached display name |
| **Cache Player Name** | Callable | -- | Store a player name in cache |
| **Clear Name Cache** | Callable | -- | Clear the name cache |

##### Get Cached Player Name

```
Get Cached Player Name
  Target: Chat State Component (self)
  Player Id: String
  Out Display Name: String (by ref)
  Return: Boolean (true if found)
```

### Whisper State

**Category:** `Chat|Whisper`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Last Whisper Target** | Pure | String | Last player who whispered us |
| **Get Last Whisper Sent** | Pure | String | Last player we whispered |
| **Set Last Whisper Target** | Callable | -- | Set the whisper reply target |
| **Set Last Whisper Sent** | Callable | -- | Set the last sent target |

### Party Info

**Category:** `Chat|Party`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Party Info** | Pure | `FFWChatPartyInfo` | Current party info |
| **Is In Party** | Pure | Boolean | Whether the player is in a party |
| **Update Party Info** | Callable | -- | Update party info |
| **Clear Party Info** | Callable | -- | Clear party info (on leave) |

### Presence

**Category:** `Chat|Presence`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Update Presence** | Callable | -- | Update a player's presence |
| **Get Presence** | Pure | Boolean + `FFWChatPresence` | Get a player's presence |
| **Get Players In Zone** | Pure | `TArray<FFWChatPresence>` | All players in a zone |

##### Get Presence

```
Get Presence
  Target: Chat State Component (self)
  Player Id: String
  Out Presence: FFWChatPresence (by ref)
  Return: Boolean (true if found)
```

### Configuration

**Category:** `Chat|Config`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Config** | Pure | `FFWChatConfig` | The chat configuration |

### Event Dispatchers

| Event | Parameters | Description |
|-------|------------|-------------|
| **On Message Received** | `Message: FFWChatMessage` | New message added to history |
| **On Unread Count Changed** | `Channel: EFWChatChannel, UnreadCount: Integer` | Unread count changed |
| **On Presence Updated** | `Presence: FFWChatPresence` | Player presence updated |
| **On Party Updated** | `PartyInfo: FFWChatPartyInfo` | Party info changed |
| **On Last Whisper Target Changed** | `TargetName: String` | Last whisper target changed |

---

## UFWChatRouterComponent

Main chat entry point. Handles slash commands and routing.

### Component Setup

**Category:** `Chat|Router`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Set Transport Component** | Callable | -- | Connect the transport |
| **Set State Component** | Callable | -- | Connect the state |
| **Get Transport Component** | Pure | Transport ref | Get the transport |
| **Get State Component** | Pure | State ref | Get the state |

##### Set Transport Component

```
Set Transport Component
  Target: Chat Router (self)
  Transport: Socket.IO Chat Transport Object Reference
```

### Input Processing

**Category:** `Chat|Router`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Submit Chat Input** | Callable | `EFWChatSendResult` | Submit raw text (with command parsing) |
| **Send Message** | Callable | `EFWChatSendResult` | Send on specific channel (no parsing) |

##### Submit Chat Input

```
Submit Chat Input
  Target: Chat Router (self)
  Raw Input: String
  Default Channel: EFWChatChannel (default: Local)
  Return: EFWChatSendResult
```

##### Send Message

```
Send Message
  Target: Chat Router (self)
  Body: String
  Channel: EFWChatChannel
  Target: String (optional, for whispers)
  Return: EFWChatSendResult
```

### Slash Commands

**Category:** `Chat|Router`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Whisper** | Callable | `EFWChatSendResult` | Send a whisper to a player |
| **Reply** | Callable | `EFWChatSendResult` | Reply to last whisper |
| **Party Chat** | Callable | `EFWChatSendResult` | Send party message |
| **Guild Chat** | Callable | `EFWChatSendResult` | Send guild message |
| **Say** | Callable | `EFWChatSendResult` | Send local/say message |
| **Global Chat** | Callable | `EFWChatSendResult` | Send global message |
| **Emote** | Callable | `EFWChatSendResult` | Send emote action |

##### Whisper

```
Whisper
  Target: Chat Router (self)
  Target Name: String
  Body: String
  Return: EFWChatSendResult
```

##### Reply

```
Reply
  Target: Chat Router (self)
  Body: String
  Return: EFWChatSendResult
```

### Default Channel

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Get Default Channel** | Pure | `EFWChatChannel` | Current default channel |
| **Set Default Channel** | Callable | -- | Change the default channel |

### System Messages

| Node | Type | Description |
|------|------|-------------|
| **Add Local System Message** | Callable | Add a client-side system message |
| **Add Local Error Message** | Callable | Add a client-side error message |

### Command Parsing

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Parse Slash Command** | Pure (Static) | Boolean + Strings | Parse a command from input |
| **Get Help Text** | Pure (Static) | String | All available commands |

##### Parse Slash Command

```
Parse Slash Command
  Input: String
  Out Command: String (by ref)
  Out Arguments: String (by ref)
  Return: Boolean (true if command found)
```

### Event Dispatchers

| Event | Parameters | Description |
|-------|------------|-------------|
| **On Help Requested** | -- | `/help` command invoked |
| **On Message Display** | `Message: FFWChatMessage` | Message ready for UI display |
| **On Input Submitted** | `RawInput: String, Channel: EFWChatChannel` | Chat input submitted |
| **On Chat Focus Changed** | `bIsFocused: Boolean` | Chat input focus toggled |
| **On Active Channel Changed** | `OldChannel, NewChannel: EFWChatChannel` | Active channel switched |
| **On Chat Window Requested** | `WindowInfo: FFWChatWindowInfo` | UI should create a window |
| **On Chat Window Closed** | `Handle: FFWChatWindowHandle` | UI should destroy a window |
| **On Focused Window Changed** | `OldHandle, NewHandle: FFWChatWindowHandle` | Focus moved to another window |

#### Message Display Binding Example

```
Event BeginPlay
  |
  Get Component by Class (Chat Router)
  |
  Bind Event to On Message Display
    Event -> Custom Event "HandleMessageDisplay"
      |
      Break FFWChatMessage
        Channel -> Get Channel Display Name (FFWChatTypeUtils)
        Sender Display Name
        Body
        |
        Format Text: "[{Channel}] {Name}: {Body}"
        |
        Add to Chat Log ScrollBox Widget
```

---

## UFWSocketIOChatTransportComponent

Socket.IO network transport layer.

### Connection Management

**Category:** `Chat|Transport`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Connect** | Callable | -- | Connect with URL and token |
| **Connect With Token Response** | Callable | -- | Connect with API response |
| **Disconnect** | Callable | -- | Disconnect from server |
| **Get Connection State** | Pure | `EFWChatConnectionState` | Current state |
| **Is Connected** | Pure | Boolean | Whether connected |

##### Connect

```
Connect
  Target: Socket.IO Chat Transport (self)
  Server Url: String
  Auth Token: String
```

##### Connect With Token Response

```
Connect With Token Response
  Target: Socket.IO Chat Transport (self)
  Token Response: FFWChatTokenResponse
```

### Message Sending

**Category:** `Chat|Transport`

| Node | Type | Returns | Description |
|------|------|---------|-------------|
| **Send Message** | Callable | `EFWChatSendResult` | Send via outgoing message struct |
| **Send Local Message** | Callable | `EFWChatSendResult` | Send local/say |
| **Send Global Message** | Callable | `EFWChatSendResult` | Send global |
| **Send Whisper** | Callable | `EFWChatSendResult` | Send whisper |
| **Send Party Message** | Callable | `EFWChatSendResult` | Send party |
| **Send Guild Message** | Callable | `EFWChatSendResult` | Send guild |
| **Send Emote** | Callable | `EFWChatSendResult` | Send emote |

### Presence and Party

| Node | Type | Description |
|------|------|-------------|
| **Update Presence** | Callable | Send presence update (zone + position) |
| **Sync Party** | Callable | Join a party chat room |
| **Leave Party** | Callable | Leave the current party chat |
| **Open DM Conversation** | Callable | Open a DM with a player |
| **Sync Guild** | Callable | Join a guild chat room |
| **Leave Guild Chat** | Callable | Leave the current guild chat |

### State Integration

| Node | Type | Description |
|------|------|-------------|
| **Set Chat State Component** | Callable | Auto-forward messages to state |
| **Set Socket IO Client** | Callable | Provide a pre-configured built-in Socket.IO client |

### Event Dispatchers

| Event | Parameters | Description |
|-------|------------|-------------|
| **On Connection State Changed** | `OldState, NewState: EFWChatConnectionState` | Connection state transition |
| **On Message Received** | `Message: FFWChatMessage` | Message from server |
| **On System Notice** | `Code: String, Text: String` | System notification |
| **On Party Updated** | `PartyInfo: FFWChatPartyInfo` | Party info from server |
| **On Guild Roster Received** | `GuildId: String, RosterData: SIOJsonValue` | Guild roster data |
| **On Guild Update Received** | `UpdateData: SIOJsonValue` | Guild change event |
| **On Error** | `Code: String, Message: String` | Error from server |

---

## Struct Types in Blueprints

### FFWChatMessage

| Pin | Type | Description |
|-----|------|-------------|
| `Channel` | `EFWChatChannel` | Chat channel |
| `Sender Id` | String | Sender's player ID |
| `Sender Display Name` | String | Sender's display name |
| `Target Id` | String | Whisper target ID |
| `Target Display Name` | String | Whisper target name |
| `Body` | String | Message content |
| `Timestamp` | Integer64 | Unix timestamp (ms) |
| `Zone Id` | String | Zone where sent |
| `Position` | Vector | World position |
| `Is Outgoing` | Boolean | True if sent by local player |

### FFWChatOutgoingMessage

| Pin | Type | Description |
|-----|------|-------------|
| `Channel` | `EFWChatChannel` | Target channel |
| `Target` | String | Target player (whispers) |
| `Body` | String | Message content |

### FFWChatPresence

| Pin | Type | Description |
|-----|------|-------------|
| `Player Id` | String | Player ID |
| `Display Name` | String | Display name |
| `Zone Id` | String | Current zone |
| `Position` | Vector | World position |
| `Is Online` | Boolean | Online status |
| `Last Update` | Integer64 | Last update timestamp |

### FFWChatPartyInfo

| Pin | Type | Description |
|-----|------|-------------|
| `Party Id` | String | Party unique ID |
| `Member Ids` | Array of String | Member player IDs |
| `Member Names` | Array of String | Member display names |
| `Leader Id` | String | Party leader ID |

### FFWChatTokenResponse

| Pin | Type | Description |
|-----|------|-------------|
| `Token` | String | JWT auth token |
| `Server Url` | String | Chat server URL |
| `Expires At` | Integer64 | Token expiration (Unix seconds) |

### FFWChatConfig

| Pin | Type | Description |
|-----|------|-------------|
| `Max History Per Channel` | Integer | Max messages per channel |
| `Max Message Length` | Integer | Max message length |
| `Name Cache Duration` | Float | Name cache TTL (seconds) |
| `Presence Update Interval` | Float | Presence update rate (seconds) |
| `Max Reconnect Attempts` | Integer | Max reconnection tries |
| `Reconnect Base Delay` | Float | Base reconnect delay (seconds) |
| `Reconnect Max Delay` | Float | Max reconnect delay (seconds) |

### FFWChatWindowHandle

| Pin | Type | Description |
|-----|------|-------------|
| `Name` | FName | Window identifier |

### FFWChatWindowInfo

| Pin | Type | Description |
|-----|------|-------------|
| `Handle` | `FFWChatWindowHandle` | Window handle |
| `Channel` | `EFWChatChannel` | Window channel |
| `Title` | String | Window title |
| `Whisper Target Id` | String | Whisper target ID |
| `Whisper Target Name` | String | Whisper target name |
| `Is Primary` | Boolean | Non-closeable primary window |
