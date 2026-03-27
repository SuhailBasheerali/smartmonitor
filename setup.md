# SmartMonitor Setup & Configuration Guide

This setup covers all components used by this project:
- Flutter app (`/lib`)
- ESP32 firmware (`/espcode/sketch_mar01a/sketch_mar01a.ino`)
- Firebase (Authentication + Realtime Database)
- Architecture-level external services (Supabase scheduled function + AI server contract)

> Note: Supabase budget monitor and AI server are documented system components, but their implementations are not present in this repository.

---

## 1) Prerequisites

### Flutter app
- Install Flutter SDK compatible with `pubspec.yaml`:
  - Dart SDK constraint: `>=3.0.0 <4.0.0`
- Verify installation:
  ```bash
  flutter --version
  ```

### Firebase
- Firebase project with:
  - Authentication enabled
  - Realtime Database enabled
- Platform configuration files:
  - Android: `android/app/google-services.json`
  - iOS/macOS: `GoogleService-Info.plist`

### ESP32 firmware toolchain
- Arduino IDE (or PlatformIO)
- ESP32 board package installed
- Required library:
  - `Firebase_ESP_Client`

---

## 2) Flutter App Setup (`/lib`)

1. From repository root, fetch dependencies:
   ```bash
   flutter pub get
   ```
2. Ensure Firebase config files are in place (see prerequisites).
3. Run the app:
   ```bash
   flutter run
   ```
4. Confirm Firebase initialization path in `lib/main.dart`:
   - Uses platform-default Firebase configuration via `Firebase.initializeApp()`
   - Uses `AuthService` and `FirebaseService` providers

### Flutter configuration notes
- App routes configured in `lib/main.dart`:
  - `/login`
  - `/register`
  - `/forgot-password`
  - `/dashboard`
- Firebase packages are declared in `pubspec.yaml`:
  - `firebase_core`
  - `firebase_auth`
  - `firebase_database`

---

## 3) Firebase Setup & Data Configuration

### Authentication
- Enable Email/Password sign-in method (used by app and firmware credential flow).

### Realtime Database structure
The app and firmware use this hierarchy:
```text
/users/{uid}/devices/{esp_id}/appliances/{appliance_id}/
  config/
  live/
  stats/
  history/
    daily/
    monthly/
```

### Appliance node conventions
- `config`: `voltage`, `calibration`, `peakCurrent`, `gpioPin`, `enabled`
- `live`: `current`, `power`, `timestamp`, `peak`
- `stats`: `todayEnergy`, `monthEnergy`, `lastCalcTime`, reset metadata
- `history.daily` and `history.monthly`: archived energy values

---

## 4) ESP32 Firmware Setup (`/espcode/sketch_mar01a/sketch_mar01a.ino`)

1. Open `espcode/sketch_mar01a/sketch_mar01a.ino` in Arduino IDE.
2. Install/select ESP32 board.
3. Configure firmware constants before flashing:
   - Wi-Fi:
     - `WIFI_SSID`
     - `WIFI_PASSWORD`
   - Firebase:
     - `API_KEY`
     - `DATABASE_URL`
   - User auth:
     - `USER_EMAIL`
     - `USER_PASSWORD`
   - Device:
     - `ESP_ID`
   - Runtime mode:
     - `TEST_MODE`
   - Time/NTP:
     - `NTP_SERVER`
     - `GMT_OFFSET_SEC`
     - `DST_OFFSET_SEC`
   - Sensor settings:
     - `ADC_REF`
     - `ADC_RESOLUTION`
     - `SCT_CALIBRATION`
     - `NO_SIGNAL_THRESHOLD`
     - `SCT_SAMPLES`
4. Flash firmware to the ESP32.
5. Validate serial logs:
   - Wi-Fi connection status
   - NTP sync success
   - Firebase authentication and base path readiness

### Important firmware behavior
- Update cycle interval: 5 seconds
- On each cycle:
  - reads appliance configuration
  - measures current / computes power
  - updates live + stats
  - performs daily/monthly reset archival

---

## 5) External Architecture Components (Not Implemented Here)

### A) Supabase budget monitor (external)
- Scheduled workflow design:
  - checks monthly usage against budget thresholds
  - can trigger notification flow (for example at 80%)
- This repository does not include Supabase function code/deployment artifacts.

### B) AI assistant server (external)
- Expected primary endpoint:
  - `POST /ask-ai`
- Contract and workflow are defined in:
  - `AI_WORKFLOW.md`
- This repository does not include AI server implementation or app client integration for this endpoint.

---

## 6) Verification Checklist

- Flutter:
  - `flutter pub get` succeeds
  - `flutter run` launches and shows auth/dashboard flow
- Firebase:
  - authentication works from app
  - appliance nodes appear under expected `/users/{uid}/devices/{esp_id}/appliances` path
- ESP32:
  - device connects to Wi-Fi
  - NTP sync completes
  - live/stats values are written in Firebase

---

## 7) Troubleshooting

- **`flutter: command not found`**
  - Install Flutter SDK and add it to `PATH`.
- **Firebase init/runtime issues**
  - Verify `google-services.json` / `GoogleService-Info.plist` placement.
  - Confirm project API keys and database URL are correct.
- **ESP32 upload failures**
  - Recheck selected board/port and USB drivers.
- **ESP32 Firebase auth failures**
  - Verify `API_KEY`, `DATABASE_URL`, `USER_EMAIL`, and `USER_PASSWORD`.
