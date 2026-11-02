# RFGRVR maintenance guide for AI and human contributors

This is the current editing/build/verification guide. Start here, not in the historical development notes. Read `README.md`, `LICENSE`, and `AGENTS.md` before making changes.

The original mod implementation was AI-generated under human direction and testing. That is provenance, not proof of correctness or a guarantee of public-domain status. Keep the project noncommercial, publicly accessible and free of memberships, paywalls and exclusivity. Preserve third-party notices. Optional donations must never grant access or special rights.

## 1. Establish what you are editing

The source root contains `src/`, `tests/`, `distribution/`, `vendor/`, `third_party/`, `BUILD-TESTS.ps1` and `BUILD-RELEASE.ps1`. A development checkout may sit at `vr-research/project` inside a game installation; the public source archive puts these files at its own root.

The game installation is NOT the repository. Do not initialize/publish a repository containing the entire game. Do not assume an old `vr-research/output` script, process ID, symbol map or machine-specific path exists in a public checkout. Those are local diagnostic artifacts, not portable dependencies.

Before editing:

1. Read the user's latest objective and preserve earlier accepted constraints.
2. Inspect applicable instructions and the actual current code, not just release notes.
3. Record the release version in `distribution/release.json`; inspect source changes and built-file hashes if relevant.
4. Identify whether game, launcher or companion is running, without closing a user's active game prematurely.
5. Back up settings and the affected source before risky changes. Use version control where available; do not erase another contributor's work.
6. State what is known, measured, inferred, unverified or blocked. Tests passing is not a headset result.

Historical material is archived in `docs/history/`. It includes abandoned multi-pose calibration, size tools and experimental renderer paths. It is useful evidence, NOT authority to reactivate those systems.

## 2. Architecture and ownership

| Area | Main files | Responsibility |
|---|---|---|
| Native 32-bit proxy | `game.cpp`, `proxy.def`, `vr_launch_mode.h` | DirectInput forwarding, explicitly gated VR worker/startup |
| Game input | `gamepad_input.cpp`, `weapon_physics_input.h` | Synthetic XInput, native movement, focus/reload filtering |
| Stereo scene | `native_scene.cpp`, `temporal_stereo.cpp`, `vr_eye_*.h` | Two engine eye renders, resources, camera/projection/state restoration |
| 64-bit XR companion | `bridge.cpp`, `ipc.h`, `vr_mapping.h` | OpenXR session/actions/poses, eye delivery, panels, calibration and runtime pacing |
| Head/body | `native_head_camera.cpp`, `native_arms.cpp` | Local-player camera anchors, tracked wrists, owned meshes and skin palettes |
| Stance/eye math | `vr_crouch_*.h`, `vr_landing_eye.h`, `vr_neutral_pose.h`, `vr_world_scale.h`, `vr_head_motion.h` | Crouch/landing transitions, calibrated references and scale/pivot separation |
| Body visibility | `local_body_geometry.h`, `vr_body_visibility.h` | Private local-player glove-only index filtering |
| Weapons | `native_weapon_physics.h`, `vr_weapon_motion.h`, `vr_two_hand.h` | Physical melee, inherited throw motion, grips and recoil paths |
| Reloads | `native_reload_engine.h`, `vr_manual_reload.h`, `reload_visual.h` | Native ammo ownership plus manual magazine state/visuals |
| Scopes | `weapon_scope.h`, `weapon_scope_gpu.h`, stereo auxiliary path | Native per-rifle HUD/effects, lens feed and focus gating |
| Vehicles/turrets | `native_vehicle_*.h`, `native_turret_rig.h`, `vr_vehicle_eye_anchor.h` | Driver/gunner ownership, mount pivots, saved heights and native physics |
| APC views | `native_apc_cameras.h`, `apc_camera_*.h`, `apc_render_targets.h`, `apc_world_screens.h` | Vehicle-relative camera feeds, world-depth monitor geometry and offline state |
| HUD/menus | `hud_group_render.h`, `wrist_map_render.h`, `vr_map_panel.h`, `vr_menu_*.h`, `main_menu_capture.h` | Native UI layers, complete world panels, per-eye world retention |
| Controller profiles | `vr_controller_profiles.h`, `vr_quest_layout.h`, `vr_index_layout.h`, `vr_weapon_selector.h` | Profile detection, context/gesture/chord state and tap/hold timing |
| Preferences | `vr_user_settings.h`, `bridge_vr_settings.h`, `bridge_desktop_settings.h` | Persistent settings, logical remap and headset/desktop panel |
| Installer/launcher | `distribution/Common.cs`, `Setup.cs`, `Launcher.cs`, `GameAssets.cs` | Supported-game validation, transactions, launch choice, updates, local extraction |
| Shipping profiles | `distribution/defaults/` | Approved eye/grip/vehicle/APC/HUD defaults; not disposable personal files |

The native DLL is x86 because the game is 32-bit. The companion is x64. Managed setup/launcher use .NET Framework. Shared structures must have identical layout across architectures; use fixed-width fields, documented sizes, explicit protocol/version handling and sequence consistency. Never put process-local pointers or C++ containers into shared packets.

## 3. Non-negotiable behavior to preserve

### Standard launch isolation

Only explicit `RFGRVR_LAUNCH_MODE=vr` may start the VR hooks/worker. Standard/unset launch must still forward native DirectInput without companion startup, VR calibration, synthetic input or VR scene changes. Keep the built-proxy smoke check in addition to math tests. Never ship modified game archives/executables as a workaround.

### Local player versus NPCs

A shared rig, a familiar mesh name or an array slot does not establish player ownership. Prior errors treated an NPC body as an owned third mesh slot and removed its head/overrode its animation.

Validate the current local `Player*`, character parent/link, instance handle, rig/model/skin identities and current lifetime. Only then publish a command or hide an owned head/eye mesh. Keep the parent-link/foreign-instance regressions. Never modify shared rig bind data or shared GPU buffers to change just the player.

### Camera, torso, feet and scale

Foot placement must not drive player eye elevation. A foot/leg-only edit must not alter the root, torso/head references, camera scale or saved eye position. Grounded crouch fitting uses native lower-body animation without a second gameplay foot solver; its fitted upper torso is isolated from pelvis/foot corrections. Idle legs use authored bind geometry rather than a captured asymmetric weapon stance. Walking/strafe, crouch, jump and landing require separate validation.

Preserve the known distinction between calibrated player/interaction scale and optical world scale. Do not silently normalize approved world/eye values. Compare final rendered head and eye positions at the actual saved scale, not just identity/100% fixtures. Do not add physical crouch displacement twice, flatten binocular eye separation during head roll, or switch to standing references before landing recovery finishes.

Use the calibrated head rotation pivot to distinguish a head turn from physical room translation. A player looking left/right should not drag their whole body. Keep on-foot settings separate from vehicle/turret eye levels. Do not restore retired arm/body scaling or advanced wrist/elbow calibration because old notes describe it.

### Ragdoll and gloves

Ragdoll view deliberately samples the owned native head/eye pose and locks view orientation to that head while preserving stereo separation. Do not add normal tracked-center translation/yaw on top or apply this to NPCs. Confirm behavior during entry, active ragdoll and recovery, not only a static quaternion test.

Gloves Only retains complete authored glove components/cuffs via private, bounded index buffers. Native player meshes use triangle strips with material boundaries and odd winding starts. Preserve restart markers, winding and material groups. Do not shrink arm bones or change vertices to hide the body; that previously stretched gloves toward the ground. Head/eye hiding is render-only and local-player-only. Glove/cuff wrist rotation must remain intact.

### Weapon/grip ownership

Saved primary/support poses are weapon-local. Acquiring or releasing a support grip must not move the primary model hand on the weapon. Do not let last-shot/reload animation automatically detach a held support hand. Keep sledge sliding bidirectional on the actual shaft, bounded by the approved upper choke point. Avoid reintroducing speculative new grip axes or overwriting saved profiles to compensate for a rig error.

Separate destructive melee from gentle collision. Preserve weapon-specific native damage/impulse profiles, swing thresholds and the two-handed trigger-held sledge area path. Prevent repeated NPC damage from one swing unless behavior explicitly calls for it.

Throw release should combine unboosted inherited rigid-body motion with smoothed hand-relative motion. Do not boost player/platform velocity with the hand throw multiplier or apply native throw velocity again after writing Havok release velocity. Keep orientation-only changes from creating false translation. Preserve early-cancel/third-tone arming semantics.

### Menus, scopes and extra views

Gameplay menus must show complete native UI over live head-responsive stereo world rendering, with the standard perspective. Do not solve them with a frozen flat screenshot, duplicate eye-locked panel, left-eye reuse for the right eye or a brief flat-screen transition on close. The large Map includes its terrain/background layer as well as icons and text. Main-menu/video/loading presentation is a separate floor-anchored screen path.

Scopes must use each rifle's native HUD/effect backend, not an invented generic reticle. Rail Driver special vision must stay inside its scope feed. Track effects/decals, shadows, zoom, per-eye parallax, format/lifetime and actual pixel content. Valid capture calls with zero draws or an empty texture are failures. Normal eye rendering must not inherit auxiliary visual state. Scope cameras remain inactive outside focus or explicit preview.

APC views must stay relative to the vehicle and respect world depth. Do not scan the entire object registry every render to rediscover APCs. Live extra feeds are occupancy-gated; offline geometry and the saved parked presentation have separate paths. Preserve paired side-camera placement and approved monitor layouts. Never use the APC work as a reason to reset driver/turret eye positions.

### Controls and transient commands

Quest and Index profiles are separate from Frame. Stick clicks are not downward axis deflections. Menu confirm/back is distinct from gameplay mappings. Preserve right-grip vehicle-height priority over mission acceptance.

Index right pad: short touch tap emits one quick-switch on release; 0.5-second hold opens selection with no extra release swap. Left-pad Pause/Map requires force, touch and release hysteresis, and locks the chosen half until release. Quest Y resolves interaction versus switch once per short tap using fresh visible native context, not every frame while a prompt fades.

Reserved controller command bits are internal transport only: strip them before forwarding native XInput. Test C4-only gating and menu/inactive resets. Do not revive old Index A+B Map/both-sticks Pause chords or map physical system buttons reserved by the runtime.

## 4. Approved release defaults

`distribution/defaults/` contains exactly the intentionally approved profiles listed in README. These include weapon grips, magazine/scope placement, vehicle eye heights, APC layout, focus/HUD settings and eye/world-scale defaults. Gloves Only is the packaged default.

The release builder hashes them as `seedOnly` records. Setup validates/stages them, creates a file only when absent, preserves existing choices on repair/update, and leaves seeded profiles after uninstall. Keep those transaction tests. Do not treat these files as local junk or remove them from Git. `.gitignore` must explicitly retain them despite excluding local `RFGRVR-*.ini` files.

Never regenerate the baseline from whatever happens to be on a developer's machine. Change an approved default only intentionally, with review of its exact diff. Do not package head-pivot recordings, elbow recordings, tracking history, test unlocks, retired propane editing data or diagnostic logs. Do not wipe other settings while updating one key or adjusting controller labels.

## 5. Build from source

Requirements: Windows; Visual Studio C++ Desktop build tools with x86/x64 compilers and Windows SDK; .NET Framework 4.x compiler; PowerShell. Keep the supplied MinHook library, OpenXR headers/loader and their notices. Use paths/tool discovery from the scripts rather than hard-coding one user's installation.

From the source root:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\BUILD-TESTS.ps1
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\BUILD-RELEASE.ps1
```

`CHECK-BUILD.ps1` builds only the native x86 DLL/map. `BUILD-RELEASE.ps1` builds the x86 DLL, x64 companion, managed launcher/setup, payload manifest, installer ZIP/checksum and editable source ZIP. Building does not install or modify Steam.

`BUILD-RELEASE.ps1 -SkipNative` reuses the existing native DLL/companion. Use it only for packaging/managed/docs changes when those binaries are known to match the source. Any changed companion or native header included by a binary requires rebuilding that binary. Keep logs; inspect actual exit codes, not just the last printed line.

Expected outputs:

- `build-check/dinput8.dll`, `dinput8.map`, `RFGRVR_OpenXR.exe`, `Launcher.exe`.
- `release/RFGRVR-Setup.exe`, `RFGRVR-Setup.zip`, `RFGRVR-Setup.zip.sha256`.
- `release/RFGRVR-Source.zip` with current source, defaults, instructions and licenses.

Use Windows PowerShell/.NET Framework for reflection-based setup tests. Loading the Framework installer assembly into PowerShell 7 can fail on System.Web types; that is a host mismatch, not proof the installer itself fails.

Windows compiler pitfalls: define `NOMINMAX` before Windows headers or use `(std::min)/(std::max)` as appropriate. Do not name a parameter `near`, a Windows macro. Preserve calling conventions, stack layout and x86 pointer widths in native hooks.

## 6. Verification workflow

1. Reproduce the current failure before changing a working subsystem. If live capture is needed, identify fresh process/module/symbol state; never reuse a stale PID or mismatched map.
2. Form a bounded hypothesis with measurable expected results. Prefer a small isolated fix to a broad body/camera/physics rewrite.
3. Add a regression that exercises the actual failure or boundary. Avoid tests that merely duplicate the implementation.
4. Run affected checks and required build scripts. The current suite contains 28 native executables plus desktop/settings/installer/updater checks; use the script's current list rather than assuming the count never changes.
5. For release changes, verify the generated payload, all default profiles, checksum, source archive and install/uninstall paths.
6. Install only after the game releases its loaded files. Ask the user to save/close if necessary; do not kill an unsaved game or repeatedly request permission already granted.
7. Back up the existing install and settings, install transactionally, compare installed hashes and verify unchanged profiles. Use the wizard, not ad hoc replacement of half the mod.
8. Request a specific headset check and inspect fresh counters/images. Distinguish built/tested, installed, and actually confirmed in headset.

Mandatory regression domains for relevant edits:

| Change | Verification |
|---|---|
| Head/body | Standing, moving/fire, physical/button crouch, jump/full landing, actual saved scale, head turn vs translation, NPC rejection |
| Gloves | Full cuffs, native strip winding/material ranges, immutable original buffers, all owned parts and foreign-part rejection |
| Controllers | Short/long press, release, menu transition while held, modifiers, inactive profile, unchanged other controllers |
| Menus | Map/store/locker/pause/death/tutorial where available, both eye backgrounds and complete panel, close transition/aspect |
| Scopes/APC | Real GPU formats/copies, per-eye content/depth, resize/lifetime, scene-state restoration and inactive gating |
| Installer | Clean install, update/repair preserving overrides, wrong game, bad hash/path, rollback, uninstall history/default retention |
| Standard mode | Actual built proxy loads and creates native DirectInput without VR startup |

`tests/test_standard_proxy.cpp` is the built-proxy smoke test and needs a candidate DLL path; the automatic list does not build/run it as a normal header-only test. Compile it with the Windows SDK and run according to its source arguments. Do not substitute source-text assertions for loading the candidate.

There is no portable claim that all hardware/GPU tests run on every CI agent. Report unavailable GPU/runtime checks instead of silently skipping them and declaring complete validation.

## 7. Performance investigation

Measure before optimizing. Compare the same scene, headset refresh, PPD, lighting, weapon, vehicle and gameplay activity. Separate companion/XR frame cadence, fresh engine eye-pair rate, CPU cost, GPU cost, reprojection, copies and allocations. Do not claim 144 new game frames because `xrWaitFrame` ran at 144 Hz.

Preserve headset-paced bounded waits and fail-open stale-clock behavior in `vr_frame_clock.h`. Do not wait while holding eye texture ownership or create circular renderer/companion waits. Keep sequence-protected shared snapshots, COM reference lifetime, keyed-mutex discipline and full D3D state restoration.

Avoid per-draw allocations, full registry scans, repeated kernel memory queries, broad GPU surveys and unbounded diagnostic logs. Scope caches to the correct thread/frame/lifetime; invalidate on device/resource/generation changes. Measure cache benefit before keeping it. The previously tested shader-deduplication experiment was slower and is not a production optimization.

Desktop uses the right eye, not a third camera. Offscreen shadows and required effects still need rendering; "HMD view only" is not permission to cull necessary shadows or always-rendered auxiliary cameras. Scope/APC activity gates are intentional. Never quietly lower visual settings to conceal a regression.

Eye-tracked foveation was withdrawn following crashes. Do not reactivate old driver hooks or an unsupported settings toggle without a separately validated backend and explicit scope. Advertised gaze support is not valid gaze; valid gaze is not proof shading rates were applied or improved frame time.

## 8. Native-engine editing discipline

Addresses, signatures, layouts and allocator calls are tied to `distribution/supported-game.json`. Never update its hash just to bypass an unsupported executable. Verify instructions/calling convention, expected bytes, object ownership and failure fallback before installing a hook. Avoid changing global engine behavior to fix only the local player.

Physics writes must use the appropriate engine queue/simulation path. Do not inject arbitrary impulse/activation calls from a GPU callback. For turrets, distinguish driver weapons, occupied gunner seats, standing pivots and Raider rear-standing offsets. Preserve gun/assembly/seat separation and native physics activation; firing must not be required to wake aiming.

Handle device/resize changes without releasing the last COM reference to a still-referenced texture. Do not retain swap-chain buffers across ResizeBuffers. Deferred and immediate D3D contexts are different paths: verify the one actually used. Never access a context concurrently without the relevant ownership/restoration scheme.

Keep diagnostics opt-in/bounded. Pointer validity is not object identity. A successful native call with no visible or physical result is not task completion.

## 9. Packaging and GitHub releases

Set `repository` in `distribution/release.json` to the real public `OWNER/REPOSITORY`; increment numeric `version` and retain `asset: RFGRVR-Setup.zip`. Empty repository intentionally performs no update download. Do not invent an address or store a token in the config.

Publish a non-prerelease GitHub Release tagged `vX.Y.Z` with:

- `RFGRVR-Setup.zip`.
- `RFGRVR-Setup.zip.sha256` from that exact ZIP.
- `RFGRVR-Source.zip` corresponding to that exact release.

The launcher shows its version, offers a manual update button, and checks after VR selection. It checks the configured HTTPS release origin, package bounds and SHA-256, then starts the same wizard. This is not an independently signed update channel; do not claim the checksum authenticates an untrusted publisher.

`.github/workflows/release.yml` builds/tests and uploads artifacts for review. It does not automatically publish a release. GPU/hardware availability on hosted runners must be confirmed. Keep public downloads and source accessible without memberships, paywalls, invitations or exclusive arrangements.

The source bundle must include LICENSE, README, MAINTAINING, AGENTS, controller guide, current tests/scripts and approved defaults. Installer/public payloads must include the user guide, custom license and upstream notices. Source archive selection must never sweep in the enclosing game installation, assets, local caches, saves or diagnostics.

Setup validates the complete payload before writing, rejects escaping paths/reparse points, backs up replacement history, and retains seeded settings on uninstall. Steam integration modifies only this game's launch choice and records/restores the previous value while respecting unrelated wrappers. Do not broaden registry/Steam edits.

## 10. Shipping checklist and handoff

Before delivery verify version labels, exact ZIP checksum, wizard contents, public source/license/defaults, seed-on-clean-install behavior, unchanged existing profiles, installed file hashes if installed, Standard isolation and relevant live results. Do not automatically install a docs-only update or restart a game just to refresh a public package.

A useful final handoff states what changed, which artifact/folder was updated, what was tested, whether it was installed, and what remains unverified. Keep current documentation aligned with code. Update this guide and controller tables whenever defaults, profile mappings, calibration stages or build/release steps change.
