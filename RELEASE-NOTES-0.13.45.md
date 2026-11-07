# Red Faction Guerrilla VR 0.13.45

- Fixes tracked gameplay arms requiring the developer-only `RFGRVR-controller-arms.enable` file, which was absent from clean installer payloads. Explicit VR launches now enable tracked arms without that file. Standard and unset launch modes remain disabled.
- Quest Touch/Touch Plus, Valve Index and Steam Frame retain their separate OpenXR bindings and gameplay layouts. Pose sampling continues through the same action set before and after calibration; this fix changes native arm activation, not controller bindings or calibration measurements.
- Adds regressions covering missing-marker native arm activation, pose publication across calibration completion, and recovery after inactive or invalid controller poses. Includes the existing calibration regression in the normal test suite.
- Preserves saved settings, placements, world scale and body/head calibration. No new activation file is required.

This corrects a reproduced clean-install defect. It does not establish the cause of every reported tracking failure. Quest and Index hardware verification remains required.

Known issue: excessive sun/sky glare can still appear while looking upward.

Install: extract RFGRVR-Setup.zip and run RFGRVR-Setup.exe, or use the standalone installer. Launch through Steam and select VR. Updates preserve existing profiles.

Free, noncommercial and open access under the included community license. Corresponding source and third-party notices are included. Requires your own supported Steam game installation.
