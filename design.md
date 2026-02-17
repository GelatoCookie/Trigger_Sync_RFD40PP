# AI_Java_SDKSample Design Document

## Overview
This project is a sample Android application demonstrating integration with the Zebra RFID API3 SDK and barcode scanning libraries. It provides a reference for connecting to Zebra RFID readers, performing inventory operations, and handling barcode scans in a modern Android environment.

## Architecture
- **MainActivity**: Handles all UI logic, user interactions, and implements the `RFIDHandler.ResponseHandlerInterface` for callbacks.
- **RFIDHandler**: Encapsulates all RFID reader logic, including connection management, inventory, and event handling. Uses an `ExecutorService` for background operations.
- **ScannerHandler**: Implements the Zebra scanner SDK delegate for barcode events and session management.
- **MainUIHandler**: Abstract class for UI update patterns, allowing for future extension or separation of UI logic.

## Key Components
- **RFID Connection**: Bluetooth-based, with support for multiple Zebra reader models. Handles connection, disconnection, and error states.
- **Inventory**: Real-time tag reading, with unique tag tracking and RSSI display. Inventory can be started/stopped via UI or hardware trigger.
- **Barcode Scanning**: Integrated with Zebra's scanner SDK, supports session management and barcode data callbacks.
- **Threading**: All device operations are performed off the UI thread using a single-threaded executor to ensure responsiveness.
- **Permissions**: Handles all required Bluetooth and location permissions, including Android 12+ requirements.

## Build & Deployment
- All Zebra .aar libraries are included in `app/libs` and referenced via Gradle `flatDir`.
- Build and deployment are automated via `build_deploy_launch.sh`.
- Project is compatible with Android Studio Flamingo or newer, and API level 28+ (minSdkVersion updated for CoreComponentFactory compatibility).
- Lint errors related to `android:onClick` in XML and minSdkVersion were resolved for successful build and deployment.

## Extensibility
- The architecture allows for easy extension to support additional reader models, new UI features, or more advanced inventory/scanning workflows.
- All main classes are documented with Javadoc for maintainability.

## Release & History
- See `history.md` and `README.md` for release notes and usage instructions.

---
For questions or contributions, see the repository at https://github.com/GelatoCookie/AI_Java_SDKSample
