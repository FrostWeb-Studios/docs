---
title: Blueprints - Faction System
---

# Blueprint Integration

FWFactionSystem exposes its entire API to Blueprints through `BlueprintCallable` / `BlueprintPure` functions, `BlueprintAssignable` delegates, and a dedicated Blueprint Function Library.

---

## Adding the Faction Component

1. Open your actor Blueprint.
2. Click **Add Component** in the Components panel.
3. Search for **FW Faction Component** and add it.
4. In the Details panel, set **Default Faction Id** to match one of your faction definitions (e.g., `Ironclad`).

!!! tip "Player Characters"
    For player characters, leave `DefaultFactionId` empty and set the faction at runtime via `SetFaction` -- typically from the server during login or spawn.

---

## Blueprint-Callable Functions

### Faction Identity

| Node | Category | Authority | Description |
|------|----------|-----------|-------------|
| **Get Current Faction Id** | Faction | Any | Returns the actor's current faction ID. |
| **Get Generic Team Id** | Faction | Any | Returns the `FGenericTeamId` mapped from the faction definition. |
| **Set Faction** | Faction | Server Only | Changes the actor's faction. Fires `OnFactionChanged`. |

### Personal Reputation

| Node | Category | Authority | Description |
|------|----------|-----------|-------------|
| **Get Personal Reputation** | Faction \| Reputation | Any | Returns the personal reputation score toward a faction. |
| **Modify Personal Reputation** | Faction \| Reputation | Server Only | Adds or subtracts from personal reputation. Fires `OnReputationChanged`. |
| **Set Personal Reputation** | Faction \| Reputation | Server Only | Sets personal reputation to an absolute value. |

### Persistence

| Node | Category | Authority | Description |
|------|----------|-----------|-------------|
| **Get Faction Player Data** | Faction \| Persistence | Any | Exports current state as a serializable struct. |
| **Apply Faction Player Data** | Faction \| Persistence | Server Only | Applies loaded data from backend. |

---

## Blueprint-Assignable Events

Bind to these events in your Blueprint's Event Graph to react to faction state changes.

### OnFactionChanged

Fired when the actor's faction changes.

| Parameter | Type | Description |
|-----------|------|-------------|
| Old Faction Id | `Name` | Previous faction. |
| New Faction Id | `Name` | New faction. |

**Example usage:** Update nameplate color, play faction-change VFX, notify party members.

---

### OnReputationChanged

Fired when personal reputation toward any faction changes.

| Parameter | Type | Description |
|-----------|------|-------------|
| Faction Id | `Name` | The faction whose reputation changed. |
| Old Score | `Float` | Previous reputation score. |
| New Score | `Float` | Updated reputation score. |

**Example usage:** Update reputation progress bar, show floating text "+10 Shadow Covenant reputation".

---

### OnAttitudeChanged

Fired when an attitude change affects this actor.

| Parameter | Type | Description |
|-----------|------|-------------|
| Event | `FFWAttitudeChangeEvent` | Struct containing source/target factions, old/new attitudes, and new score. |

**Example usage:** Play alert sound when a previously neutral faction becomes hostile.

---

## Blueprint Function Library

The `UFWFactionBlueprintLibrary` provides static helper nodes available anywhere in Blueprints without needing a component reference.

### GetFactionComponent

Finds the `UFWFactionComponent` on an actor. Searches the actor itself, and if the actor is a Controller, checks its controlled Pawn (and vice versa).

**Input:** Actor (defaults to Self)
**Output:** FW Faction Component (or null)

---

### AreActorsHostile

Returns true if two actors are hostile to each other based on their faction components.

```
[Actor A] ---> [Are Actors Hostile] ---> Boolean
[Actor B] ---|
```

---

### AreActorsFriendly

Returns true if two actors are friendly (Allied or Friendly attitude).

---

### AreActorsSameFaction

Returns true if both actors share the same `CurrentFactionId`.

---

### GetAttitudeDisplayName

Converts an `EFWFactionAttitude` enum value to a localized display name (e.g., `Hostile` to "Hostile").

---

### GetAttitudeColor

Returns a `FLinearColor` representing an attitude level. Useful for coloring nameplates, health bars, or minimap markers.

| Attitude | Default Color |
|----------|---------------|
| Allied | Green |
| Friendly | Light Green |
| Neutral | Yellow |
| Unfriendly | Orange |
| Hostile | Red |

---

## Subsystem Access in Blueprints

To access `UFWFactionSubsystem` from Blueprints:

1. Use the **Get World Subsystem** node.
2. Set the class to `FW Faction Subsystem`.
3. Call any of its `BlueprintPure` or `BlueprintCallable` functions.

### Commonly Used Subsystem Nodes

| Node | Description |
|------|-------------|
| **Get Faction Definition** | Look up a faction DataAsset by FactionId. |
| **Get All Factions** | Get an array of all loaded faction definitions. |
| **Get Faction Attitude** | Query the global attitude between two factions. |
| **Get Attitude Between Actors** | Query the resolved attitude between two actors (personal rep first, then global). |
| **Modify Faction Reputation** | Change the global reputation between two factions (with optional propagation). |
| **Get Components In Faction** | Get all faction components belonging to a specific faction. |
| **Save Player Faction Data** | Trigger a save operation via the persistence handler. |
| **Load Player Faction Data** | Trigger a load operation via the persistence handler. |

---

## Blueprint Example: Faction Selection UI

```
Event BeginPlay
  |
  v
Get World Subsystem (FWFactionSubsystem)
  |
  v
Get All Factions
  |
  v
For Each Loop
  |
  v
  +-- Get DisplayName --> Create Widget (Faction Button)
  +-- Get FactionColor --> Set Button Color
  +-- Get Icon --> Set Button Image
  |
  v
Add to Scroll Box

// On Button Click:
Event OnFactionSelected(FactionId)
  |
  v
Get Faction Component (Self)
  |
  v
Set Faction (FactionId)  // Server-only; use Server RPC from client
```

!!! warning "Authority"
    `SetFaction` and all reputation modification functions are `BlueprintAuthorityOnly`. In multiplayer, call them from server-side logic (GameMode, server RPCs) or use the built-in `Server_SetFaction` RPC.
