---
title: Guild System Configuration
description: Plugin settings, project configuration, and backend API setup for FWGuildSystem.
---

# Guild System Configuration

This page covers all configuration options for the FWGuildSystem plugin, including project settings, backend API endpoints, and default values.

---

## Plugin Settings

Guild system settings are configured in your project's `DefaultGame.ini` or via the Project Settings editor under **Plugins > FW Guild System**.

```ini
[/Script/FWGuildSystem.FWGuildSystemSettings]
; Backend API base URL for guild operations
ApiBaseUrl=https://ayndora.frostweb.dev/api/v1/guilds

; Maximum members per guild
MaxMembersPerGuild=100

; Invitation expiry time in hours
InvitationExpiryHours=72

; Minimum guild name length
MinGuildNameLength=3

; Maximum guild name length
MaxGuildNameLength=32

; Maximum guild description length
MaxDescriptionLength=256

; Enable audit logging
bEnableAuditLog=true

; Maximum audit log entries stored per guild
MaxAuditLogEntries=1000

; Guild chat channel prefix (used by chat integration)
ChatChannelPrefix=guild

; HTTP request timeout in seconds
RequestTimeoutSeconds=30

; Enable automatic state refresh interval
bAutoRefreshState=true

; State refresh interval in seconds
AutoRefreshIntervalSeconds=60
```

---

## Settings Reference

### Network

| Setting | Type | Default | Description |
|---|---|---|---|
| `ApiBaseUrl` | `FString` | `""` | Base URL for the guild HTTP API. Must be set for the plugin to function. |
| `RequestTimeoutSeconds` | `float` | `30.0` | Timeout for HTTP requests. |
| `bAutoRefreshState` | `bool` | `true` | Whether to periodically poll for guild state updates. |
| `AutoRefreshIntervalSeconds` | `float` | `60.0` | Interval between automatic state refresh polls. |

### Guild Limits

| Setting | Type | Default | Description |
|---|---|---|---|
| `MaxMembersPerGuild` | `int32` | `100` | Maximum number of members a guild can have. |
| `MinGuildNameLength` | `int32` | `3` | Minimum characters for a guild name. |
| `MaxGuildNameLength` | `int32` | `32` | Maximum characters for a guild name. |
| `MaxDescriptionLength` | `int32` | `256` | Maximum characters for a guild description. |

### Invitations

| Setting | Type | Default | Description |
|---|---|---|---|
| `InvitationExpiryHours` | `int32` | `72` | Hours before a pending invitation expires. |

### Audit Log

| Setting | Type | Default | Description |
|---|---|---|---|
| `bEnableAuditLog` | `bool` | `true` | Whether to record audit log entries. |
| `MaxAuditLogEntries` | `int32` | `1000` | Maximum entries per guild before oldest are pruned. |

### Chat Integration

| Setting | Type | Default | Description |
|---|---|---|---|
| `ChatChannelPrefix` | `FString` | `guild` | Prefix for auto-created guild chat channels. |

---

## Enabling the Plugin

### Via .uproject File

Add the plugin to your project's `.uproject` file:

```json
{
    "Plugins": [
        {
            "Name": "FWGuildSystem",
            "Enabled": true
        },
        {
            "Name": "FWChatSystem",
            "Enabled": true
        }
    ]
}
```

### Via Plugin Manager

1. Open the UE Editor.
2. Navigate to **Edit > Plugins**.
3. Search for "FWGuildSystem".
4. Check the **Enabled** checkbox.
5. Restart the editor.

---

## Build Configuration

### Module Dependencies

Add the module dependency to your game module's `Build.cs`:

```csharp
public class MyGame : ModuleRules
{
    public MyGame(ReadOnlyTargetRules Target) : base(Target)
    {
        PublicDependencyModuleNames.AddRange(new string[]
        {
            "Core",
            "CoreUObject",
            "Engine",
            "FWGuildSystem"
        });

        // Optional: Add FWChatSystem for guild chat integration
        if (Target.bBuildEditor || IsPluginEnabled("FWChatSystem"))
        {
            PublicDependencyModuleNames.Add("FWChatSystem");
            PublicDefinitions.Add("WITH_FWCHATSYSTEM=1");
        }
    }
}
```

---

## Backend API Configuration

The guild system communicates with a REST API for persistence. The following endpoints must be available at the configured `ApiBaseUrl`.

### Required Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/` | Create a new guild |
| `DELETE` | `/{guildId}` | Disband a guild |
| `GET` | `/{guildId}` | Get guild details |
| `PATCH` | `/{guildId}` | Update guild info |
| `PUT` | `/{guildId}/ranks` | Replace rank hierarchy |
| `POST` | `/{guildId}/invite` | Send an invitation |
| `POST` | `/{guildId}/kick` | Kick a member |
| `POST` | `/{guildId}/promote` | Promote a member |
| `POST` | `/{guildId}/demote` | Demote a member |
| `GET` | `/search?q={query}&page={page}&size={size}` | Search guilds |
| `GET` | `/{guildId}/audit?page={page}&size={size}` | Get audit log |

### Authentication

All API requests include the player's authentication token in the `Authorization` header:

```
Authorization: Bearer <player_jwt_token>
```

!!! info "Token Source"
    The guild manager component retrieves the auth token from the Identity system. Ensure the player is authenticated before performing guild operations.

---

## Default Rank Configuration

When a guild is created, the following default rank hierarchy is applied:

```cpp
TArray<FFWGuildRank> FFWGuildTypeUtils::GetDefaultRanks()
{
    return {
        { TEXT("rank_leader"),  TEXT("Guild Leader"), 0, 0xFF },  // All permissions
        { TEXT("rank_officer"), TEXT("Officer"),      1, 0x83 },  // Invite|Kick|ViewAuditLog
        { TEXT("rank_veteran"), TEXT("Veteran"),      2, 0x01 },  // Invite
        { TEXT("rank_member"), TEXT("Member"),        3, 0x00 }   // None
    };
}
```

!!! tip "Custom Defaults"
    To customize the default ranks for your project, override `GetDefaultRanks()` in a subclass of the guild settings or modify the backend API to return custom defaults on guild creation.

---

## Logging

The plugin uses the `LogGuild` log category:

```cpp
DECLARE_LOG_CATEGORY_EXTERN(LogGuild, Log, All);
```

Control verbosity via command line or `DefaultEngine.ini`:

```ini
[Core.Log]
LogGuild=Verbose
```

| Verbosity | Content |
|---|---|
| `Error` | HTTP failures, deserialization errors, critical state issues |
| `Warning` | Permission denials, missing components, FWChatSystem not found |
| `Log` | Operation completions, state updates |
| `Verbose` | HTTP request/response details, state cache operations |
