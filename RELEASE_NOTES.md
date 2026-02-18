# Release Notes

## v1.1.3 (2026-02-17)

- Performed focused code cleanup across core app classes
  (`MainActivity`, `RFIDHandler`, `ScannerHandler`).
- Removed unused `MainUIHandler` and stale placeholder methods from RFID
  flow.
- Simplified reader health checks and trigger menu action logic.
- Consolidated duplicated busy/retry handling and trigger value logging.
- Updated `README.md`, `design.md`, and
  `DesignDoc_Trigger_Event_Handling.md` to reflect current architecture
  and trigger behavior.
- Validated with build/deploy/launch (`./build_deploy_launch.sh`) on
  connected device.

## v1.0.1 (2026-02-17)

- Renamed branding to Trigger Sync RFD_P+.
- Improved `build_deploy_launch.sh` for multi-device `adb` handling and
  macOS Bash compatibility.
- Added `.docx` exports for all project Markdown documentation.
- Updated README and release-note consistency.

## v1.0.0 (2026-02-17)

- Initial public GitHub publish for Trigger Sync RFD_P+.
- Repository initialized and pushed to `main` on GitHub.
- Added deadlock-safe trigger synchronization documentation for
  RFID/Barcode switching.
- Documented `bRfidBusy` busy-state guard behavior and bounded idle wait
  strategy.
- Added explicit safe switch sequence guidance:
  - RFID -\> Barcode: disable RFID handheld events, then switch
    keylayout to barcode.
  - Barcode -\> RFID: switch keylayout to RFID, then re-enable RFID
    handheld events.
- Added deadlock-avoidance contract and failure-handling behavior for
  trigger configuration paths.
- Created and pushed annotated release tag `v1.0.0`.

## v2.0.2 (2026-02-18)
- Major release for dev2 branch: Trigger Sync and deadlock-safe switching fully documented and validated.
- Improved concurrency and lock-guarded trigger switching (see Trigger_Sync_Doc.md).
- Updated all documentation and markdown files for new architecture and release.
- All code and docs ready for dev2.0.2 tag and push.
