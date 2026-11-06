# Controller support

The OpenXR runtime selects the active interaction profile for each hand.
Steam Frame, Valve Index, Oculus Touch and Quest Touch Plus profiles receive
their own suggested bindings. Touch Plus is enabled only when the runtime
advertises its extension; the ordinary Touch profile remains available.
Controller labels in VR Settings update when the active profile changes.

| Action | Steam Frame | Quest 3 Touch / Touch Plus | Valve Index |
|---|---|---|---|
| Fire / secondary fire | Right / left trigger | Same | Same |
| Hold / support grip | Grip | Grip trigger | Force-sensitive squeeze |
| Move / turn | Left / right stick | Same | Same |
| Jump / interact | Right A / B | A / tap Y with use prompt | Right A / Left A |
| Reload | X | Right B | Right B |
| Quick switch / hold weapon selector | Right bumper | Tap Y without use prompt / hold Y | Tap right pad: swap / hold: selector |
| Map | View | Left grip + Menu | Press lower left trackpad |
| Pause | Menu | Left Menu | Press upper left trackpad |
| Crouch / vehicle recenter | Left stick click | RS click on foot / LS click in vehicle | RS click on foot / LS click in vehicle |
| Vehicle aim overlay | Right stick click | Right stick click | Right stick click |

Frame and Quest retain the half-second hold delay and gesture selection.
Index: tap the right trackpad to quick-switch; hold for 0.5 seconds to open gesture selection.
The Index system buttons remain owned by the VR runtime. Missing bumpers are
identified in the remapping screen. VR Settings can be navigated with the left
stick and A/B, so a physical D-pad is not required.

Index gripping uses squeeze force, rather than merely resting fingers on the
controller. A half-scale firm squeeze reaches full grip, with the existing
grab/release hysteresis.

Pose tracking is never remapped. The existing Controls tab can remap gameplay
actions. Controllers that provide native finger-joint data use that data;
otherwise the existing analog-grip open/closed fallback remains available.

Quest 3 and Index hardware verification is still required: the binding and
fallback tests do not constitute headset testing on those devices.

## Release placements
Setup hotkeys are retired. Approved weapon, grip, scope, magazine, vehicle-eye
and APC placements ship in distribution/defaults and seed clean installs.
Updates preserve existing settings. Normal VR Settings remain available.

## Profile references

- [Khronos OpenXR interaction profiles](https://registry.khronos.org/OpenXR/specs/1.0-khr/html/xrspec.html)
- [Meta Touch Plus controller profile](https://developers.meta.com/vr/documentation/native/pc/native-touch-plus-controllers/)

Quest additions: left stick click sprints on foot; pushing the right stick down accepts a mission offer. Right grip + right stick up/down retains vehicle eye-height adjustment and does not accept missions. Left X detonates C4 only. A/B retain confirm/back in menus. Y taps resolve on release; holding for 0.5 seconds opens the selector without interacting or quick-switching first. Steam Frame and Index layouts are unchanged.

Index: LS click sprints on foot. Left B detonates C4 only. Pushing RS down
accepts mission offers unless right-grip vehicle eye-height adjustment owns
the axis. The left pad uses force/touch hysteresis and latches the pressed
half until release. The right pad quick-switches on a short tap and opens selection after a
half-second hold. In menus A/B retain confirm/back; in VR Settings the left
pad's upper/lower presses switch tabs. Runtime system buttons are untouched.

Free-standing turret eye height: hold RIGHT GRIP and move RIGHT STICK up/down. The setting saves for all free-standing turrets. Left-stick turret orbit and right-hand pivot aiming remain unchanged.

Private tank setup: left stick drives/reverses/steers, right stick traverses/elevates the driver gun; RT fires the main cannon and A fires the secondary turret. Right grip retains the vehicle eye-height modifier and suppresses right-stick gun aiming while held. Tanks stay first person. Other vehicle/controller layouts are unchanged.

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

Release 0.13.44: approved placement defaults saved from the author session; private head/arm recordings excluded. RFGRVR_DEVELOPER_TOOLS gates tank editors/spawner and is undefined in distribution builds. Normal controls and VR Settings remain. Known sun-glare issue documented; no GitHub publication performed by packaging.
