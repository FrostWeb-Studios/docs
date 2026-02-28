---
title: Migration Guide - Faction System
---

# Migration Guide

This page documents breaking changes and migration steps between versions of FWFactionSystem.

---

## Version 1.0 (Current)

This is the initial release. No migration steps are required.

---

## Future Migration Notes

When upgrading between major versions, this page will document:

- **Breaking API changes** -- renamed or removed functions, changed signatures.
- **DataAsset schema changes** -- new required fields, deprecated properties.
- **Replication changes** -- any modifications to replicated property conditions.
- **Configuration changes** -- new project settings, changed defaults.
- **Step-by-step upgrade instructions** with code examples.

!!! tip "Before Upgrading"
    Always follow these steps when upgrading the plugin:

    1. Back up your project (source control recommended).
    2. Read the [Changelog](changelog.md) for the target version.
    3. Review this migration guide for breaking changes.
    4. Update the plugin files.
    5. Regenerate project files (right-click `.uproject` > Generate Project Files).
    6. Build and resolve any compile errors using this guide.
    7. Test faction resolution, replication, and persistence in a PIE session.
