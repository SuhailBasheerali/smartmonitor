# Setup Guide for SmartMonitor

## Repository Structure
- **/flutter_app**: Contains the Flutter application.
- **/esp32_code**: Contains the firmware code for the ESP32.
- **/node_backend**: Contains the Node.js backend code.

## Installation Instructions

### Windows
1. **Install Flutter:** Download from the [Flutter website](https://flutter.dev/docs/get-started/install/windows).
2. **Install Node.js:** Download from the [Node.js website](https://nodejs.org/).
3. **Install ESP32 SDK:** Follow the instructions [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html).

### macOS
1. **Install Flutter:** Download from the [Flutter website](https://flutter.dev/docs/get-started/install/macos).
2. **Install Node.js:** Download from the [Node.js website](https://nodejs.org/).
3. **Install ESP32 SDK:** Follow the instructions [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html).

### Linux
1. **Install Flutter:** Follow the instructions [here](https://flutter.dev/docs/get-started/install/linux).
2. **Install Node.js:** Use your package manager (e.g., `sudo apt install nodejs npm`).
3. **Install ESP32 SDK:** Follow the instructions [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html).

## Quick Start Guides

### Flutter
- Run `flutter pub get` to install dependencies.
- Use `flutter run` to start the application.

### ESP32
- Flash the ESP32 with the firmware using the ESP-IDF tools.

### Node.js
- Run `npm install` to install dependencies.
- Start the server using `node server.js`.

## Testing Procedures

### Unit Tests
- Navigate to each respective directory and run `flutter test`, `npm test`, or the appropriate command for the ESP32.

### Widget Tests
- Use `flutter test` to run widget tests in the Flutter app.

### Integration Tests
- Run integration tests using a testing framework suitable for your chosen platform.

## Configuration Details

### Firebase
- Set up Firebase using [Firebase Console](https://console.firebase.google.com/).

### Flutter
- Update `firebase_options.dart` with your Firebase project settings.

### ESP32
- Configure Wi-Fi and Firebase credentials in the code.

### Node.js
- Use environment variables to configure Firebase and other services.

### Supabase
- Configure Supabase connection details in your codebase.

## API Documentation
- Available in the `/docs` folder.

## Security Considerations
- Always validate inputs on the server-side.
- Use environment variables to keep sensitive data out of the codebase.

## Troubleshooting Guide
- If you encounter issues, please check:
  - Console logs for errors.
  - GitHub Issues for similar problems.
  - Verify your configurations for Firebase and Supabase.

For further information, refer to the respective documentation of Flutter, ESP32, Node.js, and Supabase.