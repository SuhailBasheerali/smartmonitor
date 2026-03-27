# SmartMonitor Setup Instructions

## Flutter App Setup
1. **Install Flutter:** Download and install Flutter from the official site: [Flutter Installation](https://flutter.dev/docs/get-started/install).
2. **Set up Environment Variables:** Add Flutter to your system PATH.
3. **Create a new Flutter project:** Run `flutter create smartmonitor`.
4. **Run the app:** Use `flutter run` command to start the Flutter app.

## ESP32 Firmware Setup
1. **Install Arduino IDE:** Download from [Arduino](https://www.arduino.cc/en/software).
2. **ESP32 Board Manager:** In Arduino IDE, go to Preferences and add the ESP32 URL to the Board Manager.
3. **Select ESP32 Board:** Choose your ESP32 board under Tools > Board.
4. **Upload Firmware:** Open the firmware project and upload it to the ESP32 using the `Upload` button.

## Firebase Configuration
1. **Create a Firebase Project:** Go to [Firebase Console](https://console.firebase.google.com/) and create a new project.
2. **Add iOS/Android App:** Follow the steps in the console to register your Flutter app.
3. **Download Configuration Files:** Download the `google-services.json` for Android and `GoogleService-Info.plist` for iOS and place them in the respective directories.
4. **Initialize Firebase in Flutter:** Add Firebase dependencies to `pubspec.yaml` and initialize Firebase in your app.

## Troubleshooting
- **Flutter Issues:** Check the output log for any errors and ensure you have the correct Flutter and Dart SDK versions.
- **ESP32 Upload Errors:** Ensure the correct board is selected and drivers are properly installed.
- **Firebase Setup Issues:** Verify your configuration files are placed correctly and app permissions are set up in the Firebase console.
