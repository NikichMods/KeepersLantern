# Keeper's Lantern — Working Rules

These rules are mandatory for development in this public repository.

## Global engineering contract

Before substantive changes, consult the current global rules in `NikichMods/DevRules`: `ENGINEERING_RULES.md`, `CI_POLICY.md`, `GIT_WORKFLOW.md`, and `PROJECT_BOOTSTRAP.md`. Repository evidence outranks chat memory.

Use the standard flow: **discover -> verify -> implement narrowly -> test -> accept**. Keep research separate from production and preserve exact source/build identity. Documentation/bookkeeping changes do not need hosted CI; research-only code may use CI when compilation, tests, or a runnable artifact materially advance the investigation.

## Project identity

- Public mod name: **Keeper's Lantern**
- Repository / project / assembly: `KeepersLantern`
- Game: `Graveyard Keeper 1.407`
- Stable primary BepInEx GUID: `nikich.gyk.keeperslantern`
- Current accepted stable runtime baseline: **1.0.12**
- `main` is the stable accepted public line.

The production assembly currently compiles exactly:

- `src/UnifiedLightingV021.cs`
- `src/BeltLanternBackPocV034.cs`
- `src/DungeonPracticalLightBoost.cs`
- `src/LanternShadowBoostOptimized.cs`
- `src/SaveNowEnvironmentCompatibility.cs`

Do not silently add old POC/runtime files to `KeepersLantern.csproj`.

## Accepted 1.0.12 contract

Treat `docs/BASELINE_1.0.12.md`, `docs/TEST_BUILD_LOG.md`, the production source compiled by `KeepersLantern.csproj`, and the frozen accepted baseline ref as authoritative.

Accepted behavior includes:

- outdoor-night ambient scale `0.70`, cool tint `0.16`;
- procedural-dungeon ambient scale `0.60`, cool tint `0.07`;
- dungeon stationary practical-light radius boost `x1.25`;
- Keeper Point target `120 / 1.50 / K1.25 / offset 0,-0.40`;
- Keeper Ground target `455 / 1.65 / K0.75`;
- normal interiors use vanilla world lighting and hard-disable the enhanced Keeper light;
- `mortuary` uses vanilla-lighting passthrough;
- Save Now direct-interior restores refresh the already-selected vanilla environment preset only after `SaveNow.Plugin.RestoreLocation()` has completed;
- the Save Now compatibility path is idle when Save Now is absent, ignores outdoor/dungeon results, and fails closed if required runtime members cannot be resolved;
- belt visual remains rear-belt mounted and animation-frame synchronized;
- shadows use the live `DynamicLights.shadows` registry;
- no custom GL/full-screen edge-darkening renderer;
- normal Keeper-light control uses native `DynamicLights` cached coefficients instead of per-frame direct intensity fighting;
- Keeper native intensity baseline is captured only from settled non-interior native output and normalized by `TimeOfDay.light_intensity_k`;
- a valid normalized baseline survives later interior/rebind transitions;
- Darker Nights, when detected, suppresses this mod's outdoor ambient-darkening pass to avoid stacking.

Do not change accepted lighting balance, lantern timing, overlap compensation, belt behavior, dungeon practical-light strength, shadow behavior, interior policy, or the accepted Save Now restore seam while implementing an unrelated fix.

## Public / private boundary

This repository is public production code. It may contain our source, documentation, workflows, and other redistributable project material.

Do **not** commit Graveyard Keeper assemblies, extracted copyrighted game assets, decompiled game source, bulk runtime dumps, research archives, temporary probes, or private reverse-engineering material here.

Reusable **derived** deep-game findings belong in the shared `NikichMods/GraveyardKeeperResearch` evidence layer; proprietary payloads, copied assemblies, full decompilation material, or other non-redistributable research data must remain outside public repositories. Production must depend on distilled verified facts, public packages, and runtime APIs—not on downloading private research data during CI.

## Runtime architecture and performance

The mod should remain effectively free relative to the game during normal play.

Preserve the accepted architecture:

- reuse native `DynamicLights` data and cached objects;
- avoid unconditional global scans in steady state;
- avoid repeated hierarchy searches, reflection, LINQ, allocations, and logging in hot paths;
- do not write Keeper `Light.intensity` every frame during normal operation;
- low-frequency maintenance is acceptable only when verified vanilla behavior can overwrite required state;
- keep shadow work narrow and local;
- keep Save Now compatibility event-driven rather than polling plugin/environment state in steady state;
- restore modified Unity/native state on release, teardown, or destruction.

The rejected custom GL edge-darkening path must not return without new evidence and explicit approval.

## Save/load and transition invariants

Lighting code must correctly handle:

- fresh/reloaded gameplay during daytime followed by night;
- first load inside a normal interior;
- Save Now direct-load into a normal interior;
- interior -> outdoors transitions, including at night;
- outdoors <-> interior transitions;
- dungeon entry/exit;
- player/light-rig recreation or rebind;
- main-menu return and subsequent reload.

Never capture an interior-attenuated or raw/unsettled light value as the full Keeper baseline. The accepted normalized-baseline design remains a release invariant unless a separately tested architecture replaces it.

Known history: 1.0.10 and 1.0.11 attempted the Save Now interior refresh before its late location restoration and were rejected. Do not reintroduce an elapsed-time-from-player-spawn approximation in place of the accepted post-`RestoreLocation()` seam without new evidence.

## Workflow

- `main` = accepted stable public state.
- Runtime changes start on `dev/X.Y.Z` or an explicitly named research branch.
- Do not promote runtime changes without explicit player acceptance such as `фиксируем`, `релизим`, `сливай`, or `можно в main`.
- For a tested numbered Keeper's Lantern build, an unqualified approval to promote it to `main` means the version is accepted as the new stable release. Complete the whole promotion without asking for a second release confirmation: merge the accepted runtime to `main`, preserve the accepted `baseline/X.Y.Z-accepted` ref, and publish the exact tested DLL as GitHub Release `vX.Y.Z`.
- Treat merge and release publication as separate technical operations but one acceptance outcome. Only leave an accepted `main` version unpublished when the user explicitly says merge-only, asks to defer publication, or publication is externally blocked; in the blocked case, record publication as unfinished work rather than as an intentional stable state.
- Every handed numbered DLL is immutable and tied to exact committed source.
- `candidate/X.Y.Z` is the immutable handed/test source state; acceptance does not retroactively turn rejected candidates into stable baselines.
- Important accepted checkpoints receive a frozen `baseline/X.Y.Z-accepted` branch/ref.
- `docs/TEST_BUILD_LOG.md` is the durable handoff/acceptance/release record.

Documentation-only changes do not require a runtime rebuild and should not trigger hosted CI. Updating the accepted-release manifest to publish an already tested, hash-verified binary is an intentional distribution action and may trigger the dedicated publication workflow.

## Build / handoff

Before handing a new runtime DLL to the user:

- version metadata is consistent;
- Release build succeeds from canonical committed source;
- no unintended diagnostics or temporary probes ship;
- exact source SHA and artifact provenance are recorded;
- the user receives a raw versioned DLL when a file handoff is needed;
- the required in-game test is concise and specific.

Standard public CI is permitted when it proves a concrete build/release property. Use the established Windows build while it remains the best fit for the verified toolchain; switch runners only for a concrete engineering benefit after equivalence is proven, not to conserve standard public runner minutes. Keep artifact retention short.

## Long-lived sources of truth

- `AGENTS.md`
- `docs/BASELINE_1.0.12.md` or a newer accepted baseline
- `docs/TEST_BUILD_LOG.md`
- `docs/MIGRATION_PROVENANCE.md`
- `README.md`
- `KeepersLantern.csproj`
- frozen accepted baseline refs and CI evidence

When chat history conflicts with accepted repository evidence, investigate before changing code.

## Shared Graveyard Keeper research

Cross-project Graveyard Keeper 1.407 host/runtime research is centralized in `NikichMods/GraveyardKeeperResearch`.

Before starting a fresh investigation into vanilla/game-engine/UI/NGUI/data/lifecycle behavior:

1. read this repository's own canonical verified-data / architecture docs first;
2. consult `NikichMods/GraveyardKeeperResearch/docs/RESEARCH_INDEX.md` and the linked shared knowledge documents;
3. search accepted local/shared test evidence and relevant history if the result has not yet been promoted;
4. perform new static/runtime research or a probe only if the question remains open.

Project-specific mechanics, product/UX decisions, release state, and build acceptance remain canonical in this repository. Reusable host/runtime facts that can serve multiple Graveyard Keeper mods should be promoted back into the shared research repository after acceptance rather than left only in chat, commit history, or a test log.

