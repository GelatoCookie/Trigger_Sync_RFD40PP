# Release Notes

## v1.0.0 (2026-02-17)

- Initial public GitHub publish for Trigger Sync RFD40PP.
- Repository initialized and pushed to `main` at `GelatoCookie/Trigger_Sync_RFD40PP`.
- Added deadlock-safe trigger synchronization documentation for RFID/Barcode switching.
- Documented `bRfidBusy` busy-state guard behavior and bounded idle wait strategy.
- Added explicit safe switch sequence guidance:
	- RFID -> Barcode: disable RFID handheld events, then switch keylayout to barcode.
	- Barcode -> RFID: switch keylayout to RFID, then re-enable RFID handheld events.
- Added deadlock-avoidance contract and failure-handling behavior for trigger configuration paths.
- Created and pushed annotated release tag `v1.0.0`.
