# 🎵 ESP32-S3 Hi-Fi Network Audio Streamer & Cyber HUD

![Platform](https://img.shields.io/badge/Platform-ESP32--S3%20(N16R8)-blue?style=for-the-badge&logo=espressif)
![ESPHome](https://img.shields.io/badge/ESPHome-2026.x%2B-00B4D8?style=for-the-badge&logo=esphome)
![Music Assistant](https://img.shields.io/badge/Music%20Assistant-Sendspin%20%2B%20DSP-9D4EDD?style=for-the-badge)
![DAC](https://img.shields.io/badge/DAC-PCM5102A%2032--bit%20I2S-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-amber?style=for-the-badge)

An audiophile-grade **Hi-Fi Network Audio Streamer** powered by **ESP32-S3 (16MB Flash / 8MB Octal PSRAM)**, an external **TI PCM5102A 32-bit / 48 kHz I2S DAC**, and a **2.8" IPS TFT ST7789 (320×240)** display.

Designed for seamless integration with **Home Assistant** and **Music Assistant**, featuring millisecond-accurate **Sendspin** multi-room synchronization, hardware **Bass / Treble DSP Parametric EQ** control via physical potentiometers, a zero-lag velocity-accelerated rotary encoder, and onboard environmental sensors (`AHT20` + `BMP280` + `DS18B20`).

---

## ✨ Key Features

- 🎧 **32-bit / 48 kHz I2S Audio Pipeline** – Direct digital I2S output to an external **PCM5102A DAC** with an internal 3-channel hardware mixer (dedicated pipelines for FLAC/PCM music, Home Assistant TTS announcements, and **Sendspin** multi-room audio).
- 🖥️ **Apple Music / Spotify "Now Playing" UI (25 FPS @ 80 MHz SPI)**:
  - **Dynamic Ambient Cover Gradient** – Real-time color sampling from the upper and lower halves of the album artwork with smooth Hermite interpolation (`lerp`) and automatic text contrast adaptation (ITU-R BT.601 luminance).
  - **Double-Buffered PSRAM Album Artwork (Crossfade)** – Dual `online_image` buffers (128×128 RGB565) stored in 8 MB Octal PSRAM for smooth track transitions.
  - **Minimalist Hi-Fi Standby Weather Screen** – Flipping the physical toggle switch to *Standby* transitions the display into a clean clock & weather station (**INDOOR / OUTDOOR** temperature, relative humidity, and atmospheric pressure).
- 🎛️ **Zero-Lag Physical Controls**:
  - **Velocity-Accelerated Rotary Encoder** – Instantaneous RAM volume accumulator (`vol_target`) with `0 ms` UI latency, FreeRTOS queue coalescing (`25 ms`), and multi-click button gestures (Click = Play/Pause or Backlight Toggle in Standby, Double-Click = Next Track, Hold = Switch Screen).
  - **Analog BASS & TREBLE Potentiometers (±6 dB)** – Dual B5k linear potentiometers with moving-average filtering, 0 dB center deadzone, and live RPC API sync to **Music Assistant Parametric EQ DSP** (`60 Hz low_shelf` & `10 kHz high_shelf`).
  - **Hardware Toggle Switch (Soft-Standby)** – Physical power switch with RGB LED status indicator.
- 🌡️ **All-in-One Onboard Environmental Sensors (Main ESP32-S3)**:
  - **AHT20 + BMP280 (I2C)** – Direct room temperature, relative humidity, and atmospheric pressure measurements on the main ESP32-S3 (`room_temp`, `room_humidity`, `atmospheric_pressure`).
  - **DS18B20 (1-Wire)** – External temperature probe support (`outdoor_temp_ds18b20`) with automatic fallback to a Home Assistant weather sensor (`outdoor_temp_ha`).
- 📶 **Ready-to-Flash Wi-Fi Hotspot & Captive Portal**:
  - Zero hardcoded credentials – on first boot the streamer broadcasts an open Wi-Fi Hotspot (**`HiFi-Streamer-Setup`**) with a Captive Portal to select your home Wi-Fi network.

---

## 🛠️ Bill of Materials (BOM)

| Component | Model / Specification | Notes |
| :--- | :--- | :--- |
| **Microcontroller** | **ESP32-S3 DevKitC-1 N16R8** | 16 MB Flash + **8 MB Octal PSRAM** (required for audio & cover buffers) |
| **Audio DAC** | **PCM5102A I2S DAC** | GY-PCM5102 module (solder jumpers: H1L=GND, H2L=GND, H3L=3.3V, H4L=GND) |
| **Display** | **2.8" TFT IPS ST7789V (320×240)** | SPI interface + ultrasonic 25 kHz PWM backlight control |
| **Rotary Encoder** | **EC11 with push-button** | Volume control + multi-click media gestures |
| **Potentiometers** | **2× B5k (5 kΩ linear)** | **BASS** and **TREBLE** hardware tone controls (±6 dB) |
| **Power Switch** | **2-pin SPST Toggle Switch** | Physical *Active / Soft-Standby* mode selector |
| **I2C Sensors** | **AHT20 + BMP280 Combo Module** | Indoor temperature, humidity, and atmospheric pressure |
| **1-Wire Probe** | **DS18B20** *(optional)* | Outdoor temperature probe (with 4.7 kΩ pull-up resistor to 3.3V) |
| **Status LED** | **RGB LED (Common Cathode)** | Green = ON, Red = Standby, Blue = Streaming Active |

---

## 📌 ESP32-S3 Pinout Reference

### 1. I2S Audio DAC (PCM5102A)
| PCM5102A Pin | ESP32-S3 Pin | Description |
| :--- | :--- | :--- |
| **BCK** | `GPIO15` | I2S Bit Clock |
| **LCK / WS** | `GPIO16` | I2S Word Select / Left-Right Clock |
| **DIN** | `GPIO17` | I2S Serial Data Out |
| **SCK** | `GND` | Tie to GND (enables internal PCM5102A PLL) |
| **VIN / GND** | `5V` / `GND` | Power supply (onboard 3.3V LDO regulator) |

### 2. 2.8" TFT ST7789V Display (Hardware SPI @ 80 MHz)
| Display Pin | ESP32-S3 Pin | Description |
| :--- | :--- | :--- |
| **SCK / CLK** | `GPIO12` | SPI Clock |
| **SDA / MOSI** | `GPIO11` | SPI MOSI Data |
| **RES / RST** | `GPIO14` | Display Reset |
| **DC / RS** | `GPIO9` | Data / Command Select |
| **CS** | `GPIO10` | Chip Select |
| **BLK / LED** | `GPIO13` | Backlight PWM (25 kHz ultrasonic – zero coil whine) |

### 3. Controls & Status RGB LED
| Control / Indicator | ESP32-S3 Pin | Entity ID | Description |
| :--- | :--- | :--- | :--- |
| **Encoder Pin A (CLK)** | `GPIO4` | `volume_encoder` | Internal pull-up enabled |
| **Encoder Pin B (DT)** | `GPIO5` | `volume_encoder` | Internal pull-up enabled |
| **Encoder Button (SW)** | `GPIO6` | `encoder_button` | Active LOW to GND (click / double-click / hold) |
| **Toggle Switch** | `GPIO7` | `power_switch` | Active LOW to GND when switched ON |
| **Bass Potentiometer** | `GPIO2` | `pot_bass` | Wiper (0 – 3.3V ADC1) |
| **Treble Potentiometer**| `GPIO3` | `pot_treble` | Wiper (0 – 3.3V ADC1) |
| **RGB LED – Green** | `GPIO8` | `led_green` | Power ON indicator |
| **RGB LED – Red** | `GPIO18` | `led_red` | Standby indicator |
| **RGB LED – Blue** | `GPIO21` | `led_blue` | Audio playback indicator |

### 4. Onboard Environmental Sensors (Main ESP32-S3)
| Sensor Bus | ESP32-S3 Pin | Sensor IDs | Description |
| :--- | :--- | :--- | :--- |
| **I2C SDA (AHT20 + BMP280)** | `GPIO41` | `room_temp`, `room_humidity` | Configurable via `i2c_sda_pin` substitution |
| **I2C SCL (AHT20 + BMP280)** | `GPIO42` | `room_temp_bmp280`, `atmospheric_pressure` | Configurable via `i2c_scl_pin` substitution |
| **1-Wire (DS18B20)** | `GPIO40` | `outdoor_temp_ds18b20` | Configurable via `onewire_pin` substitution |

---

## 🚀 Quick Start & Wi-Fi Provisioning

### Step 1: Customize `substitutions` (Optional)
At the top of [`streamer.yaml`](./streamer.yaml), adjust your Home Assistant / Music Assistant IP and player entity if desired:

```yaml
substitutions:
  device_name: "hifi-streamer"
  friendly_name: "Hi-Fi Streamer"
  ha_ip: "192.168.1.100"
  ha_port: "8123"
  ma_port: "8095"
  ma_token: "Bearer YOUR_MUSIC_ASSISTANT_TOKEN"
  ma_player_entity: "media_player.hifi_streamer"
```

### Step 2: Flash the Firmware
Compile and upload [`streamer.yaml`](./streamer.yaml) to your **ESP32-S3 N16R8** board:

```bash
esphome run streamer.yaml
```

### Step 3: Connect to Wi-Fi via Hotspot
1. Once flashed, the streamer broadcasts an open Wi-Fi access point:
   - **SSID:** `HiFi-Streamer-Setup`
2. Connect to **`HiFi-Streamer-Setup`** using your smartphone or computer.
3. A **Captive Portal** window will pop up automatically (or open `http://192.168.4.1` in your browser).
4. Select your Wi-Fi network, enter your password, and click **Save**.
5. The credentials are saved to NVS flash and the streamer will automatically appear in **Home Assistant** under *Settings → Devices & Services → ESPHome*.

---

## 🎚️ Controls & Encoder Button Gestures

| Device State | Encoder Action | Function |
| :--- | :--- | :--- |
| **Powered ON** | **Rotate CW / CCW** | Smooth volume adjustment with 3-stage velocity acceleration |
| **Powered ON** | **Single Click** | `Play / Pause` active media player |
| **Powered ON** | **Double Click** | Skip to `Next Track` |
| **Powered ON** | **Long Press (> 0.45 s)**| Toggle screen: *Now Playing* ↔ *Clock & Weather* |
| **Standby (OFF)** | **Single Click** | Toggle display backlight completely `OFF (0% PWM)` / `ON (70%)` |

---

## 👤 Author & License

- **Author:** **Maciej_Kow** (`Maciej_Kow.HiFi_Streamer`)
- **License:** Released under the [MIT License](./LICENSE).
