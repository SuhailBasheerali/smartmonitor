# SmartMonitor

SmartMonitor is an IoT-based energy monitoring system with:
- **ESP32 firmware** for acquisition and sync
- **Flutter mobile/web/desktop app** for authentication, monitoring, and appliance management
- **Firebase Realtime Database** as the real-time data backbone

> Note: The architecture below also includes a **Supabase budget-monitoring Edge Function** and an **AI Assistant server** (`/ask-ai` flow) as system components. Those backend services are part of the overall workflow design but are **not implemented in this repository**.

---

## Components

### 1) ESP32 Firmware (`/espcode/sketch_mar01a/sketch_mar01a.ino`)
- Connects to Wi-Fi and syncs time via NTP.
- Authenticates with Firebase and builds base path:
  - `/users/{uid}/devices/{esp_id}/appliances`
- Runs a periodic cycle (5s):
  - Reads appliance config/stats.
  - Measures current (or generates test readings in `TEST_MODE`).
  - Computes power and energy.
  - Handles daily/monthly reset + history archival.
  - Writes live and stats back to Firebase.

### 2) Flutter App (`/lib`)
- Firebase auth flow (`AuthService`) and guarded app entry.
- Dashboard for real-time appliance monitoring.
- Add/edit/delete appliance configuration.
- Appliance history view (daily/monthly energy history).
- Firebase stream-based UI updates via `FirebaseService`.

### 3) Firebase Realtime Database
Primary data hierarchy used by firmware and app:
```text
/users/{uid}/devices/{esp_id}/appliances/{appliance_id}/
  config/
  live/
  stats/
  history/
    daily/
    monthly/
```

### 4) External System Components (architecture-level)
- **Supabase Edge Function** (scheduled): budget threshold checks (e.g., 80%) + email alerts.
- **AI Assistant server** (Node.js/Express): natural-language command/advice endpoint(s), LLM integration, conversation memory, and interaction logs.

---

## System Design & Architecture

### High-level architecture
1. ESP32 uploads live and aggregated energy data to Firebase.
2. Flutter app subscribes to Firebase data for live visualization and user actions.
3. Supabase scheduled workflow evaluates monthly usage vs budget and triggers notifications.
4. AI assistant server receives app chat requests, optionally executes appliance/device actions, and returns natural-language responses.

### End-to-end operational flow
1. User authenticates and configures device/appliances via app.
2. ESP32 collects current, calculates energy, and updates Firebase every 5 seconds.
3. App renders live values and historical energy history.
4. AI chat (external server) processes user requests and replies/executes commands.
5. Budget monitor (external scheduled function) sends threshold alerts.

---

## Repository Structure

```text
smartmonitor/
├── espcode/
│   └── sketch_mar01a/sketch_mar01a.ino      # ESP32 firmware
├── lib/                                      # Flutter application code
│   ├── main.dart
│   ├── models/
│   ├── screens/
│   │   ├── auth/
│   │   ├── dashboard/
│   │   └── appliance/
│   ├── services/
│   └── theme/
├── test/
│   └── widget_test.dart
├── android/ ios/ web/ linux/ macos/ windows/ # Flutter platform targets
├── pubspec.yaml
└── README.md
```

---

## Setup

## Prerequisites
- Flutter SDK (matching Dart constraint in `pubspec.yaml`: `>=3.0.0 <4.0.0`)
- Firebase project with:
  - Authentication enabled
  - Realtime Database configured
- Platform Firebase config files:
  - Android: `android/app/google-services.json`
  - iOS/macOS: `GoogleService-Info.plist` (not currently committed)
- ESP32 toolchain (Arduino IDE or PlatformIO) with required libraries:
  - `Firebase_ESP_Client`
  - ESP32 Wi-Fi/time support

## Flutter app setup
1. From repo root:
   ```bash
   flutter pub get
   ```
2. Run on target platform:
   ```bash
   flutter run
   ```
3. Verify Firebase is initialized in `lib/main.dart`.

## ESP32 firmware setup
1. Open `espcode/sketch_mar01a/sketch_mar01a.ino`.
2. Configure Wi-Fi/Firebase/auth/device constants.
3. Flash firmware to ESP32.
4. Observe serial logs for Wi-Fi, NTP sync, and Firebase auth status.

---

## Configuration Reference

### Firmware configuration (compile-time constants)
In `sketch_mar01a.ino`:
- Wi-Fi: `WIFI_SSID`, `WIFI_PASSWORD`
- Firebase: `API_KEY`, `DATABASE_URL`
- User auth: `USER_EMAIL`, `USER_PASSWORD`
- Device identity: `ESP_ID`
- Runtime mode: `TEST_MODE`
- NTP: `NTP_SERVER`, `GMT_OFFSET_SEC`, `DST_OFFSET_SEC`
- Sensor settings: `ADC_REF`, `ADC_RESOLUTION`, `SCT_CALIBRATION`, etc.

### Appliance data model
App and firmware share nested appliance nodes:
- `config`: voltage, calibration, peakCurrent, gpioPin, enabled
- `live`: current, power, timestamp
- `stats`: todayEnergy, monthEnergy, lastCalcTime, reset metadata
- `history.daily` and `history.monthly`: archived energy values

---

## API Documentation

This repo contains no standalone REST backend implementation. APIs are:

### A) Firebase Realtime Database contract
- **Read/subscribe**:
  - `/users/{uid}/devices` (device + appliance hierarchy)
  - `/users/{uid}/devices/{deviceId}/appliances/{applianceId}/history/daily`
  - `/users/{uid}/devices/{deviceId}/appliances/{applianceId}/history/monthly`
- **Write/update**:
  - Add appliance under:
    - `/users/{uid}/devices/{deviceId}/appliances/{applianceName}`
  - Update appliance config under:
    - `/users/{uid}/devices/{deviceId}/appliances/{applianceId}/config`
  - Delete appliance node.

### B) AI Assistant service contract (external)
Expected endpoint from architecture:
- `POST /ask-ai`
  - Request: `{ "userId": "...", "message": "..." }`
  - Response: natural-language answer and/or command execution confirmation.

Expected auxiliary endpoints:
- memory stats
- clear conversation history
- health check

See **`AI_WORKFLOW.md`** for detailed AI workflow and sequence.

---

## AI Workflow Documentation

A dedicated AI workflow document is available at:
- **`/AI_WORKFLOW.md`**

It describes:
- app ↔ AI server request/response lifecycle
- command execution path
- LLM response generation path
- memory lifecycle and observability
- security and guardrails
