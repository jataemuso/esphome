# ESPHome Home Automation Configs

A collection of ESPHome firmware configurations for various ESP32/ESP8266 devices integrated with Home Assistant.

---

## Devices

### `temp.yaml` — Indoor Room Sensor (ESP32)
Main room sensor using an **SHT4x** sensor over I²C.

- **Hardware:** ESP32 + SHT4x (GPIO21 = SDA, GPIO22 = SCL)
- **Sensors:**
  - Temperature & Humidity (sliding average, 15-sample window)
  - Dew Point (Magnus formula, computed on-device)
- **Text Sensors:**
  - **Comfort Level** — classifies dew point into: `Muito seco`, `Confortável`, `Começando a grudar`, `Desagradável`, `Muito abafado`
  - **Recommended AC Mode** — suggests `COOL`, `DRY`, or `OFF` based on temperature and dew point
- **Switch:** GPIO23 — Oil heater relay (`mdi:radiator`)
- **Static IP:** none (DHCP)

---

### `display-temp.yaml` — TTGO T-Display with Sensor (ESP32)
Combined sensor + display device. Reads temperature locally and shows it on a color TFT screen.

- **Hardware:** TTGO T-Display (ST7789V 135×240) + SHT4x (I²C on GPIO21/22) + SPI display (CLK=18, MOSI=19)
- **Display layout (rotation 90°, 240×135):**
  - Temperature (large, white)
  - Humidity (medium, cyan)
  - Dew Point with dynamic color coding:
    | Range | Color | Label |
    |-------|-------|-------|
    | < 10°C | Blue | MUITO SECO |
    | 10–15.5°C | Green | CONFORTAVEL |
    | 15.5–18.3°C | Yellow | A GRUDAR |
    | 18.3–21.1°C | Red | DESAGRADAVEL |
    | > 21.1°C | Purple | MUITO ABAFADO |
- **Physical Buttons:** GPIO0 (active-low) — fires `esphome.botao_pressionado` / `esphome.botao_solto` events to Home Assistant
- **Text Sensors:** Same comfort + AC mode logic as `temp.yaml`

---

### `display.yaml` — Display Only (ESP32, HA-fed sensors)
Simpler display device that pulls sensor data **from Home Assistant** instead of reading locally.

- **Hardware:** TTGO T-Display (ST7789V 135×240)
- **Data source:** `sensor.temp_temperatura` and `sensor.temp_umidade` via HA API
- **Display layout:** Temperature, Humidity, divider line, footer ("Home Assistant")

---

### `outdoor.yaml` — Outdoor Sensor (ESP8266 NodeMCU)
External temperature/humidity sensor with comfort classification.

- **Hardware:** NodeMCU v2 + SHT41 (D2 = SDA, D1 = SCL)
- **Sensors:**
  - Temperature & Humidity (100-sample sliding average, sends every 15 readings)
  - Dew Point (`Ponto de Orvalho Externo`)
- **Text Sensor — Conforto Climático Externo:**
  | Dew Point | Label |
  |-----------|-------|
  | < 7°C | Ar muito seco |
  | 7–13°C | Agradável |
  | 13–16°C | Levemente úmido |
  | 16–19°C | Úmido |
  | 19–22°C | Muito úmido |
  | > 22°C | Abafado / sensação pesada |
- **Static IP:** none (DHCP)

---

### `secadora.yaml` — Dryer Sensor (ESP8266 NodeMCU)
Monitors conditions inside or near a clothes dryer.

- **Hardware:** NodeMCU v2 + SHT45 at address `0x44` (D2 = SDA, D1 = SCL)
- **Static IP:** `192.168.1.148`
- **Sensors:**
  - Temperature & Humidity (15-sample sliding average)
  - Dew Point (`Secadora Dew Point`)
  - Delta T — difference between temperature and dew point, useful for assessing drying efficiency

---

### `portao.yaml` — Garage Door Controller (ESP8266 NodeMCU)
Controls a garage door via a relay and reports its position via a reed switch.

- **Hardware:** NodeMCU v2 — Relay on D1, Reed switch on D2
- **Static IP:** `192.168.1.50`
- **Cover entity:** `Portão Garagem` (device class: `garage`)
  - Open / Close / Stop all pulse the relay for 500 ms
  - Position derived from the binary sensor state (open/closed)
- **Binary Sensor:** `Portão - Sensor` (device class: `garage_door`, 20 ms debounce)

---

### `espdht.yaml` — Base ESP32 Template
Minimal ESP32 config — no sensors defined. Used as a base or placeholder device.

- **Hardware:** ESP32 dev board
- Includes WiFi, API (hardcoded encryption key), OTA, captive portal fallback

---

## Secrets

All devices rely on a `secrets.yaml` file (not tracked in git). Required keys:

```yaml
wifi_ssid: "your_wifi"
wifi_password: "your_password"
ota_password: "your_ota_password"
fallback_passoword: "your_fallback_password"   # note: typo preserved for compatibility
api_encryption_key_portao: "..."
api_encryption_key_secadora: "..."
api_encryption_key_outdoor: "..."
api_encryption_key_temp: "..."
api_encryption_key_temp_display: "..."
```

---

## Dew Point Formula

All devices compute dew point using the **Magnus formula**:

```
α = ln(RH/100) + (a·T) / (b + T)
Td = (b·α) / (a − α)

where a = 17.27, b = 237.7
```

---

## Requirements

- [ESPHome](https://esphome.io/) ≥ 2024.x
- Home Assistant (for API integration)
- Fonts: `gfonts://Roboto` (auto-downloaded by ESPHome for display devices)
