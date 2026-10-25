# Architecture Decision Records (ADR)

This document records the key architectural and engineering decisions implemented in the **D1 Pellet Level Monitor** firmware (`Wemos D1 Mini ESP8266 + VL53L0X ToF`), detailing physical constraints, signal processing mathematics, and hardware reliability trade-offs.

---

## Index of Architecture Decisions

* [ADR-01: Dual-Stage Median + SMA Filter Pipeline for Sawdust Rejection](#adr-01-dual-stage-median--sma-filter-pipeline-for-sawdust-rejection)
* [ADR-02: Optical Time-of-Flight (VL53L0X) vs Ultrasonic Ranging (HC-SR04)](#adr-02-optical-time-of-flight-vl53l0x-vs-ultrasonic-ranging-hc-sr04)
* [ADR-03: I2C Bus Frequency Locked at 100 kHz for Stove EMI Immunity](#adr-03-i2C-bus-frequency-locked-at-100-khz-for-stove-emi-immunity)
* [ADR-04: Flash-Persisted Runtime Calibration vs Hardcoded Lambda Constants](#adr-04-flash-persisted-runtime-calibration-vs-hardcoded-lambda-constants)

---

## ADR-01: Dual-Stage Median + SMA Filter Pipeline for Sawdust Rejection

### Status
**Accepted** (Implemented in `packages/pellet_sensor.yaml` & `d1-livello-pellet_standalone.yaml`)

### Context
Inside a pellet stove hopper, two distinct physical disturbances affect optical distance measurements:
1. **Impulse Noise (Airborne Sawdust):** Pouring a $15\text{ kg}$ bag of pellets generates a temporary cloud of suspended wood dust particles passing directly in front of the $940\text{ nm}$ laser aperture, causing sudden single-sample short-distance spikes (e.g., $d = 0.035\text{ m}$ instead of $0.180\text{ m}$).
2. **Surface Ripple (Funnel Depletion):** As the auger screw rotates intermittently, pellets slide down the hopper walls in small avalanches, causing $\pm 5\text{ mm}$ surface jitter.

### Mathematical Problem with Single-Stage Moving Average
The original firmware used only a 10-sample `sliding_window_moving_average` ($\text{SMA}_{10}$):
$$\bar{d}_k = \frac{1}{N} \sum_{i=0}^{N-1} d_{k-i}$$
Because the arithmetic mean has an **outlier breakdown point of $0\%$**, a single sawdust reflection spike of $\Delta d = -0.10\text{ m}$ corrupts the output by $10\%$ for the next $10 \times 30\text{s} = 5\text{ minutes}$!

### Decision
Implement a **two-stage cascaded digital filter**:
1. **Stage 1 — Non-Linear Median Filter ($W_m = 5$, breakdown point $50\%$):**
   $$y_k = \text{median}(d_k, d_{k-1}, d_{k-2}, d_{k-3}, d_{k-4})$$
   Any 1 or 2 consecutive dust-cloud outliers within a 5-sample window are **100% discarded** with zero amplitude distortion.
2. **Stage 2 — Linear Sliding Window Moving Average ($W_a = 6$):**
   $$\hat{d}_k = \frac{1}{6} \sum_{i=0}^{5} y_{k-i}$$
   Smooths the legitimate millimeter-scale surface avalanches of the pellet cone.

### Consequences
* **Positive:** Complete immunity to transient dust spikes during hopper refills; rock-steady percentage gauge in Home Assistant.
* **Negative:** Step changes (such as refilling the tank) settle smoothly over $3\text{--}4$ cycles ($90\text{--}120\text{ s}$), which can be bypassed immediately by pressing the `Measure Pellet Level Now` button.

---

## ADR-02: Optical Time-of-Flight (VL53L0X) vs Ultrasonic Ranging (HC-SR04)

### Status
**Accepted** (Hardware Architecture Baseline)

### Context
Hopper level sensing inside compact domestic pellet stoves ($20\text{ cm} \times 30\text{ cm}$ cross-section with sloped sheet-metal walls) can be approached using ultrasonic transducers (`HC-SR04` / `JSN-SR04T`) or optical Time-of-Flight (`VL53L0X`).

### Technical Comparison

| Parameter | Ultrasonic (`HC-SR04`) | Optical ToF (`VL53L0X`) - **Selected** |
| :--- | :--- | :--- |
| **Sensing Cone (FoV)** | Wide ($30^\circ\text{--}60^\circ$ acoustic lobe) | Narrow ($25^\circ$ optical cone) |
| **Granular Absorption** | Wood pellets scatter/absorb $40\text{ kHz}$ sound waves | $940\text{ nm}$ photons measure time-of-flight regardless of surface angle |
| **Temperature Drift** | Speed of sound varies by $v \approx 331.3 + 0.606 \cdot T_c$ ($\approx 5\%$ error from $20^\circ\text{C}$ to $50^\circ\text{C}$) | Speed of light $c$ is invariant to stove hopper air temperature |
| **Wall False Echoes** | High risk of side-wall multipathing | Zero side-wall echo when mounted $\ge 5\text{ cm}$ from walls |

### Consequences
* **Positive:** Millimeter-level accuracy across temperature swings ($15^\circ\text{C}\text{--}55^\circ\text{C}$) without needing a temperature compensation thermistor.
* **Negative:** Optical lens must be wiped clean of fine sawdust buildup every $1\text{--}2$ months (monitored by the Home Assistant lens maintenance automation).

---

## ADR-03: I2C Bus Frequency Locked at 100 kHz for Stove EMI Immunity

### Status
**Accepted** (Implemented in `packages/pellet_sensor.yaml`)

### Context
A pellet stove contains inductive $230\text{V AC}$ loads (auger gearmotor, combustion exhaust fan, room convection blower, and $300\text{W}$ resistive igniter). Switching these loads generates common-mode electromagnetic interference (EMI) along internal sensor harnesses.

### Analysis
On the ESP8266 (`Wire` software bit-banged I2C implementation), running the bus at Fast Mode ($400\text{ kHz}$) reduces the clock high time ($T_{\text{HIGH}} \approx 0.6\,\mu\text{s}$) and makes the open-drain `SDA`/`SCL` lines sensitive to capacitive cable loading ($C_{\text{bus}}$) and inductive noise spikes.
At Standard Mode ($100\text{ kHz}$):
$$T_{\text{HIGH}} \approx 4.0\,\mu\text{s}, \quad T_{\text{LOW}} \approx 4.7\,\mu\text{s}$$
Additionally, disabling continuous startup bus scanning (`scan: false`) eliminates 127 unnecessary address probes at boot.

### Consequences
* **Positive:** High noise margin over $20\text{--}50\text{ cm}$ sensor cables inside the stove chassis; eliminates I2C NACK timeouts during auger motor starts.
* **Negative:** Negligible ($< 2\text{ ms}$ difference per 30-second measurement cycle).

---

## ADR-04: Flash-Persisted Runtime Calibration vs Hardcoded Lambda Constants

### Status
**Accepted** (Implemented in `packages/core.yaml` & `packages/pellet_sensor.yaml`)

### Context
Previously, `MIN_DIST` ($0.100\text{ m}$) and `MAX_DIST` ($0.202\text{ m}$) were hardcoded as `const float` values inside the C++ lambda expression. Adjusting calibration after moving the sensor or changing the mounting bracket required editing YAML, recompiling the C++ firmware, and flashing OTA.

### Decision
Expose 4 configuration `number` entities (`cal_min_distance`, `cal_max_distance`, `cal_tank_capacity_kg`, `cal_low_threshold_pct`) with `restore_value: true` backed by `esp8266: restore_from_flash: true`.

### Consequences
* **Positive:** Zero-compile live calibration directly from the Home Assistant UI; instant recalculation of both `%` and `kg` via `on_value` triggers; reusable across any pellet stove model without touching code.
* **Negative:** Flash write wear is only incurred when the user manually changes a calibration slider (negligible over device lifetime).

