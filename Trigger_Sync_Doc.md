# Trigger Sync Doc: bRfidBusy and Deadlock-Safe Trigger Switching

## Purpose

This document defines the runtime contract for safely switching the
RFD40 trigger between RFID inventory and barcode scanning modes.

## Core Problem

The trigger is shared by two subsystems: - RFID inventory
(`reader.Actions.Inventory.perform/stop`) - Barcode scanner pull trigger
(`DCSSDK_DEVICE_PULL_TRIGGER`)

If the trigger mode is changed while RFID is still active, SDK calls can
fail or become unstable.

## Concurrency Model

- Single worker thread:
  `ExecutorService executor = Executors.newSingleThreadExecutor()` for
  serialized operational tasks.
- Lock for trigger config: `ReentrantLock resourceLock` wraps trigger
  configuration methods.
- Busy state signal: `volatile boolean bRfidBusy` tracks RFID operation
  state from status events.


## bRfidBusy: Busy Guard Logic

### Purpose
`bRfidBusy` is a `volatile boolean` flag in `RFIDHandler` that acts as a concurrency guard, indicating whether the RFID reader is currently busy (performing inventory or other operations). It prevents trigger configuration changes and inventory operations from being attempted while the reader is active, ensuring thread safety and hardware stability.

### Lifecycle
- `bRfidBusy = true` on `INVENTORY_START_EVENT`
- `bRfidBusy = false` on `INVENTORY_STOP_EVENT`

### Usage
- Before any trigger mode/configuration change (e.g., in `setTriggerEnabled` or `restoreDefaultTriggerConfig`), the code checks `bRfidBusy`. If `true`, the operation is rejected, inventory is stopped, and the UI is notified to retry.
- The `waitForReaderIdle()` helper method polls `bRfidBusy` with a timeout to ensure the reader is idle before proceeding with configuration changes.

### Example
```java
if (bRfidBusy) {
  Log.e(TAG, "setTriggerEnabled failed: Reader is busy");
  stopInventory();
  uiHandler.post(() -> {
    if (context != null) context.showSnackbar(BUSY_RETRY_MESSAGE, false);
  });
  return false;
}
```

### Rationale
- Prevents race conditions and SDK errors by ensuring only one operation is active at a time.
- Ensures that trigger mode switches and configuration changes are only performed when the reader is idle.
- Provides a robust, thread-safe mechanism for synchronizing hardware state with application logic.

### State Table
| Event                  | bRfidBusy Value | Allowed Operations                |
|------------------------|-----------------|-----------------------------------|
| Inventory started      | true            | No config changes, no new inventory|
| Inventory stopped      | false           | Config changes allowed            |
| During config attempt  | checked         | Blocked if true                   |

---
This logic is essential for safe, deadlock-free trigger switching and reliable operation in concurrent environments.

## Deadlock-Safe Rules

1.  `setTriggerEnabled(...)` and `restoreDefaultTriggerConfig()` must
    always acquire `resourceLock`.
2.  Any lock acquisition path must release lock in `finally`.
3.  Waiting for idle must be bounded:
    - `waitForReaderIdle()` retries 10 times with 200ms sleep (max 2s).
    - Throws `TimeoutException` on timeout.
4.  No blocking dependency on event callbacks while lock is held.
5.  On timeout/failure, return `false` and keep system responsive.

## Safe Switch Sequences


## subscribeRfidHardwareTriggerEvents: Event Subscription Logic

### Purpose
`subscribeRfidHardwareTriggerEvents(boolean enable)` controls whether the application receives `HANDHELD_TRIGGER_EVENT` notifications from the SDK. This allows the app to dynamically enable or disable trigger event handling based on the current mode (RFID or Barcode).

### Usage
- **Enable (`true`)**: The app receives callbacks when the trigger is pressed/released. This is essential for RFID mode to programmatically start/stop inventory.
- **Disable (`false`)**: The app ignores trigger events. This is useful when switching to Barcode mode to prevent the RFID logic from reacting to trigger presses intended for the scanner.

### Integration in Mode Switch
- When switching from RFID to Barcode:
  1. Call `subscribeRfidHardwareTriggerEvents(false)` to immediately stop handling trigger events in RFID logic.
  2. Call `setTriggerEnabled(false)` to reconfigure the hardware for barcode scanning.
- When switching from Barcode to RFID:
  1. Call `setTriggerEnabled(true)` to reconfigure the hardware for RFID.
  2. On success, `subscribeRfidHardwareTriggerEvents(true)` is called automatically to resume RFID trigger event handling.

### Rationale
- Prevents race conditions and unwanted inventory operations during mode transitions.
- Ensures that trigger events are only handled by the correct subsystem (RFID or Barcode) at the right time.

### Example
```java
public void subscribeRfidHardwareTriggerEvents(boolean enable){
    if(reader != null && reader.isConnected()) {
        Log.v(TAG, "### subscribeRfidTriggerEvents: rfid=" + enable);
        reader.Events.setHandheldEvent(enable);
    }
}
```

---
This method is a key part of the safe trigger mode switch workflow and must be called in the correct sequence to avoid spurious or missed trigger events.

## Failure Handling Contract

- If reader is disconnected/null: return `false`.
- If reader remains busy beyond timeout: return `false` and log timeout.
- If SDK throws (`InvalidUsageException`, `OperationFailureException`):
  return `false` and log error.
- Never leave `resourceLock` locked.

## Relevant Methods

- `waitForReaderIdle()`
- `setTriggerEnabled(boolean isRfidEnabled)`
- `restoreDefaultTriggerConfig()`
- `subsribeRfidTriggerEvents(boolean bRfidHardwareTriggerEvent)`
- `eventStatusNotify(...)` for inventory state transitions

## Verification Checklist

  and return `false`.
  a failure.

- Press trigger repeatedly during mode switch; app should not freeze.
- Force inventory active while switching; function should timeout safely and return `false`.
- Confirm lock release by attempting subsequent switch operations after a failure.
- Confirm RFID trigger events are ignored in barcode mode.
- Confirm trigger events resume in RFID mode after re-subscription.

## Release & History

- **dev2.0.2 (2026-02-18):**
  - Major release for Trigger Sync and deadlock-safe switching.
  - Improved concurrency and lock-guarded trigger switching.
  - Updated all documentation and markdown files for new architecture and release.
  - All code and docs ready for dev2.0.2 tag and push.
- See `history.md` and `README.md` for previous release notes and usage instructions.
