# FrSky Wireless Trainer Bridge – ESP32-S3 / ESP-NOW / SBUS 16ch

> **Status:** ✅ Working prototype – tested on real hardware (April 2026)

Wireless trainer link between two FrSky radios using two ESP32-S3 Zero modules communicating via ESP-NOW. Passes all **16 SBUS channels** – bypassing the built-in PARA BLE trainer limitation of only 8 channels.

---

## Motivation

FrSky X20 (Ethos) supports Bluetooth PARA trainer protocol, but it is limited to **8 channels only**. This project was born out of the need to have a full 16-channel wireless trainer link between:

- **Master radio:** FrSky X20 (Ethos)
- **Student radio:** FrSky X9D / X9D+ / X9D+ 2019 (EdgeTX)

All three X9D variants use **identical JR bay pinout and wiring** – the solution works on all of them without any modification.

---

## Features

- ✅ Full **16 SBUS channels** over ESP-NOW (2.4 GHz Wi-Fi)
- ✅ Works with X9D, X9D+, X9D+ 2019 – same wiring on all
- ✅ **Bidirectional** SBUS relay (both modules run identical firmware)
- ✅ BLE telemetry output (S.Port sensors forwarded to Android GCS app)
- ✅ Powered from radio via 7805 voltage regulator + smoothing capacitor
- ✅ No modifications to the radio required

---

## Hardware

| Component | Quantity | Notes |
|---|---|---|
| Waveshare ESP32-S3 Zero | 2 | One per radio |
| 7805 voltage regulator | 2 | 5V stabilization |
| Electrolytic capacitor | 4 | Voltage smoothing |
| Resistors, wires | – | Standard prototyping parts |

---

## Wiring

### FrSky X9D / X9D+ / X9D+ 2019
### ESP32-S3 Zero GPIO Assignment

``

UART signal inversion is handled in software via `uart_set_line_inverse()` – no external inverter needed.

---

## SBUS Protocol

- **Baud rate:** 100000
- **Format:** 8E2 (8 data bits, Even parity, 2 stop bits)
- **Polarity:** Inverted
- **Packet size:** 25 bytes
- **Channels:** 16 × 11-bit values
- **Flags byte:** channel 17/18 digital + failsafe + frame lost

SBUS is a fixed-size protocol – 16 channels maximum by design. No extension exists within standard SBUS.

---

## Communication

Both ESP32-S3 modules run **identical firmware**. Each module simultaneously:
1. Reads SBUS on GPIO4 (Serial1)
2. Forwards the frame via ESP-NOW broadcast
3. Receives ESP-NOW frame from the other module
4. Outputs SBUS on GPIO5 (Serial2)

ESP-NOW uses broadcast MAC address – no pairing required.

---

## Firmware

- Platform: **PlatformIO** (Cursor IDE)
- Library: **ESP-NOW** (built-in ESP-IDF)
- SBUS parsing: custom 11-bit channel packing/unpacking
- Debug: USB CDC via Serial monitor

See [`FIRMWARE.md`](FIRMWARE.md) for full details and source code.

---

## Telemetry – Android GCS App

A companion Android app (`TelemetryView`) was developed in parallel:

- Connects to radio via **BLE** (NimBLE-Arduino)
- Displays real-time: GPS, altitude, heading, pitch, roll, RSSI, battery, variometer
- Custom `ArtificialHorizonView` – aviation-style rendering
- Dark HUD UI – Orbitron / Share Tech Mono fonts

BLE GATT structure (FrSky PARA protocol reverse-engineered via nRF Connect):
```
Device name:     Hello
Service:         0xFFF0
Characteristic:  0xFFF6  (Notify – outbound)
Characteristic:  0xFFF3  (Write No Response – inbound)
```

X9D BLE Bridge firmware advertises as `"X9D BLE Bridge"` using unique UUIDs to coexist with X20 BLE on the same device.

See [`TELEMETRY.md`](TELEMETRY.md) for S.Port sensor ID mapping and full details.

---

## Development Timeline

| Period | Milestone |
|---|---|
| September 2024 | Project started – Android GCS app TelemetryView |
| Oct–Nov 2024 | BLE PARA protocol reverse engineering (nRF Connect) |
| Nov–Dec 2024 | X9D BLE bridge firmware (NimBLE-Arduino) |
| Jan–Feb 2025 | S.Port telemetry parsing, INAV 8.0.1 sensor IDs |
| Mar 2025 | ESP-NOW SBUS bridge design and firmware |
| Apr 2025 | Hardware prototype – 7805 regulator, real wiring |
| Apr 2026 | Full testing on real hardware – all 16ch working ✅ |

See [`CHANGELOG.md`](CHANGELOG.md) for detailed history.

---

## Related Projects / Background

- **INAV fixed-wing platform:** SpeedyBee F405 Wing + INAV 8.0.1 + FrSky Archer Plus (ACCST/S.Port)
- **S.Port GPS sensor:** NEO-6M emulated as FrSky S.Port sensor (half-duplex, byte stuffing, CRC)
- **OpenIPC FPV:** Investigated air unit + GCS bonnet (RPi4 + WFB-ng + GStreamer)
- **mLRS / R9M:** Evaluated as ELRS alternative for MAVLink/MSP telemetry with INAV GCS

---

## License

MIT – free to use, modify, share.
