---
title: FAQ - FWCustomizationSystem
---

# Frequently Asked Questions

---

## General

### Does FWCustomizationSystem require any other plugins?

No. FWCustomizationSystem has no external dependencies. It works standalone in any UE5 C++ project.

### Can I add custom races beyond Human, Dwarf, and Orc?

The `EFWRace` enum is defined in plugin source. To add new races, you would need to extend the enum and add corresponding entries in the database. Since this modifies plugin source, plan for merge conflicts during plugin updates.

### Does this plugin support Blueprint-only projects?

No. FWCustomizationSystem contains C++ modules and requires a C++ project. If your project is Blueprint-only, convert it by adding any C++ class through the Unreal Editor.

---

## DMI and Materials

### My textures are not updating when I change slots. What is wrong?

The most common cause is a parameter name mismatch. The component sets DMI parameters by name (e.g., `Diffuse`, `Normal`, `MetallicRoughness`, `Mask`). Verify that your base materials expose Texture Parameter nodes with these exact names.

!!! tip "Debugging DMI Parameters"
    Select the character mesh at runtime and inspect the material instances in the Details panel. You can see which parameters are set and their current values. Compare these against your material's expected parameter names.

### Can I use custom parameter names instead of the defaults?

The current version uses fixed parameter names internally. If your materials use different naming conventions, you will need to either rename your material parameters to match or modify the component source.

### My character flickers when changing customization options. Why?

This typically happens when `CreateDynamicMaterialInstances` is being called every frame or on every slot change. The component should only recreate DMIs when the underlying mesh changes (race/gender swap). Check that you are not calling `RefreshVisuals` on tick or in a loop.

### What happens to the DMIs when the mesh is replaced?

DMI references are invalidated when the owning mesh component changes. The component automatically recreates DMIs when you call `RefreshVisuals` after a race or gender change. Do not cache DMI pointers long-term.

---

## Race and Gender

### What happens if I change race at runtime?

Calling `SetRace` resolves a new `UFWRaceConfig` from the database. Since different races have different option arrays (and potentially different counts), all slot indices reset to 0. You must call `RefreshVisuals` after changing race.

### Can I have gender-specific option sets?

The race config itself is not gender-separated, but individual option assets can contain gender-specific data internally. For entirely different option lists per gender, filter the arrays in your UI logic using the component's `GetGender()` return value.

---

## Profiles and Replication

### How large is the FFWCustomizationProfile struct?

Approximately 8 bytes. It packs the race, gender, and all slot indices into a compact binary format. This makes it efficient for both SaveGame serialization and network replication.

### Can I replicate customization across the network?

Yes. Add `FFWCustomizationProfile` as a `UPROPERTY(Replicated)` on your PlayerState. When the value changes on a client, call `SetProfile` on that client's customization component. See the [tutorial](tutorials.md#part-5----saving-and-loading-profiles) for details.

### What if the profile references an option index that no longer exists?

If an option is removed from a race config after profiles have been saved, the component clamps out-of-range indices to the valid range. This prevents crashes but may result in a different visual than expected. Consider versioning your profiles if you frequently change option arrays.

---

## Performance

### How many customization options can I have per race before performance suffers?

The component itself has no hard limit. The main constraint is memory -- each texture-based option adds texture assets that may be loaded on demand. Practical recommendation: keep each slot under 32 options to limit total texture memory. Use 1024x1024 textures for body/head and 512x512 for accessories.

### Are textures loaded all at once or on demand?

On demand. All texture references in `FFWCustomizationTextureSet` use `TSoftObjectPtr`, which means textures are loaded asynchronously when first accessed. Only the currently selected options consume GPU memory.

### Is there a frame cost to changing customization slots?

Changing a DMI-driven slot (skin tone, eye color, etc.) involves setting texture parameters on an existing material instance, which is a trivial GPU operation. Mesh swap slots (hair, facial hair, tusks) involve component creation/destruction, which has a small one-time cost. Neither should cause visible hitches.

---

## Unlock System

### Does the plugin enforce unlock requirements automatically?

No. The plugin stores `FFWUnlockRequirement` data on each option asset, but your game logic is responsible for checking requirements before allowing selection. The component will apply any index passed to `SetSlotSelection` regardless of lock state.

### How should I integrate unlock checking with my UI?

Query each option's `UnlockRequirement` when building your UI list. Grey out or hide options that the player has not unlocked. Only call `SetSlotSelection` for options the player is allowed to use.

---

## Troubleshooting

### The component compiles but no visuals change at runtime.

Checklist:

1. Is a `UFWCustomizationDatabase` assigned? Call `GetDatabase()` to verify.
2. Does the database contain a `UFWRaceConfig` for the current race? Call `GetCurrentRaceConfig()` to verify.
3. Are there options in the race config arrays? Check that arrays are not empty.
4. Does the owning Actor have a Skeletal Mesh Component? The component needs a mesh to create DMIs on.
5. Did you call `RefreshVisuals` after initial setup?

### Mesh attachments (hair, tusks, earrings) are not appearing.

Verify that:

- The option asset's `Mesh` field is not null.
- The mesh asset is valid and has proper LODs.
- The owning Actor's skeleton is compatible with the attachment mesh (if using skeletal meshes).

### The character looks correct in editor but wrong at runtime.

This usually means `RefreshVisuals` is being called before the database is assigned, or the race/gender are still at default values when the visual refresh runs. Ensure your initialization order is: `SetDatabase` > `SetRace` > `SetGender` > `RefreshVisuals`.
