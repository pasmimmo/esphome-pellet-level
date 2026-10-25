# 🚀 Release Notes - v1.0.0

**Release Date:** October 25, 2026  
**Target Hardware:** Wemos D1 Mini (ESP8266) + STMicroelectronics VL53L0X (`CJVL53L0XV2`) Time-of-Flight Laser Sensor  
**Minimum ESPHome Version:** `2024.10.0+`

---

## ✨ Highlights of v1.0.0

### 1. Dual-Architecture Firmware (Modular + Standalone)
* **Modular `packages/` Layout** ([`d1-livello-pellet.yaml`](d1-livello-pellet.yaml)): Cleanly separates system core ([`packages/core.yaml`](packages/core.yaml)), optical ToF telemetry & calibration ([`packages/pellet_sensor.yaml`](packages/pellet_sensor.yaml)), and diagnostics ([`packages/diagnostics.yaml`](packages/diagnostics.yaml)).
* **Standalone Monolithic Configuration** ([`d1-livello-pellet_standalone.yaml`](d1-livello-pellet_standalone.yaml)): Synchronized 1:1 single-file configuration for rapid copy-paste installation in the Home Assistant ESPHome Dashboard.

### 2. Dual-Stage Anti-Dust Optical Filter Pipeline
* Replaced single-stage moving average with a cascaded **`filter_out: nan` → `median (window 5)` → `sliding_window_moving_average (window 6)`** pipeline.
* Completely rejects transient optical reflection spikes caused by airborne sawdust clouds during hopper refills while smoothing pellet funnel avalanches.

### 3. Zero-Compile Live Calibration in Home Assistant
* Added 4 flash-persisted configuration entities (`restore_from_flash: true`):
  * `Full Tank Distance (100%)` (default `0.100 m`)
  * `Empty Tank Distance (0%)` (default `0.202 m`)
  * `Tank Capacity` (default `15.0 kg`)
  * `Low Pellet Alert Threshold` (default `20%`)

### 4. Rich Hopper Telemetry & Maintenance Controls
* **`Pellet Remaining (kg)`**: Real-time fuel mass estimation based on hopper capacity.
* **`Low Pellet Reserve`**: Binary `problem` sensor for Home Assistant automations.
* **`Hopper Status`**: Human-readable text status (`Full`, `OK`, `Low`, `Reserve (Refill)`).
* **`Measure Pellet Level Now`**: Instant manual sampling button for immediate updates right after pouring a bag of pellets.
* **Lightweight Native API & OTA**: Removed legacy OTA password and API encryption overhead, reducing ESP8266 flash footprint by **>50 KB** (`37.6%` Flash, `40.4%` RAM) and enabling `power_save_mode: LIGHT` for cooler operation inside the IP55 hopper enclosure.

### 5. Documentation, ADRs & Home Assistant Examples
* Added ready-to-use automations ([`examples/home-assistant/automations.yaml`](examples/home-assistant/automations.yaml)) integrating with **[SmartDisplay ESP32-S3](https://github.com/pasmimmo/SmartDisplay)** (visual notification cards + TTS voice announcements) and mobile push alerts.
* Added Lovelace dashboard gauge and calibration card ([`examples/home-assistant/dashboard_card.yaml`](examples/home-assistant/dashboard_card.yaml)).
* Documented 4 formal Architecture Decision Records in [`docs/ARCHITECTURE_DECISIONS.md`](docs/ARCHITECTURE_DECISIONS.md) and real hardware installation in [`docs/HARDWARE.md`](docs/HARDWARE.md).

