# Minecraft 1.21.1 -> 1.21.8 Mod Migration Primer (consolidated)

This primer consolidates the per-version migration notes from 1.21.2 through 1.21.8 into a single, in-depth reference so a mod written for 1.21.1 can be upgraded directly to 1.21.8. It reconciles conflicts between intermediate primers and prefers the final behavior present in 1.21.8 when earlier primers diverge.

This is a high-level but detailed guide — it lists every major topic and change referenced in the 1.21.2 → 1.21.8 primers and gives concrete migration guidance and actionable steps so you don't have to upgrade step-by-step through intermediate minor versions.

License: Creative Commons Attribution 4.0 International.

---

## How to use this consolidated primer

- Read the "Migration checklist" and apply the steps in order.
- Consult each topic section for "What changed" and "Action" items relevant to your mod.
- Use the "Resolved conflicts" section first — it summarizes places where earlier primers differed and states which final behavior to adopt.
- If you need example code for a particular change, request the specific section and I will provide snippet(s) for that migration.

---

## Quick summary

- Major shifts are in registry/holder handling, rendering/model pipeline and item render states, data-driven codecs/registries (recipes/consumables/instruments), shader and pipeline rewrites, GUI internals, game test overhauls, and Blaze3D uniform/buffer handling.
- Many classes and methods were renamed, privatized, or removed over the period; adopt the 1.21.8 API shape.
- Prefer holder-backed registry access and the new codec/registry-driven datapack formats.
- Replace removed and deprecated client render primitives with the Tracking/RenderPipelines/Blaze3D APIs introduced through 1.21.7 and finalized in 1.21.8.

---

## Migration checklist (practical order)

1. Update mappings and workspace to 1.21.8.
2. Compile and list errors; fix missing types/methods first (constructors removed, privatizations).
3. Migrate registry access to holder-backed patterns.
4. Update BlockEntityTypes registration/lookup for privatization changes.
5. Convert data providers / JSON reload listeners to codecs / registry-backed formats.
6. Replace `ItemStackRenderState` usage with `TrackingItemRenderState` and update custom pipelines.
7. Update model baking and item model hooks to match new model pipeline and render state shapes.
8. Update custom shader programs and shader file handling.
9. Switch OpenGL/Blaze3D buffer and uniform logic to new helpers (DirectStateAccess, CommandEncoder, Uniform types).
10. Test client and server thoroughly (rendering, recipes, block entities, commands, game tests).
11. Address any remaining runtime warnings or deprecations.

---

## Resolved conflicts — prefer these 1.21.8-final decisions

- Item rendering: 1.21.7/1.21.8 use TrackingItemRenderState; ItemStackRenderState has been partially removed (clearModelIdentity removed). Migrate to TrackingItemRenderState and update calls to getModelIdentity.
- Render pipelines: The 1.21.7+ pipeline and blend function changes (GUI_TEXT_HIGHLIGHT using ADDITIVE and the new GUI_INVERT pipeline) are the authoritative changes — adapt custom render pipelines accordingly.
- BlockEntityType access: Intermediate primers discuss changes that culminated in privatization; use the final registry-backed approach in 1.21.8.
- Data providers / codecs: 1.21.2–1.21.6 moved many structures to registry/codecs; 1.21.8 expects codec/registry-backed formats. Prefer codecs and registry formats over older JSON/provider styles.
- Shader & GL helpers: Prefer the final helpers (DirectStateAccess.copyBufferSubData, CommandEncoder.copyToBuffer, GraphicsWorkarounds) when replacing older manual sequences.

If any other explicit conflict is discovered in your codebase, list it and this primer will be updated to reconcile.

---

## Detailed consolidated topics

For each entry below you'll find "What changed" and "Action" subsections. Read the Action and apply it to your mod.

### Pack Changes
What changed:
- Multiple changes to pack layout/metadata and how client assets and resource packs are loaded across the intermediate versions.
Action:
- Use the final 1.21.8 pack metadata expectations and resource loader hooks. Validate your pack.mcmeta and asset paths against 1.21.8 behavior and update any custom resource reload listeners to use codecs where applicable.

---

### The Holder Set Transition (registries)
What changed:
- Movement from direct registry singletons to holder-backed, holder-set, holder-getter patterns. Many APIs now return holders/holder references rather than raw registry objects.
Action:
- Replace direct registry.get() or static access with the holder-backed API: use Holder, HolderSet, HolderLookup, and registry accessors provided by the 1.21.8 mappings. Audit code that caches raw references — replace with holders or safe lookups that tolerate registry reloads.

---

### Gui Render Types / GUI internals / GuiGraphics
What changed:
- Rework of GUI rendering internals. New render pipelines (GUI_INVERT, GUI_TEXT_HIGHLIGHT changes), GuiGraphics helpers (textHighlight), element ordering and strata, scissoring behavior and render-state encapsulation (GuiElementRenderState, GuiItemRenderState changes).
Action:
- Replace direct calls to deprecated GUI internals with GuiGraphics APIs. Update custom pipelines to use RenderPipelines and the new blend functions. If you relied on old scissor/ordering mechanics, test and update element ordering and scissoring according to the new strata/tree rules.

---

### Shader Rewrites / Shader Files / Shader Programs
What changed:
- Shader file handling and program pipeline shapes were rewritten; shader inputs and uniform handling changed with additional Blaze3D/Uniform helper types.
Action:
- Rework custom shader files and program creation to match new shader program registration and uniform interfaces. Replace direct GL uniform setting with the new typed Uniform interfaces where available.

---

### Entity Render States / Model Baking / Model Rework
What changed:
- Multiple rewrites of model baking pipeline, dynamic item models, and how entity and item render states are tracked. New TrackingItemRenderState introduced; model identity methods moved.
Action:
- Migrate any custom model baking handlers to the 1.21.8 model bake hooks. Replace ItemStackRenderState usage with TrackingItemRenderState and update model identity calls. Revisit special dynamic models and conditional/composite model handling described below.

---

### Equipments and Items, Models and All (Item model system)
What changed:
- Item model system rework: new special models, composite and conditional property models, ranged/select/conditional property model types, tint sources, custom model definitions, dynamic models, and changes to rendering an item.
Action:
- If you add custom item models, reauthor them to follow the newer json model variations (ranged/select/conditional/composite). Use the provided tint source conventions and adapt baker code for special dynamic models. For runtime item rendering, use the updated item render states and the RenderPipelines expected in 1.21.7/1.21.8.

---

### Item Names and Models / Enchantable, Repairable Items
What changed:
- Adjustments to how item display names, model lookups and repair/enchant properties are represented (data components).
Action:
- Migrate to the new item data components where applicable. Check custom items for compatibility with new equipable, repairable and enchantable component shapes.

---

### Elytras -> Gliders
What changed:
- The Elytra concept was renamed/expanded into a more generic Glider/flight system in the intermediate primers and finalized by later versions.
Action:
- If your mod referenced Elytra-specific types, migrate to the Glider/flight interfaces and registration patterns expected by 1.21.8.

---

### Tools, via Tool Materials / ArmorMaterial / Equipment
What changed:
- Tool and armor materials were refactored to data-driven or registry-backed materials; equipment models and textures and equipment registration changed.
Action:
- Move tool/armor material definitions to the new material interfaces or registries. Ensure your equipment registration uses the new Equippable data components and any registries introduced in the final version.

---

### The Data Components / Consumables / ConsumableListener / ConsumeEffect
What changed:
- Consumables and data component systems were converted into richer, codec-backed components with listeners, effects, on-override-sound hooks, and cooldown handling.
Action:
- Convert consumable JSON/data definitions into codec-backed registries and implement ConsumableListener and ConsumeEffect where required. Handle OnOverrideSound and cooldown behavior through the new component hooks.

---

### Interaction Results
What changed:
- Interaction result enumeration/semantics were refined in several iterations.
Action:
- Verify code expecting older InteractionResult semantics and adapt to the final result shapes and server/client handling in 1.21.8.

---

### Instruments, Trial Spawner Configurations (Data pack changes)
What changed:
- Instruments and spawner configs were moved to datapack formats and now expect registry/codecs.
Action:
- Convert custom instrument or trial spawner definitions to datapack registrations and codecs. Use the registry-backed providers to supply your values.

---

### Recipe Providers / The Ingredient Shift / Recipes -> Registry format
What changed:
- Recipes moved toward registry formats with codecs; recipe displays and placements changed; recipe books and book categories required registry integration.
Action:
- Convert recipe JSON/providers to registry-backed recipes and implement codecs for custom recipe types. Update recipe book categories and displays to match the new registry format.

---

### BlockEntityTypes Privatized / Handling Removal of Block Entities Properly
What changed:
- BlockEntityType direct fields were privatized; removal and lifecycle behavior of BlockEntities changed.
Action:
- Update BlockEntityType registration to use the new registry patterns and avoid direct field access. Handle removal and cleanup according to the new lifecycle APIs and ensure your BlockEntity code checks isValidBlockState where applicable.

---

### Voxel Shape Helpers
What changed:
- New helpers and utilities were added for shape handling.
Action:
- Replace ad-hoc voxel shape code with provided helpers to improve compatibility and performance.

---

### Weighted List Rework / Tickets / Ender Pearl Chunk Loading
What changed:
- Systems like weighted lists, ticketing and chunk loading semantics were reworked.
Action:
- Migrate to the new APIs for weighted lists and ticketing. Check code that relied on older chunk loading behaviors for ender pearls and adapt to the new guarantees/semantics.

---

### Game Test Overhaul (tests, environment, custom types)
What changed:
- Game test APIs, environment, test instances, function composites, custom types, test data and test instance creation were overhauled heavily.
Action:
- Update any game tests to the new environment model and test function formats. Recreate custom test instances and data following the 1.21.8 test API.

---

### Data Component Getters (Items, Entities), Spawn Conditions, Variant Datapack Registries
What changed:
- Data component getter shapes changed and new registry patterns for variants/spawn conditions introduced.
Action:
- Update getter usage for items and entities, and migrate any spawn condition or variant registries to the registry-backed datastructure.

---

### Client Items — Basic Model / Tint Sources / Ranged/Select/Conditional/Composite Models / Special Dynamic Models
What changed:
- New JSON model types and rendering conventions were introduced for items, including tint sources and property-driven models.
Action:
- Reauthor item model JSONs for the new model types. If you have dynamic models you must implement the new baker contracts and ensure your model state resolves through the TrackingItemRenderState pipeline.

---

### Ranged Property Model / Select Property Model / Conditional Property Model / Composite Model
What changed:
- These new model types allow dynamic selection of sub-models by property ranges, select lists, conditions, or composites.
Action:
- Replace older custom model selection code with these standardized model JSON types. Test combinations of properties to ensure deterministic model selection.

---

### Custom Item Model Definitions / Rendering an Item
What changed:
- Custom item model definition formats and rendering invocation points changed.
Action:
- Convert custom model definitions to the updated format and ensure you use the new render entry points and states for rendering the items.

---

### Mob Replacing Current Items
What changed:
- Hooks where mobs replaced items in-world were updated.
Action:
- Verify your mob/item replacement logic against the new event/behavior hooks.

---

### Particles, Render Types
What changed:
- Particle rendering moved further into render types and pipelines.
Action:
- Update particle renderers to use the new RenderTypes and pipeline registration to ensure correct blending and z-ordering.

---

### SimpleJsonResourceReloadListener / MetadataSectionSerializer → Codecs
What changed:
- The legacy resource reload listener and metadata serializers were replaced by codecs and codec-based reload listeners.
Action:
- Rewrite your resource reload listeners to use codecs and codec-backed readers. Remove MetadataSectionSerializer usages and adopt the codec-based approach.

---

### Music Volume Controls / Map Textures / Texture Atlas Reworks
What changed:
- Music handling gained volume control options; texture atlas and map texture handling were reworked.
Action:
- Update custom music providers to expose volume controls if needed. Migrate atlas registration and texture usage to the new atlas rework API.

---

### Tag Providers: Appender Rewrite / Copying Tags
What changed:
- Tag provider APIs (appenders) and copy utilities were rewritten.
Action:
- Replace old tag provider extension points with the new appender-style APIs and if you used copying helpers adapt to the new block/item tag copy utilities.

---

### Generic Encoding and Decoding: Replacing Direct NBT Access
What changed:
- New generic encoding/decoding systems were introduced to replace direct NBT manipulation in many data paths.
Action:
- Where you previously serialized to NBT directly for custom registry/data types, adopt the new codec- or encoding-based approach and use the provided ProblemReporter to handle errors.

---

### Consecutive Executors / Codecable JSON Reload Listener / Reload Instance Creation
What changed:
- Reload and executor systems were updated for deterministic order and codec support.
Action:
- Update reload listeners and any custom executor usage to the new consecutive/deterministic executors and codec-compatible reload mechanisms.

---

### Game Profiler / Tracy Client / Tick Throttler / Profilers
What changed:
- Profiling tools and tick throttler systems were introduced/refined.
Action:
- If you integrated with profiling, update to the final profiling client hooks. Adjust tick throttling usage to the new throttler APIs.

---

### Context Keys / Permission Sources / Descoping Player Arguments
What changed:
- Several command and context semantics changed: player arguments were descoped, permission sources consolidated, and keys added for contexts.
Action:
- Update commands and selectors to use the new argument parsers and permission checks (including EntitySelectorParser#allowSelectors). Ensure commands use new context keys and respect the permission source model.

---

### MacOsUtil#IS_MACOS / Util#isAarch64 / GraphicsWorkarounds
What changed:
- Utilities for platform detection and graphics workarounds were added (Util.isAarch64, GraphicsWorkarounds).
Action:
- Use `Util.isAarch64` for architecture checks and prefer `GraphicsWorkarounds` for vendor/driver specific behavior. Remove fragile OS/driver detection code in favor of these utilities.

---

### Blaze3D Changes: Buffer Slices / DirectStateAccess / CommandEncoder / Uniform Rework / Texel Buffers
What changed:
- Buffer slicing APIs, direct state access helpers, command encoders, typed uniform interfaces and texel buffer support were added/reworked.
Action:
- Replace manual GL buffer slice and copy logic with DirectStateAccess and CommandEncoder facilities. Use the typed Uniforms APIs and new uniform type/blocks for shader data. Convert any texel buffer handling to the new APIs.

---

### Render Pass Scissoring now only OpenGL
What changed:
- Scissoring behavior was restricted to OpenGL backends for some render passes.
Action:
- Guard OpenGL-specific scissoring code with backend checks or use engine-provided scissoring abstractions.

---

### Render Pipeline Rework / Abstracting OpenGL / Object References / Post Effects
What changed:
- The render pipeline was reworked to abstract OpenGL and handle post effects, pipeline composition, and object references differently.
Action:
- Refactor custom pipeline code to use RenderPipelines compositional builders and avoid direct OpenGL calls except through the supported abstractions.

---

### Model Rework / Block Generators / Variant Mutator
What changed:
- Model generator and variant mutator infrastructure changed; block model generation tools were changed to support the reworked model pipeline.
Action:
- Rebuild block model generator code and variant mutators using the new generator APIs and ensure texture/model variant generation uses the updated mutator API.

---

### Tag Changes / New Tags
What changed:
- Many tags were renamed, removed, or added.
Action:
- Reconcile tag names used by your mod with 1.21.8 tag sets. Replace or add tag suppliers where necessary.

---

### Fuel Values / Light Emissions / Orientations / Minecart Behavior / Explosions / Carving Generation Step removal
What changed:
- Small but important behavior changes: fuel values, light emission semantics, block orientations, minecart behavior, explosion parameters and the carving generation step removal.
Action:
- Test gameplay-affecting code and adapt to new defaults. For world-gen and terrain algorithms, account for removal of carving as a separate step.

---

### Client Assets / Client Items / Entities / Client Player / Server Player Changes
What changed:
- Client/server player handling and entity client asset loading changed; server->client boundaries may have been altered in a few fields.
Action:
- Test multiplayer interactions and revise any use of client-only fields. Audit reflective accesses that target player internals.

---

### Consumables: OnOverrideSound / ConsumableListener / ConsumeEffect / On Use Conversion / Cooldowns
What changed:
- Consumable data components and listener models were formalized into codecs and registries including override sound, effects, and cooldown semantics.
Action:
- Move consumable definitions into codec-backed registries and implement listeners/effects as provided by 1.21.8 APIs.

---

### Recipes: Recipe Book Changes / Displays / Placements / Technical Changes / Creating Recipe Book Categories
What changed:
- Recipe book rendering and category creation were changed along with recipe displays and how placements are requested.
Action:
- Register recipe categories via the new registry format and update recipe display providers to the new display contract.

---

### Instruments and Trial Spawner Configurations in Datapacks
What changed:
- Instruments and trial spawners moved to datapack-driven registries.
Action:
- Register these datapack objects via the registry codec and validate parsing on reload.

---

### Tag Providers / Copying Tags (Block and Item)
What changed:
- Tag provider appender rewriting and tag copying helpers updates.
Action:
- Use appender-pattern tag providers and the new copy APIs for block/item tags.

---

### Generic Encoding/Decoding / NBT Impl / Problem Reporter
What changed:
- The code path for encoding/decoding generic objects was reworked; NBT impls and problem reporting were introduced or revised.
Action:
- Replace direct NBT writes/reads for registry data with codec-based encoding and use the ProblemReporter for robust reload-time error messages.

---

### Game Test Instance / Test Data / Test Functions / Custom Types
What changed:
- Game test environment internals changed; test instance creation and custom test functions are updated.
Action:
- Update your tests and test data format to the final 1.21.8 game test API.

---

### Saved Data now with Types / Registry Context Swapper / Reload Instance Creation
What changed:
- Saved data gained stronger typing and registry contexts became swappable for reloads.
Action:
- Convert saved data implementations to the typed saved data pattern and ensure your reload logic uses the registry context swapper to be safe during reloads.

---

### JOML Backing Interfaces / Very Technical Changes
What changed:
- Internal math and vector types (JOML) received backing interface changes that may affect method signatures.
Action:
- Update any custom vector/matrix code that bridged into Minecraft internals to use the new backing interfaces.

---

### Timer Callbacks now Codecs / Animation Baking / ChunkSectionLayers
What changed:
- Timer callback definitions moved to codec-backed registries, animation baking internals changed, chunk section internals were renamed/adjusted.
Action:
- Rework timer callback providers, update baked animation exporters, and adapt chunk section referencing.

---

### Entity References / Mob Conversions / Spawn Conditions
What changed:
- Entity referencing and mob conversion behavior changed; spawn conditions are now registries or holders.
Action:
- Migrate entity references to holder-backed references and convert any mob conversion tables to the new registries.

---

### Ender Pearl Chunk Loading / Profiler/tracy / Tick Throttler
What changed:
- Edge-case handling and profiling tools were improved.
Action:
- Validate ender-pearl chunk load boundaries and adapt profiling integration points; update tick throttler usage.

---

### Minor Migrations (lists of additions/changes/removals)
What changed:
- Many small API additions (several Blaze3D and util helpers), renames, deprecations and removals across the versions.
Action:
- Review compiler errors and adapt per-symptom: replace removed methods with the recommended replacements listed above (e.g., DirectStateAccess.copyBufferSubData, CommandEncoder.copyToBuffer, Util.isAarch64, GuiGraphics#textHighlight, RenderPipelines#GUI_INVERT, etc.)

---

## Examples and quick conversions (choose which to expand)

- TrackingItemRenderState migration (Item rendering): update imports, replace calls to getModelIdentity, remove uses of clearModelIdentity, adapt pipeline construction.
- BlockEntityType registration (privatization): show registry-backed registration pattern and lookup via HolderLookup.
- JSON->Codec example for a custom recipe or consumable: show codec definitions and registration.

---

## Testing and validation checklist

- Start Minecraft client with your mod and watch console errors/warnings on startup.
- Test resource reloads and verify codecs deserialize your custom types.
- Check model baking and item rendering across contexts (inventory, hand, GUI).
- Validate recipes and recipe book categories load and display.
- Validate block entity creation/removal and world reload behavior.
- Run game tests (if you have them) and update them to the new test environment API.
- Test multiplayer scenarios for permission/selector changes (EntitySelectorParser#allowSelectors) and verify commands behave as expected.

---
