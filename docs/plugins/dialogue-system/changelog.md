---
title: Changelog - FWDialogueSystem
---

# Changelog

All notable changes to FWDialogueSystem are documented on this page.

The format follows [Keep a Changelog](https://keepachangelog.com/). Versions use [Semantic Versioning](https://semver.org/).

---

## [1.0.0] -- 2026-02-27

### Added

- **UFWDialogueComponent** -- Core actor component with full Blueprint API for managing NPC conversations. Supports starting, navigating, and ending dialogues.
- **UFWDialogueTree** -- Data asset for defining complete conversation flows as arrays of dialogue nodes.
- **FFWDialogueNode** -- Struct representing a single dialogue node with speaker name, dialogue text, entry actions, and player responses.
- **FFWDialogueResponse** -- Struct representing a player response option with text, next node navigation, conditions, and actions.
- **Condition system** -- Extensible base class `UFWDialogueConditionBase` with virtual `Evaluate` method for gating response visibility.
- **FWDlgCondition_HasItem** -- Built-in condition: checks item ownership.
- **FWDlgCondition_SkillLevel** -- Built-in condition: checks minimum skill level.
- **FWDlgCondition_QuestState** -- Built-in condition: checks quest state (NotStarted, InProgress, Completed).
- **FWDlgCondition_HasCurrency** -- Built-in condition: checks minimum currency amount.
- **Action system** -- Extensible base class `UFWDialogueActionBase` with virtual `Execute` method for triggering gameplay effects from dialogue.
- **FWDlgAction_OpenVendor** -- Built-in action: opens vendor interface.
- **FWDlgAction_GiveItem** -- Built-in action: grants items to the interactor.
- **FWDlgAction_StartQuest** -- Built-in action: starts a quest for the interactor.
- **FWDlgAction_GiveCurrency** -- Built-in action: grants currency to the interactor.
- **Event delegates** -- OnDialogueStarted, OnDialogueEnded, OnNodeChanged for UI integration.
- **Full Blueprint exposure** -- All component functions, structs, enums, conditions, and actions are accessible from Blueprints.
- **FText support** -- All dialogue text fields use FText for Unreal localization pipeline compatibility.
