# Firmware

## Platform

- **IDE:** Cursor + PlatformIO
- **Target:** Waveshare ESP32-S3 Zero
- **Framework:** Arduino (ESP-IDF under the hood)
- **Libraries:** ESP-NOW (built-in), NimBLE-Arduino (BLE)

---

## GPIO Assignment

```
GPIO4  – SBUS IN  (Serial1, inverted, 100kbaud 8E2)
GPIO5  – SBUS OUT (Serial2, inverted, 100kbaud 8E2)
```

---

## SBUS Configuration

```cpp
// Serial1 – SBUS input (inverted)
Serial1.begin(100000, SERIAL_8E2, GPIO_NUM_4, -1);
uart_set_line_inverse(UART_NUM_1, UART_SIGNAL_RXD_INV);

// Serial2 – SBUS output (inverted)
Serial2.begin(100000, SERIAL_8E2, -1, GPIO_NUM_5);
uart_set_line_inverse(UART_NUM_2, UART_SIGNAL_TXD_INV);
```

UART inversion is handled in software – no external inverter hardware needed.

---

## SBUS Packet Structure

```
Byte 0:     0x0F  (start byte)
Bytes 1-22: 16 channels × 11 bits, packed LSB first
Byte 23:    flags (ch17, ch18, frame lost, failsafe)
Byte 24:    0x00  (end byte)
```

Total: **25 bytes** per frame at 100Hz refresh rate.

### Channel Packing (11-bit, LSB first)

```cpp
// Pack 16 channels into bytes 1..22
void sbus_pack(uint16_t ch[16], uint8_t *buf) {
    buf[0] = 0x0F;
    // 11-bit packing across byte boundaries
    // ...
    buf[23] = 0x00; // flags
    buf[24] = 0x00; // end byte
}

// Unpack
void sbus_unpack(uint8_t *buf, uint16_t ch[16]) {
    ch[0]  = ((buf[1]    ) | (buf[2] <<8))  & 0x07FF;
    ch[1]  = ((buf[2] >>3) | (buf[3] <<5))  & 0x07FF;
    // ...
}
```

---

## ESP-NOW Communication

Both modules use **identical firmware**. No master/slave distinction.

```cpp
// Broadcast MAC – no pairing required
uint8_t broadcastMac[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

void setup() {
    WiFi.mode(WIFI_STA);
    esp_now_init();
    esp_now_register_recv_cb(onDataRecv);
    esp_now_register_send_cb(onDataSent);
    // Add broadcast peer
    esp_now_peer_info_t peer = {};
    memcpy(peer.peer_addr, broadcastMac, 6);
    peer.channel = 0;
    peer.encrypt = false;
    esp_now_add_peer(&peer);
}

void loop() {
    // Read SBUS frame from Serial1
    if (readSBUS(rxBuf)) {
        // Forward via ESP-NOW
        esp_now_send(broadcastMac, rxBuf, 25);
    }
}

void onDataRecv(const uint8_t *mac, const uint8_t *data, int len) {
    // Write received SBUS frame to Serial2
    if (len == 25) {
        Serial2.write(data, 25);
    }
}
```

---

## Debug

USB CDC debug output via `Serial` (native USB on ESP32-S3):

```cpp
Serial.begin(115200); // USB CDC
Serial.printf("CH1: %d CH2: %d CH3: %d\n", ch[0], ch[1], ch[2]);
```

---

## Build & Flash

```bash
# PlatformIO CLI
pio run --target upload

# Or via Cursor IDE with PlatformIO extension
```

### platformio.ini

```ini
[env:waveshare_esp32s3_zero]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200
lib_deps =
    h2zero/NimBLE-Arduino
```
