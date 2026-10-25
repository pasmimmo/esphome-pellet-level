# ⚡ Quickstart Guide (3 Steps)

Get your **D1 Pellet Level Monitor** running in Home Assistant in under 3 minutes.

---

## Step 1: Wire the VL53L0X Sensor to Wemos D1 Mini

Connect the 4 I2C pins between the **Wemos D1 Mini (ESP8266)** and the **VL53L0X ToF Sensor**:

| VL53L0X Pin | Wemos D1 Mini Pin | ESP8266 GPIO | Function |
| :--- | :--- | :--- | :--- |
| **VIN / VCC** | `3V3` | — | 3.3V Regulated Power |
| **GND** | `GND` | — | Common Ground |
| **SCL** | `D1` | `GPIO5` | I2C Clock (100 kHz) |
| **SDA** | `D2` | `GPIO4` | I2C Data |

> [!TIP]
> Leave `XSHUT` and `GPIO1` disconnected on the VL53L0X breakout board (they have onboard pull-up resistors for single-sensor operation).

---

## Step 2: Flash the Firmware

### Option A: Single-File Install (ESPHome Dashboard)
1. Open your **ESPHome Dashboard** in Home Assistant and click **+ New Device**.
2. Name it `d1-livello-pellet` and paste the contents of [`d1-livello-pellet_standalone.yaml`](../d1-livello-pellet_standalone.yaml).
3. Ensure your `secrets.yaml` contains:
   ```yaml
   wifi_ssid: "YOUR_WIFI_SSID"
   wifi_password: "YOUR_WIFI_PASSWORD"
   ```
4. Click **Install** -> **Plug into this computer** (for first flash via USB) or **Wirelessly** (OTA).

### Option B: Modular Package Install
Copy `d1-livello-pellet.yaml` and the `packages/` folder into your `/config/esphome/` directory and click **Install**.

---

## Step 3: Calibrate Live in Home Assistant (No Re-Flashing Needed!)

Once discovered in **Settings -> Devices & Services -> ESPHome**:

1. **Empty Hopper Calibration (`0%`)**:
   - With the pellet hopper empty (or down to the auger grid), check the `Pellet Distance` sensor reading (e.g., `0.202 m`).
   - Enter that value into **`Empty Tank Distance (0%)`**.
2. **Full Hopper Calibration (`100%`)**:
   - Fill the hopper completely with pellets, press **`Measure Pellet Level Now`**, and check `Pellet Distance` (e.g., `0.100 m`).
   - Enter that value into **`Full Tank Distance (100%)`**.
3. **Set Hopper Capacity**:
   - Set **`Tank Capacity`** to your stove's hopper capacity in kg (e.g., `15.0 kg` or `30.0 kg`).
   - All values are saved permanently to the ESP8266 flash memory!

