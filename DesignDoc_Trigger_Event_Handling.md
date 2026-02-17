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
To ensure thread safety and hardware stability, we employ a **"Wait-for-Idle"** pattern protected by a `ReentrantLock`.

*   **`resourceLock` (ReentrantLock):** Ensures mutual exclusion. Only one configuration operation can proceed at a time.
*   **`bRfidBusy` (volatile boolean):** Acts as a guard. Configuration commands are blocked until this flag is `false` (indicating the reader is idle).
*   **`waitForReaderIdle()`:** A helper method that polls `bRfidBusy` with a timeout mechanism to prevent deadlocks.

## 3. Detailed Design

### 3.1. `setTriggerEnabled(boolean isRfidEnabled)`
This method physically reconfigures the reader's hardware trigger behavior.

**Workflow:**
1.  **Acquire Lock:** Enters `resourceLock` to block other configuration attempts.
2.  **Wait for Idle:** Calls `waitForReaderIdle()`.
    *   If the reader is busy (scanning), it waits up to 2 seconds.
    *   If the timeout is reached, it throws a `TimeoutException` and aborts to prevent hardware errors.
3.  **Determine Mode:**
    *   `true` -> `ENUM_NEW_KEYLAYOUT_TYPE.RFID` (Trigger starts RFID Inventory).
    *   `false` -> `ENUM_NEW_KEYLAYOUT_TYPE.SLED_SCAN` (Trigger activates Barcode Scanner).
4.  **Apply Configuration:** Calls `reader.Config.setKeylayoutType(mode, mode)`.
5.  **Release Lock:** Ensures the lock is released in a `finally` block.

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
2.  **Subscribe Events:** Call `subsribeRfidTriggerEvents(true)`.
    *   *Reason:* Now that the hardware is ready, start listening for press events to trigger `performInventory()`.

## 5. Safety Mechanisms

| Mechanism | Purpose |
| :--- | :--- |
| **`ReentrantLock`** | Prevents `restoreDefaultTriggerConfig` and `setTriggerEnabled` from running concurrently. |
| **`waitForReaderIdle`** | Prevents sending configuration commands while the radio is active (Inventorying), which causes `OperationFailureException`. |
| **Timeout (2s)** | Prevents the UI thread or background executor from hanging indefinitely if the reader gets stuck in a busy state. |
| **Try-Catch-Finally** | Ensures locks are always released, even if the SDK throws an unexpected exception. |

## 6. Code Reference

```java
public boolean setTriggerEnabled(boolean isRfidEnabled) {
    resourceLock.lock();
    try {
        if (reader == null || !reader.isConnected()) return false;

        try {
            waitForReaderIdle();
        } catch (TimeoutException e) {
            Log.e(TAG, "setTriggerEnabled failed: " + e.getMessage());
            return false;
        }

        ENUM_NEW_KEYLAYOUT_TYPE mode = isRfidEnabled ? 
            ENUM_NEW_KEYLAYOUT_TYPE.RFID : ENUM_NEW_KEYLAYOUT_TYPE.SLED_SCAN;
            
        RFIDResults result = reader.Config.setKeylayoutType(mode, mode);
        return result == RFIDResults.RFID_API_SUCCESS;
    } finally {
        resourceLock.unlock();
    }
}
```