# Instructions for AI agents editing RFGRVR

Read `README.md`, `LICENSE`, `MAINTAINING.md` and `CONTROLLER-SUPPORT.md` before editing. `MAINTAINING.md` is the full current technical guide. Files under `docs/history/` describe experiments and are not current operating instructions.

- Respect the user's current request and preserve accepted behavior outside its scope.
- Keep the mod and distributed derivatives noncommercial, publicly accessible and free from paid/member-only/exclusive access. Optional donations buy nothing. Do not claim AI generation automatically establishes public-domain ownership.
- Preserve upstream licenses and never publish game assets, the game executable/archives, extracted caches, saves, private tracking data, credentials or the enclosing installation folder.
- Do not spawn subagents unless the user or applicable higher-level instructions explicitly authorize delegation.
- Changes must target verified local-player ownership; never mutate a shared rig or guessed mesh slot to alter the player. Preserve NPC animation and heads.
- Foot/leg edits must not change camera/torso height. Preserve crouch/jump/landing ownership, calibrated head pivot, glove/cuff rotation, independent world scale and saved eye/grip/vehicle positions.
- `distribution/defaults/` is approved shipping data, not disposable local settings. Setup seeds only missing profiles; repairs/updates/uninstall preserve existing files. Never regenerate defaults from an arbitrary developer session.
- Preserve separate Quest/Index/Frame layouts and menu versus gameplay semantics. Stick clicks and axis directions are different. Test tap/hold, release, modifiers and transitions while held.
- Standard mode must not run VR hooks/input/companion. Do not bypass executable signatures or supported-game validation.
- Build from these sources. Do not patch release binaries by hand or replace loaded files. Finish and verify a candidate before requesting game closure for installation.
- Run checks appropriate to actual edits. Compiled, tested, packaged, installed and headset-confirmed are different claims. Report the distinction honestly.
- Avoid speculative rewrites. Measure a failure, make a narrow change, add a meaningful regression, and inspect final rendered/physical behavior when required.
- Do not reactivate withdrawn foveation, advanced multi-pose body fitting, setup hotkeys, frozen-world menu workarounds, per-render APC searches or slower shader experiments because historical notes mention them.
- Keep source/installer documentation, license files, controller tables and release contents aligned. Publish corresponding source with distributed binaries under LICENSE.
