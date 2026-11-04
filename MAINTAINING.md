# RFGRVR maintenance guide for AI and human contributors

This is the current editing/build/verification guide. Start here, not in the historical development notes. Read `README.md`, `LICENSE`, and `AGENTS.md` before making changes.

The 0.10.82 menu candidate rasterizes native main-menu UI at 6144 � 3456, returning to 3840 � 2160 for gameplay popups. Surface replacement is transactional; failed allocations retain the previous surface and retry after five seconds. The 0.10.84 candidate renders backdrop and native menu together into a single opaque projection layer. Never submit an opaque video quad as an underlay behind projection on SteamVR: it composites quads above the scene. If either eye draw fails, fall back to the established two-quad path. Gameplay/popup projection remains unchanged. Main-menu sharpness is headset-confirmed in 0.10.84. The 0.10.85 candidate applies the same eye renderer to existing flat loading/front-end fallback screens, preserving the 4K source and fixed world plane. Never overlay the old full-screen quad after a successful projection draw. Calibration, gameplay, pause/store/locker ownership is unchanged; loading-screen clarity remains unverified. The 0.10.86 candidate corrects the native main-menu linear-UNORM to sRGB conversion: do not byte-copy UNORM captures into sRGB surfaces merely because the formats share storage. Equal-sized surfaces use the plain copy shader and sRGB RTV encoding; size-changing surfaces retain cubic reconstruction.

The 0.10.87 popup recovery reuses the existing main-menu UI swapchain/surface for gameplay popups, captures their native UI at 6144 x 3456, and projects the foreground over the native stereo world. Do not allocate an independent popup swapchain set, discard cached gameplay buffers when opening a popup, or let the transient pause debounce select the loading-buffer resize. Draw/copy work is gated by menu visibility. Popup diagnostics record capture, world and both eye draws. Popup visibility, sharpness and world-space behavior are headset-confirmed in 0.10.87.

The 0.10.88 sky candidate borrows the signature-verified scene camera accessor for the complete stereo/auxiliary draw, not only the material refresh. Nested refreshes restore the outer eye; the outer scope restores engine ownership. Never mutate celestial world directions to compensate for head rotation. Moon appearance remains unverified. The current Steam Frame connection reports IsMotionSmoothingSupported=false and IsMotionSmoothingEnabled=false; no custom frame-generation backend is installed or advertised.

The 0.10.88 camera accessor change did NOT fix the headset-reported orbital texture spin. The 0.10.90 candidate targets the verified orbital quad builder (RVA 0x1313f0), which takes camera up/right directly. It supplies a private 0x38-byte camera image with world-meridian axes derived from the existing orbital direction. No scene-camera or orbital-record writes; Standard mode passes through. Appearance remains unverified.

Current finalized DLC behavior: see `DLC-TESTING.md`. The old filename is
retained for bundle compatibility. The 0.10.81 candidate disables setup keys,
native HUD visibility borrowing and all automatic arsenal grants. Earlier
testing descriptions below are history, not active behavior. IPC protocol 12
retains DLC campaign detection and independent placements. Never enable global
developer tools or restore grants during ordinary maintenance.

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

### Blocking-notification input ownership
Tutorial pauses may keep the native game-state stack at GAMEPLAY while the pause counter is positive. Route controller navigation before gameplay locomotion/weapon filtering. Preserve native START unchanged: the salvage and PAUSED prompts request START, not A/Confirm. Mask held modal buttons until released on return, and reset desktop navigation ownership after keyboard dismissal. Regression coverage is in test_vr_gamepad_input.cpp.

Magazine attachment caching must not freeze a simulation fallback or the transient zero-offset EDF-pistol clip tag seen during save loading. Only capture an authored, initialized mount; retain that stable seat during firing and invalidate by parent/child object handles. Save-comparison measurements: bad clip offset zero; working clip approximately (0, .048, .153) weapon-local. Do not hard-code that measured offset as a universal grip.

### Magazine initialization and handgun laser (0.10.57)

The seated-magazine cache rejects fallback, invalid and zero-offset initialization samples for every slot-1 magazine, not only the EDF pistol. Native parent/child pointers and instance handles remain the cache key. The later valid authored tag seeds the mount; firing animation cannot move that stored receiver-local pose. Regression coverage repeats the save-load sequence for multiple weapon names. This is shared-path coverage, not proof that every weapon has been headset-tested.

`weapon_laser.h` holds a bounded, lock-protected simulation result. Only the locally held EDF pistol publishes a valid muzzle, derived from the same muzzle-to-root binding used for shot direction. `updateLaser` performs one 200-metre native query per eligible simulation tick, filters the player/held weapon/owned attachments, refines human hits against the skeleton and frees native hit storage. No queries execute in GPU callbacks. A short 12-cm beam and a 2-mm to 25-mm radius red spot render into each eye against selected world depth via a private deferred context. The laser expires after 150 ms without valid simulation data. `test_weapon_laser` verifies real-GPU output, ordinary/reversed depth occlusion, stale-state rejection and output release; `test_native_reload` covers query ownership gating, hit distance, storage release and radius limits. Headset alignment still requires live verification. No companion protocol or shipping placement profile changes are needed.

### Gun-local laser placement and visibility (0.10.58)

The pistol emitter uses `pistolLaserPose` for both simulation queries and current rendered-weapon attachment. Arrow keys adjust only the EDF pistol emitter: Right/Left forward/back, Up/Down vertical in 2-mm increments with bounded repeat. `RFGRVR-handgun-laser.ini` stores the local placement; writes debounce after release. Input requires tracked, foreground gameplay and does not consume the keys while notification placement is active. Updates seed missing laser settings only.

The render path now obtains the latest owned pistol pose, applies the same per-eye local-player visual shift to the short beam as the gun, and re-intersects the latest ray with the last valid collision plane to avoid simulation-pose lag. World impact dots never receive the avatar-only shift. Missing collision normals use an opposing-ray normal rather than discarding an otherwise valid hit. The spot gets a brighter core, a pixel-size floor within the existing physical size limit, and a small bounded surface/eye bias to reduce z-fighting. It remains depth-tested. Do not replace this with a through-wall HUD dot or render-thread physics queries. Alignment and reliability require live headset verification.

### Laser presentation stabilization (0.10.59)

The user-approved emitter position is now the seeded default: forward 0.266 m, vertical 0.060 m. Existing placement files remain untouched by updates. Laser queries and render reprojection share a 50-metre maximum; the dot fades from 40 to 50 metres. Beam width and physical dot divergence increase by 50%, with a 37.5-mm maximum radius. The shader uses derivative-aware edge coverage.

Eye selection freezes the laser collision state for the entire stereo pair. A simulation-only stabilizer bridges at most 35 ms from the original hit for a near-stationary ray (under 5 mm origin movement, dot product above 0.9999), resets on weapon changes or invalid tracking, and never extends its retention window through repeated misses. Depth testing remains active. Tests cover retention expiry, weapon changes, range clipping, fade and normal/reversed depth. Flicker improvement still requires headset verification.

### Adjustable emissive laser and surface bias (0.10.60)

With the handgun equipped in active foreground gameplay, Numpad * / - multiplies/divides the laser spot size by 1.1 (0.25-4 times). Numpad 8 / 9 raises/lowers emissive intensity by 0.25 (0.1-12). `Appearance=size emission` in `RFGRVR-handgun-laser.ini` persists both values; absent fields use size 1 and intensity 2 without changing the saved emitter position. The dot is an unlit additive emission pass, not a scene light. Keyboard adjustments require Num Lock.

A 60-second live sample of v0.10.59 had active tracking and selected world depth for all 600 samples; 577 contained collision hits. This does not prove a single cause for perceived flicker. The renderer now uses a small clamped, sign-correct slope/constant depth bias for conventional and reversed depth to improve surface coincidence. The query/range logic and existing surface bias remain bounded; nearer geometry still occludes the dot. Hardware regressions verify size/brightness actually change rendered pixels and that both depth directions still occlude. Headset flicker improvement remains unverified.

### Laser range/beam and shop audit (0.10.61)

Laser maximum range is reduced by one quarter from 50 to 37.5 metres. The same normalized final-20% fade now spans 30-37.5 metres. The visible muzzle beam extends from 12 to 18 cm from the unchanged emitter origin, with a smooth fade at both ends. The current saved mount (forward 0.276 m, vertical 0.060 m) is the seeded placement; existing local position, size and emission values are preserved.

The shop audit found no active production calls to the retired test unlock helper. The observed save did not match its uniform-max record pattern (1 of 51 inspected purchase records fully maxed). This is not proof that every native shop listing is correct, nor does it undo previous intentional changes to a save. The dormant `native_gear_unlock.h`, its production include and its obsolete grant tests are removed entirely. A replacement regression runs the actual VR player update and checks that purchase metadata and native arsenal records remain unchanged. Do not introduce unlock hooks, edit shop tables or relock earned/save-stored progression as a VR fix. The installer/launcher contain no grant path.

### HUD/menu recovery (0.10.63)

Reverts the 0.10.62 raid/timer HUD capture expansion and restores the exact 0.10.61 capture chain and companion layer handling after a live menu transport regression. The mission control and timer additions are withdrawn pending independent integration. Bloom/exposure and all approved placement defaults remain unchanged. Do not restore the withdrawn additions from the 0.10.62 package without reproducing menu publication and consumer acknowledgement during live transitions.


### Shared mission-control panel (0.10.64)

The existing RFGRVRMissionHud capture accepts its original mission-offer hash and native raid_main (0x6b70456e). No HUD hook instances, marker ranges, mappings, OpenXR swapchains or layers are added; wrist_map_render.h and wrist_map_protocol.h remain exactly at the 0.10.63 recovery baseline. Mission-only frame gating captures both native groups together at the existing bounded rate, with independent crop bounds and metadata; one canvas clears once per accepted native frame. The native control panel occupies the upper 2.5:1 slot; the existing offer occupies the lower 4:1 slot, retaining its original world position/size. Native visibility, score text and animations are retained. Menu capture and rendering branches are unchanged. GPU regression covers simultaneous native groups, ended-offer clearing and two independent menu publish/consume handoffs after HUD activity. Live menu transition and scoreboard visibility remain required. Timers are not added by this candidate.


### Decorative corona occlusion (0.10.65)

The live spotlight-through-wall capture (PixelProbe-39012) identified native rfg+12B222 corona draws using 176-byte vertices, six indices and a bound world DSV; native 12A9F0 explicitly disables depth. vr_corona_depth.h scopes only native corona pass 0 during stereo views. The existing indexed-draw hook borrows a cached read-only depth state only when the bound DSV exactly matches the geometry-selected scene depth, with its verified conventional/reversed comparison. It restores native depth/stencil state immediately after the draw. Query passes 1/2, ordinary rendering, other effects, templates, light illumination and native visibility/alpha are unchanged. No render-thread physics queries or scene searches are added. Hardware tests verify wall occlusion and visible glow with read-only depth, both depth directions, no-world-depth fallback and state restoration. Final native-shader/headset behavior still needs verification.


### Corona activation/depth-source correction (0.10.66)

The 0.10.65 live counters were zero: its stereo-marker gate never activated the draw guard, and ordinary gameplay does not collect APC/scope/laser scene-depth candidates. Native VR hook installation now explicitly enables corona pass-0 correction independently of that transient marker; Standard never enables it. A bounded 32-view ledger records native depth clears across immediate/deferred contexts, retains each exact DSV with conventional/reversed comparison inferred only from endpoint depth clears, and expires entries after two seconds. The draw guard uses the currently bound DSV only if that exact view has a valid recorded clear; it requires neither APC/scopes/laser nor full-scene depth scoring. Query passes and noncorona effects stay native. Hardware tests now exercise immediate-clear to deferred-glow handoff, missing/stale/ambiguous clear rejection and stencil-only clear preservation in addition to normal/reversed occlusion and state restoration. Verify queued/applied/missingWorldDepth counters live after installation; previous release's headset occlusion failed.


### Actual deferred hook integration (0.10.67)

The 0.10.66 live run recorded 86,946 native glow scopes but zero applied/missing-depth attempts. PixelProbe source context was deferred; its indexed draws use RFGRVRDeferredProbe::indexed, not gpu_pipeline_trace.cpp's immediate HIndexed. The same depth guard and clear ledger are now wired into the actual deferred indexed-draw and depth-clear wrappers, keeping per-context originals intact. The real-GPU regression attaches the production deferred shadow table, calls DrawIndexed through that table while the corona scope is active, asserts the applied counter increments, executes the resulting command list, and verifies wall occlusion. Previous helper-only GPU tests did not prove this integration. Live native shader appearance remains to be verified; no menu, HUD, light-template or illumination changes.


Live confirmation: user verified 0.10.67 hides the portable spotlight glow behind solid walls. Preserve the deferred indexed-draw/depth-clear integration when changing graphics instrumentation.



Vehicle exposure correction (0.10.68): first-person driver, gunner and passenger views use a 0.5 minimum luminance during native HDR metering. The user confirmed the stronger temporary comparison reduced over-bright exterior reflections; 0.15 was insufficient. The adaptation call receives a private HDR parameter copy, with only its luminance floor changed. On-foot, third-person, cinematic and Standard rendering retain native parameters. Spotlight occlusion, lighting sources, bloom composition and saved placements are unchanged. Build validation does not establish the permanent candidate's headset result.


0.10.68 upload preparation: laser keyboard editing is disabled at its input entry point, retaining settings loading and rendering. Grip/scope/APC/body setup shortcuts were already disabled in shipping input paths. The approved laser appearance (size 4, emission 4.1) is seeded for clean installs; existing profiles are not overwritten. Normal menu keyboard/controller controls and VR Settings remain available. Native vehicle exposure still requires permanent-build headset verification.


0.10.69 mission timer: the existing mission capture matches native hud_timer_group (0x183d2fe6) and the separately owned convoy_timer text (0xf166e650) into the previously empty top 128 pixels of its 512-pixel canvas. It preserves native timer visibility/text/alpha. The companion crops that strip into an 80-pixel header above the wrist map in its existing atlas/layer. Map, health and controls retain their prior world positions. The chest mission panel excludes the timer strip. No capture hook, mapping, epoch range, protocol field or XR layer is added. Regressions must cover all three simultaneous mission groups, timer ending, cropped wrist drawing and menu texture handoff. Native timer appearance remains unverified until live testing.


0.10.69 HUD layout editor: vr_hud_layout.h is companion-only. Numpad 0 toggles 12 labeled previews, + switches 9-key pages, decimal toggles right-controller translation and saves on placement, */- scales; edits persist separately in RFGRVR-hud-layout.ini. Native HUD visibility/state is never forced. Preview uploads occur only on editor revision changes and are invalidated on exit. Gameplay buttons/grips are suppressed while editing; tracking continues. Individual offsets remain local to their native anchor; map parts project hand motion into the map plane and clamp their rendered rectangle inside the atlas. The map assembly scales as a whole. Mission-control pixels are sampled from the existing mission texture into the same wrist atlas as the timer; only mission offers remain in its chest quad. Wrist atlas is 768x1280, retaining map pixel density and existing world positions of map/health/controls. This adds no native hooks, channels or OpenXR layers. It does allocate a larger atlas, so do not claim measured memory savings. HUD layout has no effect in Standard mode.


0.10.70 fixes companion raster-state leakage introduced by HUD atlas scissoring. WristAtlas::draw restores the caller's rasterizer and exact scissor list; StereoScaleBlit explicitly binds its own cull-none, scissor-disabled fullscreen state through RFGRFullscreenRaster. Previously the next eye copy inherited a small HUD scissor, leaving stale regions in the projection swapchain. Regression: draw a cropped HUD first, verify state restoration, deliberately leave a tiny scissor active and use the production fullscreen raster binding before drawing a complete eye target; check corners and center. HUD layout, timer, placements and native capture remain unchanged. Headset confirmation is still required.


0.10.71 removes synthetic HUD editor label images. A new end-of-Bus hudPreviewTick heartbeat requests synchronous native group rendering for at most 500 ms after the last companion update, only for the matching game PID. RFGRHudVisibility temporarily borrows the verified group-visible/alpha and copied draw-state alpha fields, restoring originals through RAII. Menus, scope UI and weapon selector are excluded. The companion always consumes real native textures while editing and retains the last captured frame when a transient element stops publishing. No GDI preview textures overwrite the HUD or map. Leaving editing resets display freshness, returning to native visibility. Empty native content is not replaced with invented values. No UI source assets are distributed. The confirmed fullscreen scissor isolation remains unchanged. Native editor coverage still needs live verification.


0.10.72 timer correction: live timed-mission inspection confirmed convoy_timer (0xf166e650) is a bitmap stopwatch; convoy_strength_number (0x36d291c1) is the countdown text. Both share container 0x2ac6d903. Capture that verified container, not the icon alone, gated on both owned children being visible. Its numeric child is also used for distance/strength in other missions, so do not capture it unconditionally. The 16-child bounded check validates parent/document ownership. General hud_timer_group remains supported. The shared traversal retains icon and text in one bounds/canvas slot. Tests cover the true hierarchy, icon-only rejection, non-timer visibility, foreign document rejection and two separate GPU batches retained together. Existing HUD layout and fullscreen scissor fixes are unchanged. The permanent candidate needs headset verification.


0.10.73: separate companion vehicle HUD profile and selection mapping through vrhudlayout::Profiles. Existing global layout remains the on-foot file. On first use, the vehicle profile inherits existing item settings; vehicle edits then save separately. Vehicle-state snapshot is selected once per XR frame and shared with map layout; entering/leaving a seat ends/saves an edit and clears stale HUD freshness. Vehicle edit previews exclude handheld ammo/charge/use elements. Wrist atlas now uses premultiplied-alpha composition and restores blend state as well as raster/scissor state. Draw map first, health last. Map rectangles uniformly fit inside atlas bounds instead of independently truncating width/height, preserving bar proportions. GPU tests check transparent health margins retain the map; profile tests check separate selection, persistence, context transitions and proportional sizing. Native timers and the full-eye clipping fix remain unchanged. Live headset results are unverified.


0.10.74 public release preparation: the author approved the final vehicle HUD and requested saving placements, removing tools and packaging for GitHub. Copy only approved RFGRVR-hud-layout.ini, RFGRVR-vehicle-hud-layout.ini and the updated vehicle-heights.ini into seed-only distribution/defaults; preserve existing approved weapon/APC/eye/laser defaults. Retain Gloves Only for fresh installs despite the developer session using Full Body. Personal head-motion/arm recordings and retired propane/menu-character profiles are not shipping defaults. distribution/defaults-manifest.json records hashes.

Public builds do not define RFGRVR_DEVELOPER_TOOLS. Layout::input is inert, native HUD visibility borrowing is excluded, camera-lab drawing/hotkeys are excluded and the backtick capture hotkey is excluded. The direct layout math/key helpers remain in editable source and tests. To do deliberate future placement work, define RFGRVR_DEVELOPER_TOOLS consistently for the native DLL and companion, rebuild, and explicitly remove it again before release. Normal VR Settings and desktop keyboard navigation are preserved. Gameplay at 0.10.73 was headset-approved; the tool-stripped 0.10.74 is build/test verified pending any later live check. No GitHub publication is performed by packaging.

0.10.76 menu clarity: the native MainUI capture now renders directly at 3840x2160, matching its existing OpenXR panel, rather than upscaling a 1024-square capture. The video backdrop captures at its existing 1920x1080 output size. Independent canvas width/height are used for render targets, published texture dimensions, viewport and horizontal scissor mapping; vertical clipping follows viewport height. HUD and weapon-scope canvas sizes, scene PPD, menu pose/size and DLC testing tools are unchanged. GPU regressions check actual capture/channel dimensions, alpha, menu handoffs and state restoration. Headset sharpness remains unverified until installation.

0.10.77: live DLC list reached its native 64-record capacity because the broad testing grant had admitted generic-inventory turret/debug weapons. Validate native array capacity and writable storage, then compact only verified accidental grants on the local DLC simulation update, preserving unknown/retained records. Grant only explicit handheld definitions through signature-checked native APIs; do not alter shop availability or campaign progression. Readable terminated native weapon and inventory names are required. Tests reproduce a saturated list and preserve record order/flags. The Impaler melee variant uses royal_sword, not spiker. Numpad 2 health placement is now available on foot in both campaigns, with separate campaign/DLC/vehicle profiles. Native turret_stomper_mg is added to the explicit standing-platform rule, preserving its rear offset under rotation; other seated turrets remain centered. Headset and repaired-locker results require live verification.

0.10.78 menu transfer: live MainMenuUI capture and shared output confirmed 3840x2160; SteamVR exposes no FB layer-quality extension. Native video resolution is unrelated to the UI complaint. The format-converting GpuPreview transfer previously inherited blend/depth/raster state. RFGRMenuCopyState now disables alpha blending and scissor/culling for a source copy and restores the caller's blend factors/mask, depth/stencil and raster state on every exit. This avoids applying premultiplied UI alpha twice before XR composition. test_menu_copy reproduces inherited alpha blending plus a one-pixel scissor on hardware and checks RGB/alpha within one 8-bit quantization level across every output pixel and state restoration. Direct same-format texture copies remain unchanged. No sharpening/edge-enhancement filter, geometry change, resolution reduction or gameplay change is introduced. UI sharpness still requires an installed headset comparison; a source capture alone is not proof of headset appearance.


0.10.79 Missilepod manual reload: live missilepod had zero loaded, 27 reserve and no magazine child. Reload::eligible rejected it because rocket loading only recognized RPG/Thermobaric names. Explicitly recognize missilepod and spawn rpg_projectile, its verified native projectile definition, through the existing safe item-prop path. Each insertion adds one rocket and subtracts one reserve round, stays ready without a reload-button press, and respects its native nine-round capacity. Tests reproduce childless acquisition and insertion plus full-capacity rejection, retaining other reloads and native button-reload mode. No further main-menu clarity change is claimed: live UI source and shared panel are 3840x2160; SteamVR overlay-quality Ultra comparison had no user-visible improvement and was restored to Auto. Menu font-source versus downstream presentation remains unresolved; do not alter the working pause/store/locker path speculatively.

0.10.80 candidate: Missilepod reload payloads are stacks of up to its native nine rockets. Fetch quantity follows empty capacity and available reserve; insertion transfers the represented count with native capacity/reserve bounds, immediately ready without Reload. Additional render-only props extend down the magazine-local Y axis; slot count, scale and captured hand orientation share one frame for simulation and queued visuals. Num 1 previews full capacity; Page Up/Down adjusts and saves bounded 0.5-cm spacing steps. Num 3 moves a rocket-loading weapon's insertion ghost using the off hand, release/press grip confirms full weapon-local pose; cancellation restores original pose. All stack children are erased on mode/weapon changes, seating completion or cancellation; retries are bounded and reset on starting preview. Existing RPG/Thermobaric behavior stays single-round. Controls remain DLC-only except main/DLC on-foot health editing. Tests cover nine-item creation, spacing persistence and transforms, full-capacity reload accounting, setup without ammo changes, ghost pose capture/cancel and cleanup. Main-menu UI is now rendered as a world plane into both application projection eyes, preserving its existing floor anchor and 4K source. Distinct eye poses/FOVs produce binocular parallax; anisotropic sampling, premultiplied alpha, private deferred state and no retained XR target resources. If a native menu eye pair is missing, generate transparent UI projection eyes directly from current XR views. Suppress the former UI quad only when both eye draws succeed; otherwise keep it as fallback. Pause/store/locker path and videos are unchanged. RFGRVR-Menu-Projection.log records actual per-eye activation. GPU test checks binocular geometry, world anchor, transparency, target-slice independence and caller-state restoration. Build/headset results must be distinguished.

0.10.81: author approved DLC setup, requested removing tools/grants and a main-menu upscale. Disable WeaponSetupInput and its worker call, companion health editing and native HUD visibility borrowing; remove native_dlc_testing inclusion and DlcTesting::update call entirely. Existing saves are not relocked. Seed only the explicitly approved current DLC weapon/HUD profiles, reload positions/spacing and vehicle heights; preserve all other approved defaults and exclude personal tracking files. defaults-manifest.json records 16 profile hashes.

MainMenuLayers source remains a genuine 3840x2160 native UI capture. Only its presentation uses the new cubic/edge-bounded UpscalePS into a 5120x2880 texture (respect runtime limits; retry 3840x2160 if a larger swapchain is rejected). Transparent coverage is retained, RGB is bounded by alpha, and solid artwork is unchanged. Shader runs only for main-menu UI, including its quad fallback, never videos, HUD, scopes or the working pause/store/locker copies. Main-menu app projection requests 25 PPD and restores the configured gameplay target on exit. Live 0.10.80 log showed fallback, so expanded diagnostics now distinguish readiness, eye validity, shouldRender and per-eye GPU failure stages rather than claiming activation from a source capture. test_menu_upscale uses actual 5K hardware rendering, checks alpha/solid regions across the image and reports GPU duration; measured 0.231 ms for its test pattern, not an in-game performance guarantee. Headset clarity is still unverified.

0.10.90 moon billboard orientation is headset-confirmed. 0.10.91 is a bounded, opt-in NPC diagnostic candidate: existing 20-pair human visibility trace additionally hashes the actual draw palette, render root and owned attachment transforms at pre/left/right phases. Circular attachment walks stop at 16 and reject foreign parents. No animation calls, pose writes or normal-frame scanning are introduced. This is not a claimed NPC twitch/slope fix; use the trace to establish whether multi-eye rendering changes poses before adding a correction.

0.10.92 removes the vehicle exposure-floor override: onAdaptation always receives its native HDR pointer, with shared once-per-frame history retained. First-person vehicles instead cap only rl_hdr_state::m_bloom_amount (+0x48) at 0.5 in a private 0x68-byte argument at the verified native HDR constant uploader RVA1259E0. Its RVA125B17 reads HDR+0x48 and writes the shader slot selected by compositor+0xB58. Verify the entry, field transfer and ret12 before hooking. RSL layout and native HDR shaders identify bloom strength separately from scene/exposure. Do not change luminance bounds, adaptation, scene colour, stored state or on-foot/Standard rendering. Tests exercise the actual upload/adaptation ABIs, both eyes, byte-for-byte retention of every other HDR field, eligibility and invalid inputs. Headset bloom balance remains unverified.

The existing final-composite hook also scopes the same native bloom coefficient selected by compositor+0xB58 in renderer+0x2C0 (bounded 84-float constant region). This covers right-eye cache restoration when adaptation/upload is skipped. Restore the coefficient and mark native constants dirty afterward. Regression invokes both full composite callbacks with restored pre-eye constants and verifies every non-bloom renderer byte remains unchanged apart from the native dirty bit.

0.10.93: user confirmed direction-dependent blue lighting disappeared and normal brightness returned during a live comparison disabling both the manual per-eye sky refresh and canonical scene-camera override. Remove those two render-time workarounds, including the call to global sky/environment setup RVA66BC90. Native scheduled sky/lighting updates remain authoritative. Keep per-eye renderer camera preparation and all projection matrices, the confirmed world-oriented orbital quad fix, vehicle bloom-only limit, sun-glare suppression and menus. The older vr_scene_eye_camera helper/test remain historical standalone coverage, not active runtime. Permanent candidate needs headset verification; do not claim that the narrower refresh-only comparison was confirmed without user feedback.

0.10.93 is REJECTED by headset testing: removing native per-eye sky preparation broke sky projection/depth. 0.10.94 restores the complete 0.10.92 sky refresh and eye preparation, but confines canonical scene-camera borrowing to the synchronous sky-material refresh. Never retain that override across the whole scene/lighting draw or auxiliary draw. Both eye renderer cameras/projection matrices remain unchanged, and the orbital image orientation remains world-oriented. Native-scene regression asserts the eye camera is available inside sky preparation and the original native camera pointer is restored before each actual scene render. This is narrower than the failed .93 removal; directional blue lighting still needs headset verification.

0.10.95 night presentation: live native ambient/directional/reflection multiplier, HDR-level and luminance-floor comparisons produced no user-visible change; do not claim those raw scene writes were effective. Night scene exposure instead runs a highlight-preserving linear-light curve in the existing companion eye-transfer shader, identically for both eyes. The optional FSR path applies it in the existing final RCAS pass; no additional draw, surface, resolution change or camera change. Black/white and alpha are retained. UI/main-menu alpha surfaces and loading/front-end presentation bypass grading; HUD/menu foreground is drawn separately. NightSceneExposure defaults to 60% (5-100), NightBloom to 50% (0-100); 100 restores native. The verified native game-clock float is read only, with smooth dawn/dusk weighting (5-7/17-19). UserSettings bus v2 appends a sequence-protected owner/timestamp/hour packet; native and companion must update together. Reject stale/foreign/invalid clock samples to neutral rendering. Native night bloom modifies only HDR+0x48 through a private upload argument, with the existing final-composite coefficient ceiling covering right-eye cache reuse; keep vehicle bloom cap. No native exposure/history, ambient globals, sky preparation, scene-camera ownership, geometry, head/body or saved placements change. Tests use the production two-eye scene transfer on hardware and cover the native/day/UI identity, highlights/alpha, clock math and FSR pass. Headset balance still requires verification.

0.10.96 IMPORTANT CORRECTION: executable RVA3C2960 is GameRenderGetTodLight, corroborated by RSL functions and its companion colour setter RVA3C2990 writing light+CC. Scene+12C/+130 are directional rl_light pointers, NOT camera pointers. The old sky 'camera' borrowing was invalid and must NEVER return. Keep native per-eye sky/material refresh and all real camera/projection preparation, but leave the time-of-day light slot native. Remove the misleading vr_scene_eye_camera helper/test. The working moon orientation is independent.
Native light controls intercept rl_light colour packing RVA C27E0 after native output, matching only the verified selected time-of-day directional light (type0). Scale RGB only, preserving native alpha, attenuation, direction and source objects. Ambient front/back setters RVA116460/116590 receive private scaled RGB inputs; retain source alpha and native globals. Native signatures and ABI tests guard these hooks; other lights, NPC rigs and Standard rendering pass through. Night main/ambient controls range0-200%, default100. Native counters mainSamples/mainWrites/ambientWrites allow live activation inspection.
Night saturation and red/green/blue grading range0-200%, default100. Share one clock/settings plan per eye pair; apply grading in the existing scene blit and existing FSR final pass, never UI/main-menu alpha surfaces. Existing night exposure/bloom and automatic twilight fading remain. New controls apply/save through the established controller/desktop VR Settings panel with 5% increments and clamping. No setup hotkeys or extra per-frame renders/allocations. Headset light balance remains unverified.

0.10.97 crash correction: dump rfg+1165C5 confirms native caller 117855/11785B retains ECX and EDX across front/back ambient setters and XMM6/7 afterward. Normal C++ hook wrappers violate that stronger native contract even when their stack ABI is correct. Production frontEntry/backEntry preserve those registers around the existing callbacks; regression deliberately clobbers them in the forwarded callback and checks both production entry wrappers. No light, camera or settings change. Live stability requires verification.

0.10.98 night gate matches GameClockGetTimeOfDay RVA3C15E0 and partition API3C15C0: subtract6 from native hour byte RVA21054B0, wrap24, divide4; partitions0-2 day,3-5 night. The old5-7/17-19 twilight windows leaked into GC_DAY. Fade only05-06 and18-19; all exposure/bloom/main/ambient/saturation/RGB are exactly neutral during06-18. Live native19:47:27 matched the existing continuous time float19.7908, so do not replace it with an unrelated lighting parameter. User-approved ambient150 and all other saved choices are preserved. Regression checks every native daytime minute with extreme controls. Headset transition remains unverified.

0.10.99 approved night preset: exposure20, bloom100, main0, ambient200, saturation110, RGB55/90/140 percent, read from the author's saved profile at request time. EnhancedNightLighting defaults1; Night lighting toggles Improved/Vanilla. No individual sliders remain. Old Night* fields are ignored, never used to override this shipping preset. The preset shares .98 engine-day gating; Vanilla returns a fully neutral Plan. Existing unrelated profiles/default files are unchanged. Missing settings key defaults to enabled on existing and clean installations.

Corona billboards: native12ACA4 uploads camera+8 rotation;12ACDB uses position+2C;12AE1B copies glow world position from source+14. Decorative pass0 now receives a private38-byte camera copy using world-up, light-to-eye radial axes rather than HMD-forward axes. Real projection, eye translation, source object, local illumination, query passes1/2 and the confirmed glow-depth guard stay native. Signature checks cover these exact native reads. No texture, allocation, new pass or registry scan. Head-yaw stability is tested mathematically; final headlight appearance needs headset confirmation.

0.10.100: .99 radial billboard alone did not fix headset rotation. Read-only extraction/disassembly of native rl_corona vertex shader confirms an additional view-dependent image rotation: angle=(dot(normalize(eye-light),camRot[0])*TEXCOORD1.x*0.5+0.5)*2pi + radians(TEXCOORD2.z). Native12AE6E copies source+64 into TEXCOORD1.x. Live62 glows had nonzero scales on59. For decorative pass0 only, forward a private118-byte source copy with +64 zero, retaining authored angle+68, color, size and all other rendering inputs. Preserve the native pass0 visibility filter write at+C4 only; queries1/2 retain original pointer and complete native state. No template/global/simulation writes or shader replacement. Bounded spinRemoved counter permits activation checks. Production callback regressions cover both profiled paths, query passthrough and precise visibility writeback. Keep .99 world-up radial camera and confirmed depth guard. Headset result remains unverified. Extracted proprietary shader files stay outside source/distribution.

0.10.101 read-only cone diagnostic, NOT a claimed fix: user clarified visible headlight cones ahead, inside/outside vehicles, not lamp coronas. Actual light-cone_s/bs native shaders are separate mesh materials. The occupied Min_LightPickup_3 and its two type0 mesh lights were identified; parked world/render roots and valid vehicle palette stayed fixed. Do not infer a cone fix from corona counters. Private device VS creation wrapper tags only three verified native light-cone DXBC checksums (no extracted bytecode shipped). Opt-in RFGRVR-cone-trace.enable captures at most32 native cone common/instance constant-buffer snapshots, spacing150ms, timeout30seconds. Deferred copy commands add no binding changes; Present maps staging without waiting and releases all snapshots on completion/timeout. No inactive GPU copies or draw Get calls, no shader/beam/light modifications. Current eye and expected eye-position origin are recorded for comparison. CPU eye preparation correctness does not establish actual GPU cone constants. Actual GPU deferred test covers state preservation and exact constants. Live diagnostic capture required before changing beam shader/geometry.

0.10.102: headset-confirmed temporary comparison identifies veh_headlight_a.effectx g_light_cone_01 mesh type12 (ROD|ADDITIVE) as the head-turning cone cause. Creation-time vr_headlight_cones.h changes only matching headlight mesh ROD to NORMAL, preserving additive/other flags and native illumination. Existing signature-verified VFX create hook is reused; no registry scan or additional render pass. Permanent candidate requires headset confirmation. Lighting preset, corona wall occlusion and approved placements remain unchanged.

0.10.103 candidate: VR Settings uses the right-hand aim pose for a visible alpha-blended ribbon and panel cursor, trigger-edge selection, and an explicit recalibration button. Pointer uses a narrow atlas strip in the existing settings swapchain, active only in settings; it never edits the eye images or consumes another swapchain. Recalibration marks an existing-game session and releases the completed room without requiring a new mono preview frame from the paused game; initial launch still requires a valid preview. Handgun emitter offsets use held weapon scale rather than calibrated optical world scale; native muzzle/grip offsets also honor model scale. Saved laser placement is not rewritten. Pistol recoil increases to5.5 degrees with slower22 return and cumulative shot-direction climb. Preserve approved headlight NORMAL-mode fix and lighting preset. Headset results pending.

0.10.104 recovery: allocating the settings pointer as another startup swapchain exhausted the runtime resource budget: live main-menu UI swap stayed null after fallback to3840, leaving no foreground for popup menus either. Store the16x64 pointer strip at x2048 in a2064x2048 settings atlas. The menu crops exactly2048x2048; the beam samples the strip from the same released swapchain image. No separate pointer swapchain or per-frame acquire/update/release is allowed. Preserve the verified sharp menu surfaces and original swapchain count. Regression checks pointer-layer sharing without any acquire callback, atlas bounds, and settings behavior. Permanent result needs headset verification.

0.10.105: handgun kick follows bounded1-4 round ammo consumption even when the native firing/aim flag is late or absent. Previously that flag had to overlap the ammo decrease; a missed overlap discarded the kick permanently. Reload/magazine removal and weapon-identity changes remain excluded. Later firing flags with unchanged ammo do not add duplicate kick; other weapon profiles retain their existing gate. Regression covers eight rapid shots with delayed flags, three rounds between samples, empty-mag removal, reload, switching and cumulative shot direction. Recoil strength unchanged from0.10.103. Menu-atlas recovery retained. Headset results pending.

Release0.10.105: user approved the installed result for public packaging. Both builds and48 native tests passed; final changed recoil/arm tests and Standard-mode built-DLL smoke passed. Main menu/popups and pointer share the original swapchain budget. Preserve all approved shipping defaults; no runtime profiles or personal calibration recordings were reseeded for this release.
