# CompassLink — Live Android Orientation

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-2.0.21-7F52FF?logo=kotlin&logoColor=white)
![Android](https://img.shields.io/badge/Android-8%2B-3DDC84?logo=android&logoColor=white)
![Connection](https://img.shields.io/badge/Connection-USB%20%7C%20Wireless-009688)

**CompassLink** is an Android-to-PC **live orientation dashboard**. Built with a **Kotlin + SensorManager** companion and a **Node.js + HTML Canvas** dashboard, it streams fused sensor quaternions through an **ADB reverse tunnel** over **USB debugging or wireless debugging**, turning your phone's movement into a live 3D handset and telemetry display.

The companion reads the sensors; ADB provides the transport. USB streaming needs no Wi-Fi, while wireless debugging uses the same Wi-Fi network as your PC. Neither mode needs internet during streaming. Initial tool installation and build dependency downloads require internet access.

## Demo

<p align = "center">
<img width="480" height="270" alt="demo" src="https://github.com/user-attachments/assets/efe669a1-41f9-46d1-bd96-b5834cdc80b9" />
</p>

## Key Features

- **Two Connection Options:** Connect by USB cable or Android 11+ wireless debugging using dedicated npm commands.
- **Live 3D Orientation:** Quaternion-driven handset rendering with drag-to-orbit controls and double-click camera reset.
- **Sensor Telemetry:** Heading, pitch, roll, quaternion components, sample count, sensor accuracy and sample age.
- **Connection-Aware Dashboard:** USB/wireless setup guides, active stream mode, waiting state and stale-data indicators.
- **Fast Reconnect:** Reuse an installed companion; explicitly reinstall after rebuilding the Android app.
- **Resilient Streaming:** Latest-sample queue, automatic socket reconnect attempts and cleanup when the app leaves the foreground.
- **Local Operation:** Local browser assets, Node built-ins and loopback-only HTTP/TCP receivers.

## Tech Stack

| Category | Technology | Badge & Version | Purpose |
| :--- | :--- | :--- | :--- |
| **Frontend** | JavaScript, HTML5 Canvas | ![JavaScript](https://img.shields.io/badge/JavaScript-ES_Modules-F7DF1E) | Handset projection, camera controls and telemetry |
| **Styling** | CSS | ![CSS](https://img.shields.io/badge/CSS-Responsive-663399) | Dark dashboard, cards and connection controls |
| **Backend** | Node.js built-ins | ![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933) | HTTP server, TCP receiver and state API |
| **Android App** | Kotlin, SensorManager | ![Kotlin](https://img.shields.io/badge/Kotlin-2.0.21-7F52FF) | Fused orientation, phone UI and lifecycle |
| **Transport** | Android Debug Bridge | ![ADB](https://img.shields.io/badge/ADB-USB%20%2F%20Wireless-009688) | Pairing, connection and reverse tunnel |
| **Android Build** | Android Gradle Plugin, Gradle | ![AGP](https://img.shields.io/badge/AGP-8.7.3-3DDC84) ![Gradle](https://img.shields.io/badge/Gradle-8.9-02303A) | APK packaging and reproducible builds |
| **Java Runtime** | Microsoft OpenJDK | ![JDK](https://img.shields.io/badge/JDK-17-orange) | Local Android build runtime |
| **Tooling** | PowerShell, Prettier, Node test runner | ![Prettier](https://img.shields.io/badge/Prettier-3.6.2-F7B93E) | Setup/build helpers, formatting and existing tests |

## Getting Started

### Prerequisites

- **Windows PowerShell**, **Node.js 18+** and npm.
- A physical Android phone with a rotation-vector or game-rotation-vector sensor.
- **USB:** Android 8+ and a USB data cable.
- **Wireless debugging:** Android 11+ and PC/phone on the same Wi-Fi.
- Internet for initial tool installation and Android build dependencies.

### 1. Install Dependencies and Build the Companion

Open PowerShell in the project folder:

```powershell
npm install
npm run setup:android
npm run build:android
```

Setup installs Microsoft OpenJDK 17, Gradle 8.9, Android command-line tools, platform-tools (ADB), Android platform 35 and Build Tools 34.0.0 into `.tools/`. It accepts the Android SDK tool licenses as part of installation. It does not replace your system Java. Build scripts select the local JDK and SDK for their own process. The Android plugins are pinned to AGP 8.7.3 and Kotlin 2.0.21. Android requires version 8.0/API 26 or newer and a rotation-vector sensor.

### Option 1: Connect over USB

1. Enable Developer options (tap Build number seven times), then enable **USB debugging**.
2. Connect exactly one phone using a **USB data cable**, unlock it and accept the debugging authorization prompt.
3. Connect and install the updated companion:

```powershell
npm run connect:usb -- -Reinstall
```

For subsequent connections, use `npm run connect:usb`. The existing `npm run connect` command also selects USB mode. The helper selects the USB device with `adb -d`, creates the reverse tunnel and opens the companion.

### Option 2: Connect with Wireless Debugging

1. Connect the PC and phone to the **same Wi-Fi**.
2. On Android 11+, enable **Developer options > Wireless debugging**.
3. Choose **Pair device with pairing code** and run:

```powershell
npm run connect:wireless -- -Pair -Reinstall
```

Enter the **pairing IP address and port**, then the pairing code when ADB prompts. Return to the main Wireless debugging page and enter its **connection IP address and port** when prompted. **The pairing and connection ports are different.** No USB cable is required.

For subsequent connections:

```powershell
npm run connect:wireless
```

Or provide the current connection address directly:

```powershell
npm run connect:wireless -- -Address 192.168.1.20:37123
```

Replace the example with the address shown on your phone. Debugging ports may change after restarting or toggling Wireless debugging. Pair again if the PC is no longer paired.

### 2. Start the Dashboard and Stream

```powershell
npm start
```

Open **[http://127.0.0.1:8000](http://127.0.0.1:8000)** and tap **Start streaming** on the phone. Keep the app in the foreground; it keeps the screen awake during streaming. Rotate the handset through all axes. Drag the PC view to orbit the camera and double-click to reset it.

The dashboard's USB/wireless buttons show setup instructions. Run the connection commands in your project terminal to connect. The active mode comes from the latest fresh sensor sample; older APKs or manual launches without mode metadata show a generic ADB stream.

### Updating an Installed Companion

Normal connection commands skip installation when the companion is already installed. After changing Android code, rebuild and explicitly reinstall once:

```powershell
npm run build:android
npm run connect:usb -- -Reinstall
# Or:
npm run connect:wireless -- -Reinstall
```

The debug APK is at `android/app/build/outputs/apk/debug/app-debug.apk`. To stop, tap **Stop streaming** and press **Ctrl+C** in the server terminal. The Android app also stops when it leaves the foreground. Repeat your chosen connection command if the tunnel is lost.

## Manual ADB commands

The npm helpers use the project's local ADB, so it does not need to be on PATH. For manual commands, select the executable in PowerShell:

```powershell
$adb = Join-Path (Get-Location) '.tools/android-sdk/platform-tools/adb.exe'
& $adb devices -l
```

### USB

```powershell
& $adb -d install -r android/app/build/outputs/apk/debug/app-debug.apk
& $adb -d reverse tcp:8765 tcp:8765
& $adb -d shell am start -n com.compasslink3d/.MainActivity --es connection_mode usb
& $adb -d reverse --list
# Optional tunnel removal after stopping:
& $adb -d reverse --remove tcp:8765
```

### Wireless

Replace both example ports with those displayed on your phone:

```powershell
& $adb pair 192.168.1.20:40001
& $adb connect 192.168.1.20:37123
& $adb -s 192.168.1.20:37123 install -r android/app/build/outputs/apk/debug/app-debug.apk
& $adb -s 192.168.1.20:37123 reverse tcp:8765 tcp:8765
& $adb -s 192.168.1.20:37123 shell am start -n com.compasslink3d/.MainActivity --es connection_mode wireless
& $adb -s 192.168.1.20:37123 reverse --list
```

After stopping, remove the wireless tunnel with `& $adb -s IP:PORT reverse --remove tcp:8765` and disconnect with `& $adb disconnect IP:PORT`.

`-d` selects USB; `-s IP:PORT` selects the wireless connection. Both modes forward phone `127.0.0.1:8765` to PC loopback port 8765. The dashboard binds only `127.0.0.1:8000`; no dashboard or sensor listener is opened to the LAN. There are no remote JavaScript, font or CDN dependencies. Android's `INTERNET` permission is required for the companion's TCP sockets.

## Android Studio

After setup, open `android/` in Android Studio. Set the Gradle JDK to the project's `.tools/jdk` (JDK 17). `android/local.properties` points to the project SDK and is generated by setup. The project includes a Gradle wrapper so the build version is reproducible. Select a connected phone and run `app`. Create the reverse tunnel before tapping Start streaming. The command-line build does not require Android Studio.

## Project Structure

| Location                                                            | Purpose                                                                           |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| `android/app/src/main/java/com/compasslink3d/MainActivity.kt`       | Sensor registration, sample serialization, foreground lifecycle and phone UI      |
| `android/app/src/main/java/com/compasslink3d/UsbSensorTransport.kt` | Disposable connection sessions, reconnect, latest-sample queue and socket cleanup |
| `pc/server.js`                                                      | Loopback TCP receiver, bounded framing, validation, HTTP assets and state API     |
| `pc/dashboard.js`                                                   | Canvas phone renderer, camera controls, sequential polling and stale status       |
| `pc/orientation.js`                                                 | Quaternion math, coordinate mapping and Android orientation angles                |
| `pc/styles.css`                                                     | Responsive dashboard styling                                                      |
| `pc/index.html`                                                     | Dashboard markup, telemetry cards and USB/wireless setup guides                  |
| `scripts/connect-phone.ps1`                                         | USB selection, wireless pairing/connect, optional install and tunnel setup        |
| `scripts/`                                                          | Local tool setup, APK build, downloads and cleanup helpers                        |
| `tests/`                                                            | Packet validation, TCP/API integration and orientation regressions                |

## Streaming Behavior Summary

| Setting | Behavior |
| :--- | :--- |
| **Sensor sampling** | Requested at 50 Hz |
| **Transmission** | Latest available sample sent at up to 50 Hz |
| **Backpressure** | Superseded samples dropped rather than queued |
| **Dashboard polling** | Sequential requests approximately every 50 ms |
| **Stale threshold** | Samples older than two seconds |
| **Socket inactivity** | Connection closed after ten seconds without data |

These are implementation settings, not measured performance guarantees. Sensor delivery, wireless conditions and browser scheduling may reduce the observed rate. The PC runtime uses Node built-ins; Prettier is the only npm development dependency.

## API Endpoints

| Method | Endpoint | Description |
| :---: | :--- | :--- |
| `GET` | `/` | Dashboard with orientation, telemetry and connection guides |
| `GET` | `/api/state` | Latest sample, receipt time, count, age, connected and stale flags; `last.connectionMode` reports `usb`, `wireless` or `unknown` |

Sensor data uses newline-delimited JSON on TCP port **8765**, through the selected ADB reverse tunnel.

## Coordinates and quaternion protocol

Android device axes are X right, Y toward the top edge, and Z outward from the screen. These axes follow the physical device's natural orientation, not its rotated UI. The handset mesh uses those same device coordinates. Android's rotation quaternion maps device vectors into East/North/Up world coordinates. The renderer first applies that quaternion, then maps world `(East, North, Up)` into view `(East, Up, -North)`, followed by an independent orbit camera. It does not accumulate Euler angles, so crossing 360 degrees does not cause pose discontinuities. `q` and `-q` render identically.

Heading, pitch and roll follow `SensorManager.getOrientation()` conventions; the heading is clockwise from magnetic north when the top edge projects onto the horizontal plane. Near vertical, heading is undefined and Euler angles have singularities; the 3D quaternion pose remains usable. Rotation-vector fusion and calibration depend on the phone. Nearby magnets and metal can affect heading. If `TYPE_ROTATION_VECTOR` is unavailable, the companion falls back to `TYPE_GAME_ROTATION_VECTOR`; its heading is relative and can drift because it does not use magnetic north.

The stream contains one UTF-8 JSON object per newline:

```json
{
  "t": 1730000000000,
  "timestampNs": "123456789000",
  "sensor": "android.sensor.rotation_vector",
  "accuracy": 3,
  "q": [0, 0, 0, 1],
  "connectionMode": "wireless"
}
```

`q` uses `[x,y,z,w]`, converted from Android's `[w,x,y,z]` output. The server validates finite components and normalizes them. `t` is phone wall-clock milliseconds; `timestampNs` is the monotonic sensor event timestamp, serialized as a string to preserve precision. The dashboard age uses PC receipt time and is not a measurement of transport latency. Accuracy codes are Android's -1/0/1/2/3 (no contact, unreliable, low, medium, high); availability varies by sensor. This version displays fused orientation, not separate raw accelerometer/gyro/magnetometer streams.

## Running Tests and Maintenance

To remove installer ZIPs, extraction leftovers, interrupted downloads and generated build caches, run `npm run clean` with builds stopped. It keeps the source, installed tools, downloaded build dependencies and debug APK. The next Android build will take longer while it regenerates its caches. The dashboard can still be started normally with `npm start`.

```powershell
npm test
npm run build:android
npx prettier --check package.json pc tests
```

The existing tests cover quaternion normalization and invalid data rejection, fragmented TCP packets reaching the HTTP API, connection shutdown, asset serving, cardinal headings and coordinate mapping. They do not verify physical wireless pairing or hardware performance. Hardware verification requires a real phone: stream, rotate each axis, stop/restart, disconnect/reconnect using either mode, and verify relative heading with the game sensor fallback. No phone measurements are simulated in the normal dashboard.

## Troubleshooting

- **Unauthorized/offline phone:** unlock it and accept the authorization prompt. For USB, check the cable; for wireless, check the current debugging address and Wi-Fi connection. Rerun the connection command for your mode.
- **Waiting for phone:** confirm `npm start` is running, rerun the connection command for your mode, and tap **Start streaming**. Inspect the reverse tunnel using the USB or wireless manual commands above.
- **No supported sensor:** this phone exposes neither rotation-vector sensor. An emulator may lack realistic fused orientation.
- **Port already in use:** stop the other server using port 8000 or 8765 and restart CompassLink3D.
- **Heading moves unexpectedly:** move away from metal/magnets and follow the phone manufacturer's calibration guidance. Game sensor heading is relative.
- **Build download fails:** check internet access and retry setup/build. The `.tools` downloads and Gradle caches are local to this project. Offline streaming does not need access to those download servers.
- **Wireless pairing fails:** open a fresh pairing-code dialog and use its pairing port. Use the separate connection port from the main Wireless debugging page when prompted to connect.
- **Wireless connection fails:** check both devices are on the same Wi-Fi and that the network allows communication between clients. Restart Wireless debugging and use its current address and port.
- **Old app behavior after rebuilding:** connect with `-Reinstall` to install the newly built APK.
- **ADB is not recognized:** use the local executable shown in Manual ADB Commands or use the npm connection helpers.

## Limitations

V1 displays fused orientation and a 3D handset representation. It does not mirror the phone's screen or show separate raw accelerometer, gyroscope and magnetometer streams. Sensor accuracy depends on hardware and calibration. Wireless debugging requires Android 11+ and a compatible local network.

## References

- [Android Debug Bridge and wireless debugging](https://developer.android.com/tools/adb)

- [Android rotation vector sensors and device coordinates](https://developer.android.com/develop/sensors-and-location/sensors/sensors_motion)
- [Android orientation and game rotation sensor](https://developer.android.com/develop/sensors-and-location/sensors/sensors_position)
- [Android Gradle Plugin 8.7 compatibility: Gradle 8.9 and JDK 17](https://developer.android.com/build/releases/agp-8-7-0-release-notes)

