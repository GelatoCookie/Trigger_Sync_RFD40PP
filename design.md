# RFID SDKSample for Trigger Test Design Document

## 

## Overview

This project is a sample Android application demonstrating integration
with the Zebra RFID API3 SDK and barcode scanning libraries. It provides
a reference for connecting to Zebra RFID readers, performing inventory
operations, and handling barcode scans in a modern Android environment.

## 

## Architecture

- **MainActivity**: Handles all UI logic, user interactions, and
  implements the `RFIDHandler.ResponseHandlerInterface` for callbacks.
- **RFIDHandler**: Encapsulates all RFID reader logic, including
  connection management, inventory, and event handling. Uses an
  `ExecutorService` for background operations.
- **ScannerHandler**: Implements the Zebra scanner SDK delegate for
  barcode events and session management.

## 

## Key Components

- **RFID Connection**: Bluetooth-based, with support for multiple Zebra
  reader models. Handles connection, disconnection, and error states.
- **Inventory**: Real-time tag reading, with unique tag tracking and
  RSSI display. Inventory can be started/stopped via UI or hardware
  trigger.
- **Barcode Scanning**: Integrated with Zebra's scanner SDK, supports
  session management and barcode data callbacks.
- **Threading**: All device operations are performed off the UI thread
  using a single-threaded executor to ensure responsiveness.
- **Permissions**: Handles all required Bluetooth and location
  permissions, including Android 12+ requirements.

## Trigger Sync (RFID ↔ Barcode) {#trigger-sync-rfid-barcode-1}

- **Busy Guard (**`bRfidBusy`**)**: Set on `INVENTORY_START_EVENT`,
  cleared on `INVENTORY_STOP_EVENT`; prevents mode switching while RFID
  radio is active.
- **Mutual Exclusion (**`resourceLock`**)**: Serializes trigger
  reconfiguration methods (`setTriggerEnabled`,
  `restoreDefaultTriggerConfig`) to avoid overlapping SDK calls.
- **Bounded Wait Safety**: `waitForReaderIdle()` uses bounded waiting
  (up to \~3s total) and throws `TimeoutException` instead of blocking
  indefinitely.
- **Safe Switch Sequence**:
  - RFID → Barcode: `subsribeRfidTriggerEvents(false)` then
    `setTriggerEnabled(false)`.
  - Barcode → RFID: `setTriggerEnabled(true)` (method internally ensures
    RFID trigger events align with mode).
- **Failure Behavior**: Any timeout or SDK exception returns `false` and
  unlocks in `finally`, guaranteeing lock release.

## Build & Deployment {#build-deployment-1}

- All Zebra .aar libraries are included in `app/libs` and referenced via
  Gradle `flatDir`.
- Build and deployment are automated via `build_deploy_launch.sh`.
- Project is compatible with Android Studio Flamingo or newer, and API
  level 28+ (minSdkVersion updated for CoreComponentFactory
  compatibility).
- Lint errors related to `android:onClick` in XML and minSdkVersion were
  resolved for successful build and deployment.

## Extensibility

- The architecture allows for easy extension to support additional
  reader models, new UI features, or more advanced inventory/scanning
  workflows.
- All main classes are documented with Javadoc for maintainability.


## Release & History

- **dev2.0.2 (2026-02-18):**
    - Major release for Trigger Sync and deadlock-safe switching (see Trigger_Sync_Doc.md).
    - Improved concurrency and lock-guarded trigger switching.
    - Updated all documentation and markdown files for new architecture and release.
    - All code and docs ready for dev2.0.2 tag and push.
- See `history.md` and `README.md` for previous release notes and usage instructions.

For questions or contributions, see the project repository on GitHub.
