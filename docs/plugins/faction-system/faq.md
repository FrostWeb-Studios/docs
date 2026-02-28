---
title: FAQ - Faction System
---

# Frequently Asked Questions

---

## General

### How many factions can I have?

There is no hard limit on the number of factions. The `TeamId` field supports 0--254 (255 is reserved for NoTeam), so you can have up to 255 factions with unique team IDs. However, the reputation matrix grows quadratically with the number of factions, so performance-sensitive projects should keep the count reasonable (under 50 is recommended for most games).

### Can an actor belong to multiple factions?

The component tracks a single `CurrentFactionId` as the primary faction. The `FFWFactionMembership` struct exists to support multi-faction scenarios, but the current attitude resolution always uses `CurrentFactionId`. Multi-faction membership can be tracked via `PersonalReputations` -- an actor with positive personal reputation toward multiple factions will be treated favorably by all of them.

### Does the plugin work in singleplayer?

Yes. All replication code gracefully degrades in standalone mode. The subsystem initializes, faction components register, and attitude resolution works identically. Server-authoritative functions execute without network validation in standalone.

---

## Replication

### What gets replicated?

| Data | Replication | Visibility |
|------|-------------|------------|
| `CurrentFactionId` | `DOREPLIFETIME` | All clients |
| `PersonalReputations` | `DOREPLIFETIME_CONDITION` | Owner only |
| Global reputation matrix | Not replicated | Server-side only |

### How do clients know about the global reputation matrix?

They do not need to. Attitude resolution happens server-side. Clients receive the results through replicated properties on the faction component (`CurrentFactionId`) and through the replicated `OnRep_` callbacks. For UI purposes, clients can query their own personal reputation (which is replicated to the owning client).

### Can clients call SetFaction directly?

No. `SetFaction` is `BlueprintAuthorityOnly`. Clients must use the `Server_SetFaction` RPC, which includes validation to ensure the request is legitimate.

---

## AI Integration

### How does SolveTeamAttitude work?

`SolveTeamAttitude` is a static function that:

1. Finds `UFWFactionComponent` on both the source and target actors.
2. If either component is missing, returns `ETeamAttitude::Neutral`.
3. Queries the `UFWFactionSubsystem` for the attitude between the two actors (personal reputation overrides checked first, then global matrix).
4. Converts the five-level `EFWFactionAttitude` to Unreal's three-state `ETeamAttitude` using `FWFactionUtils::ConvertToTeamAttitude`.

### Do I need to implement IGenericTeamAgentInterface?

You need to override `GetTeamAttitudeTowards()` in your AI Controller to call `UFWFactionComponent::SolveTeamAttitude()`. The `FGenericTeamId` is automatically resolved from the faction component's TeamId via `GetGenericTeamId()`.

### Will AI Perception automatically detect faction changes?

AI Perception caches team affiliations. When a faction changes at runtime, the AI Perception system should re-evaluate targets on its next update tick. For immediate response, you may need to manually request a perception refresh on the AI Controller.

---

## Reputation

### What is the difference between personal and global reputation?

- **Global reputation** lives in the subsystem's reputation matrix and affects all actors in those factions. Changing it shifts the relationship between entire factions.
- **Personal reputation** lives on individual `UFWFactionComponent` instances. It overrides the global matrix for that specific actor, allowing individual standing.

### How is attitude resolved when both personal and global reputation exist?

Personal reputation takes priority. If an actor has a personal reputation entry for a target faction, that score is used. If no personal entry exists, the global matrix score is used.

### Can reputation go below or above any limits?

The raw score has no built-in clamp. You can set reputation to any float value. The thresholds determine which attitude band the score falls into. If you need clamping, implement it in your game logic before calling `ModifyPersonalReputation` or `ModifyFactionReputation`.

### How does reputation propagation work?

When `ModifyFactionReputation` is called with `bPropagate = true`:

1. The delta is applied to the source-target pair.
2. The subsystem checks the source faction's `PropagationRules`.
3. For each rule, the delta is multiplied by the `FalloffMultiplier` and applied to the rule's `TargetFaction`.
4. Propagation does not chain (only one level deep).

!!! warning
    Personal reputation changes (`ModifyPersonalReputation`) do **not** trigger propagation. Propagation only applies to global reputation changes.

---

## Persistence

### What happens if I do not configure a persistence handler?

Faction data exists only at runtime. When the server shuts down or a player disconnects, their faction state is lost. This is fine for session-based games or games that manage persistence externally.

### Can I use Unreal's SaveGame system?

Yes. Create a subclass of `UFWFactionPersistenceHandler` that uses `USaveGame` / `UGameplayStatics::SaveGameToSlot`. The handler receives `FFWFactionPlayerData` which is a simple struct that serializes cleanly.

### When should I call SavePlayerFactionData?

Common patterns:

- On player disconnect (in `GameMode::Logout`)
- After significant reputation changes
- At periodic auto-save intervals
- On level transitions

---

## Debugging

### The subsystem reports zero factions at startup

Check that:

1. The `FactionDef` PrimaryAssetType is registered in **Project Settings > Asset Manager**.
2. The scan directories include the folder where your DataAssets live.
3. The DataAssets actually inherit from `UFWFactionDefinition`.
4. `Apply Recursively` is checked if your assets are in subdirectories.

### Personal reputation is not replicating to the client

`PersonalReputations` replicates only to the **owning client**. If you are testing in PIE with a listen server, check that you are querying from the correct player's perspective. Autonomous proxy pawns receive their own personal reputations; simulated proxies do not.

### AI NPCs do not react to faction changes

Ensure your AI Controller overrides `GetTeamAttitudeTowards` and calls `UFWFactionComponent::SolveTeamAttitude`. Also verify that the AI Perception component has the correct detection-by-affiliation settings (Detect Enemies checked).
