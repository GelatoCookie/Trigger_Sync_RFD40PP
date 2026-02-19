Design Document: RFID Trigger Configuration & Event Subscription
Version: dev2.0.2
Release Date: 2026-02-18

1. Executive Summary
The RFIDHandler class manages the Zebra RFID reader's hardware trigger, which serves two distinct functions depending on the application state:

RFID Mode: The trigger starts and stops RFID tag inventory.

Barcode (Sled Scan) Mode: The trigger activates the barcode scanner.

To support these modes safely, the application dynamically reconfigures the hardware key layout and manages software event subscriptions.

2. Architecture & Synchronization
Reconfiguring the hardware trigger requires sending commands directly to the reader. If the reader is performing an inventory, or if multiple threads attempt simultaneous configuration, the SDK may throw exceptions or enter an undefined state.

To ensure thread safety and hardware stability, the architecture utilizes a lock and busy-guard strategy:

resourceLock (ReentrantLock): Ensures mutual exclusion so only one configuration operation proceeds at a time.

bRfidBusy (volatile boolean): Acts as a guard that blocks configuration commands until the reader is idle.

waitForReaderIdle(): A bounded helper used by default restore operations to prevent indefinite waiting.

3. State Management Variables
The bRfidBusy Guard

This volatile boolean flag indicates whether the RFID reader is currently performing an inventory or other operations.

Lifecycle Activation: Set to true upon receiving an INVENTORY_START_EVENT.

Lifecycle Deactivation: Set to false upon receiving an INVENTORY_STOP_EVENT.

Configuration Guarding: The code checks this flag before any trigger mode change; if true, the operation is rejected, inventory is stopped, and the UI prompts a retry.

State Matrix for bRfidBusy

Event	bRfidBusy Value	Allowed Operations
Inventory started	true	No config changes, no new inventory
Inventory stopped	false	Config changes allowed
During config attempt	Checked	Blocked if true
The bSwitchFromRfidToBarcode Transition Flag

This volatile boolean flag controls the transition of trigger event handling, preventing RFID trigger events from being processed while switching to barcode scanning.

Activation: Set to true in test mode after an inventory stop event, signaling that subsequent trigger events should be handled by the barcode logic.

Deactivation: Reset to false after successfully configuring the hardware back to RFID mode.

Execution: The RFID trigger event handler ignores all hardware events while this flag is true.

4. Core Configuration Methods
setTriggerEnabled(boolean isRfidEnabled)

This method physically reconfigures the reader's hardware trigger behavior.

Acquires resourceLock to block other attempts.

Checks bRfidBusy; rejects the operation if the reader is active.

Determines the mode: ENUM_NEW_KEYLAYOUT_TYPE.RFID or ENUM_NEW_KEYLAYOUT_TYPE.SLED_SCAN.

Applies configuration via reader.Config.setKeylayoutType(mode, mode).

Calls subscribeRfidHardwareTriggerEvents on success to align software events with the hardware mode.

Releases the lock in a finally block.

subscribeRfidHardwareTriggerEvents(boolean enable)

This method controls whether the application receives HANDHELD_TRIGGER_EVENT notifications from the SDK.

Enabled (true): The app receives callbacks to programmatically start or stop RFID inventory.

Disabled (false): The app ignores events, preventing the RFID logic from reacting to scanner presses during barcode mode.

5. Integration Workflow
Switching to Barcode Mode

Unsubscribe Events: Call subscribeRfidHardwareTriggerEvents(false) to immediately stop receiving pressed events.

Reconfigure Hardware: Call setTriggerEnabled(false) to map the trigger to the scanner button.

Switching to RFID Mode

Reconfigure Hardware: Call setTriggerEnabled(true) to map the trigger to RFID functionality.

Subscribe Events: The setTriggerEnabled method automatically calls subscribeRfidHardwareTriggerEvents(true) upon success.

6. Deadlock-Avoidance Contract & Safety
To keep trigger mode changes safe under concurrent events, the implementation strictly adheres to the following rules:

Mechanism / Rule	Description & Purpose
No Indefinite Blocking	waitForReaderIdle() uses bounded retries and fails fast with a TimeoutException (approx. 3 seconds).
Volatile State	bRfidBusy is updated directly by status events without requiring event thread locks.
Guaranteed Unlocking	Operations inside setTriggerEnabled utilize a try-finally block to ensure locks drop even during SDK exceptions.
Strict Switch Order	Subscriptions drop before hardware switches to Barcode, and activate after hardware switches to RFID.