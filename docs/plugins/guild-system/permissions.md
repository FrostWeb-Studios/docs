---
title: Guild Permissions Guide
description: Deep dive into the FWGuildSystem rank permission system, bitmask operations, and rank hierarchy design.
---

# Guild Permissions Guide

The FWGuildSystem uses a bitmask-based permission system that provides granular control over what actions each guild rank can perform. This guide covers the permission model, rank hierarchy design, and implementation patterns.

---

## Permission Model

### Bitmask Overview

Each guild rank has a `uint8` permissions field that stores up to 8 permission flags. Permissions are combined using bitwise OR and checked using bitwise AND.

```cpp
// Permission values
Invite       = 1   (0b00000001)
Kick         = 2   (0b00000010)
Promote      = 4   (0b00000100)
Demote       = 8   (0b00001000)
EditInfo     = 16  (0b00010000)
EditRanks    = 32  (0b00100000)
Disband      = 64  (0b01000000)
ViewAuditLog = 128 (0b10000000)
```

### All Permissions

| Flag | Value | Binary | Description |
|---|---|---|---|
| `Invite` | 1 | `00000001` | Invite new members to the guild |
| `Kick` | 2 | `00000010` | Remove members from the guild |
| `Promote` | 4 | `00000100` | Promote members to a higher rank |
| `Demote` | 8 | `00001000` | Demote members to a lower rank |
| `EditInfo` | 16 | `00010000` | Edit guild name and description |
| `EditRanks` | 32 | `00100000` | Modify rank hierarchy and permissions |
| `Disband` | 64 | `01000000` | Permanently disband the guild |
| `ViewAuditLog` | 128 | `10000000` | View the guild audit log |

---

## Bitmask Operations

### Combining Permissions

=== "C++"

    ```cpp
    // Using enum cast
    uint8 OfficerPerms =
        static_cast<uint8>(EFWGuildPermission::Invite) |
        static_cast<uint8>(EFWGuildPermission::Kick) |
        static_cast<uint8>(EFWGuildPermission::ViewAuditLog);
    // Result: 131 (0b10000011)

    // All permissions
    uint8 LeaderPerms = 0xFF; // 255 (0b11111111)

    // No permissions
    uint8 MemberPerms = 0x00; // 0 (0b00000000)
    ```

=== "Blueprint"

    In Blueprint, use the **Make Bitmask** node with `EFWGuildPermission` and check the desired flags. The resulting integer can be assigned to the rank's `Permissions` field.

### Checking Permissions

```cpp
// Check if a rank has a specific permission
bool bCanKick = (Rank.Permissions & static_cast<uint8>(EFWGuildPermission::Kick)) != 0;

// Using the state component (recommended)
bool bCanKick = GuildState->HasPermission(EFWGuildPermission::Kick);
```

### Adding a Permission

```cpp
Rank.Permissions |= static_cast<uint8>(EFWGuildPermission::Invite);
```

### Removing a Permission

```cpp
Rank.Permissions &= ~static_cast<uint8>(EFWGuildPermission::Invite);
```

### Toggling a Permission

```cpp
Rank.Permissions ^= static_cast<uint8>(EFWGuildPermission::Invite);
```

---

## Rank Hierarchy

### Priority System

Ranks are ordered by a `Priority` integer field where lower values represent higher authority:

- Priority `0` = Highest rank (Guild Leader)
- Priority `N` = Lowest rank (default for new members)

### Rank Enforcement Rules

The following rules are enforced by both the client and the backend API:

1. **Cannot act on equal or higher rank** -- A rank 2 member cannot kick, promote, or demote a rank 1 or rank 2 member.
2. **Cannot promote above own rank** -- An officer (rank 1) cannot promote a member to rank 0 (leader).
3. **Cannot modify own rank** -- A member cannot promote or demote themselves.
4. **Disband is leader-only by default** -- While the `Disband` permission can technically be granted to any rank, the default configuration restricts it to rank 0.

```cpp
// Rank enforcement check
bool FFWGuildTypeUtils::CanActOnRank(
    const FFWGuildRank& SourceRank,
    const FFWGuildRank& TargetRank)
{
    return SourceRank.Priority < TargetRank.Priority;
}
```

!!! warning "Priority Gaps"
    It is valid to have non-contiguous priority values (e.g., 0, 1, 5, 10). The system only compares relative ordering, not sequential numbering. However, promotion and demotion move members to the next adjacent rank in the sorted hierarchy, not by incrementing the priority value.

---

## Rank Design Patterns

### Standard MMO Hierarchy

```
Priority 0: Guild Leader    (255 - all permissions)
Priority 1: Co-Leader       (255 - all permissions except Disband = 191)
Priority 2: Officer         (Invite|Kick|Promote|Demote|ViewAuditLog = 143)
Priority 3: Veteran         (Invite|ViewAuditLog = 129)
Priority 4: Member          (None = 0)
Priority 5: Initiate        (None = 0)
```

### Flat Organization

```
Priority 0: Leader           (255)
Priority 1: Member           (Invite = 1)
```

### Officer-Heavy Structure

```
Priority 0: Guild Master     (255)
Priority 1: Raid Leader      (Invite|Kick|ViewAuditLog = 131)
Priority 2: Class Leader     (Invite|ViewAuditLog = 129)
Priority 3: Recruiter        (Invite = 1)
Priority 4: Raider           (None = 0)
Priority 5: Trial            (None = 0)
```

---

## Permission UI Implementation

### Permission Editor Widget

When building a rank editor UI, present each permission as a checkbox:

```cpp
void URankEditorWidget::PopulatePermissions(const FFWGuildRank& Rank)
{
    InviteCheckbox->SetIsChecked(
        (Rank.Permissions & static_cast<uint8>(EFWGuildPermission::Invite)) != 0);
    KickCheckbox->SetIsChecked(
        (Rank.Permissions & static_cast<uint8>(EFWGuildPermission::Kick)) != 0);
    PromoteCheckbox->SetIsChecked(
        (Rank.Permissions & static_cast<uint8>(EFWGuildPermission::Promote)) != 0);
    // ... repeat for each permission
}

uint8 URankEditorWidget::BuildPermissionMask() const
{
    uint8 Mask = 0;
    if (InviteCheckbox->IsChecked())
        Mask |= static_cast<uint8>(EFWGuildPermission::Invite);
    if (KickCheckbox->IsChecked())
        Mask |= static_cast<uint8>(EFWGuildPermission::Kick);
    if (PromoteCheckbox->IsChecked())
        Mask |= static_cast<uint8>(EFWGuildPermission::Promote);
    // ... repeat for each permission
    return Mask;
}
```

### Conditional UI Based on Permissions

Hide or disable UI elements based on the current player's permissions:

```cpp
void UGuildPanel::RefreshActionButtons()
{
    auto* GuildState = GetGuildState();
    if (!GuildState) return;

    InviteButton->SetVisibility(
        GuildState->HasPermission(EFWGuildPermission::Invite)
            ? ESlateVisibility::Visible
            : ESlateVisibility::Collapsed);

    KickButton->SetVisibility(
        GuildState->HasPermission(EFWGuildPermission::Kick)
            ? ESlateVisibility::Visible
            : ESlateVisibility::Collapsed);

    DisbandButton->SetVisibility(
        GuildState->HasPermission(EFWGuildPermission::Disband)
            ? ESlateVisibility::Visible
            : ESlateVisibility::Collapsed);
}
```

---

## Security Considerations

!!! danger "Server-Side Validation"
    Permission checks in the client are for UI convenience only. The backend API must independently validate all permissions before executing operations. Never trust client-side permission checks for security-critical operations.

### Defense in Depth

The guild system enforces permissions at three layers:

1. **UI Layer** -- Hide buttons and menu items for operations the player cannot perform.
2. **Client Component Layer** -- `UFWGuildManagerComponent` checks `HasPermission()` before sending the HTTP request and fires `OnGuildOperationFailed` with `InsufficientPermission` if denied.
3. **Backend API Layer** -- The server validates the player's rank and permissions against the database before executing any operation.

### Permission Escalation Prevention

The rank system prevents privilege escalation through these rules:

- A rank cannot grant permissions it does not itself possess (enforced by `EditGuildRanks`).
- A rank cannot create or modify ranks with equal or higher priority.
- The guild leader rank (priority 0) can only be transferred, not duplicated.
- `EditRanks` permission does not allow modifying the leader rank's permissions.
