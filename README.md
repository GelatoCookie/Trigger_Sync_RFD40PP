
# Trigger Sync RFD_P+

**Release Version: v1.0.1**  
Branch: `main`  
Tag: `v1.0.1`
Release Notes: [RELEASE_NOTES.md](RELEASE_NOTES.md)

Trigger Sync RFD_P+ demonstrates how to integrate and use the Zebra RFID API3 SDK for Android. It provides a practical implementation for connecting to Zebra RFID readers and safely switching hardware trigger behavior between RFID inventory and barcode scanning.

---

## v1.0.0 Highlights

- Initial public GitHub release for Trigger Sync RFD_P+.
- Added trigger synchronization documentation for safe RFID/Barcode switching.
- Documented `bRfidBusy` busy-state guard with bounded idle wait to avoid deadlocks.
- Added release workflows and release notes alignment for tag-based publishing.

## Features

- **Modern UI Notifications:** Centered, pill-shaped Snackbars with auto-dismiss and manual close options.
- **RFID Reader Connection:** Connect and disconnect from Zebra RFID readers via Bluetooth.
- **RFID Inventory:** Perform real-time inventory to discover RFID tags.
- **Tag Data Display:** View unique tag IDs along with their peak RSSI values in a list.
- **Barcode Scanning:** Utilize the reader's scanner to capture barcode data.
- **Hardware Trigger Support:** Handle hardware trigger presses for starting/stopping inventory or scanning barcodes.
- **Settings Configuration:** Basic demonstration of modifying antenna settings and singulation control.

## Project Structure

- `MainActivity.java`: Handles the UI logic, user interactions, and the modern notification system.
- `RFIDHandler.java`: Manages the lifecycle and operations of the Zebra RFID reader.
- `ScannerHandler.java`: Handles barcode scanning functionality.
- `MainUIHandler.java`: Helper for UI-related updates.

## Getting Started

### Prerequisites

- Android Studio Flamingo or newer.
- Android device running API level 26 (Oreo) or higher.
- A compatible Zebra RFID Reader (e.g., RFD8500, RFD40, RFD90).
- Zebra RFID SDK for Android (ensure `.aar` or `.jar` files are placed in `app/libs`).

### Installation

1. Clone or download this repository.
2. Open the project in Android Studio.
3. Sync project with Gradle files.
4. Build and deploy the application to your Android device.

## Usage

1. **Connect:** Launch the app and tap the status text at the top to search and connect to an available Zebra RFID reader.
2. **Inventory:** Once connected, tap **Start Inventory** to begin reading tags. Tap **Stop Inventory** to end the session.
3. **Barcode Scan:** Tap the **Scan** button or use the hardware trigger (if configured) to scan barcodes.
4. **View Tags:** Discovered tags will appear in the list with their EPC and RSSI.
5. **Dismiss Messages:** Pop-up messages in the center of the screen can be dismissed manually by tapping the 'X' button or will auto-dismiss after 3 seconds if they show a loading icon.

## Permissions

The application requires the following permissions:
- `BLUETOOTH`
- `BLUETOOTH_ADMIN`
- `BLUETOOTH_SCAN` (Android 12+)
- `BLUETOOTH_CONNECT` (Android 12+)
- `ACCESS_FINE_LOCATION` (Required for Bluetooth scanning on some Android versions)

## License

This project is for demonstration purposes. See the [LICENSE](LICENSE) file for details.
