# Red Faction Guerrilla VR 0.13.58

Implements the supplied controls.txt Quest layout and configurable activation in VR Settings > Controls.

- B: tap to reload/chamber; hold to use or skip cutscenes. Use sends native Y without prompt detection.
- X: tap to detonate active C4; hold to accept a mission. Holds do not detonate first.
- Y: weapon mode, with quick-switch tap and the existing held selector.
- Menu: tap for world map; hold to pause. A jumps, right trigger fires, left stick moves, horizontal right stick turns and headset aims. Left-stick press sprints while moving; physical crouch and right-stick press both crouch. Melee remains weapon swinging; fine aim remains disabled.
- Every action has a saved Single press, Double press or Hold setting. Click the press-type column or use left/right on a selected control row. Bindings can share a source without automatic swapping.
- Hold is 500 ms; the double-press window is 300 ms. Shared single presses wait for conflicting gesture decisions. Controls held through menus, rebinding or mode changes must return to neutral. Analog inputs retain normal values unless a configured gesture requires arbitration.
- Quest bindings use a separate QuestControls section. Existing general bindings, comfort settings and other profiles' defaults remain. Menu A/B and vehicle thumb controls remain unchanged.

Tests cover default mappings, native use/detonate separation, shared gestures, boundary timing, mode/input changes, analog safety, cutscene transitions, persistence and actual desktop/controller settings interaction. Quest hardware verification remains pending. The unrelated test_modern_generated edge-overlay assertion is an existing environment limitation, also reproduced with untouched published 0.13.45 source.

Retains the user-confirmed turret behavior and native seated body/hands. Shared protocol is 14 with the same 632-byte packet, so native DLL and companion must be updated together. Install only with game and companion closed.

Free, noncommercial and open access under the included community license. Corresponding source and third-party notices are included. Requires your own supported Steam game installation.
