# Hardware

## Components

| Component | Notes |
|---|---|
| Waveshare ESP32-S3 Zero (×2) | One per radio |
| 7805 voltage regulator | 5V stabilization from radio battery |
| Electrolytic capacitor | Voltage smoothing |
| 1kΩ resistor | S.Port half-duplex bus protection |
| Hookup wire, solder | Standard prototyping |

---

## FrSky X9D / X9D+ / X9D+ 2019 – JR Bay

All three variants have **identical JR bay pinout and wiring**. No differences between X9D, X9D+, and X9D+ 2019 in this regard.

```
Pin 1 – VCC (3.3V)
Pin 2 – GND
Pin 3 – S.Port  (57600 baud, inverted polarity)
Pin 4 – SBUS out (100kbaud, 8E2, inverted)
```

Physical orientation: connector is on the back of the radio, to the right of the antenna. Pin 1 is closest to the top edge of the radio.

> **Note:** The JR bay connector may appear to have 5 pins physically. The 5th pin is a mechanical stabilizing pin with no electrical contact.

---

## FrSky X20 – JR Bay

Same standard JR bay connector. SBUS output on pin 4 carries the full 16-channel frame from the internal trainer system.

---

## ESP32-S3 Zero – Pin Assignment

```
GPIO4  – SBUS IN  (from radio JR pin 4, Serial1)
GPIO5  – SBUS OUT (to radio JR pin 4 on other side, Serial2)
3.3V   – from 7805 output (via regulator)
GND    – common ground
```

---

## Power Supply

Radio battery bay provides ~7.4V (2S LiPo). This is regulated down to stable 5V via **7805 linear regulator**, then to 3.3V by the ESP32-S3 Zero's onboard regulator.

```
Radio battery (~7.4V) → 7805 → 5V → ESP32-S3 Zero (3.3V onboard reg)
```

Electrolytic capacitor on the 7805 output smooths any ripple.

---

## Signal Inversion

SBUS uses inverted UART polarity (logic 1 = low voltage). The ESP32-S3 handles this in software:

```cpp
uart_set_line_inverse(UART_NUM_1, UART_SIGNAL_RXD_INV); // SBUS IN
uart_set_line_inverse(UART_NUM_2, UART_SIGNAL_TXD_INV); // SBUS OUT
```

No external inverter (transistor or 74HC inverter) is needed.

---

## Photos

*(Add photos of completed prototype here)*

- `photos/overview.jpg` – full assembly
- `photos/x9d_wiring.jpg` – X9D JR bay connection
- `photos/esp32s3_closeup.jpg` – ESP32-S3 Zero wiring detail
- `photos/power_supply.jpg` – 7805 regulator assembly
