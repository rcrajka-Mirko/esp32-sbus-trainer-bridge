# Changelog – Development History

Full chronological development log from project start to working prototype.

---

## September 2024 – Project Start: Android GCS App

- Started development of **TelemetryView** Android GCS app (`com.example.telemetryview`)
- Goal: display real-time FrSky S.Port telemetry from X9D on Android phone via BLE
- Initial BLE connection to FrSky X20 investigated
- Basic telemetry display implemented (GPS, altitude, RSSI, battery)

---

## October–November 2024 – BLE PARA Protocol Reverse Engineering

- Used **nRF Connect** to scan FrSky X20 GATT structure
- Discovered correct UUIDs:
  - Device name: `Hello`
  - Service: `0xFFF0`
  - Characteristic `0xFFF6` – Notify (outbound)
  - Characteristic `0xFFF3` – Write No Response (inbound)
- Earlier attempts with assumed UUIDs `FFF1/FFF2` were incorrect
- Developed **NimBLE-Arduino** firmware to emulate PARA BLE client
- Confirmed 4-channel limitation of native PARA BLE trainer protocol

---

## November–December 2024 – X9D BLE Bridge Hardware

- Physically soldered ESP32-S3 Zero to X9D **battery bay UART**
  - GPIO4/GPIO5, 57600 baud, inverted polarity
- Firmware advertises as `"X9D BLE Bridge"` with unique UUIDs
- Coexistence with X20 BLE confirmed on same Android device
- Investigated HC-05 as alternative – incompatible (inverted UART, grounding issues)

---

## January–February 2025 – S.Port Telemetry Parsing

- Deep work on INAV 8.0.1 S.Port sensor ID mappings
- Implemented Kotlin parsing code for all standard and INAV-specific DIY sensor IDs
- Added `ArtificialHorizonView.kt` – custom aviation-style horizon renderer
- Dark HUD UI styled with Orbitron / Share Tech Mono fonts
- Variometer, heading, pitch/roll all working in real-time

---

## March 2025 – ESP-NOW SBUS Bridge Design

- Decision: bypass PARA BLE 4ch limit entirely with custom ESP-NOW bridge
- Architecture: two ESP32-S3 Zero modules, identical firmware, broadcast MAC
- SBUS protocol studied: 25 bytes, 16 channels × 11-bit, 8E2 inverted 100kbaud
- Firmware designed: simultaneous SBUS read (GPIO4) + ESP-NOW TX + ESP-NOW RX + SBUS write (GPIO5)
- UART inversion via `uart_set_line_inverse()` – no external inverter needed
- X9D JR bay pinout confirmed: pin 4 = SBUS out, pin 3 = S.Port

---

## April 2025 – Hardware Prototype Build

- Two ESP32-S3 Zero modules wired to X20 and X9D JR bays
- Power supply: **7805 voltage regulator** + smoothing electrolytic capacitor
- Stable 5V supply from radio battery bay confirmed
- Photos of complete wiring taken

---

## April 2026 – Full Hardware Test ✅

- Complete end-to-end test on real hardware
- All **16 SBUS channels** passing correctly through ESP-NOW bridge
- Bidirectional operation confirmed
- BLE telemetry running in parallel
- Compatible with X9D, X9D+, X9D+ 2019 (identical JR bay on all variants)
- **Prototype declared functional – testing phase complete**

---

## Related Work (Parallel Threads)

### S.Port GPS Sensor Emulation
- NEO-6M + ESP32-S3 emulating FrSky S.Port GPS sensor
- Half-duplex S.Port with byte stuffing and CRC
- TinyGPS++ for NMEA parsing, FrSky lat/lon encoding

### R9M / mLRS Investigation
- Evaluated ELRS for R9M: ELRS 4.x dropped STM32 support
- Viable options: ELRS 3.x or **mLRS** (preferred for MAVLink/MSP + INAV GCS)

### OpenIPC FPV System
- Air unit: OpenIPC Thinker AIO (IMX335/IMX415, SSC338Q/SSC30KQ)
- GCS: Raspberry Pi 4 (4GB) + ALFA AWUS036ACH + WFB-ng + GStreamer
- Android integration via LibVLC/ExoPlayer investigated

### GPS Logger Evaluation
- Canmore GP-102: evaluated, impractical (USB only, no external compass access)
- Recommended alternative: **Beitian BN-880** (u-blox M8N + HMC5883L)
