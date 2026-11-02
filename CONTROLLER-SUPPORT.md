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
