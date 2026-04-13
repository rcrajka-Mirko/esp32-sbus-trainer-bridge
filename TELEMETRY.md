# Telemetry

## Overview

S.Port telemetry from the flight controller is forwarded via BLE to the Android GCS app **TelemetryView** (`com.example.telemetryview`).

---

## Hardware – X9D BLE Bridge

An ESP32-S3 Zero is physically soldered to the X9D battery bay UART:

```
GPIO4 / GPIO5 – 57600 baud, inverted (S.Port half-duplex)
```

Firmware advertises as `"X9D BLE Bridge"` using unique UUIDs to coexist with X20 BLE on the same Android device.

---

## BLE GATT – FrSky PARA Protocol

Reverse-engineered via **nRF Connect**:

```
Device name:     Hello
Service:         0xFFF0
Characteristic:  0xFFF6  (Notify – outbound, S.Port data → app)
Characteristic:  0xFFF3  (Write No Response – inbound)
```

> Early development used assumed UUIDs (FFF1/FFF2) which were incorrect. Correct UUIDs were confirmed by scanning the X20 with nRF Connect.

---

## S.Port Protocol

- **Baud rate:** 57600
- **Polarity:** Inverted (RS-232 style)
- **Interface:** Half-duplex on single wire
- **Packet:** `0x7E [sensor_id] [data_id_low] [data_id_high] [value 4 bytes] [CRC]`

### Byte Stuffing

| Sequence | Meaning |
|---|---|
| `0x7D 0x5E` | Escaped `0x7E` |
| `0x7D 0x5D` | Escaped `0x7D` |

---

## INAV 8.0.1 S.Port Sensor IDs

### Standard IDs

| Data ID | Sensor | Notes |
|---|---|---|
| `0x0100` | Altitude | cm |
| `0x0200` | Current | 0.1A steps |
| `0x0210` | VBAT | mV |
| `0x0800` | GPS (lat/lon) | FrSky encoding |
| `0x0840` | Heading | degrees |

### INAV-specific DIY IDs

| Data ID | Sensor | Notes |
|---|---|---|
| `0x0420` | Home distance | m |
| `0x0450` | FPV heading | degrees ÷ 10 |
| `0x0460` | Azimuth to home | degrees |
| `0x0470` | Flight mode / arm state | bitmask |
| `0x0480` | GNSS status | fix type |

---

## Android App – TelemetryView

### Displayed Data

- GPS position (lat/lon)
- Altitude
- Heading
- Pitch / Roll (Artificial Horizon)
- RSSI
- Battery voltage
- Variometer (climb rate)

### Artificial Horizon

Custom `ArtificialHorizonView.kt` – aviation-style rendering with:
- Sky/ground split
- Pitch ladder
- Roll arc indicator

### UI Style

- Dark HUD theme
- Fonts: **Orbitron**, **Share Tech Mono**

---

## GPS Sensor Emulation (Separate Project)

A separate ESP32-S3 firmware emulates a **FrSky S.Port GPS sensor** using a **NEO-6M** module:

- NMEA parsing via TinyGPS++
- FrSky lat/lon encoding (degrees + minutes × 10000, with hemisphere flag)
- Half-duplex S.Port output with byte stuffing and CRC
- 1kΩ resistor on the bus for safe half-duplex operation

This allows any FrSky receiver with S.Port to display GPS data on the radio screen without a dedicated FrSky GPS sensor.
