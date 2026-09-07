# BCI Mobile — Build Instructions

This is a real, runnable Flutter project. It builds and runs standalone
right now (in-memory local data, no backend required) so you can see it on
a device today, then swap in the real backend from `bci-backend/` when ready.

## 1. One-time setup (only if you don't have Flutter yet)
1. Install Flutter: https://docs.flutter.dev/get-started/install (pick your OS)
2. Run `flutter doctor` and resolve anything it flags (Android SDK/licenses)
3. Plug in an Android phone (USB debugging on) OR start an emulator

## 2. Get this project running
```bash
cd bci-mobile
flutter pub get
flutter run
```
That installs it straight onto your connected phone/emulator and opens it —
fastest way to see the app today.

## 3. Build an installable APK file
```bash
flutter build apk --release
```
The APK lands at:
```
bci-mobile/build/app/outputs/flutter-apk/app-release.apk
```
Copy that file to your phone (or `adb install app-release.apk`) to install it
without a USB-connected dev session.

## What works right now
- Splash → Login (UI only, not yet calling the real backend) → Dashboard
- Dashboard quick action → New Sale Invoice screen (fully functional UI,
  saves to an in-memory store, queues a sync mutation)
- The Cloud Sync Engine plumbing (`lib/sync/sync_service.dart`) is real and
  wired up, currently pointed at `NoopSyncApiClient` (no server yet)

## Wiring up the real backend (bci-backend/)
Two files to swap, both in `lib/core/local_stubs.dart`:
- `InMemoryInvoiceRepository` → a Drift (SQLite) implementation using the
  schema in `bci-backend/src/database/migrations/001_init_schema.sql`
- `NoopSyncApiClient` → an `http`-based client calling `POST /sync/push`
  and `GET /sync/pull` on your deployed BCI backend (see
  `bci-backend/src/modules/sync/sync.controller.ts` for the exact contract)

Everything else (auth, GST engine, accounting, reports, etc.) is built on
the backend side and ready to connect via the API architecture in
`BCI-01-Architecture-Database-API.md`.

## Renaming the package before a real release
Currently `com.bci.app` (placeholder). Before publishing, pick your real
package id and update it in:
- `android/app/build.gradle` (`applicationId`)
- `android/app/src/main/AndroidManifest.xml` (implicitly via the folder path)
- `android/app/src/main/kotlin/com/bci/app/MainActivity.kt` (`package` line
  and folder structure to match)
