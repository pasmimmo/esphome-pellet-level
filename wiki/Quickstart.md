# ⚡ Quickstart Guide

Get your **D1 Pellet Level Monitor** running in Home Assistant in 3 simple steps.

---

## 1. Wire the Sensor (4 Wires)

| VL53L0X Pin | Wemos D1 Mini Pin | ESP8266 GPIO |
| :---: | :---: | :---: |
| `VCC` | `3V3` | 3.3V Power |
| `GND` | `GND` | Ground |
| `SCL` | `D1` | `GPIO5` |
| `SDA` | `D2` | `GPIO4` |

---

## 2. Flash with ESPHome

1. Add `wifi_ssid` and `wifi_password` to your ESPHome `secrets.yaml`.
2. Paste [`d1-livello-pellet_standalone.yaml`](../d1-livello-pellet_standalone.yaml) into a new ESPHome device named `d1-livello-pellet`.
3. Click **Install**.

---

## 3. Live Calibration in Home Assistant

Open the device in **Settings -> Devices & Services -> ESPHome -> D1 Livello Pellet**:
* Set **`Empty Tank Distance (0%)`** to the `Pellet Distance` reading when the hopper is empty (default `0.202 m`).
* Set **`Full Tank Distance (100%)`** to the `Pellet Distance` reading when the hopper is full (default `0.100 m`).
* Set **`Tank Capacity`** to your hopper's total capacity in kilograms (default `15.0 kg`).
