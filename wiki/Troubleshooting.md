# 🛠️ Troubleshooting & Maintenance

## 1. Sensor Reads `Unavailable` or `0%` Continuously
* **Cause**: Fine wood dust film covering the VL53L0X optical aperture on the underside of the IP55 junction box.
* **Fix**: Wipe the sensor window gently with a dry cotton swab or microfiber cloth during routine stove cleaning, then press **`Measure Pellet Level Now`** in Home Assistant.

## 2. Distance Reading Shown in Feet (`ft`) Instead of Meters (`m`)
* **Fix**: The firmware removes `device_class: distance` to prevent Home Assistant from auto-converting units. If Home Assistant cached `ft` from an older install, open the `Pellet Distance` entity in Home Assistant -> **Gear Icon ⚙️** -> **Unit of measurement** -> select **`m`** (or **`cm`**).

## 3. Changing Home Wi-Fi Router Without Opening the Junction Box
* If the Wemos D1 Mini cannot reach your Wi-Fi network for 60 seconds, it automatically broadcasts a fallback hotspot named **`D1 Livello Pellet Hotspot`** (password: `d1-livello-pellet`).
* Connect with your phone to configure the new Wi-Fi credentials via the Captive Portal without opening the stove enclosure!

