# Red Faction Guerrilla VR

An unofficial OpenXR VR mod for **Red Faction Guerrilla Re-MARS-tered**, Windows / Steam app 667720.

Release 0.13.44. Includes the current lighting, tank cameras and approved placement defaults. Known issue: looking toward the sun can still produce excessive sky glare.

## [I WANNA PLAY NOW!](https://github.com/ElderMiner/Red-Faction-Guerrilla-Re-MARS-tered-VR/releases/latest/download/RFGRVR-Setup.exe)

Download the installer, run **RFGRVR-Setup.exe**, and follow the setup wizard. Requires your own Steam copy of Red Faction Guerrilla Re-MARS-tered.

## Made with AI. Made for everyone. Never paywalled.

**The original mod implementation was made entirely using AI, under human direction and extensive hands-on testing.** This describes the mod code, not the original game or third-party libraries.

This project is intended for everyone. You may use, study, improve, fork and reupload it under the [community license](LICENSE). **No version, fork, update or reupload may be sold, privatized for financial gain, made exclusive, or gated behind payments, memberships, subscriptions or paid early access.** Distributed versions must keep these free-access terms and publicly provide their corresponding mod source.

AI authorship does not automatically establish legal public-domain status or ownership. This is a **noncommercial, source-available project under a custom license**, not a public-domain dedication or a license to the underlying game. The license applies only to rights the project can grant; third-party components retain their own terms.

### A note of appreciation

Although AI generated the implementation, getting this mod to where it is took a substantial amount of time, testing, feedback, troubleshooting and repeated refinement. If you enjoy it and would like to show appreciation for that effort, you can do so at **[Buy Me a Coffee](https://buymeacoffee.com/ghostelder)**.

Donations are entirely optional. They never buy access, early releases, exclusive features, ownership or special control. The mod and its distributed derivatives must remain freely available to everyone.

## Features and status

- Stereo VR with tracked headset and motion-controller input.
- Controller-aimed weapons, support grips, recoil and heavier one-hand handling for large weapons.
- Physical magazines, hip pickup, insertion and chambering, with weapon-specific exceptions.
- Handgun laser with a short red beam and a depth-tested impact spot that grows with range to a fixed size limit; its 37.5-metre range fades out over the last 7.5 metres. Your approved mount position and appearance are included as defaults; developer placement shortcuts are disabled.
- Motion-based melee and throws, including inherited player/platform movement.
- First-person vehicle, mech and turret handling with saved type-specific eye positions.
- Vehicle-relative APC camera monitors, wrist/body HUD and world-positioned gameplay menus. Active mission timers appear above the wrist map.
- DLC Missilepod stack reloading and approved DLC weapon/vehicle/HUD placements. Temporary setup hotkeys and automatic testing unlocks are disabled; progression remains native.
- Main-menu artwork rasterized to a 6144 ï¿½ 3456 surface and drawn through both eye views. Loading/fallback screens use the same world-plane renderer. In-game popup menus reuse the menu surfaces with 6K native UI capture and per-eye foreground drawing while open; main-menu, loading-screen and popup presentation are headset-confirmed.
- Startup calibration, VR Settings with a right-hand laser pointer and trigger selection, a Recalibrate button, and right-eye desktop mirroring.
- Quest Touch/Touch Plus, Valve Index and Steam Frame controller layouts.
- A VR/Standard selector and install/update/uninstall wizard.

**Experimental:** automated tests do not prove correct behavior in every mission or headset. Quest and Index hardware verification is still needed. Foot/body, scope and vehicle edge cases remain under investigation. The attempted eye-tracked foveation backend was withdrawn after crashes; this release does not promise working foveated rendering.

## Requirements

You need your own **Steam Windows Re-MARS-tered release**, not the original 2009 game; Windows 10/11; a working PC OpenXR runtime; a VR headset with tracked motion controllers; and a Direct3D 11-capable PC with enough performance for stereo rendering.

Setup verifies the game executable against `distribution/supported-game.json`. Different builds are rejected rather than guessing at native hook addresses. Quest support is through PC VR, not a standalone Quest application. No game executable, assets or saves are included.

## Download and installation

1. Download `RFGRVR-Setup.zip` from the [public Releases page](https://github.com/ElderMiner/Red-Faction-Guerrilla-Re-MARS-tered-VR/releases).
2. Extract it outside the game folder; keep that copy for repair/uninstall.
3. Close the game and companion. For initial Steam integration, exit Steam using **Steam > Exit**.
4. Run **RFGRVR-Setup.exe**, select the folder containing `rfg.exe`, and click **Install / Update**.
5. Keep the Steam launch-choice checkbox enabled to get the selector when pressing Play in Steam.
6. Restart Steam, launch the game, and choose **Play in VR** or **Standard game**.

Launch through Steam to preserve Steamworks initialization. An existing third-party Steam launch wrapper may conflict; Setup reports that instead of silently replacing it.

The wizard is unsigned. Use packages from a source you trust. It validates hashes, stages changes, backs up replaced mod files and retains uninstall history. It does not patch `rfg.exe` or replace game archives.

## Calibration and startup

Connect the headset/controllers and start the appropriate OpenXR runtime before choosing VR.

1. In the calibration room, T-pose and hold both triggers for one second.
2. Follow the head-calibration dots left/right and up/down while keeping shoulders/body reasonably still. The dot follows your head; Mason is a visual reference.
3. The room fades and the normal videos/menus begin when calibration completes.

Normal VR launches calibrate each session. The launcher owns the companion: canceling launch or exiting the game should close it too. Close both before retrying a stuck session.

The retired wrist/elbow/overhead multi-pose fitting and body-size setup tools are not the shipping calibration process. Current calibration aligns the session and measures the head pivot; players do not need to recreate release weapon grips or vehicle eye levels.

## Controller layouts

**Click** means pushing a stick inward. **Push down** means deflecting it downward. They are different inputs. The runtime selects each controller's OpenXR profile; system buttons remain reserved for the runtime.

### Valve Index

| Control | Gameplay action |
|---|---|
| Left / right stick | Move or steer / turn |
| Right A / Left A | Jump / interact |
| Right B / Left B | Reload or chamber / detonate C4 in active C4 context |
| Left stick click | Sprint while moving; vehicle view reset |
| Right stick click | Crouch on foot; vehicle aiming overlay |
| Push right stick down | Accept mission offer |
| Press upper / lower left trackpad | Pause / large Map |
| Tap right trackpad | Quick-switch on release |
| Hold right trackpad contact 0.5 seconds | Weapon selector; gesture with right hand |

Left-pad commands require pressure and stay latched until release. Right-pad selection uses touch/contact. Index grips use squeeze force rather than fingers merely resting on the controller.

### Quest Touch / Touch Plus

| Control | Gameplay action |
|---|---|
| Left / right stick | Move or steer / turn |
| A / B / X | Jump / reload or chamber / detonate C4 in active C4 context |
| Tap Y | Interact with a fresh use prompt; otherwise quick-switch |
| Hold Y 0.5 seconds | Weapon selector; gesture with right hand |
| Left stick click | Sprint while moving; vehicle view reset |
| Right stick click | Crouch on foot; vehicle aiming overlay |
| Push right stick down | Accept mission offer |
| Left Menu / left grip + Menu | Pause / large Map |

Y taps resolve on release. A hold does not also generate a tap. The interaction decision stays fixed for that tap so a disappearing prompt cannot turn it into a second action.

### Steam Frame

| Control | Gameplay action |
|---|---|
| Left / right stick | Move or steer / turn |
| Right A / B | Jump / interact |
| X | Reload / chamber |
| Left bumper | Sprint while moving |
| Tap / hold right bumper | Quick-switch / weapon selector after 0.5 seconds |
| Left / right stick click | Crouch or vehicle reset / vehicle aiming overlay |
| View / Menu | Large Map / Pause |
| D-pad right | Accept mission offer |

### Shared input and menus

- Right trigger fires, prepares an immersive throw, or enables two-handed sledge area mode.
- Left trigger handles magazines and applicable secondary actions; it can also detonate C4 in its active context.
- Squeeze grips hold weapons and attach a support hand.
- **Right grip + right-stick up/down** adjusts vehicle eye height. On Quest/Index this takes priority over downward-stick mission acceptance.
- Turrets use right-hand aiming and left-stick rotation. Native mount/seat behavior differs by turret type.
- **Menus retain right A confirm / right B back**, even when gameplay uses those buttons differently. Navigate with the left stick.
- VR Settings tabs: Quest X/Y; Index upper/lower left-pad presses; Frame bumpers. Desktop arrows, Enter, Escape, clicks and mouse wheel also work.
- The Controls tab remaps logical gameplay actions. Profile-specific shared gestures/chords should be tested after remapping.

See [CONTROLLER-SUPPORT.md](CONTROLLER-SUPPORT.md) for binding details and verification limits.

## Weapons and interactions

### Immersive reloading

For most magazine-fed weapons:

1. An empty magazine does not automatically refill.
2. Press reload to eject it, or use the supported off-hand trigger interaction to remove it.
3. Hold the left trigger near either hip to fetch a magazine when reloadable. A left-hand pulse indicates an eligible pickup region.
4. Bring it into the visible insertion guide; it transitions into place.
5. Press reload again to chamber/engage ammunition.

Weapon exceptions exist. Rocket insertion adds one ready-to-fire rocket without a separate chamber press. Empty disc-launcher magazines are removed automatically. Button Reload in VR Settings restores button-driven loading instead of the immersive sequence.

### Aiming, melee and throwing

Two hands improve heavy-weapon handling. Automatic focus uses firing-hand proximity to the eyes with the applicable support grip. Sniper and Rail Driver have weapon-specific scope paths; scope cameras are gated to focus or explicit preview, not permanently active.

The sledge requires firm two-handed motion for damaging attacks. Hold right trigger while double-gripping for larger-area mode, indicated by the focus vignette. Shaft sliding and approved grip placements are retained.

For immersive C4/held-object throws, prepare with the trigger and release once armed. The higher third tone marks readiness; early release cancels. Release smoothing and inherited movement are part of velocity. Object orientation alone is not intended to launch a dropped object forward.

Report the exact weapon and interaction mode when something fails: native reload, melee, throwing and scopes differ across weapons.

## Vehicles, turrets and HUD

Vehicle control hints appear below the wrist map when available, including the vehicle's attacks. Type-specific eye heights are part of the release. Right grip + right-stick up/down adjusts height; the vehicle view-reset input recenters. Driver guns with an aiming overlay use look-directed aiming only when that mode is enabled; an inactive overlay should retain normal forward/native firing behavior.

Vehicle gunner seats, standing turrets and the Raider's rear-standing platform are distinct mounts. Hand aiming should not drag the player's torso or force crouching. Left-stick turret rotation is separate from right-hand fine aiming. APC monitors are vehicle-relative world objects; camera feeds are occupancy-gated and the parked presentation uses offline screens.

The minimap hides during support gripping. Use prompts appear near the left hand; mission offers are placed above the health region, vehicle health below the map, and faction updates are transient notifications. Visibility and fade timing often follow the native game: absence outside a change event is not necessarily a failure. Report the exact HUD element when something is missing.

## VR Settings

Open **Pause > VR SETTINGS** in the headset or desktop window. Point with the right hand and pull the right trigger to select. Use Previous / More Options to reach additional rows, or navigate with the controller stick/buttons. The Recalibrate button restarts calibration and returns to the running game.

| Option | Purpose |
|---|---|
| Recalibrate | Repeat calibration |
| Crouching | Full physical, half physical or button only |
| Jumping | Physical only, physical or button, or button only |
| Reloading / Throwing | Immersive or button interaction |
| Vehicle view / play mode | First-/third-person preference; standing/seated |
| Turning | Smooth/snap, speed and angle |
| Player visibility | Gloves Only default, or Full Body |
| Scene resolution | Low 11 PPD, Default approximately 16.8 PPD, High 20 PPD |
| Vibration / vignette | Feedback preferences |
| Eye height | On-foot adjustment; vehicle heights stay separate |
| Show on desktop | Right-eye mirror, or a small black window |

The desktop mirror reuses the right-eye image rather than rendering another world camera. High resolution increases workload. Runtime submission/reprojection frequency is not the same as fresh game-frame frequency; headset refresh is not a guaranteed game frame rate.

## Included defaults and saved settings

**Clean installs include the maintainer-approved eye levels and placements.** Players should not have to redo the weapon grips, magazines, scopes, seats or APC layout established for the base mod.

The 11 versioned profiles in `distribution/defaults/` are:

| File | Contains |
|---|---|
| `RFGRVR-eye-offset.ini` | Eye/world-scale/elevation defaults |
| `RFGRVR-vehicle-heights.ini` | Type-specific vehicle heights |
| `RFGRVR-weapon-alignment.ini` | Weapon and grip placements |
| `RFGRVR-reload-positions.ini` | Magazine/reload placements |
| `RFGRVR-weapon-scopes.ini` | Scope placements |
| `RFGRVR-apc-cameras.ini` | APC camera/screen layout |
| `RFGRVR-c4-alignment.ini` | C4 alignment |
| `RFGRVR-hand-alignment.ini` | Hand alignment |
| `RFGRVR-focus-distance.ini` | Focus distances |
| `RFGRVR-notification-layout.ini` | HUD notification layout |
| `RFGRVR-settings.ini` | VR preferences/bindings; Gloves Only default |

Setup seeds missing files only. Updates/repairs preserve existing files; uninstall retains them. To adopt one new default, close the game, back up that profile outside the game folder, rename/remove only that profile, and rerun Setup. Do not delete all settings as a first troubleshooting step.

Per-user head-pivot/elbow recordings are not universal defaults. Fresh headset calibration remains necessary. Developer placement shortcuts are disabled; finalized layouts are included.

## Updates, Standard mode and uninstall

The selector shows the installed version and **Check for updates**, and also checks when VR is selected. This release is configured to use the public `ElderMiner/Red-Faction-Guerrilla-Re-MARS-tered-VR` GitHub repository. An empty repository setting in a custom build disables downloads.

Once configured, updates use the latest published release's `RFGRVR-Setup.zip` and `.sha256`, verify the download, and run the same wizard. Close game/companion first. Manual downloads work too. Do not mix DLLs/companions from different versions. Backups are in `RFGRVR/backups`.

Choose **Standard game** to bypass VR startup/hooks and calibration. The companion does not start; native DirectInput forwarding remains available.

To uninstall, close game/companion, exit Steam if launch options need restoration, then run the extracted wizard or installed `RFGRVR/Setup.exe` and choose **Uninstall**. Its backup history restores prior files and its own Steam launch option. Profiles remain.

A manual wipe of the game folder deletes those files too. Reinstall the base game through Steam before the mod: the ZIP cannot install Red Faction itself. Keep later adjustments outside the game folder if you want them across a wipe.

## Troubleshooting and reports

| Symptom | First check |
|---|---|
| No headset view | PC-VR connection and active OpenXR runtime; close both processes and relaunch VR |
| Calibration stuck | Current prompt, tracked controllers and completed trigger/head sequence |
| Controls stop after switching windows | Bring game to foreground; report runtime and any pause/settings involvement |
| Unsupported executable | Supported Steam build; do not bypass hash/signature checks |
| Files in use | Close game/companion instead of forcing a loaded-file replacement |
| Old placement after updating | Existing profiles are preserved intentionally |
| Low frame rate | Compare Low/Default PPD in the same scene; record weapon, vehicle and extra views |
| Broken menu/stereo | Exact menu and whether one eye, both eyes or desktop is affected |
| Update unavailable | Repository and matching release assets actually configured |

Include mod version, headset/controllers, runtime/connection, game build, weapon/vehicle/menu, reproduction steps and whether Standard mode is affected. Review logs/dumps for private information before sharing. A right-eye capture helps but cannot prove binocular stereo correctness.

Areas still needing live verification include foot contact/clipping, crouch/body transitions, some scope effects, vehicle/turret edge cases, incoming-fire performance and head-locked ragdoll view. Ragdoll view follows native head orientation and may be disorienting.

## Source and maintenance

The editable release source is `RFGRVR-Source.zip`. Read [MAINTAINING.md](MAINTAINING.md); AI agents must also read [AGENTS.md](AGENTS.md). Historical notes under `docs/history/` are not current instructions.

**Do not upload the installed game folder to GitHub.** Upload mod source, not `rfg.exe`, game archives, extracted caches, saves, tracking recordings, dumps or credentials. Publish installer ZIP, checksum and corresponding source archive with the license/notices.

Third-party terms are in `distribution/THIRD-PARTY-NOTICES.txt` in source and `THIRD-PARTY-NOTICES.txt` beside the installer. This is an unofficial community modification, not an official game or headset-vendor release.

Mission offers retain their chest placement; mission-control graphics and timers use the approved wrist HUD layout.


Decorative light glows use the scene depth buffer in VR so nearby solid geometry can occlude them; actual spotlight illumination is retained.



Vehicle lighting preserves normal exposure and limits bloom separately. Improved nighttime lighting is the default; choose Vanilla in VR Settings to restore native nighttime values. Decorative light glows respect walls, and headlight cone meshes stay attached to their vehicle when you turn your head.


## Finalized HUD layouts

The installer includes the tested on-foot and vehicle HUD layouts, weapon grips, scope/laser placements, APC screen positions and vehicle eye heights. Fresh installs receive these defaults; updates preserve existing settings. On-foot and vehicle HUD profiles remain separate. Mission countdowns include their numbers alongside the stopwatch, and the EDF/Guerrilla mission-control display is in the wrist-map area.

Developer HUD, weapon and camera placement hotkeys are disabled in this release. Normal keyboard/controller menu navigation and VR Settings remain available. Saved layouts are loaded from `RFGRVR-hud-layout.ini` and `RFGRVR-vehicle-hud-layout.ini`. Personal head-pivot/arm recordings are not distributed; each player completes startup calibration.


### Night lighting

Pause > VR Settings > **Night lighting** selects **Improved (default)** or **Vanilla**. Improved uses the approved lighting and color preset; Vanilla restores native night lighting and color. The individual lighting sliders are removed. This choice saves automatically and applies live.

Both modes keep native daytime lighting from 06:00 to 18:00. The improved preset fades in from 18:00 to 19:00 and fades out from 05:00 to 06:00. HUD and menu artwork retain their brightness.

## Private tank setup (0.13.28)

Tanks stay in first person. Left stick drives/reverses/steers; right stick rotates/elevates the gun. Right trigger fires the main cannon; A fires the secondary turret. Numpad 0 on foot opens the tank-only spawn list (8/2 selects, 5 spawns). In a tank, Numpad 1–5 selects front/barrel/left/right/rear; Control switches screen/camera editing. Arrows position; camera mode Page Up/Down moves forward/back and */minus changes FOV. Screen mode Page Up/Down tilts forward/back, Home/End turns left/right and */minus changes curvature. Screen-mode Numpad 0 starts right-hand placement, right grip confirms. Num Lock does not affect physical keypad selection. All edits save automatically. These are private setup controls, separate from the locked APC layout.

Tank setup: Heavy tanks only: Insert raises / Delete lowers the hull-anchored body and attached eyes. Tank camera housings and their forward pointers are visible. Head movement does not aim the tank gun.

0.13.30 tank editor: physical Numpad keys stay separate from dedicated navigation keys regardless of Num Lock. In screen mode [ / ] decreases/increases width; ; / apostrophe decreases/increases height. Camera markers show through the hull. Full-circle tank mounts wrap at the rear angle.

Tank screen depth: regular . brings the selected screen closer; regular / moves it farther away. These are not numpad keys. Screen mode only; changes save automatically.

Tank update: left trigger does not fire; RT remains main cannon and A remains secondary turret. Regular . / slash edit depth in either selected screen or camera mode.

0.13.34 tank cabin: seated body, eyes and monitor assembly follow native seat/cannon traversal. Left-stick driving remains relative to the hull front; cameras retain hull/barrel mounts. Saved placement and heavy-only height adjustment are retained.

Private 0.13.35: local tank input uses native hull-relative tread mode only within the drive call, restoring the camera setting afterward. Native tank camera axes 6/7 are suppressed for controller 0 while the independent RS joint aim remains active. Rotating cabin/body/screens and saved placements remain unchanged. Other vehicles and Standard mode are unaffected. Headset results pending.

Private 0.13.36: Insert/Delete adjusts body and eyes in every tank, saved per canonical tank family. LT fires the secondary turret (A remains an alias), RT main gun, LS hull-relative driving and RS gun aim retained. Occupied tank screens draw over the hull; parked/APC panels retain depth occlusion. A one-time backed-up migration copies medium-tank screen layout to heavy/artillery families, preserving seats, heights and camera mounts/pitch/FOV.

Private 0.13.37: heavy/artillery body, eyes and screens stay hull-fixed; medium cabin retains traversal. Full-circle tank yaw stays continuous across +/-180 degrees instead of wrapping the motor target by a full turn. Partial arcs remain clamped. Height, LT/RT controls, hull-relative steering, saved layouts and camera mounts retained. Headset verification pending.

Private 0.13.38: artillery gun camera uses stable launcher assembly origin/orientation rather than alternating native muzzle tubes; other tank gun cameras retain muzzle attachment. Tank feed scheduling: gun every game frame, front/left/right/rear every second frame; three reduced-resolution passes per frame. APC remains four feeds. Saved camera offsets, body anchoring, screen layout and controls retained. Overall FPS and headset result unverified.

Private 0.13.38 follow-up: screens 1/front and 2/gun update each game frame (2.5 times the previous two-of-five cadence); left/right/rear rotate at one third rate. Left camera position is authoritative on load, with right mirrored; editing either side updates its counterpart. Screen transforms remain independent. Tank aim input expires after 60 ms instead of 250 ms and is rejected while paused, reducing stale-input carryover. Residual motion after cannon movement remains unverified: no live tracking map was available for capture.

Private 0.13.39: artillery camera 2 uses the verified mechanical elevation pivot (yaw pivot fallback, then turret root), never a firing weapon/tube position. Gun orientation still controls its viewing direction; saved camera offsets remain applied. Tests now vary weapon position while keeping pivot fixed. Existing feed cadence, mirrored side positions and tank controls retained. Live result pending.

Private 0.13.40: LT directly overrides native tank secondary held/pressed actions 0x2A for local controller 0, with release/expiry/pause handling; A is removed from tank gameplay. APC front feed updates every game frame; others rotate at one third rate, reducing total to two passes. Tank feed cadence retained. Camera setup markers hidden. Includes pending artillery pivot correction. Saved settings retained; live results pending.

Private 0.13.41: artillery camera 2 locks its position to the exact approved 0.13.40 live capture, in hull-local coordinates (-0.152348364, 1.506873195, 3.504900077); gun orientation remains live. Firing pivot, tube and weapon position changes cannot relocate it. Camera editor offsets remain usable. Camera profile loader canonicalizes all tank variants to heavy/medium/artillery family entries; current saved profiles already use those three families. Tests cover tube/weapon/pivot movement, direction and variant reload. Live result pending.

Private 0.13.42: measured artillery weapon roll differs from the pitch rigid-body frame although forward matches. Camera 2 now uses the verified physical pitch body basis (+70), not the tube weapon basis; approved captured position is stored in native hull units, removing calibration-scale relocation. Per-family CameraMountScale keeps authored camera offsets independent of later seat calibration changes; canonical profiles already share all variants. Auxiliary view passes temporarily disable native sun generation and restore it afterward to prevent shared glare-texture contamination; main-eye shafts retain their path. Live results pending.

Private 0.13.43: artillery camera 2 is a full rigid attachment to the verified physical launcher model body (+70). Position and orientation use the same frame; editor position is a fixed local offset, preserving the native rotation pivot. Supersedes the hull lock in 0.13.41/42. Derived initial offset uses the approved position and recorded model geometry; live placement still needs verification. Glare observation captured zero SunGlare01 values throughout; remaining sky wash is not confirmed fixed.
