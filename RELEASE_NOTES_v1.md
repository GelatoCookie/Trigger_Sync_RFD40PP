# Release Notes

## 1.1.2 (2026-02-16)

- Updated strings.xml and minor UI text improvements.
- Merged latest changes from remote and resolved all merge conflicts.
- Updated documentation and release notes for v1.1.2.
- Tagged and released as v1.1.2.

## 1.1.1 (2026-02-15)

- Updated app name to display version number (Trigger v1.1.1) for better version identification.
- Refactored trigger handling architecture: trigger events now route through handleTriggerPress method for better control.
- Improved tag clearing behavior: added clearTagData calls before scanning and in response handler to ensure clean state.
- Code cleanup: removed unused testFunction method from MainActivity.
- Development improvements: added VS Code build tasks configuration for streamlined workflow.
- Successfully tested on device 59040DLCH003LK.

## rc1 (2026-02-13)

- Project ready for public release: GitHub templates, CI workflow, and documentation updated.
- Modernized Pop-up UI: Centered, pill-shaped Snackbars with custom close icon and auto-dismiss.
- Hourglass loading indicator for auto-disappearing messages.
- Improved stability and UI alignment.
- Code cleanup, refactoring, and style compliance.
- All docs and release notes updated for rc1.

## Overview
This is the initial release (v1) of the Zebra RFID SDK Sample Application, providing a working Android app for RFID inventory and barcode scanning using Zebra's SDKs.

## Major Features
- RFID inventory and tag reading with unique tag count
- Barcode scanning integration
- UI for connecting/disconnecting RFID reader
- Start/Stop inventory controls
- Real-time tag and barcode display
- Device connection status and error handling
- Automated build, deploy, and launch script (auto.sh)
- Automated build, deploy, and launch script (build_deploy_launch.sh)

## Improvements
- Refactored UI logic for button enable/disable
- Consistent naming for UI elements (StartButton, btnStop)
- Added/updated Javadoc for all major classes and methods
- Improved error handling and user feedback
- Cleaned up and organized codebase

## Files Changed
- MainActivity.java: UI logic, Javadoc, button wiring
- RFIDHandler.java: Javadoc, event handling, logic improvements
- activity_main.xml: Button IDs, onClick wiring, layout updates
- build_deploy_launch.sh: New script for build/deploy/launch automation

## Known Issues
- None reported for this release

## Getting Started
See README.md for build and usage instructions.

----
