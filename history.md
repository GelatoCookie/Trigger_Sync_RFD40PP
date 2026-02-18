# History

## 2026-02-17 - Version 1.1.3

- Completed code cleanup across `MainActivity`, `RFIDHandler`, and
  `ScannerHandler`.
- Removed unused `MainUIHandler` and other stale placeholder code.
- Updated documentation (`README.md`, `design.md`,
  `DesignDoc_Trigger_Event_Handling.md`, `RELEASE_NOTES.md`).
- Successfully built and deployed app to device `59040DLCH003LK`.

## 2026-02-16 - Version 1.1.2

- Updated strings and minor UI text improvements.
- Merged latest remote changes and resolved conflicts.
- Updated release documentation for v1.1.2.

## 2026-02-10

- Build and run process completed successfully on macOS.
- Fixed Android SDK location error by creating local.properties.
- Removed android:onClick from XML to resolve lint errors.
- Increased minSdkVersion to 28 to resolve API compatibility issues.
- Application installed and launched on device using
  build_deploy_launch.sh.

## 2026-02-10 (post-release)

- Major code cleanup and refactoring for maintainability and style
  compliance.
- Reduced method complexity and improved naming conventions in all main
  Java classes.
- Removed unused fields, improved exception handling, and modernized
  code style.
- Ready for commit and further development.

## 2026-02-18 - Version 2.0.2 (dev2 branch)
- Major release for Trigger Sync and deadlock-safe switching (see Trigger_Sync_Doc.md).
- Improved concurrency and lock-guarded trigger switching.
- Updated all documentation and markdown files for new architecture and release.
- All code and docs ready for dev2.0.2 tag and push.
