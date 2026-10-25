# 🏠 Home Assistant Integration & SmartDisplay Alerts

## 1. Lovelace Dashboard Card

Paste the contents of [`examples/home-assistant/dashboard_card.yaml`](../examples/home-assistant/dashboard_card.yaml) into a Manual Card in Home Assistant to get:
* **Color-Coded Hopper Level Gauge** (`Green` $\ge 40\%$, `Yellow` $20\%\text{--}39\%$, `Red` $< 20\%$)
* **Remaining Pellet Weight (`kg`)** & **Hopper Status** badge
* **`Measure Pellet Level Now`** button for instant post-refill updates
* **Live Calibration Panel** (`Full Tank Distance`, `Empty Tank Distance`, `Tank Capacity`, `Low Pellet Alert Threshold`)

---

## 2. SmartDisplay ESP32-S3 Integration

Using [`examples/home-assistant/automations.yaml`](../examples/home-assistant/automations.yaml), when `binary_sensor.d1_livello_pellet_low_pellet_reserve` turns `ON`:
1. **Visual Notification Card**: Sends an orange `thermometer` alert card to **SmartDisplay** showing both `%` and remaining `kg`.
2. **TTS Voice Announcement**: Announces the low pellet reserve over the SmartDisplay speaker during daytime hours (`08:00–22:00`).
3. **Mobile Push Notification**: Sends an alert to the Home Assistant Companion App.

