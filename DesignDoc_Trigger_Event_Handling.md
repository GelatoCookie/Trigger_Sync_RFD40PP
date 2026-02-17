# Design Document: RFID Trigger Configuration & Event Subscription

## 1. Overview
The `RFIDHandler` class manages the Zebra RFID reader's hardware trigger, which serves two distinct functions depending on the application state:
1.  **RFID Mode:** The trigger starts/stops RFID tag inventory.
2.  **Barcode (Sled Scan) Mode:** The trigger activates the barcode scanner.

To support these modes, the application must dynamically reconfigure the hardware key layout and manage event subscriptions. This document details the design of `setTriggerEnabled` (hardware config) and `subsribeRfidTriggerEvents` (software event masking).

## 2. Architecture & Synchronization

### 2.1. The Challenge
Reconfiguring the hardware trigger (`setKeylayoutType`) involves sending commands to the reader. If the reader is currently performing an inventory (busy) or if multiple threads attempt to configure it simultaneously, the SDK may throw exceptions or the reader may enter an undefined state.

### 2.2. Synchronization Strategy
To ensure thread safety and hardware stability, we use a **lock + busy-guard** strategy, with bounded waiting reserved for default restore operations.

*   **`resourceLock` (ReentrantLock):** Ensures mutual exclusion. Only one configuration operation can proceed at a time.
*   **`bRfidBusy` (volatile boolean):** Acts as a guard. Configuration commands are blocked until this flag is `false` (indicating the reader is idle).
*   **`waitForReaderIdle()`:** A bounded helper used by `restoreDefaultTriggerConfig()` to avoid waiting indefinitely while the reader is busy.

## 3. Detailed Design

### 3.1. `setTriggerEnabled(boolean isRfidEnabled)`
This method physically reconfigures the reader's hardware trigger behavior.

**Workflow:**
1.  **Acquire Lock:** Enters `resourceLock` to block other configuration attempts.
2.  **Check Busy Guard:** If `bRfidBusy` is `true`, operation is rejected immediately, inventory stop is requested, and UI gets a retry message.
3.  **Determine Mode:**
    *   `true` -> `ENUM_NEW_KEYLAYOUT_TYPE.RFID` (Trigger starts RFID Inventory).
    *   `false` -> `ENUM_NEW_KEYLAYOUT_TYPE.SLED_SCAN` (Trigger activates Barcode Scanner).
4.  **Apply Configuration:** Calls `reader.Config.setKeylayoutType(mode, mode)`.
5.  **Align Event Subscription:** On success, calls `subsribeRfidTriggerEvents(isRfidEnabled)` to keep software events consistent with hardware mode.
6.  **Release Lock:** Ensures the lock is released in a `finally` block.

### 3.2. `subsribeRfidTriggerEvents(boolean enable)`
This method controls whether the application receives `HANDHELD_TRIGGER_EVENT` notifications from the SDK.

**Purpose:**
*   **Enable (`true`):** The app receives callbacks when the trigger is pressed/released. This is essential for RFID mode to programmatically start/stop inventory.
*   **Disable (`false`):** The app ignores trigger events. This is useful when switching to Barcode mode to prevent the RFID logic from reacting to trigger presses intended for the scanner.

## 4. Integration Workflow

When switching modes (e.g., from RFID to Barcode), these methods should be used in a specific sequence to ensure a clean transition.

### Scenario: Switching to Barcode Mode
1.  **Unsubscribe Events:** Call `subsribeRfidTriggerEvents(false)`.
    *   *Reason:* Stop the `RFIDHandler` from receiving "Pressed" events immediately, preventing it from trying to start an RFID inventory while we are reconfiguring.
2.  **Reconfigure Hardware:** Call `setTriggerEnabled(false)`.
    *   *Reason:* Tell the hardware that the trigger should now act as a scanner button.

### Scenario: Switching to RFID Mode
1.  **Reconfigure Hardware:** Call `setTriggerEnabled(true)`.
    *   *Reason:* Tell the hardware the trigger is for RFID.
2.  **Subscription Alignment:** `setTriggerEnabled(true)` enables RFID trigger events on success.
    *   *Reason:* Keeps app event handling synchronized with hardware mode.

## 5. Safety Mechanisms

| Mechanism | Purpose |
| :--- | :--- |
| **`ReentrantLock`** | Prevents `restoreDefaultTriggerConfig` and `setTriggerEnabled` from running concurrently. |
| **`waitForReaderIdle`** | Prevents sending restore-default commands while the radio is active (inventorying). |
| **Timeout (~3s)** | Prevents hangs if the reader remains busy/stuck during restore-default workflow. |
| **Try-Catch-Finally** | Ensures locks are always released, even if the SDK throws an unexpected exception. |

## 6. Deadlock-Avoidance Contract

To keep trigger mode changes safe under concurrent events, the implementation follows these rules:

1. **Never block indefinitely while holding `resourceLock`**
    - `waitForReaderIdle()` has bounded retries and fails fast with `TimeoutException`.
2. **Never require event thread locks for busy-state transitions**
    - `bRfidBusy` is `volatile` and updated directly by status events.
3. **Always release lock on every path**
    - `setTriggerEnabled` and `restoreDefaultTriggerConfig` unlock in `finally`.
4. **Switch order is intentional**
    - RFID→Barcode unsubscribes RFID trigger events before hardware switch.
    - Barcode→RFID applies hardware mode and enables RFID trigger events on successful configuration.

These rules prevent circular waiting between trigger event handling and trigger configuration APIs.

## 7. Code Reference

```java
public boolean setTriggerEnabled(boolean isRfidEnabled) {
    resourceLock.lock();
    try {
        if (reader == null || !reader.isConnected() || context == null) return false;
        if (bRfidBusy) {
            context.showSnackbar("BUSY and Retry Set Trigger Again!!!", false);
            stopInventory();
            return false;
        }

        ENUM_NEW_KEYLAYOUT_TYPE mode = isRfidEnabled
            ? ENUM_NEW_KEYLAYOUT_TYPE.RFID
            : ENUM_NEW_KEYLAYOUT_TYPE.SLED_SCAN;
        RFIDResults result = reader.Config.setKeylayoutType(mode, mode);
        if (result == RFIDResults.RFID_API_SUCCESS) {
            subsribeRfidTriggerEvents(isRfidEnabled);
            return true;
        }
        return false;
    } finally {
        resourceLock.unlock();
    }
}
```