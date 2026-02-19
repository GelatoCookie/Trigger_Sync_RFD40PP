# Trigger Sync Doc: bRfidBusy and Deadlock-Safe Trigger Switching

---
**Current Git Info**
- Tag: v2.1.0
- Version: v2.1.0-1-gfaf5192
- Branch: main
- Local directory: /Users/chucklin/myprojects/MS_Code/Trigger_Sync_RFD40PP
- Remote: https://github.com/GelatoCookie/Trigger_Sync_RFD40PP.git
---

## Purpose
This document defines the runtime contract for safely switching the RFD40 trigger between RFID inventory and barcode scanning modes.

## Core Problem
The trigger is shared by two subsystems:
- RFID inventory (`reader.Actions.Inventory.perform/stop`)
- Barcode scanner pull trigger (`DCSSDK_DEVICE_PULL_TRIGGER`)

If the trigger mode is changed while RFID is still active, SDK calls can fail or become unstable.

## Concurrency Model
- Single worker thread: `ExecutorService executor = Executors.newSingleThreadExecutor()` for serialized operational tasks.
- Lock for trigger config: `ReentrantLock resourceLock` wraps trigger configuration methods.
- Busy state signal: `volatile boolean bRfidBusy` tracks RFID operation state from status events.

## Busy-State Lifecycle
- `bRfidBusy = true` on `INVENTORY_START_EVENT`
- `bRfidBusy = false` on `INVENTORY_STOP_EVENT`

This state is used as a guard before any trigger-layout switch.

## Deadlock-Safe Rules
1. `setTriggerEnabled(...)` and `restoreDefaultTriggerConfig()` must always acquire `resourceLock`.
2. Any lock acquisition path must release lock in `finally`.
3. Waiting for idle must be bounded:
   - `waitForReaderIdle()` retries 10 times with 200ms sleep (max 2s).
   - Throws `TimeoutException` on timeout.
4. No blocking dependency on event callbacks while lock is held.
5. On timeout/failure, return `false` and keep system responsive.

## Safe Switch Sequences
### RFID → Barcode
1. `subsribeRfidTriggerEvents(false)`
2. `setTriggerEnabled(false)`

Why: Prevent RFID trigger callback handling while hardware keylayout is changing.

### Barcode → RFID
1. `setTriggerEnabled(true)`
2. `subsribeRfidTriggerEvents(true)`

Why: Ensure hardware behavior is ready before consuming RFID handheld trigger events.

## Failure Handling Contract
- If reader is disconnected/null: return `false`.
- If reader remains busy beyond timeout: return `false` and log timeout.
- If SDK throws (`InvalidUsageException`, `OperationFailureException`): return `false` and log error.
- Never leave `resourceLock` locked.

## Relevant Methods
- `waitForReaderIdle()`
- `setTriggerEnabled(boolean isRfidEnabled)`
- `restoreDefaultTriggerConfig()`
- `subsribeRfidTriggerEvents(boolean bRfidHardwareTriggerEvent)`
- `eventStatusNotify(...)` for inventory state transitions

## Verification Checklist
- Press trigger repeatedly during mode switch; app should not freeze.
- Force inventory active while switching; function should timeout safely and return `false`.
- Confirm lock release by attempting subsequent switch operations after a failure.
- Confirm RFID trigger events are ignored in barcode mode.
- Confirm trigger events resume in RFID mode after re-subscription.
