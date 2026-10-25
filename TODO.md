# 📋 D1 Pellet Level Monitor (Wemos D1 Mini + VL53L0X) - Piano di Lavoro & TODO

Progetto firmware open-source basato su ESPHome per il monitoraggio ottico Time-of-Flight (VL53L0X) del livello di pellet nel serbatoio della stufa tramite Wemos D1 Mini (ESP8266), integrato nativamente con Home Assistant e SmartDisplay.

---

## 🧭 Fasi del Progetto

```
[Fase 1: Riorganizzazione Codice] ──> [Fase 2: Test & Fix] ──> [Fase 3: Nuove Migliorie]
                                                                        │
[Fase 6: Portfolio & Engineering] <── [Fase 5: Documentazione] <────────┘
```

---

## 📌 Checklist Dettagliata

### Fase 1: Riorganizzazione del Codice (Modularizzazione & Sicurezza)
- [x] Inizializzare repository Git locale (`git init`)
- [x] Creare `.gitignore` ottimizzato per ESPHome (`.esphome/`, build files, `secrets.yaml`, ecc.)
- [x] Creare `secrets.yaml.example` snello (solo `wifi_ssid` e `wifi_password`, identico a SmartDisplay)
- [x] Progettare e implementare architettura modulare a `packages/`:
  - [x] `packages/core.yaml`: ESP8266 `d1_mini`, `restore_from_flash: true`, logging, Native API leggera, OTA passwordless, Wi-Fi con `power_save_mode: LIGHT` e Captive Portal di emergenza
  - [x] `packages/pellet_sensor.yaml`: Bus I2C a 100kHz, sensore ToF `vl53l0x` con filtro anti-polvere a doppio stadio, calibrazione dinamica da HA (`number`), calcolo percentuale con `clamp`, stima peso residuo in kg, allarme binario riserva e stato testuale serbatoio
  - [x] `packages/diagnostics.yaml`: Wi-Fi RSSI, IP, SSID, MAC, Uptime, Versione ESPHome, pulsanti `Measure Pellet Level Now` e `Restart`
- [x] Refactoring generale: commenti tecnici in inglese in tutti i file, `substitutions` centralizzate
- [x] Sincronizzazione continua della copia monolitica all-in-one (`d1-livello-pellet_standalone.yaml`)

---

### Fase 2: Test & Verifica Hardware
- [x] Rimozione definitiva via OTA v2 della vecchia password legacy (`0x41 RESPONSE_AUTH_OK` passwordless) e risparmio di oltre 50 KB di Flash (`37.6%` Flash, `40.4%` RAM)
- [x] Flash OTA sul Wemos D1 Mini e verifica connessione API con Home Assistant
- [x] Verifica stabilità lettura I2C `VL53L0X` (`D1`/`D2`) e assenza di errori `NaN`
- [x] Fix unità di misura distanza (`m` fisso, rimosso `device_class: distance` che forzava la conversione in piedi `ft` su HA)
- [x] Verifica taratura live da Home Assistant (`Full Tank Distance`, `Empty Tank Distance`, `Tank Capacity`) e persistenza al riavvio

---

### Fase 3: Migliorie Tecniche & Nuove Funzionalità Firmware
- [x] **Ottimizzazione A - Pipeline Anti-Polvere a Doppio Stadio (Median + SMA)**:
  - Aggiunto `filter_out: nan` per scartare letture fuori portata
  - Aggiunto filtro `median` (`window_size: 5`) a monte della media mobile per eliminare gli spike ottici istantanei causati dalla segatura sollevata durante il carico del pellet
  - Mantenuto `sliding_window_moving_average` (`window_size: 6`) a valle per smussare l'avvallamento a cono del pellet
- [x] **Ottimizzazione B - Calibrazione Live da Home Assistant senza Ricompilare**:
  - Sostituite le costanti cablate `MIN_DIST`/`MAX_DIST` con 4 entità `number` (`restore_value: true` su flash): distanza pieno (`0.100 m`), distanza vuoto (`0.202 m`), capacità serbatoio (`15.0 kg`) e soglia riserva (`20%`)
- [x] **Ottimizzazione C - Stima Peso Residuo (kg) & Stato Serbatoio**:
  - Sensore `Pellet Remaining (kg)` calcolato dinamicamente in base alla capacità configurata
  - Sensore testuale `Hopper Status` (`Full`, `OK`, `Low`, `Reserve (Refill)`)
  - Sensore binario `Low Pellet Reserve` (`device_class: problem`) per automazioni di notifica
- [x] **Ottimizzazione D - Campionamento Istantaneo Post-Rabbocco**:
  - Pulsante `Measure Pellet Level Now` per forzare la lettura del sensore ToF subito dopo aver caricato un sacco senza attendere il ciclo di 30s
- [x] **Ottimizzazione E - Immunità EMI I2C & Risparmio Termico**:
  - Bus I2C fissato a `100kHz` con `scan: false` in produzione e `power_save_mode: LIGHT` sul Wi-Fi per ridurre il riscaldamento dell'ESP8266 nella scatola stagna IP55

---

### Fase 4: Esempi di Utilizzo, Dashboard & Automazioni Home Assistant
- [x] Creare cartella `examples/home-assistant/`
- [x] Automazioni pronte all'uso in [`examples/home-assistant/automations.yaml`](examples/home-assistant/automations.yaml):
  - Allarme Riserva Pellet integrato con **SmartDisplay ESP32-S3** (card visiva + annuncio vocale TTS) e notifica push smartphone
  - Rilevamento automatico rabbocco sacco pellet con conferma su SmartDisplay
  - Controllo preventivo pre-accensione serale (17:30)
  - Avviso manutenzione lente ottica VL53L0X (se ostruita da polvere)
- [x] Card Lovelace pronta all'uso in [`examples/home-assistant/dashboard_card.yaml`](examples/home-assistant/dashboard_card.yaml) (Gauge livello %, stima kg, stato serbatoio, pulsante lettura immediata e pannello taratura live)

---

### Fase 5: Documentazione Doppia (Lapidaria & Tecnica) e Release
- [x] **Guida Rapida / Quickstart in 3 passi** in [`docs/QUICKSTART.md`](docs/QUICKSTART.md)
- [x] **Documentazione Hardware & Montaggio Ottico** in [`docs/HARDWARE.md`](docs/HARDWARE.md) (pinout Wemos D1 Mini ↔ CJVL53L0XV2, calcolo matematico del cono di visione $25^\circ\text{ FoV}$, montaggio in scatola stagna IP55 sulla griglia salvamano del serbatoio e passaggio cavo nell'asola maniglia)
- [x] **README.md principale di livello GitHub** in [`README.md`](README.md) con banner, vetrina fotografica reale, diagramma Mermaid (ToF -> D1 Mini -> HA -> SmartDisplay), badge e tabella entità
- [x] Licenza MIT in [`LICENSE`](LICENSE)
- [x] Note di rilascio ufficiali in [`RELEASE_NOTES.md`](RELEASE_NOTES.md) (`v1.0.0`)
- [x] Documentazione Wiki completa in [`wiki/`](wiki/Home.md) con sidebar e 5 pagine dedicate

---

### Fase 6: Portfolio & Engineering Excellence (Senior Dev Quality)

#### 1. ⚙️ CI/CD & Pipeline di Automazione
- [x] Workflow GitHub Actions ([`.github/workflows/ci.yml`](.github/workflows/ci.yml)), [`.yamllint.yml`](.yamllint.yml) e [`requirements-ci.txt`](requirements-ci.txt) per validazione automatica su push/PR

#### 2. 🧠 ADR (Architecture Decision Records)
- [x] Documento [`docs/ARCHITECTURE_DECISIONS.md`](docs/ARCHITECTURE_DECISIONS.md):
  - **ADR-01**: Filtro Cascata Median + Moving Average contro lo scattering ottico della polvere di legno (analisi del breakdown point al 50%)
  - **ADR-02**: Confronto fisico-ottico Time-of-Flight (`VL53L0X`) vs Ultrasuoni (`HC-SR04`) e invarianza termica
  - **ADR-03**: Frequenza I2C bloccata a 100kHz per immunità ai disturbi elettromagnetici (candeletta di accensione e motore coclea)
  - **ADR-04**: Calibrazione runtime persistente in Flash vs costanti C++ hardcoded

#### 3. 🤝 Standard Open Source & Dev Experience (DX)
- [x] Guida per i contributori in [`CONTRIBUTING.md`](CONTRIBUTING.md)
- [x] Template per le Issue di GitHub ([`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/)) e Pull Request ([`.github/pull_request_template.md`](.github/pull_request_template.md))
- [x] Codice di condotta Contributor Covenant v2.1 in [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)

#### 4. 📸 Visual Showcase & Media Reali
- [x] Cartella [`.github/assets/`](.github/assets/README.md) con guida asset
- [x] Foto reali dell'hardware e del montaggio nella stufa (`d1_pellet_components.jpg`, `d1_pellet_hopper_mount.jpg`, `d1_pellet_closed_lid.jpg`)
- [x] Galleria fotografica integrata in `README.md`, `docs/HARDWARE.md` e `wiki/`
- [x] Immagine banner per anteprima social su GitHub ([`.github/assets/banner.png`](.github/assets/banner.png) $1280 \times 640$)

#### 3. 🤝 Standard Open Source & Dev Experience (DX)
- [x] Guida per i contributori in [`CONTRIBUTING.md`](CONTRIBUTING.md)
- [x] Template per le Issue di GitHub ([`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/)) e Pull Request ([`.github/pull_request_template.md`](.github/pull_request_template.md))
- [x] Codice di condotta Contributor Covenant v2.1 in [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)

#### 4. 📸 Visual Showcase & Media Reali
- [x] Cartella [`.github/assets/`](.github/assets/README.md) con guida asset
- [x] Foto reali dell'hardware e del montaggio nella stufa (`d1_pellet_components.jpg`, `d1_pellet_hopper_mount.jpg`, `d1_pellet_closed_lid.jpg`)
- [x] Galleria fotografica integrata in `README.md`, `docs/HARDWARE.md` e `wiki/`
- [x] Immagine banner per anteprima social su GitHub ([`.github/assets/banner.png`](.github/assets/banner.png) $1280 \times 640$)

